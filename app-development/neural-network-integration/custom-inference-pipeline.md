# Custom inference pipeline

## Introduction

Sometimes it is needed to organize [custom data processing pipeline](./nn-training-deployment/nn-training-deployment.md#scheme-3-use-supervisely-as-backend-mlops-platform-for-organizing-custom-data---neural-network-pipelines) with using neural networks. This quide illustrates how to import image, process it with detection model and separate predictions with high and low confidences. 

In this tutorial, you'll learn how to infer deployed models **from your code** with the `sly.nn.inference.Session` class and process the images.
This class is a convenient wrapper for a low-level API. It under the hood is just a communication with the serving app via `requests`.

The entire integration Python script takes only 👍 95 lines of code (including comments) and can be found in [GitHub repository](https://github.com/supervisely-ecosystem/example-inference-session) for this tutorial.

**Table of Contents**:

- [Custom inference pipeline](#custom-inference-pipeline)
  * [Introduction](#introduction)
- [How to debug this tutorial](#how-to-debug-this-tutorial)
- [Tutorial](#python-code)
  * [1. Import libraries](#import-libraries)
  * [2. Init API client](#init-api-client)
  * [3. Initialize `sly.nn.inference.Session`](#initialize-sly.nn.inference.session)
  * [4. Create project](#create-project)
  * [5. Create and add new tags to the project metadata](#add-new-tags-to-the-project-metadata)
  * [6. Prepare source images](#prepare-image-links-and-classes-you-want-to-collect)
  * [7. Process images and predictions](#processing-images-and-object-detections)

**Before starting you have to deploy your model with a Serving App (e.g. [Serve YOLOv5](https://ecosystem.supervise.ly/apps/yolov5/supervisely/serve))**

## How to debug this tutorial

**Step 1.** Prepare  `~/supervisely.env` file with credentials. [Learn more here.](../../getting-started/basics-of-authentication.md#use-.env-file-recommended)

**Step 2.** Clone [repository](https://github.com/supervisely-ecosystem/example-inference-session) with source code and demo data and create [Virtual Environment](https://docs.python.org/3/library/venv.html).

```bash
git clone https://github.com/supervisely-ecosystem/example-inference-session
cd example-inference-session
./create_venv.sh
```

**Step 3.** Open repository directory in Visual Studio Code.&#x20;

```bash
code -r .
```

**Step 4.**   change ✅ workspace ID ✅ in `local.env` file by copying the ID from the context menu of the workspace. A new project with annotated images will be created in the workspace you define. [Learn more here.](../../getting-started//environment-variables.md#workspace_id)

```python
WORKSPACE_ID=680 # ⬅️ change value
```

![Copy workspace ID from context menu](https://user-images.githubusercontent.com/79905215/235677740-c117a63d-52a4-4524-8c10-34628557c588.gif)

**Step 5.** Start debugging `src/main.py`&#x20;

![Debug tutorial in Visual Studio Code](https://user-images.githubusercontent.com/79905215/235683475-23838c4c-29b1-4606-a29f-44095253e65a.gif)

## Python Code

### Import libraries

```python
import os

import supervisely as sly
from dotenv import load_dotenv
```

### Init API client

Init api for communicating with Supervisely Instance. First, we load environment variables with credentials and workspace ID:


```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))
api = sly.Api()
```

*(for more info see [Basics of authentication](../../getting-started/basics-of-authentication.md#use-.env-file-recommended) tutorial)*

With next lines we will check the you did everything right - API client initialized with correct credentials and you defined the correct workspace ID in `local.env`.

```python
workspace_id = sly.env.workspace_id()
workspace = api.workspace.get_info_by_id(workspace_id)
if workspace is None:
    print("you should put correct workspaceId value to local.env")
    raise ValueError(f"Workspace with id={workspace_id} not found")
```

### Initialize `sly.nn.inference.Session`

First serve the model you want (e.g. [Serve YOLOv5](https://ecosystem.supervise.ly/apps/yolov5/supervisely/serve)) and copy the `task_id` from the `App sessions` section in the Supervisely platform:

![Copy the Task ID here](https://user-images.githubusercontent.com/79905215/235680558-09372857-fcb9-49ff-971d-247ff025ec96.png)

**Create an Inference Session, a connection to the model:**

*(for more info see [Inference API](./inference-api-tutorial.md) tutorial)*

```python
# Get your Serving App's task_id from the Supervisely platform
task_id = 33172

# Create session
session = sly.nn.inference.Session(api, task_id=task_id)
```

### Create project

Create empty project with name **"Model predictions"** with one dataset **"Week # 1"** in your workspace on server. If the project with the same name exists in your dataset, it will be automatically renamed (_Week # 1\_001, Week # 1\_002, etc ..._) to avoid name collisions.

```python
project_info = api.project.create(workspace_id, "Model predictions", change_name_if_conflict=True)
dataset_info = api.dataset.create(project_info.id, "Week # 1")

print(f"Project has been sucessfully created, id={project_info.id}")
# Output: Project has been sucessfully created, id=20924
```


### Create 2 new tags: "high confidence" and "need validation"

```python
meta_high_confidence = sly.TagMeta("high confidence", sly.TagValueType.NONE)
high_confidence_tag = sly.Tag(meta_high_confidence)

meta_need_validation = sly.TagMeta("need validation", sly.TagValueType.NONE)
need_validation_tag = sly.Tag(meta_need_validation)
```

### Add new tags to the project metadata

Add tag metas to ProjectMeta.

```python
model_meta = model_meta.add_tag_metas(new_tag_metas=[meta_high_confidence, meta_need_validation])
```

Set up tags in our project on server:

```python
api.project.update_meta(id=project_info.id, meta=model_meta)
```

### Prepare image links and classes you want to collect

```python
links = [
    "https://live.staticflickr.com/1578/24294187606_89069ac7dd_k_d.jpg",
    "https://live.staticflickr.com/5491/9127573526_2999fafead_k_d.jpg",
    "https://live.staticflickr.com/6161/6175302372_76c4db94d0_k_d.jpg",
    "https://live.staticflickr.com/5601/15309578219_aa39bbfad2_k_d.jpg",
    "https://live.staticflickr.com/2465/3622848494_bad3b7ebe1_k_d.jpg",
    "https://live.staticflickr.com/557/19806156284_3ebb5a4046_k_d.jpg",
    "https://live.staticflickr.com/8403/8668991964_7969e1be9f_k_d.jpg",
    "https://live.staticflickr.com/1924/43556503550_f79978a134_k_d.jpg",
    "https://live.staticflickr.com/3799/20240807568_fcdab6a529_k_d.jpg",
    "https://live.staticflickr.com/7344/9886706776_16f9656162_k_d.jpg",
]
target_class_names = ["person", "bicycle", "car"]
```

### Processing images and object detections

It this section we will make predictions on images and applies tags based on the prediction confidence.
If the confidence of the current label is below 0.8, both the label and the current image will be tagged as "need validation," otherwise, the image will be tagged as "high confidence." 

{% hint style="success" %}
By setting tags based on the prediction confidence level, this script enables the separation of the dataset into "high confidence" and "need validation" images.
This allows for efficient and automated image processing. ✅
{% endhint %}

```python
CONFIDENCE_THRESHOLD = 0.8

for i, link in enumerate(links):
    # Upload current image from given link to Supervisely server
    image_info = api.image.upload_link(dataset_info.id, f"image_{i}.jpg", link)
    print(f"Image successfully uploaded, id={image_info.id}")

    # Get image inference
    prediction = session.inference_image_url(link)

    # Check confidence of predictions and set relevant tags.
    # If the prediction confidence is lower than the defined threshold,
    # both the image and the current label will be marked with the 'need validation' tag.
    image_need_validation = False
    new_labels = []

    for label in prediction.labels:
        # Skip the label if object class name is not in list of target class names.
        if label.obj_class.name not in target_class_names:
            continue
        confidence_tag = label.tags.get("confidence")
        if confidence_tag.value < CONFIDENCE_THRESHOLD:
            new_label = label.add_tag(need_validation_tag)
            image_need_validation = True
            new_labels.append(new_label)
        else:
            new_labels.append(label)

    prediction = prediction.clone(labels=new_labels)

    if image_need_validation is False:
        prediction = prediction.add_tag(high_confidence_tag)
    else:
        prediction = prediction.add_tag(need_validation_tag)

    api.annotation.upload_ann(image_info.id, prediction) # Upload annotations to server
```

![Result images with objects and tags](https://user-images.githubusercontent.com/79905215/235687566-4bfc7ce5-392e-49a1-9d77-f2c30091eb75.gif)
