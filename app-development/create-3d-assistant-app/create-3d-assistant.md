# How to build a Custom 3D AI Assistant app

Supervisely's 3D AI assistant is a universal tool for automating 3D point cloud labeling. It covers all types of labeling scenarios for 3D point clouds: 3D object detection, ground segmentation, 3D cuboid tracking, transfer of 2D annotations from photo context images to original 3D point clouds. But sometimes user may want to use its own algorithms for these task. In this tutorial, you will know how to build custom 3D AI Assistant app and release it to your instance as a private app.

This repository — [custom-3d-ai-assistant](https://github.com/supervisely-ecosystem/custom-3d-ai-assistant) — is a minimal end-to-end example. You can refer to this example in order to understand which endpoints have to be implemented, which information do they receive from the server and what should they return in response.


## What's in this guide

1. [Overview](#1-overview)
2. [Set up environment for development](#2-set-up-environment-for-development)
3. [Create code base](#3-create-code-base)
4. [Advanced debug](#4-advanced-debug)
5. [App configuration](#5-app-configuration)
6. [App release](#6-app-release)

---

## 1. Overview

The 3D AI Assistant exposes six HTTP endpoints. Each one is invoked by a different action in the labeling tool:

- **`POST /interactive_3d_detection`** — predicts a cuboid for area circled by Smart Lasso tool.
- **`POST /track`** — propagates cuboids across consecutive frames of a point cloud episode.
- **`POST /generate_clusters`** — pre-computes labeling-proposal clusters for a whole point cloud.
- **`POST /get_labeling_proposal`** — automatically corrects manually created cuboid (can use clusters generated from the previous endpoint).
- **`POST /segment_ground`** — returns indices of points belonging to the ground.
- **`POST /transfer_masks_to_pcd`** — transfers 2D figures drawn on a photo-context image to 3D point clouds.

Before going further, read the [Supervisely Point Cloud Annotation Format](https://developer.supervisely.com/getting-started/supervisely-annotation-format/point-clouds) — it explains the geometries JSON shape, project structure and how photo-context images are tied to point clouds with extrinsic/intrinsic matrices.

---

## 2. Set up environment for development

The fastest way to get a reproducible dev environment is VS Code's [Dev Containers](https://code.visualstudio.com/docs/devcontainers/containers) extension. The repo ships a [.devcontainer/](.devcontainer/) folder that is ready to use — open the project in VS Code and choose **"Reopen in Container"**. Alternatively, you can use Dockerfile provided below in order to prepare development environment.

### Development Dockerfile

```dockerfile
FROM supervisely/base-py-sdk:6.73.560
ENV DEBIAN_FRONTEND=noninteractive

RUN pip3 install open3d==0.18.0

RUN apt-get -y install curl
RUN apt-get update && apt -y install wireguard iproute2

LABEL "role"="development"
```

Two things to notice:

- The base image `supervisely/base-py-sdk:6.73.560` already contains the Supervisely Python SDK.
- `wireguard` and `iproute2` are installed only because we want to run the app in advanced debug mode, which uses a WireGuard VPN tunnel into the Supervisely platform (see [§4 Advanced debug](#4-advanced-debug)). When you publish the released private app, you do not need these packages — see the slimmer production Dockerfile below.

### Dev container run arguments

Whether you use VS Code's Dev Containers extension or not, if you are developing your app inside some Docker container it is important to remember to run it with some specific arguments. You can see `devcontainer.json` example below, pay attention to `runArgs` - those are just arguments for running Docker container:

```json
{
    "name": "custom_3d_ai_assistant_devcontainer",
    "build": {
        "dockerfile": "Dockerfile"
    },
    "customizations": {
        "vscode": {
            "extensions": [
                "ms-python.python",
                "ms-python.black-formatter"
            ]
        }
    },
    "runArgs": [
        "--gpus",
        "all",
        "--ipc=host",
        "--cap-add",
        "NET_ADMIN",
        "--runtime=nvidia"
    ]
}
```

Each `runArgs` entry matters:

- **`--gpus all` + `--runtime=nvidia`** — give the container access to the host GPU. Required for any real deep-learning model (we do not use any neural networks in our example app, but you may want to use one in your implementation).
- **`--ipc=host`** — share host IPC namespace; needed by PyTorch DataLoaders that use shared memory.
- **`--cap-add NET_ADMIN`** — required by WireGuard so the container can bring up the VPN tunnel during advanced debug.

### Environment files

You will need two env files:

- `~/supervisely.env` — your real credentials (`SERVER_ADDRESS` and `API_TOKEN`).
- [debug.env](debug.env) — placeholders for `TEAM_ID` and `WORKSPACE_ID`. Replace them with your own IDs before running advanced debug.

You can get information about environment variables [here](https://developer.supervisely.com/getting-started/environment-variables).

---

## 3. Create code base

You can find source code for implementing all endpoints in [main.py](https://github.com/supervisely-ecosystem/custom-3d-ai-assistant/blob/master/src/main.py).

The app is a standard FastAPI server bootstrapped through Supervisely:

```python
import supervisely as sly

app = sly.Application()
server = app.get_server()
```

Every handler receives a FastAPI `Request` whose `state` is populated by Supervisely's middleware. Three attributes matter:

- **`request.state.api`** — an authenticated `sly.Api` you can use to download point clouds, create figures, notify progress, etc.
- **`request.state.state`** — per-call payload from the labeling tool (e.g. `pcd_id`, `click_coordinate`).
- **`request.state.context`** — per-call context for tracking jobs (e.g. `trackId`, `pointCloudIds`, `objectIds`).

The response convention is a JSON object with two keys: `{"result": <payload>, "error": <null or string>}`. Synchronous endpoints wrap their body in `try/except` and return `{"result": None, "error": repr(e)}` on failure so the UI can surface the error.

Below is a per-endpoint reference. For full implementations including the random-stub bodies, see [main.py](https://github.com/supervisely-ecosystem/custom-3d-ai-assistant/blob/master/src/main.py).

### `POST /interactive_3d_detection`

Called when the user circles target object with Smart Lasso tool:

{% embed url="https://github.com/user-attachments/assets/dd5618d8-410f-4783-b7d7-d5c80974147a" %}

**Receives** (under `request.state.state`):

| Field | Type | Description |
| --- | --- | --- |
| `pcd_id` | int | ID of the point cloud the user is labeling. |
| `indices` | list[int] | Point indices of the user's mask. |

**Returns**: `{"result": <Cuboid3d JSON>, "error": null}`. Use [Cuboid3d / Vector3d](https://supervisely.readthedocs.io/en/latest/sdk/supervisely.geometry.cuboid_3d.Cuboid3d.html) from `supervisely.geometry.cuboid_3d`. Serialize with `.to_json()`.

```python
@server.post("/interactive_3d_detection")
def detect_cuboids(request: Request):
    api = request.state.api
    state = request.state.state
    current_pcd_id = state["pcd_id"]
    current_pcd, _ = f.read_pcd(current_pcd_id, api)
    mask_indices = state["indices"]

    geometry = f.generate_random_cuboid(current_pcd, mask_indices)  # replace with your model

    return {"result": geometry.to_json(), "error": None}
```

Function `read_pcd` (in [functions.py](https://github.com/supervisely-ecosystem/custom-3d-ai-assistant/blob/master/src/functions.py)) downloads the `.pcd` file once and caches it on disk under `input_pcds/` directory.

### `POST /track`

Called when the user starts a tracking job. The platform expects an immediate acknowledgement — the actual work runs in a FastAPI `BackgroundTasks` worker. Progress and results are reported back over the Supervisely API as the work proceeds.

{% embed url="https://github.com/user-attachments/assets/93ede72a-7003-4e1e-b9d6-aa790487c346" %}

**Receives** (under `request.state.context`):

| Field | Type | Description |
| --- | --- | --- |
| `trackId` | str | UUID of the tracking job; required for progress and error notifications. |
| `datasetId` | int | ID of the episode dataset. |
| `pointCloudIds` | list[int] | Frames to track through, in chronological order. |
| `direction` | str | `"forward"` or `"backward"`; reverse `pointCloudIds` if backward. |
| `objectIds` | list[int] | Supervisely object IDs being tracked. |
| `figureIds` | list[int] | Source figure IDs; used to size the progress bar. |

**Returns** (immediate): `{"message": "Track task started."}`.

**Background work must**:

1. Load the source cuboids from the first frame via `api.pointcloud.annotation.download(first_pcd_id)`, deserialized with `sly.deserialize_geometry`.
2. For each subsequent frame, predict a new cuboid per object and write it back via `api.pointcloud_episode.figure.create(pcd_id, object_id, geom_json, "cuboid_3d", track_id)`.
3. Notify the UI after each step via `api.pointcloud_episode.notify_progress(track_id, dataset_id, pcd_ids, current, total)`.
4. Catch all exceptions inside the background task and report them through `point-clouds.episodes.notify-annotation-tool` — otherwise the UI's tracking spinner will hang forever. The repo provides a `send_error_data` decorator that does this; reuse it.

```python
@server.post("/track")
def start_track(request: Request, task: BackgroundTasks):
    task.add_task(track_cuboids, request)
    return {"message": "Track task started."}


@send_error_data
def track_cuboids(request: Request):
    api = request.state.api
    context = request.state.context
    track_id = context["trackId"]
    pcd_ids = context["pointCloudIds"]
    if context["direction"] == "backward":
        pcd_ids = pcd_ids[::-1]
    object_ids = context["objectIds"]
    # ... run your model, then for each frame:
    #     api.pointcloud_episode.figure.create(...)
    #     api.pointcloud_episode.notify_progress(...)
```

See the full `start_track` handler, the `track_cuboids` background worker, and the `send_error_data` decorator in [main.py](https://github.com/supervisely-ecosystem/custom-3d-ai-assistant/blob/master/src/main.py).

### `POST /generate_clusters`

Called once per point cloud, before the user starts requesting labeling proposals. It is a side-effect endpoint: it pre-computes proposal clusters and stores them in memory keyed by `pcd_id`.

**Receives** (under `request.state.state`):

| Field | Type | Description |
| --- | --- | --- |
| `pcd_id` | int | ID of the point cloud to cluster. |

**Returns**: should not return anything.

```python
@server.post("/generate_clusters")
def generate_clusters(request: Request):
    api = request.state.api
    state = request.state.state
    pcd_id = state["pcd_id"]
    pcd, _ = f.read_pcd(pcd_id, api)
    f.labeling_proposals[pcd_id] = f.generate_random_clusters(pcd)  # replace with your model
```

The clusters are kept in the module-level `labeling_proposals` dict. Your real implementation of `/get_labeling_proposal` (next) can use these generated clusters, but it depends on your implementation. If you don't need it, you can just skip implementation of this endpoint.

### `POST /get_labeling_proposal`

Called when user enables `Smart Auto-Fit` option in AI Assistance tools and places manually created cuboid after that - a request to this endpoint will be sent in order to try to automatically fit cuboid to target object instead of correcting it manually%

{% embed url="https://github.com/user-attachments/assets/984e91ba-1237-428f-9857-82f9470c48a7" %}

**Receives** (under `request.state.state`):

| Field | Type | Description |
| --- | --- | --- |
| `pcd_id` | int | ID of the point cloud. |
| `click_coordinate` | list[float] | `[x, y, z]` of the user's click. |

**Returns**: `{"result": <Cuboid3d JSON> | null, "error": null}`. Return `null` for `result` when no labeling proposals are available near the click.

```python
@server.post("/get_labeling_proposal")
def get_labeling_proposal(request: Request):
    api = request.state.api
    state = request.state.state
    pcd_id = state["pcd_id"]
    click_coordinate = np.asarray(state["click_coordinate"])
    # Pick a cluster from f.labeling_proposals[pcd_id] near click_coordinate,
    # build a Cuboid3d from it, and return its JSON. Or use any other
    # algorithm in order to generate cuboid near given coordinate
    return {"result": cluster_cuboid.to_json(), "error": None}
```

### `POST /segment_ground`

Called when the user selects `Ground Segmentation` option in AI Assistance tools:

{% embed url="https://github.com/user-attachments/assets/4fc84ce7-60ac-432e-a75a-1257daf88828" %}

**Receives** (under `request.state.state`):

| Field | Type | Description |
| --- | --- | --- |
| `pcd_id` | int | ID of the point cloud to segment. |

**Returns**: `{"result": [int, int, ...], "error": null}` — a list of indices into the point cloud's `points` array that belong to the ground plane.

```python
@server.post("/segment_ground")
def get_ground_indices(request: Request):
    api = request.state.api
    state = request.state.state
    pcd_id = state["pcd_id"]
    pcd, _ = f.read_pcd(pcd_id, api)

    ground_indexes = run_your_ground_segmentation_model(pcd)  # returns numpy int array

    return {"result": ground_indexes.tolist(), "error": None}
```

### `POST /transfer_masks_to_pcd`

Called when the user has drawn 2D annotations (rectangles, polygons, bitmaps) on a photo-context image and wants to transfer them to 3D space (e.g. bounding boxes -> cuboids):

{% embed url="https://github.com/user-attachments/assets/84b744e5-ade0-49d1-81f1-61e0748726f8" %}

**Receives** (under `request.state.state`):

| Field | Type | Description |
| --- | --- | --- |
| `pcd_id` | int | ID of the target point cloud. |
| `image_id` | int | ID of the photo-context image carrying the 2D figures. |
| `figure_ids` | list[int] *(optional)* | If present, only transfer these figures. Otherwise transfer all figures on the image. |

**Returns**: `{"result": [<entry>, ...]}` where each entry is

```json
{
  "geometryType": "cuboid_3d",
  "geometry": { "...": "Cuboid3d JSON" },
  "srcFigureId": 12345
}
```

`srcFigureId` is mandatory — the labeling tool uses it to link the resulting 3D cuboid back to the originating 2D figure.

```python
@server.post("/transfer_masks_to_pcd")
def transfer_masks_to_pcd(request: Request):
    api = request.state.api
    state = request.state.state
    pcd_id = state["pcd_id"]
    photo_context_img_id = state["image_id"]
    figure_ids = state.get("figure_ids", [])
    dataset_id = api.pointcloud.get_info_by_id(pcd_id).dataset_id

    figures = api.image.figure.download(dataset_id, [photo_context_img_id])[photo_context_img_id]
    if figure_ids:
        figures = [fig for fig in figures if fig.id in figure_ids]

    anns_3d = []
    for fig in figures:
        cuboid = run_your_2d_to_3d_lifting(fig, ...)
        anns_3d.append({
            "geometryType": "cuboid_3d",
            "geometry": cuboid.to_json(),
            "srcFigureId": fig.id,
        })
    return {"result": anns_3d}
```

For real implementations you will probably need the photo-context image and its camera calibration, plus a way to rasterize the 2D figures. The repo ships two helpers in [functions.py](https://github.com/supervisely-ecosystem/custom-3d-ai-assistant/blob/master/src/functions.py) that the random-stub `transfer_masks_to_pcd` handler does not call but that you will almost certainly want.

`load_photo_context_data` downloads the related image and pulls the 3×4 extrinsic and 3×3 intrinsic matrices out of the point cloud's image metadata:

```python
def load_photo_context_data(dataset_id, image_id, api):
    filters = [{"field": ApiField.ID, "operator": "=", "value": image_id}]
    img_info = api.pointcloud.get_list_all_pages(
        "point-clouds.images.list",
        {ApiField.DATASET_ID: dataset_id, ApiField.FILTER: filters},
        convert_json_info_cb=lambda x: x,
    )[0]

    photo_context_img_path = f"related_images/{image_id}.jpg"
    api.pointcloud.download_related_image(img_info["id"], photo_context_img_path)
    photo_context_img = sly.image.read(photo_context_img_path)

    extrinsic_matrix = np.asarray(img_info["meta"]["sensorsData"]["extrinsicMatrix"]).reshape((3, 4))
    intrinsic_matrix = np.asarray(img_info["meta"]["sensorsData"]["intrinsicMatrix"]).reshape((3, 3))

    return {
        "image": photo_context_img,
        "extrinsic_matrix": extrinsic_matrix,
        "intrinsic_matrix": intrinsic_matrix,
    }
```

`get_2d_anns` turns each Supervisely 2D figure into a numpy mask (for bitmaps and polygons), a `[left, top, right, bottom]` bbox (for rectangles), or a list of `[col, row]` points (for polylines):

```python
def get_2d_anns(image_id, dataset_id, photo_context_img, api, figure_ids):
    figures = api.image.figure.download(dataset_id, [image_id])[image_id]
    anns_2d = []
    for figure in figures:
        if len(figure_ids) > 0 and figure.id not in figure_ids:
            continue
        if figure.geometry_type == "bitmap":
            geometry = sly.Bitmap.from_json(figure.geometry)
            mask = np.zeros(photo_context_img.shape, dtype=np.uint8)
            geometry.draw(bitmap=mask, color=[1, 1, 1])
            anns_2d.append(("bitmap", mask[:, :, :2], figure.id))
        elif figure.geometry_type == "rectangle":
            geometry = sly.Rectangle.from_json(figure.geometry)
            bbox = [geometry.left, geometry.top, geometry.right, geometry.bottom]
            anns_2d.append(("rectangle", bbox, figure.id))
        elif figure.geometry_type == "polygon":
            polygon_label = sly.Label(
                sly.Polygon.from_json(figure.geometry),
                sly.ObjClass("polygon", sly.Polygon),
            )
            bitmap_label = polygon_label.convert(sly.ObjClass("bitmap", sly.Bitmap))[0]
            mask = np.zeros(photo_context_img.shape, dtype=np.uint8)
            bitmap_label.geometry.draw(bitmap=mask, color=[1, 1, 1])
            anns_2d.append(("polygon", mask[:, :, :2], figure.id))
        elif figure.geometry_type == "line":
            geometry = sly.Polyline.from_json(figure.geometry)
            line_points = [[p.col, p.row] for p in geometry.exterior]
            anns_2d.append(("line", line_points, figure.id))
    return anns_2d
```

A real `transfer_masks_to_pcd` handler typically chains the two: call `load_photo_context_data` to get the image and matrices, call `get_2d_anns` to get rasterized 2D figures, then project each figure into the point cloud using the extrinsic/intrinsic matrices and fit a `Cuboid3d` around the resulting 3D points.

> **See also:** the [Supervisely Point Cloud Annotation Format](https://developer.supervisely.com/getting-started/supervisely-annotation-format/point-clouds) for the canonical JSON shape of `cuboid_3d`, dataset structure, and photo-context fields.

---

## 4. Advanced debug

Once the code is written, it's time to test it right in the Supervisely platform as a debugging app. In advanced debug mode the app runs locally on your machine but is reachable from the platform through a WireGuard VPN tunnel — so you can hit the real labeling-tool requests against your local breakpoints.

The repo already ships the launch configuration. [.vscode/launch.json](.vscode/launch.json):

```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Advanced debug",
            "type": "debugpy",
            "cwd": "${workspaceFolder}",
            "request": "launch",
            "module": "uvicorn",
            "args": [
                "src.main:app",
                "--host",
                "0.0.0.0",
                "--port",
                "8000",
                "--ws",
                "websockets"
            ],
            "jinja": true,
            "justMyCode": false,
            "env": {
                "PYTHONPATH": "${workspaceFolder}:${PYTHONPATH}",
                "LOG_LEVEL": "DEBUG",
                "ENV": "production",
                "FOLDER": "./my_model",
                "DEBUG_WITH_SLY_NET": "1",
                "SLY_APP_DATA_DIR": "${workspaceFolder}/results"
            }
        }
    ]
}
```

You can read more about advanced debug mode [here](https://developer.supervisely.com/app-development/advanced/advanced-debugging).

After that:

1. If you develop in a Docker container, run the container with `--cap-add NET_ADMIN` — already configured in [.devcontainer/devcontainer.json](.devcontainer/devcontainer.json).
2. Install `wireguard` and `iproute2` — already done in [.devcontainer/Dockerfile](.devcontainer/Dockerfile). On macOS use `brew install wireguard-tools`.
3. Define your `TEAM_ID` (and `WORKSPACE_ID`) in [debug.env](debug.env). The other env variables that advanced debug needs are already set in `.vscode/launch.json`.
4. Switch the `launch.json` config to **"Advanced debug"**:

![Advanced Debug in Supervisely](https://user-images.githubusercontent.com/31512713/224290229-5da93fd2-dc97-4911-abb5-66ce890485a2.png)

5. Run the code.

✅ It will deploy the app in the Supervisely platform as a REST API service.

Here is how an advanced-debug launch looks like:

{% embed url="https://user-images.githubusercontent.com/91027877/257574738-6c07c37a-7b20-4e02-8fba-9f4fb5b98bef.mp4" %}

After advanced debug launch you must be able to debug your app via the **Develop & Debug** app (just select Develop & Debug session in session selector).

{% embed url="https://github.com/user-attachments/assets/9f0041d5-0504-40e7-a5ea-839b774157b7" %}

---

## 5. App configuration

Use the [config.json](https://github.com/supervisely-ecosystem/custom-3d-ai-assistant/blob/master/config.json) from example repository as a starting point:

```json
{
    "name": "Custom 3D AI assistant",
    "type": "app",
    "version": "2.0.0",
    "categories": [
        "neural network",
        "pointclouds",
        "detection & tracking",
        "serve"
    ],
    "description": "Deploy custom labeling assistant for 3D point clouds as REST API service",
    "docker_image": "supervisely/custom-3d-ai-assistant:1.0.0",
    "entrypoint": "python3 -m uvicorn src.main:app --host 0.0.0.0 --port 8000 --ws websockets",
    "port": 8000,
    "task_location": "application_sessions",
    "min_instance_version": "6.15.35",
    "isolate": true,
    "session_tags": [
        "3d_smart_tool",
        "sly_point_cloud_tracking"
    ],
    "community_agent": false,
    "headless": true,
    "is3DAIAssistant": true
}
```

Two fields are critical for an app to be recognized as a 3D AI Assistant:

- **`"is3DAIAssistant": true`** — without this flag the platform will not surface your app as a 3D assistant in the labeling tool.
- **`"session_tags": ["3d_smart_tool", "sly_point_cloud_tracking"]`** — the labeling tool and the tracker look up running sessions by these tags. Drop either tag and the corresponding feature (interactive smart tool or episode tracking) will not be wired to your app.

Other fields to keep aligned:

- **`"headless": true`** — this is a serving app with no UI.
- **`"port"`** and **`"entrypoint"`** must match the uvicorn command (8000 / `src.main:app`).
- **`"docker_image"`** must point to a published production image. Do not forget to build and push it before releasing app.

---

## 6. App release

Once you've tested the code, it's time to release it into the platform. It can be released as an App that is shared with the all Supervisely community, or as your own private App.

Refer to [How to Release your App](https://developer.supervisely.com/app-development/basics/from-script-to-supervisely-app) for all releasing details. For a private app check also [Private App Tutorial](https://developer.supervisely.com/app-development/basics/add-private-app).
