---
description: How to create bounding boxes, polygons, masks, points and polylines in Python
---

# Spatial labels

## Introduction

In this tutorial, you will learn how to programmatically create classes and labels of different shapes and upload them to Supervisely platform. Supervisely supports different types of shapes / geometries for image annotation:

* bounding box (rectangle)
* polygon
* mask (also known as bitmap)
* polyline
* point
* keypoints (also known as graph, skeleton, landmarks) - will be covered in other tutorials
* cuboids - will be covered in other tutorials &#x20;

![Polygon, Rectangle and Masks](https://user-images.githubusercontent.com/12828725/181513800-0883846d-8916-422f-8e3a-e42eb2a1a961.gif)

![Points and polyline](https://user-images.githubusercontent.com/12828725/181513722-1d8e44ad-9580-460c-aebe-8e836920cc1b.png)

{% hint style="info" %}
Everything you need to reproduce [this tutorial is on GitHub](https://github.com/supervisely-ecosystem/spatial-labels): source code, Visual Studio Code configuration, and a shell script for creating virtual env.
{% endhint %}

## How to debug this tutorial

**Step 1.** Prepare `.env` files with credentials and ID of a demo project. [Learn more here](../intro-to-python-sdk.md).

**Step 2.** Clone [repository](https://github.com/supervisely-ecosystem/spatial-labels) with source code and demo data and create [Virtual Environment](https://docs.python.org/3/library/venv.html).

```bash
git clone https://github.com/supervisely-ecosystem/spatial-labels
cd spatial-labels
./create_venv.sh
```

**Step 3.** Open repository directory in Visual Studio Code.&#x20;

```bash
code -r .
```

**Step 4.** New project with annotated images will be created in the workspace you define in `local.env` file.&#x20;

Just ✅ change workspace ID in  `local.env` file ✅ by copying the ID from the context menu of the workspace:

```
context.workspaceId=506 # ⬅️ change value
```

![](https://user-images.githubusercontent.com/12828725/181525295-f9f85fde-a90b-4322-80db-be8c497594c0.gif)

**Step 5.** Start debugging `src/main.py`

## Python Code

### &#x20;Import libraries

```python
import os
import cv2
import supervisely as sly
from dotenv import load_dotenv
```

### &#x20;Init API client

Init api for communicating with Supervisely Instance. First, we load environment variables with credentials and workspace ID:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))
api = sly.Api.from_env()
```

With next lines we will check the you did everything right - API client initialized with correct credentials and you defined the correct workspace ID in `local.env`.

```python
workspace_id = int(os.environ["context.workspaceId"])
workspace = api.workspace.get_info_by_id(workspace_id)
if workspace is None:
    print("you should put correct workspaceId value to local.env")
    raise ValueError(f"Workspace with id={workspace_id} not found")
```

### Create project

Create empty project with name **"Demo"** with one dataset **"berries"** in your workspace on server. If the project with the same name exists in your dataset, it will be automatically renamed (Demo\_001, Demo\_002, etc ...) to avoid name collisions.&#x20;

```python
project = api.project.create(workspace.id, name="Demo", change_name_if_conflict=True)
dataset = api.dataset.create(project.id, name="berries")
print(f"Project has been sucessfully created, id={project.id}")
```

### Create annotation classes

```python
strawberry = sly.ObjClass("strawberry", sly.Rectangle, color=[0, 0, 255])
raspberry = sly.ObjClass("raspberry", sly.Polygon, color=[0, 255, 0])
blackberry = sly.ObjClass("blackberry", sly.Bitmap, color=[255, 255, 0])
berry_center = sly.ObjClass("berry_center", sly.Point, color=[0, 255, 255])
separator = sly.ObjClass("separator", sly.Polyline)  # color will be generated randomly
```

Color will be automatically generated if the class was created without `color` argument.

The next step is to create ProjectMeta - a collection of annotation classes and tags that will be available for labeling in the project.

```python
project_meta = sly.ProjectMeta(
    obj_classes=[strawberry, raspberry, blackberry, berry_center, separator]
)
```

And finally, we need to set up classes in our project on server:

```python
api.project.update_meta(project.id, project_meta.to_json())
```

### Create rectangle

Strawberry will be labeled with a bounding box.

```python
bbox = sly.Rectangle(top=127, left=1726, bottom=1087, right=2560)
label1 = sly.Label(geometry=bbox, obj_class=strawberry)
```

### Create polygon

Raspberry will be labeled with a polygon.

```python
polygon = sly.Polygon(
    exterior=[
        [941, 663],  # row, col
        [976, 874],
        [934, 1096],
        [819, 1196],
        [698, 1228],
        [527, 1081],
        [439, 1090],
        [331, 980],
        [359, 808],
        [452, 698],
        [549, 612],
        [762, 564],
        [879, 605],
    ]
)
label2 = sly.Label(geometry=polygon, obj_class=raspberry)
```

### Create masks

Every blackberry will be labeled with a mask. So we are going to create three masks from the following black and white images:

![Three black-and-white masks for every blackberry](https://user-images.githubusercontent.com/12828725/181560719-6e4ea40d-23f0-4841-a3fa-5511da4debe1.gif)

Supervisely SDK allows creating masks from NumPy arrays with the following values:

* `0` - nothing, `1` - pixels of target mask
* `0` - nothing, `255` - pixels of target mask
* `False` - nothing, `True` - pixels of target mask

{% hint style="info" %}
Mask has to be the same size as the image&#x20;
{% endhint %}

```python
labels_masks = []
for mask_path in [
    "data/masks/blackberry_01.png",
    "data/masks/blackberry_02.png",
    "data/masks/blackberry_03.png",
]:
    # read only first channel of an image
    image_black_and_white = cv2.imread(mask_path)[:, :, 0]
    
    # supports masks with values (0, 1) or (0, 255) or (False, True)
    mask = sly.Bitmap(image_black_and_white)
    label = sly.Label(geometry=mask, obj_class=blackberry)
    labels_masks.append(label)
```
