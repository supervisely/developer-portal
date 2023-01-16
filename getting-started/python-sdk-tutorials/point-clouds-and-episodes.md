# Working with 3D Point Clouds and Point Cloud Episodes

## Introduction

In this tutorial we will focus on working with Point Clouds and Point Cloud Episodes using Supervisely SDK.

You will learn how to:

1. [Upload point clouds and photo context to Supervisely](#Upload-point-clouds-and-photo-context-to-Supervisely)
2. [Get information about Point Clouds and image contexts](#Get-information-about-Point-Clouds-and-related-context-Images)
3. [Download point clouds and image contexts to local directory](#Download-point-clouds-and-context-images-from-Supervisely)
4. [Working with Point Cloud Episodes](#Working-with-Point-Cloud-Episodes)

📗 Everything you need to reproduce [this tutorial is on GitHub](https://github.com/supervisely-ecosystem/tutorial-pointclouds): source code and demo data.

## How to debug this tutorial

**Step 1.** Prepare `~/supervisely.env` file with credentials. [Learn more here.](../basics-of-authentication.md)

**Step 2.** Clone [repository](https://github.com/supervisely-ecosystem/tutorial-pointclouds) with source code and demo data and create [Virtual Environment](https://docs.python.org/3/library/venv.html).

```
git clone https://github.com/supervisely-ecosystem/tutorial-pointclouds.git

cd tutorial-pointclouds

./create_venv.sh
```

**Step 3.** Open repository directory in Visual Studio Code.

```
code -r .
```

**Step 4.** Change workspace ID in `local.env` file by copying the ID from the context menu of the workspace.

```
context.workspaceId=654 # ⬅️ change value
```

<figure><img src="https://user-images.githubusercontent.com/79905215/209327856-e47fb82b-c207-48fc-bb36-1fe795d45f6f.png" alt=""><figcaption></figcaption></figure>

**Step 5.** Start debugging `src/main.py`.

### Import libraries

```python
import os
import json
from pathlib import Path
from dotenv import load_dotenv
import supervisely as sly
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance.

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))
api = sly.Api()
```

### Get variables from environment

In this tutorial, you will need an workspace ID that you can get from environment variables. [Learn more here](../environment-variables.md#workspace_id)

```python
workspace_id = sly.env.workspace_id()
```

## Create new project and dataset

Create new project.

**Source code:**

```python
project = api.project.create(
    workspace_id,
    name="Point Clouds Tutorial",
    type=sly.ProjectType.POINT_CLOUDS,
    change_name_if_conflict=True,
)
print(f"Project ID: {project.id}")
```

**Output:**

```python
# Project ID: 16197
```

Create new dataset.

**Source code:**

```python
dataset = api.dataset.create(project.id, name="dataset_1")

print(f"Dataset ID: {dataset.id}")
```

**Output:**

```python
# Dataset ID: 54539
```


## Upload point clouds and photo context to Supervisely

### Upload single point cloud.

**Source code:**

```python
pcd_file = "src/input/pcd/000000.pcd"
pcd_info = api.pointcloud.upload_path(dataset.id, name="pcd_0", path=pcd_file)
print(f'Point cloud "{pcd_info.name}" uploaded to Supervisely with ID:{pcd_info.id}')
```

**Output:**

```python
# Point cloud "pcd_0" uploaded to Supervisely platform with ID:17539453
```

<figure><img src="https://user-images.githubusercontent.com/31512713/211832231-81103088-7062-46c2-b93e-99241be3d28f.png" alt="first-pcd-uploaded"><figcaption></figcaption></figure>

**Now you can explore and label it in [Supervisely labeling tool](https://ecosystem.supervise.ly/annotation_tools/pointcloud-labeling-tool)**:

<figure><img src="https://user-images.githubusercontent.com/31512713/211832212-73569ed7-d77f-4519-bd63-e02de63e4f18.png" alt="first-in-labeling-tool"><figcaption></figcaption></figure>

### Upload related context image to Supervisely.

#### Extrinsic and intrinsic matrices

If you have a photo context taken with a LIDAR image, you can attach the photo to the point cloud.
To do that, we need two additional matrices. They are used for matching 3D coordinates in the point cloud to the 2D coordinates in the photo context:

<figure><img src="https://user-images.githubusercontent.com/31512713/212629303-209af3f0-49fc-4a73-9125-233d616cd583.png" alt="matrices_labeled", width="300px"><figcaption></figcaption></figure>

**Parameters meaning**

- **f<sub>x</sub>, f<sub>y</sub>** are the focal lengths expressed in pixel units
- **c<sub>x</sub>, c<sub>y</sub>** is a principal point that is usually at the image center
- **r<sub>ij</sub>** and **t<sub>i</sub>** from the `extrinsicMatrix` are the rotation and translation parameters

The dot product of the matrices and XYZ coordinate in 3D space gives us the coordinate of a point *(x=u, y=v)* in the photo context:

<figure><img src="https://user-images.githubusercontent.com/31512713/212630179-13315291-0e69-4099-a6ae-824da3e4598e.png" alt="dot_product_matrices", width="300px"><figcaption></figcaption></figure>

#### Uploading context photo to the Supervisely.
For attaching a photo, it is needed to provide the matrices in a `meta` **dict** with the `deviceId` and `sensorsData` fields.
The matrices must be included in the `meta` dict as **flattened** lists.

**Example of a meta dict:**

```python
# src/input/cam_info/000000.json
{
    "deviceId": "CAM_2",
    "sensorsData": {
        "extrinsicMatrix": [
            0.007533745,
            -0.9999714,
            -0.000616602,
            -0.004069766,
            0.01480249,
            0.0007280733,
            -0.9998902,
            -0.07631618,
            0.9998621,
            0.00752379,
            0.01480755,
            -0.2717806,
        ],
        "intrinsicMatrix": [721.5377, 0, 609.5593, 0, 721.5377, 172.854, 0, 0, 1],
    }
}
```

#### A full code for uploading and attaching the context image

**Source code:**

```python
# input files:
img_file = "src/input/img/000000.png"
cam_info_file = "src/input/cam_info/000000.json"

# 0. Read cam_info with matrices (a meta dict).
with open(cam_info_file, "r") as f:
    cam_info = json.load(f)

# 1. Upload an image to the Supervisely. It generates us a hash for image
img_hash = api.pointcloud.upload_related_image(img_file)
# 2. Create img_info needed for matching the image to the point cloud by its ID
img_info = {"entityId": pcd_info.id, "name": "img_0", "hash": img_hash, "meta": cam_info}
# 3. Run the API command to attach the image
api.pointcloud.add_related_images([img_info])

print("Context image has been uploaded.")
```

**Output:**

```python
# Context image has been uploaded.
```

<figure><img src="https://user-images.githubusercontent.com/31512713/212670489-d9a3660e-9df1-464f-8d72-9f3f99d3ab48.png" alt="first-in-labeling-tool-context"><figcaption></figcaption></figure>

More about the format of a photo context: [Supervisely annotation JSON format](https://developer.supervise.ly/api-references/supervisely-annotation-json-format/point-clouds#photo-context-image-annotation-file)

More about calibration and matrix transformations: [OpenCV 3D Camera Calibration Tutorial](https://docs.opencv.org/2.4/modules/calib3d/doc/camera_calibration_and_3d_reconstruction.html).


### Upload list of point clouds and context images.

✅ Supervisely API allows uploading multiple point clouds in a single request. The code sample below sends fewer requests and it leads to a significant speed-up of our original code.

**Source code:**

```python
# Upload a batch of point clouds and related images
paths = ["src/input/pcd/000001.pcd", "src/input/pcd/000002.pcd"]
img_paths = ["src/input/img/000001.png", "src/input/img/000002.png"]
cam_paths = ["src/input/cam_info/000001.json", "src/input/cam_info/000002.json"]

pcd_infos = api.pointcloud.upload_paths(dataset.id, names=["pcd_1", "pcd_2"], paths=paths)
img_hashes = api.pointcloud.upload_related_images(img_paths)
img_infos = []
for i, cam_info_file in enumerate(cam_paths):
    # reading cam_info
    with open(cam_info_file, "r") as f:
        cam_info = json.load(f)
    img_info = {
        "entityId": pcd_infos[i].id,
        "name": f"img_{i}",
        "hash": img_hashes[i],
        "meta": cam_info,
    }
    img_infos.append(img_info)
result = api.pointcloud.add_related_images(img_infos)
print("Batch uploading has finished:", result)
```

**Output:**

```python
# Batch uploading has finished: {'success': True}
```


## Get information about Point Clouds and related context Images

### Get info by name

Get information about point cloud from Supervisely by name.

**Source code:**

```python
pcd_info = api.pointcloud.get_info_by_name(dataset.id, name="pcd_0")
print(pcd_info)
```

**Output:**

```python
PointcloudInfo(
    id=17553684,
    frame=None,
    description="",
    name="pcd_0",
    team_id=440,
    workspace_id=662,
    project_id=16108,
    dataset_id=54365,
    link=None,
    hash="rxl9ioCcNobe1z7q1dA6idsebCM77G0wlrZd1Be28ng=",
    path_original="/h5un6l2bnaz1vj8a9qgms4-public/point_clouds/f/x/kC/5JwCwSNouz7u3sNVDWOIURf44HRAridOKsf3lDGjo9bEHcj22gCejQIULbZHblG9Ns6GWD4Vmc3I0KdBagpmZKovKikN50Ij7utyw5aUaCTtM10sLiX4BVqPRssx.pcd",
    cloud_mime="image/pcd",
    figures_count=0,
    objects_count=0,
    tags=[],
    meta={},
    created_at="2023-01-08T07:15:50.332Z",
    updated_at="2023-01-08T07:15:50.332Z",
)
```

### Get info by ID

You can also get information about image from Supervisely by id.

**Source code:**

```python
pcd_info = api.pointcloud.get_info_by_id(pcd_info.id)
print("Point cloud name:", pcd_info.name)
```

**Output:**

```python
# Point cloud name: pcd_0
```

### Get information about context images

Get information about related context images. For example it can be a photo from front/back cameras of a vehicle.

**Source code:**

```python
img_infos = api.pointcloud.get_list_related_images(pcd_info.id)
img_info = img_infos[0]
print(img_info)
```

**Output:**

```python
{'pathOriginal': '/h5un6l2bnaz1vj8a9qgms4-public/images/original/S/j/hJ/PwhtY7x4zRQ5jvNETPgFMtjJ9bDOMkjJelovMYLJJL2wxsGS9dvSjQC428ORi2qIFYg4u1gbiN7DsRIfO3JVBEt0xRgNc0vm3n2DTv8UiV9HXoaCp0Fy4IoObKMg.png',
 'id': 473302,
 'entityId': 17557533,
 'createdAt': '2023-01-09T08:50:33.225Z',
 'updatedAt': '2023-01-09T08:50:33.225Z',
 'meta': {'deviceId': 'cam_2'},
 'fileMeta': {'mime': 'image/png',
  'size': 893783,
  'width': 1224,
  'height': 370},
 'hash': 'vxA+emfDNUkFP9P6oitMB5Q0rMlnskmV2jvcf47OjGU=',
 'link': None,
 'preview': '/previews/q/ext:jpeg/resize:fill:50:0:0/q:50/plain/h5un6l2bnaz1vj8a9qgms4-public/images/original/S/j/hJ/PwhtY7x4zRQ5jvNETPgFMtjJ9bDOMkjJelovMYLJJL2wxsGS9dvSjQC428ORi2qIFYg4u1gbiN7DsRIfO3JVBEt0xRgNc0vm3n2DTv8UiV9HXoaCp0Fy4IoObKMg.png',
 'fullStorageUrl': 'https://dev.supervise.ly/h5un6l2bnaz1vj8a9qgms4-public/images/original/S/j/hJ/PwhtY7x4zRQ5jvNETPgFMtjJ9bDOMkjJelovMYLJJL2wxsGS9dvSjQC428ORi2qIFYg4u1gbiN7DsRIfO3JVBEt0xRgNc0vm3n2DTv8UiV9HXoaCp0Fy4IoObKMg.png',
 'name': 'img00'}
```


### Get list of all point clouds in the dataset

You can list all point clouds in the dataset.

**Source code:**

```python
pcd_infos = api.pointcloud.get_list(dataset.id)
print(f"Dataset contains {len(pcd_infos)} point clouds")
```

**Output:**

```python
# Dataset contains 3 point clouds
```


## Download point clouds and context images from Supervisely

### Download a point cloud

Download point cloud from Supervisely to local directory by id.

**Source code:**

```python
save_path = "src/output/pcd_0.pcd"
api.pointcloud.download_path(pcd_info.id, save_path)
print(f"Point cloud has been successfully downloaded to '{save_path}'")
```

**Output:**

```python
# Point cloud has been successfully downloaded to 'src/output/pcd_0.pcd'
```

### Download a related context image

Download a related context image from Supervisely to local directory by image id.

**Source code:**

```python
save_path = "src/output/img_0.png"
img_info = api.pointcloud.get_list_related_images(pcd_info.id)[0]
api.pointcloud.download_related_image(img_info["id"], save_path)
print(f"Context image has been successfully downloaded to '{save_path}'")
```

**Output:**

```python
# Context image has been successfully downloaded to 'src/output/img_0.png'
```

------

## Working with Point Cloud Episodes

Working with Point Cloud Episodes is similar, except the following:
1. There is `api.pointcloud_episode` for working with episodes.
2. Create new projects with type `sly.ProjectType.POINT_CLOUD_EPISODES`.
3. Put the frame index in meta while uploading a pcd: `meta = {"frame": idx}`.

**Note:**
in Supervisely each episode is treated as a dataset. Therefore, create a separate dataset every time you want to add a new episode.

### Create new project and dataset

Create new project.

**Source code:**

```python
project = api.project.create(
    workspace_id,
    name="Point Cloud Episodes Tutorial",
    type=sly.ProjectType.POINT_CLOUD_EPISODES,
    change_name_if_conflict=True,
)
print(f"Project ID: {project.id}")
```

**Output:**

```python
# Project ID: 16197
```

Create new dataset.

**Source code:**

```python
dataset = api.dataset.create(project.id, "dataset_1")
print(f"Dataset ID: {dataset.id}")
```

**Output:**

```python
# Dataset ID: 54539
```

### Upload one point cloud to Supervisely.

**Source code:**

```python
meta = {"frame": 0}  # "frame" is a required field for Episodes
pcd_info = api.pointcloud_episode.upload_path(dataset.id, "pcd_0", "src/input/pcd/000000.pcd", meta=meta)
print(f'Point cloud "{pcd_info.name}" (frame={meta["frame"]}) uploaded to Supervisely')
```

**Output:**

```python
# Point cloud "pcd_0" (frame=0) uploaded to Supervisely
```

### Upload entire point clouds episode to Supervisely platform.

**Source code:**

```python
def read_cam_info(cam_info_file):
    with open(cam_info_file, "r") as f:
        cam_info = json.load(f)
    return cam_info


# 1. get paths
input_path = "src/input"
pcd_files = list(Path(f"{input_path}/pcd").glob("*.pcd"))
img_files = list(Path(f"{input_path}/img").glob("*.png"))
cam_info_files = Path(f"{input_path}/cam_info").glob("*.json")

# 2. get names and metas
pcd_metas = [{"frame": i} for i in range(len(pcd_files))]
img_metas = [read_cam_info(cam_info_file) for cam_info_file in cam_info_files]
pcd_names = list(map(os.path.basename, pcd_files))
img_names = list(map(os.path.basename, img_files))

# 3. upload
pcd_infos = api.pointcloud_episode.upload_paths(dataset.id, pcd_names, pcd_files, metas=pcd_metas)
img_hashes = api.pointcloud.upload_related_images(img_files)
img_infos = [
    {"entityId": pcd_infos[i].id, "name": img_names[i], "hash": img_hashes[i], "meta": img_metas[i]}
    for i in range(len(img_hashes))
]
api.pointcloud.add_related_images(img_infos)

print("Point Clouds Episode has been uploaded to Supervisely")
```

**Output:**

```python
# Point Clouds Episode has been uploaded to Supervisely
```

**Now you can explore and label it in [Supervisely labeling tool for Episodes](https://ecosystem.supervise.ly/annotation_tools/pointcloud-episodes-labeling-tool)**:

<figure><img src="https://user-images.githubusercontent.com/31512713/211832234-115e1280-e3b7-4b3f-80e2-1b17a3868a76.png" alt="episodes"><figcaption></figcaption></figure>
