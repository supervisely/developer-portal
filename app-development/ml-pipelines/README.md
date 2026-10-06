---
description: How the ML Pipelines app works inside — the pipeline graph, its JSON format, how a run processes data, how to start and run it from code, and how to extend it
---

# ML Pipelines

[ML Pipelines](https://ecosystem.supervisely.com/apps/data-nodes) is a Supervisely app in which you build a data pipeline from nodes on a canvas: one or more inputs, transformations, filters, neural networks, and one or more outputs. The source code is [supervisely-ecosystem/data-nodes](https://github.com/supervisely-ecosystem/data-nodes). How to use the app is described in the user documentation, [Pipelines](https://docs.supervisely.com/data-organization/operations-with-data/pipelines). This page is for developers: what a pipeline is, how the app runs it, how to start it from code and how to add your own nodes.

## In this section

* [Add your own node to ML Pipelines](custom-nodes.md): fork the app, add nodes with your own code and packages, and release the fork as a private app.

## Modalities

One app session works with one modality: **images** or **videos**. The node list, the presets folder and the item type all depend on it. The modality comes from the project the app was started from, or from the choice in the launch window when it is started from the Ecosystem (`modal.state.modalityType`). To work with the other modality, start another session.

## A pipeline is a list of layers

Every node on the canvas is a **layer**. The whole pipeline is a JSON list of layers; this is also what a preset file contains. A layer has four fields:

| Field | Meaning |
| --- | --- |
| `action` | the node type, for example `images_project`, `resize`, `if`, `create_new_project` |
| `src` | where the data comes from: a list of `"<project name>/<dataset name>"` (`*` for all datasets) for an input layer, or a list of connection names `"$..."` for any other layer |
| `dst` | where the data goes: one connection name, or a list of them for a node with several outputs, in the order of the outputs |
| `settings` | the node's settings; every node validates them with a JSON schema before the run |

A connection name starts with `$`. Two layers are connected when the `dst` of one is in the `src` of the other. A node with several outputs gets one name per output: for example `If` writes `["$if_3__true", "$if_3__false"]`, and the item goes to one of them. An output that is not connected to anything drops its items. Several connections into one input merge the data. Output layers have no outgoing connections: **Create New Project** uses its `dst` as the name of the new project.

```json
[
  {
    "action": "images_project",
    "src": ["my-project/*"],
    "dst": "$images_project_1",
    "settings": { "classes_mapping": "default", "tags_mapping": "default" }
  },
  {
    "action": "resize",
    "src": ["$images_project_1"],
    "dst": "$resize_2",
    "settings": { "width": 640, "height": -1, "aspect_ratio": { "keep": true } }
  },
  {
    "action": "create_new_project",
    "src": ["$resize_2"],
    "dst": "my-project (resized)",
    "settings": { "project_name": "my-project (resized)" }
  }
]
```

Input layers choose classes and tags with `classes_mapping` and `tags_mapping`: `"default"` keeps all of them, and a dict maps every class or tag to a new name or to `"__ignore__"` to drop it. Presets saved by the app also store `scene_location`, the node's position on the canvas. Most nodes' documentation in the app ends with a **JSON view** of their settings, but the surest way to get the exact format is to build the pipeline in the app and press **SAVE**.

## How a run works

**RUN** collects the layers from the canvas and runs them in a background thread of the app's container:

1. **Validate.** Each layer checks its settings against its schema, and the graph is checked: it needs at least one input and one output layer, and its connections and class mappings must be consistent.
2. **Calculate metas.** The project meta (classes and tags) is passed from the inputs through every layer, so that each layer knows what it will receive and what it outputs.
3. **Preprocess.** Each layer prepares once: for example an output node creates its project.
4. **Process items in batches.** Items are listed dataset by dataset from the inputs and pushed through the graph batch by batch. Each layer gets the item, or the whole batch if it implements batch processing, and yields it to one of its outputs.
5. **Postprocess.** Each layer finishes once: for example a deploy node stops its model if **Auto stop model on pipeline finish** is on.
6. **Results.** Exported archives are uploaded to Team Files, the results panel shows the new projects, archives and labeling jobs, and the run is recorded in the Workflow of the inputs and outputs.

An item is a pair `(descriptor, annotation)`: an image with its `Annotation`, or a video with its `VideoAnnotation`. The descriptor carries the item's info (`ImageInfo` or `VideoInfo`), its dataset and project, and the pixels or the file path. Layers can set extra attributes on the descriptor to pass values to the next layers.

What is downloaded depends on the nodes:

* If no node reads or changes the media (for example only filters, annotation and dataset nodes), no media is downloaded and items go in batches of 500. When no node needs the pixels, images are added to the output project by reference, without uploading them again.
* For images, a node that needs the pixels makes the run download every image; the batch size is 50.
* For videos, a video is downloaded only when a node first reads its file, so videos dropped by a filter are never downloaded. The batch has one video unless a node asks for more (see [Run videos in parallel](custom-nodes.md#run-videos-in-parallel)).

An error in one batch does not stop the run: the batch is skipped, the error is written to the task log with its traceback, and the next batch is processed. Check the log when the output has fewer items than the input. **STOP** ends the run early; the results can be incomplete.

## Starting the app

The app reads its starting state from the environment of the task:

| Started from | What the canvas shows |
| --- | --- |
| Ecosystem | an empty canvas for the modality chosen in the launch window |
| a project or dataset (context menu, **Pipelines** tab) | an **Images Project** or **Videos Project** input with that project or dataset |
| the project's filters, or selected items | a **Filtered Project** input with the filtered items |
| a preset file in Team Files (context menu of a `.json` file) | the pipeline from that file |
| a ready pipeline from the project's **Run pipeline** menu (`modal.state.pipelineTemplate`) | a built-in pipeline for that project: `copy`, `move`, `basic-detection-augmentations`, `basic-segmentation-augmentations` (images only) |

## Presets

**SAVE** writes the pipeline JSON to Team Files, to `/data-nodes/presets/<modality>/<name>.json`. **LOAD** lists that folder. With **Save as template**, the input layer of the preset is marked as a template: when the preset is loaded in a session started from another project, the input takes that project instead of the saved one. Because a preset is plain JSON, you can generate presets with code, keep them in git, or upload them with `api.file.upload` and open them from Team Files.

Results that are files go to `/data-nodes/archives/<modality>/<task id>/` in Team Files and are attached to the task as its output.

## Running a pipeline from code

The app has two HTTP endpoints. Start a session with the pipeline loaded, for example from a preset file, then call them with `send_request`:

```python
import time
import supervisely as sly

api = sly.Api.from_env()
task_id = 12345  # the ML Pipelines session, started with the pipeline loaded

api.task.send_request(task_id, "run_pipeline", data={})  # starts the pipeline on the canvas

while True:
    status = api.task.send_request(task_id, "get_pipeline_status", data={})
    print(status["result"])  # "Pipeline status is 120/500", or "pipeline is not running"
    if status["result"] == "pipeline is not running":
        break
    time.sleep(10)
```

* `run_pipeline` runs the pipeline that is on the canvas. It starts the run and returns; it does not wait for the run to finish. If a run is already going, it says so and starts nothing.
* `get_pipeline_status` returns the number of processed items out of the total, or `pipeline is not running`.

The result of the run is in the session's UI, in the task log and in the Workflow.

## Nodes

The nodes for each modality, grouped as in the app. The id is the `action` of the layer.

| Group | Images | Videos |
| --- | --- | --- |
| Input | Images Project `images_project`, Input Labeling Job `input_labeling_job`, Filtered Project `filtered_project` (started from filters only) | Videos Project `videos_project` |
| Pixel-level transforms | Anonymize, Blur, Contrast / Brightness, Noise, Random Color | — |
| Spatial-level transforms | Crop, Flip, Instances Crop, Multiply, Orientation, Resize, Rotate, Sliding Window | — |
| ImgAug Augmentations | ImgAug Studio, imgcorruptlike Noise / Blur / Weather / Color / Compression, Elastic Transformation, Perspective Transform | — |
| Annotation transforms | Approx Vector, Background, Bounding Box, BBox to Polygon, Bitwise Masks, Change Class Color, Drop Lines by Length, Drop Noise, Drop Object by Class, Duplicate Objects, Image Tag, Line to Mask, Mask Morphology, Mask to Lines, Mask to Polygon, Merge Classes, Merge Masks, Objects Filter, Objects Filter by Area, Polygon to Mask, Rasterize, Rename Classes, Skeletonize, Split Masks | Background, Bounding Box, BBox to Polygon |
| Video transforms | — | Split Video by Duration |
| Filters and conditions | Filter Images by Objects, Filter Images by Tags, Filter Images without Objects, If | Filter Videos by Objects, Filter Videos by Tags, Filter Videos without Object Classes, Filter Videos without Annotations, Filter Videos by Duration |
| Neural networks | Apply NN Inference, Deploy YOLOv5, Deploy YOLO v8 - v11, Deploy YOLO v8 - v26, Deploy MMDetection, Deploy MMSegmentation, Deploy RT-DETR, Deploy RT-DETRv2, Deploy DEIM | — |
| Other | Dataset, Split Data, Dummy, Copy, Move | — |
| Custom | your nodes from `src/custom_nodes/` | your nodes from `src/custom_nodes/` |
| Output | Output Project, Create New Project, Add to Existing Project, Export Archive, Export Archive with Masks, Copy Annotations, Create Labeling Job | Create New Project, Add to Existing Project, Export Archive, Create Labeling Job |

The documentation of every node, with its settings and their JSON, is in the node's folder in the repository and in the app (the **?** icon on the node).

## How the code is organized

| Path | What is there |
| --- | --- |
| `src/main.py` | the app: builds the initial canvas from the environment, the `run_pipeline` and `get_pipeline_status` endpoints |
| `src/ui/dtl/actions/<group>/<node>/` | the UI of each node (`Action` subclass), its settings widgets and its `README.md` |
| `src/ui/dtl/__init__.py` | the list of nodes for each modality and their groups |
| `src/ui/tabs/` | the canvas and sidebar (`configure.py`), **SAVE** / **LOAD** (`presets.py`), **RUN** (`run.py`) |
| `src/compute/layers/{data,processing,save}/` | the compute class of each node (`Layer` subclass): input, processing and output layers |
| `src/compute/Net.py` | the engine: builds the graph from the JSON, validates it, lists the items and pushes them through the layers |
| `src/compute/Layer.py` | the base class of every compute layer |
| `src/preconfigured/templates.py` | the built-in pipelines |
| `src/custom_nodes/` | your own nodes, registered automatically |

A node is a UI class and a compute class linked by the same name (`Action.name` and `Layer.action`). How to write one is in [Add your own node to ML Pipelines](custom-nodes.md).

## Run the app locally

Clone the repository, install `dev_requirements.txt` into a Python 3.12 virtual environment, put `TEAM_ID`, `WORKSPACE_ID`, `USER_ID` and `modal.state.modalityType` into `local.env`, and start it with `uvicorn src.main:app --host 0.0.0.0 --port 8000`. The app takes the server address and token from `~/supervisely.env` and works with the data on your instance. The steps are in Step 1 of [Add your own node to ML Pipelines](custom-nodes.md).
