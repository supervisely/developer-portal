---
description: How to create bounding boxes, masks on video frames in Python
---

# Video figures

## Introduction

In this tutorial, you will learn how to programmatically create classes and figures for vdeo frames and upload them to Supervisely platform. Supervisely supports different types of shapes / geometries for video annotation:

* bounding box (rectangle)
* mask (also known as bitmap)
* polygon - will be covered in other tutorials
* polyline - will be covered in other tutorials
* point - will be covered in other tutorials
* keypoints (also known as graph, skeleton, landmarks) - will be covered in other tutorials

Learn more [about Supervisely Annotation JSON format here](../../api-references/supervisely-annotation-json-format/).

![Bounding box and masks](https://user-images.githubusercontent.com/79905215/230330904-0a5eae31-db8d-4c0c-810a-c29d020a91ac.gif)

{% hint style="info" %}
Everything you need to reproduce [this tutorial is on GitHub](https://github.com/supervisely-ecosystem/video-figures): source code, Visual Studio Code configuration, and a shell script for creating virtual env.
{% endhint %}

## How to debug this tutorial

**Step 1.** Prepare  `~/supervisely.env` file with credentials. [Learn more here.](../basics-of-authentication.md#use-.env-file-recommended)

**Step 2.** Clone [repository](https://github.com/supervisely-ecosystem/video-figures) with source code and demo data and create [Virtual Environment](https://docs.python.org/3/library/venv.html).

```bash
git clone https://github.com/supervisely-ecosystem/video-figures
cd video-figures
./create_venv.sh
```

**Step 3.** Open repository directory in Visual Studio Code.&#x20;

```bash
code -r .
```

**Step 4.**   change ✅ workspace ID ✅ in `local.env` file by copying the ID from the context menu of the workspace. A new project with annotated videos will be created in the workspace you define:

```python
WORKSPACE_ID=507 # ⬅️ change value
```

![Copy workspace ID from context menu](https://user-images.githubusercontent.com/12828725/181572645-f042c4d0-fcb5-48db-bf11-b74b3c37e031.gif)

**Step 5.** Start debugging `src/main.py`&#x20;

![Debug tutorial in Visual Studio Code](https://user-images.githubusercontent.com/79905215/230344981-3734f92b-3cce-4209-b57d-3da8b0b33214.gif)

## Python Code

### &#x20;Import libraries

```python
import os
from os.path import join

from dotenv import load_dotenv

import supervisely as sly
```

### &#x20;Init API client

Init api for communicating with Supervisely Instance. First, we load environment variables with credentials and workspace ID:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))
api = sly.Api()
```

With next lines we will check the you did everything right - API client initialized with correct credentials and you defined the correct workspace ID in `local.env`.

```python
workspace_id = sly.env.workspace_id()
workspace = api.workspace.get_info_by_id(workspace_id)
if workspace is None:
    print("you should put correct workspaceId value to local.env")
    raise ValueError(f"Workspace with id={workspace_id} not found")
```

### Create project

Create empty project with name **"Demo"** with one dataset **"orange & kiwi"** in your workspace on server. If the project with the same name exists in your dataset, it will be automatically renamed (Demo\_001, Demo\_002, etc ...) to avoid name collisions.&#x20;

```python
project = api.project.create(
    workspace.id,
    name="Demo",
    type=sly.ProjectType.VIDEOS,
    change_name_if_conflict=True,
)
dataset = api.dataset.create(project.id, name="orange & kiwi")
print(f"Project has been sucessfully created, id={project.id}")
```

### Create annotation classes and video objects

Color will be automatically generated if the class was created without `color` argument.

```python
kiwi_obj_cls = sly.ObjClass("kiwi", sly.Rectangle, color=[0, 0, 255])
orange_obj_cls = sly.ObjClass("orange", sly.Bitmap, color=[255, 255, 0])

orange = sly.VideoObject(orange_obj_cls)
kiwi = sly.VideoObject(kiwi_obj_cls)
```

The next step is to create ProjectMeta - a collection of annotation classes and tags that will be available for labeling in the project.

```python
project_meta = sly.ProjectMeta(obj_classes=[kiwi_obj_cls, orange_obj_cls])
```

And finally, we need to set up classes in our project on server:

```python
api.project.update_meta(project.id, project_meta.to_json())
```

### Prepare source data

```python
video_path = "data/orange_kiwi.mp4"
masks_dir = "data/masks"

# prepare rectangle points for 10 demo frames
points = [
    [136, 632, 350, 817],
    [139, 655, 355, 842],
    [145, 672, 361, 864],
    [158, 700, 366, 885],
    [153, 700, 367, 885],
    [156, 724, 375, 914],
    [164, 745, 385, 926],
    [177, 770, 396, 944],
    [189, 793, 410, 966],
    [199, 806, 417, 980],
]
```

### Create masks, rectangles, frames and figures

We are going to create ten masks from the following black and white images:

![Three black-and-white masks for every orange](https://user-images.githubusercontent.com/79905215/230339269-0f1c20c3-d0a5-4f96-b661-bb3d92aa86d7.png)

{% hint style="info" %}
Mask has to be the same size as the video&#x20;
{% endhint %}

Supervisely SDK allows creating masks from NumPy arrays with the following values:

* `0` - nothing, `1` - pixels of target mask
* `0` - nothing, `255` - pixels of target mask
* `False` - nothing, `True` - pixels of target mask

```python
frames = []
for mask in os.listdir(masks_dir):
    fr_index = int(sly.fs.get_file_name(mask).split("_")[-1])
    mask_path = join(masks_dir, mask)

    # orange will be labeled with a masks.
    # supports masks with values (0, 1) or (0, 255) or (False, True)
    bitmap = sly.Bitmap.from_path(mask_path)

    # kiwi will be labeled with a bounding box.
    bbox = sly.Rectangle(*points[fr_index])

    mask_figure = sly.VideoFigure(orange, bitmap, fr_index)
    bbox_figure = sly.VideoFigure(kiwi, bbox, fr_index)

    frame = sly.Frame(fr_index, figures=[mask_figure, bbox_figure])
    frames.append(frame)
```

### Get video file info

```python
frame_size, vlength = sly.video.get_image_size_and_frames_count(video_path)
```

### Create `VideoObjectCollection` and `FrameCollcetion`

```python
objects = sly.VideoObjectCollection([kiwi, orange])
frames = sly.FrameCollection(frames)
```

### Create video annotation

```python
video_ann = sly.VideoAnnotation(
    img_size=frame_size,
    frames_count=vlength,
    objects=objects,
    frames=frames,
)
```

### Upload video with annotation

Upload video to the dataset on server:

```python
video_name = sly.fs.get_file_name_with_ext(video_path)
video_info = api.video.upload_path(dataset.id, video_name, video_path)
print(f"Video has been sucessfully uploaded, id={video_info.id}")
```

Upload annotation to the video on server:

```python
api.video.annotation.append(video_info.id, video_ann)
print(f"Annotation has been sucessfully uploaded to the video {video_name}")
```

In the [GitHub repository for this tutorial](https://github.com/supervisely-ecosystem/video-figures), you will find the [full python script](https://github.com/supervisely-ecosystem/video-figures/blob/master/src/main.py).

## Recap

In this tutorial we learned how to&#x20;

* quickly configure python development for Supervisely
* how to create a project and dataset with classes of different shapes
* how to initialize rectangles, masks for video frames
* how to construct Supervisely annotation and upload it with an videos to server
