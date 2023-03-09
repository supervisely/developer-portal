---
description: This guide explains how to bind (group) objects on images
---

# Objects binding

## Introduction

For some labeling scenarios, it is needed to group objects. Let's consider the case when you need to label object parts and then group them together. Such binding will help you distinguish parts of different objects. In this tutorial, you will learn how to programmatically group objects together (binding) and how to work with existing bindings.

{% hint style="info" %}
Everything you need to reproduce [this tutorial is on GitHub](https://github.com/supervisely-ecosystem/tutorial-object-binding): source code and demo data.
{% endhint %}

In this tutorial, we will create binding for the labeled parts of a single car:

<figure><img src="https://user-images.githubusercontent.com/12828725/193514807-3769b26a-bca5-4200-9698-a31117e0c280.gif" alt=""><figcaption><p>Imput image and masks</p></figcaption></figure>

This tutorial consists of two parts:

****[**Part 1**](objects-binding.md#part-1.-create-labels-with-binding-and-upload-them-to-server)**.** Create labels with binding and upload them to Supervisely server.&#x20;

****[**Part 2**](objects-binding.md#part-2-work-with-existing-binding)**.** Methods needed to work with existing bindings.

## How to debug this tutorial

**Step 1.** Prepare  `~/supervisely.env` file with credentials. [Learn more here.](../getting-started/basics-of-authentication.md#use-.env-file-recommended)

**Step 2.** Clone [repository](https://github.com/supervisely-ecosystem/tutorial-object-binding) with source code and demo data and create [Virtual Environment](https://docs.python.org/3/library/venv.html).

```bash
git clone https://github.com/supervisely-ecosystem/tutorial-object-binding
cd tutorial-object-binding
./create_venv.sh
```

**Step 3.** Open repository directory in Visual Studio Code.&#x20;

```bash
code -r .
```

**Step 4.**   change ✅ workspace ID ✅ in `local.env` file by copying the ID from the context menu of the workspace. A new project with demo data will be created in the workspace you define:

```python
WORKSPACE_ID=619 # ⬅️ change value
```

![Copy workspace ID from context menu](https://user-images.githubusercontent.com/12828725/181572645-f042c4d0-fcb5-48db-bf11-b74b3c37e031.gif)

**Step 5.** Start debugging `src/main.py`&#x20;

## **Part 1.** Create labels with binding and upload them to server

### Import libraries

```python
import os
from typing import List
from dotenv import load_dotenv
import cv2
import uuid
import supervisely as sly
```

### Init API client

Init API for communicating with Supervisely Instance. First, we load environment variables with credentials and workspace ID:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))
api = sly.Api()
```

### Create project

Create empty project with name **"tutorial-bindings"** with one dataset **"dataset-01"** in your workspace on server. If the project with the same name exists in your dataset, it will be automatically renamed (tutorial-bindings\_001, tutorial-bindings\_002, etc ...) to avoid name collisions.&#x20;

```python
workspace_id = int(os.environ"WORKSPACE_ID)

project = api.project.create(workspace_id, name="tutorial-bindings", change_name_if_conflict=True)
dataset = api.dataset.create(project.id, name="dataset-01")
print(f"Open project in browser: {project.url}")
```

### Define annotation classes

Let's define annotation classes and upload them to our new project on server:

```python
class_car = sly.ObjClass(name="car", geometry_type=sly.Bitmap, color=[255, 0, 255])
meta = sly.ProjectMeta(obj_classes=[class_car])
api.project.update_meta(project.id, meta)
```

### Upload demo image to server

```python
image_path = "images/image.jpg"
image_name = sly.fs.get_file_name_with_ext(image_path)
image_info = api.image.upload_path(dataset.id, image_name, image_path)
print(f"Image has been successfully uploaded: id={image_info.id}")

# will be needed later for creating annotation
height, width = cv2.imread(image_path).shape[0:2]
```

### Read masks and create sly.Annotation

More details about how to create labels can be found in [this tutorial](../getting-started/python-sdk-tutorials/spatial-labels.md).

```python
# create Supervisely annotation from masks images
labels_masks: List[sly.Label] = []
for mask_path in [
    "images/car_masks/car_01.png",
    "images/car_masks/car_02.png",
    "images/car_masks/car_03.png",
]:
    # read only first channel of the black-and-white image
    image_black_and_white = cv2.imread(mask_path)[:, :, 0]

    # supports masks with values (0, 1) or (0, 255) or (False, True)
    mask = sly.Bitmap(image_black_and_white)
    label = sly.Label(geometry=mask, obj_class=class_car)
    labels_masks.append(label)

ann = sly.Annotation(img_size=[height, width], labels=labels_masks)
```

### Create bindings

We know that all three masks are parts of a single car object. Let's bind them together. It is important to notice that any unique string can be label's `binding_key`.

```python
key = uuid.uuid4().hex  # key can be any unique string
for label in ann.labels:
    label.binding_key = key
```

### Upload annotation with binding to server

```python
api.annotation.upload_ann(image_info.id, ann)
```

As a result, we will have three objects of class car grouped together:

<figure><img src="https://user-images.githubusercontent.com/12828725/193523881-009c8550-0f8d-4300-8b19-5d04a07ec787.gif" alt=""><figcaption><p>Result annotation with binding</p></figcaption></figure>

## Part 2: Work with existing binding

### Download annotation

Let's download existing annotation (we created it in [part 1](objects-binding.md#part-1.-create-labels-with-binding-and-upload-them-to-server)) from server.

<pre class="language-python"><code class="lang-python">project_meta = sly.ProjectMeta.from_json(api.project.get_meta(project.id))
<strong>
</strong><strong>ann_json = api.annotation.download_json(image_info.id)
</strong>ann = sly.Annotation.from_json(ann_json, meta)</code></pre>

### Access to binding keys

If `binding_key` is None then the label does not belong to any group.

```python
print("Labels bindings:")
for label in ann.labels:
    print(label.binding_key)
```

The output will be the following:

```
Labels bindings:
62245779
62245779
62245779
```

### Access all binding groups in annotation

```python
groups = ann.get_bindings()
for i, (binding_key, labels) in enumerate(groups.items()):
    if binding_key is not None:
        print(f"Group # {i} [key={binding_key}] has {len(labels)} labels")
    else:
        # Binding key None defines all labels that do not belong to any binding group
        print(f"{len(labels)} labels do not have binding")
```

### Discard binding

Let's remove bindings on objects of class car:

```python
for label in ann.labels:
    if label.obj_class.name == "car":
        label.binding_key = None
```

Or you can discard binding for all labels in annotation:

```python
ann.discard_bindings()
```

Let's upload updated annotation (without bindings) back to the server:

```python
api.annotation.upload_ann(image_info.id, ann)
```
