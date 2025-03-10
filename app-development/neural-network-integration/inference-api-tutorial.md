# Inference API

## Inference API

### Introduction

**There are two ways how you can infer your models:**

* From Supervisely platform with the APPs like [Apply NN to Images](https://ecosystem.supervisely.com/apps/nn-image-labeling/project-dataset) and [Apply Classifier to Images](https://ecosystem.supervisely.com/apps/apply-classification-model-to-project).
* Right from your code.

In this tutorial, you'll learn how to infer deployed models **from your code** with the `sly.nn.inference.Session` class. This class is a convenient wrapper for a low-level API. It under the hood is just a communication with the serving app via `requests`.

**Before starting you have to deploy your model with a Serving App (e.g.** [**Serve YOLOv5**](https://ecosystem.supervisely.com/apps/yolov5/supervisely/serve)**)**

Try with Colab: [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/supervisely-ecosystem/tutorial-inference-session/blob/master/nn\_inference\_tutorial\_colab.ipynb)

**Table of Contents**:

* [Inference API](inference-api-tutorial.md#inference-api)
  * [Introduction](inference-api-tutorial.md#introduction)
* [Quick overview](inference-api-tutorial.md#quick-overview)
  * [List of all inference methods](inference-api-tutorial.md#list-of-all-inference-methods)
    * [Image inference methods:](inference-api-tutorial.md#image-inference-methods)
    * [Video inference methods:](inference-api-tutorial.md#video-inference-methods)
* [A Complete Tutorial](inference-api-tutorial.md#a-complete-tutorial)
  * [1. Initialize `sly.nn.inference.Session`](inference-api-tutorial.md#1.-initialize-sly.nn.inference.session)
  * [2. Get the model info](inference-api-tutorial.md#2.-get-the-model-info)
    * [Session info](inference-api-tutorial.md#session-info)
    * [Model Meta. Classes and tags](inference-api-tutorial.md#model-meta.-classes-and-tags)
    * [Inference settings](inference-api-tutorial.md#inference-settings)
    * [Set the inference settings](inference-api-tutorial.md#set-the-inference-settings)
  * [3. Image Inference](inference-api-tutorial.md#3.-image-inference)
    * [Inspecting the model prediction](inference-api-tutorial.md#inspecting-the-model-prediction)
    * [Visualize model prediction](inference-api-tutorial.md#visualize-model-prediction)
    * [Upload prediction to the Supervisely platform](inference-api-tutorial.md#upload-prediction-to-the-supervisely-platform)
  * [4. Video Inference](inference-api-tutorial.md#4.-video-inference)
    * [Method 1. Inferring video with iterator](inference-api-tutorial.md#method-1.-inferring-video-with-iterator)
    * [Method 2. Inferring video without iterator](inference-api-tutorial.md#method-2.-inferring-video-without-iterator)
* [Advanced. Working with raw JSON output](inference-api-tutorial.md#advanced.-working-with-raw-json-output)

Let's start with a quick example of how you can connect and make inference of your model!

## Quick overview

_(for detailed tutorial go to the_ [_next section_](inference-api-tutorial.md#a-complete-tutorial)_)_

**Example usage: visualize prediction**

```python
import os
from dotenv import load_dotenv
import supervisely as sly


# Get your Serving App's task_id from the Supervisely platform
task_id = 27209

# init sly.Api
load_dotenv(os.path.expanduser("~/supervisely.env"))
api = sly.Api()

# Create Inference Session
session = sly.nn.inference.Session(api, task_id=task_id)

session.get_session_info()
```

```
{'app_name': 'Serve YOLOv5',
 'session_id': 27209,
 'model_files': '/sly-app-data/model/yolov5s.pt',
 'number_of_classes': 80,
 'sliding_window_support': 'advanced',
 'videos_support': True,
 'async_video_inference_support': True,
 'task type': 'object detection',
 'model_name': 'YOLOv5',
 'checkpoint_name': 'yolov5s',
 'pretrained_on_dataset': 'COCO train 2017',
 'device': 'cuda',
 'half': 'True',
 'input_size': 640}
```

```python
# Inference image_id
image_id = 19386161
prediction = session.inference_image_id(image_id)  # prediction is a `sly.Annotation` object

# Download and load the image that was inferred
save_path = "demo_image.jpg"
api.image.download_path(image_id, path=save_path)
image_np = sly.image.read(save_path)

# Draw the annotation and save it to the disk
save_path_predicted = "demo_image_pred.jpg"
predicted_annotation.draw_pretty(bitmap=image_np, output_path=save_path_predicted, fill_rectangles=False, thickness=7)
```

```python
# Show
from matplotlib import pyplot as plt
image_pred = sly.image.read(save_path_predicted)
plt.imshow(image_pred)
plt.axis('off')
```

![Image with predictions of the YOLOv5 model](https://user-images.githubusercontent.com/31512713/218431952-6183b5b0-19cc-4ba0-9ff9-0493b0bb4424.png)

### List of all inference methods

#### Image inference methods:

```python
# Infer single image by local path
pred = session.inference_image_path("image_01.jpg")

# Infer batch of images by local paths
pred = session.inference_image_paths(["image_01.jpg", "image_02.jpg"])

# Infer image by ID
pred = session.inference_image_id(17551748)

# Infer batch of images by IDs
pred = session.inference_image_ids([17551748, 17551750])

# Infer image by url
url = "https://images.unsplash.com/photo-1674552791148-c756b0899dba?ixlib=rb-4.0.3&ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&auto=format&fit=crop&w=387&q=80"
pred = session.inference_image_url(url)

# Infer the entire Project by ID
predictions = session.inference_project_id(18635)

# Infer the entire Project by ID with iterator
for ann_info in session.inference_project_id_async(18635):
    print(ann_info)
```

#### Video inference methods:

```python
from tqdm import tqdm

video_id = 18635803

# Infer video getting each frame as soon as it's ready
for frame_pred in tqdm(session.inference_video_id_async(video_id)):
    print(frame_pred)

# Infer video without iterator
pred = session.inference_video_id(video_id)
```

## A Complete Tutorial

### 1. Initialize `sly.nn.inference.Session`

First serve the model you want (e.g. [Serve YOLOv5](https://ecosystem.supervisely.com/apps/yolov5/supervisely/serve)) and copy the `task_id` from the `App sessions` section in the Supervisely platform:

![Copy the Task ID here](https://user-images.githubusercontent.com/31512713/218194505-b161be1e-5a05-488b-8eb7-9bc0f24141e2.png)

**Init your sly.Api:**

_(for more info see_ [_Basics of authentication_](../../getting-started/basics-of-authentication.md) _tutorial)_

```python
import os
from dotenv import load_dotenv
import supervisely as sly

# init sly.API
load_dotenv(os.path.expanduser("~/supervisely.env"))
api = sly.Api()
```

**Create an Inference Session, a connection to the model:**

```python
# Get your Serving App's task_id from the Supervisely platform
task_id = 27209

# create session
session = sly.nn.inference.Session(api, task_id=task_id)
```

**(Optional) You can pass the inference settings in init:**

```python
# pass settings by dict
inference_settings = {
    "conf_thres": 0.45
}
session = sly.nn.inference.Session(api, task_id=task_id, inference_settings=inference_settings)
```

Or with a `YAML` file:

```python
# pass settings by YAML
inference_settings_yaml = "settings.yml"
session = sly.nn.inference.Session(api, task_id=task_id, inference_settings=inference_settings_yaml)
```

### 2. Get the model info

#### Session info

Each app with a deployed model has its own unique **task\_id** (or **session\_id** which is the same), **model\_name**, **pretrained\_dataset** and other useful info that can be obtained with the `get_session_info()` method.

```python
session.get_session_info()
```

```
{'app_name': 'Serve YOLOv5',
 'session_id': 27209,
 'model_files': '/sly-app-data/model/yolov5s.pt',
 'number_of_classes': 80,
 'sliding_window_support': 'advanced',
 'videos_support': True,
 'async_video_inference_support': True,
 'task type': 'object detection',
 'model_name': 'YOLOv5',
 'checkpoint_name': 'yolov5s',
 'pretrained_on_dataset': 'COCO train 2017',
 'device': 'cuda',
 'half': 'True',
 'input_size': 640}
```

#### Model Meta. Classes and tags

The model may be pretrained on various datasets, like a **COCO**, **ImageNet** or even your **custom data**. Datasets are different in classes/tags they have. Therefore each dataset has its own meta information called `project_meta` in Supervisely. The model also contains this information and it's called `model_meta`. You can get the `model_meta` with method `get_model_meta()`:

```python
model_meta = session.get_model_meta()
print("The first 10 classes of the model_meta:")
[cls.name for cls in model_meta.obj_classes][:10]
```

```
The first 10 classes of the model_meta:

['person',
 'bicycle',
 'car',
 'motorcycle',
 'airplane',
 'bus',
 'train',
 'truck',
 'boat',
 'traffic light']
```

The `model_meta` will be used later, when we will visualize model predictions.

#### Inference settings

Each model has its own inference settings, like a `conf_thres`, `iou_thres` and others. You can get the full list of supported settings with `get_default_inference_settings()`:

```python
default_settings = session.get_default_inference_settings()
default_settings
```

```
{'conf_thres': 0.25,
 'iou_thres': 0.45,
 'augment': False,
 'debug_visualization': False}
```

#### Set the inference settings

**There are 3 ways to set the inference settings:**

* `update_inference_settings(**kwargs)`
* `set_inference_settings(dict)`
* `set_inference_settings(YAML)`

Also you can pass it earlier at creating the `Session`.

**a) Update only the parameters you need:**

```python
session.update_inference_settings(conf_thres=0.4, iou_thres=0.55)
session.inference_settings
```

Output:

```
{'conf_thres': 0.4, 'iou_thres': 0.55}
```

**b) Set parameters with a dict:**

```python
settings = {
    "conf_thres": 0.25
}
session.set_inference_settings(settings)
session.inference_settings
```

Output:

```
{'conf_thres': 0.25}
```

**c) Set parameters with a `YAML` file:**

```python
session.set_inference_settings("settings.yml")
session.inference_settings
```

Output:

```
{'conf_thres': 0.55, 'augment': False}
```

### 3. Image Inference

**There are several ways how to infer an image:**

* by Supervisely ID
* by local path
* by URL from the web

```python
# Infer image by local path
pred = session.inference_image_path("image_01.jpg")

# Infer image by ID
pred = session.inference_image_id(image_id=17551748)

# Infer image by url
url = "https://images.unsplash.com/photo-1674552791148-c756b0899dba?ixlib=rb-4.0.3&ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&auto=format&fit=crop&w=387&q=80"
pred = session.inference_image_url(url)
```

**And you can also infer a batch of images:**

```python
# Infer batch of images by local paths
pred = session.inference_image_paths(["image_01.jpg", "image_02.jpg"])

# Infer batch of images by IDs
pred = session.inference_image_ids([17551748, 17551750])
```

#### Inspecting the model prediction

The prediction is a `sly.Annotation` object. It contains all labels and tags for an image and can be uploaded directly to the Supervisely platform.

_(see more in_ [_SDK reference_](https://supervisely.readthedocs.io/en/latest/sdk/supervisely.annotation.annotation.Annotation.html#supervisely.annotation.annotation.Annotation)_)_

```python
image_id = 19386163
prediction = session.inference_image_id(image_id)
prediction.to_json()
```

```
{'annotation': {'description': '',
  'size': {'height': 800, 'width': 1200},
  'tags': [],
  'objects': [{'classTitle': 'umbrella',
    'description': '',
    'tags': [{'name': 'confidence', 'value': 0.85693359375}],
    'points': {'exterior': [[540, 363], [694, 468]], 'interior': []},
    'geometryType': 'rectangle',
    'shape': 'rectangle'},
   {'classTitle': 'car',
    'description': '',
    'tags': [{'name': 'confidence', 'value': 0.86376953125}],
    'points': {'exterior': [[724, 380], [1198, 708]], 'interior': []},
    'geometryType': 'rectangle',
    'shape': 'rectangle'},
   {'classTitle': 'person',
    'description': '',
    'tags': [{'name': 'confidence', 'value': 0.8740234375}],
    'points': {'exterior': [[562, 442], [661, 685]], 'interior': []},
    'geometryType': 'rectangle',
    'shape': 'rectangle'},
   {'classTitle': 'car',
    'description': '',
    'tags': [{'name': 'confidence', 'value': 0.89501953125}],
    'points': {'exterior': [[4, 408], [509, 699]], 'interior': []},
    'geometryType': 'rectangle',
    'shape': 'rectangle'}],
  'customBigData': {}},
 'data': {}}
```

#### Visualize model prediction

`sly.Annotation` has a `draw_pretty()` method for convenient visualization routines:

```python
# Draw the annotation and save it to disk
save_path_predicted = "demo_image_pred.jpg"
prediction.draw_pretty(bitmap=image_np, output_path=save_path_predicted, fill_rectangles=False, thickness=7)

# Show
from matplotlib import pyplot as plt
image_pred = sly.image.read(save_path_predicted)
plt.imshow(image_pred)
plt.axis('off');
```

![Image with predictions of the YOLOv5 model](https://user-images.githubusercontent.com/31512713/218431952-6183b5b0-19cc-4ba0-9ff9-0493b0bb4424.png)

#### Upload prediction to the Supervisely platform

**Now you can upload the image with predictions to the Supervisely platform:**

```python
workspace_id = 662

# Create new project and dataset
project_info = api.project.create(workspace_id, "My model predictions", change_name_if_conflict=True)
dataset_info = api.dataset.create(project_info.id, "First dataset")

# Update project meta with model's classes
api.project.update_meta(project_info.id, model_meta)
api.project.pull_meta_ids(project_info.id, model_meta)

# Upload the image
image_name = os.path.basename(image_path)
img_info = api.image.upload_path(dataset_info.id, name=image_name, path=image_path)

# Upload model predictions to Supervisely
api.annotation.upload_ann(img_info.id, prediction)
```

**Note:** when you update a `project_meta` with `api.project.update_meta()` the server generates ids for the classes and tags that have pushed for the first time and you have to update the `model_meta` too for the further uploading a prediction. This is where `api.project.pull_meta_ids()` method is helpful. It assigns the ids directly to the `model_meta` object. Because of all predictions have a reference to the `model_meta`, without this step we can't upload the predictions to the platform as predictions' `ProjectMeta` will not have the ids.

**Result on the Supervisely platform:**

![Result in the Supervisely Labeling Tool](https://user-images.githubusercontent.com/31512713/218533323-47ce4ee0-a2e2-480d-bfc0-06caa21ccc8a.png)

### 4. Video Inference

#### Method 1. Inferring video with iterator

**The video inference is simple too.**

The first way is to infer the video with `inference_video_id_async` method. It returns an iterator, which can be useful in processing predictions frame by frame. As soon as the model done with a one frame it will be yielded by the iterator:

```python
from tqdm import tqdm

video_id = 18635803

pred_frames = []
for frame_ann in tqdm(session.inference_video_id_async(video_id)):
    pred_frames.append(frame_ann)
```

There are some parameters can be passed to the video inference:

* `start_frame_index`: the first frame to start
* `frames_count`: total frames to infer
* `frames_direction`: video playback direction, either "forward" or "backward"

**Getting more information about the inference process:**

```python
video_id = 18635803
video_info = api.video.get_info_by_id(video_id)

frame_iterator = session.inference_video_id_async(video_id)
total_frames = video_info.frames_count
for i, frame_ann in enumerate(frame_iterator):
    labels = frame_ann.labels
    predicted_classes = [x.obj_class.name for x in labels]
    print(f"Frame {i+1}/{total_frames} done. Predicted classes = {predicted_classes}")
```

```
{"message": "The video is preparing on the server, this may take a while...", "timestamp": "2023-02-13T16:10:03.827Z", "level": "info"}
{"message": "Inference has started:", "progress": {"current": 0, "total": 10}, "is_inferring": true, "cancel_inference": false, "result": null, "pending_results": [], "timestamp": "2023-02-13T16:10:13.014Z", "level": "info"}


Frame 1/10 done. Predicted classes = ['car']
Frame 2/10 done. Predicted classes = ['car', 'car']
Frame 3/10 done. Predicted classes = ['car', 'car']
Frame 4/10 done. Predicted classes = ['car', 'car']
Frame 5/10 done. Predicted classes = ['car', 'car']
Frame 6/10 done. Predicted classes = ['car', 'car']
Frame 7/10 done. Predicted classes = ['car', 'car']
Frame 8/10 done. Predicted classes = ['car', 'car']
Frame 9/10 done. Predicted classes = ['car']
Frame 10/10 done. Predicted classes = ['car']
```

**Stop video inference**

If you need to stop the inference, use `session.stop_async_inference()`:

```python
from tqdm import tqdm

video_id = 18635803

for i, frame_ann in enumerate(tqdm(session.inference_video_id_async(video_id))):
    if i == 2:
        session.stop_async_inference()
```

```
{"message": "The video is preparing on the server, this may take a while...", "timestamp": "2023-02-09T23:15:47.232Z", "level": "info"}
{"message": "Inference has started:", "progress": {"current": 0, "total": 10}, "is_inferring": true, "cancel_inference": false, "result": null, "pending_results": [], "timestamp": "2023-02-09T23:15:55.878Z", "level": "info"}
 20%|██        | 2/10 [00:03<00:13,  1.63s/it]{"message": "Inference will be stopped on the server", "timestamp": "2023-02-09T23:16:01.559Z", "level": "info"}
 30%|███       | 3/10 [00:05<00:13,  1.88s/it]
```

#### Method 2. Inferring video without iterator

If you don't need to iterate every frame, you can use the `inference_video_id` method:

```python
video_id = 18635803

predictions_list = session.inference_video_id(
    video_id, start_frame_index=5, frames_count=15, frames_direction="forward"
)
```

**Note:** it is recommended to use this method for very small videos, because the code will wait until the whole video has been inferred and you even can't to track the progress.

### 5. Project Inference

#### Method 1. Inferring project with iterator

```python
from tqdm import tqdm

project_id = 18635

pred_ann_infos = []
for ann_info in tqdm(session.inference_project_id_async(project_id)):
    pred_ann_infos.append(ann_info)
```

There is extra parameter that can be passed to the project inference:

* `dest_project_id`: destination project id. If not passed, iterator will return annotation infos. If dest_project_id is equal to project_id, iterator will upload annotations to images in the project. If it is different from project_id, iterator will copy images and upload annotations to the new project.


#### Method 2. Inferring project without iterator

If you don't need to iterate every image, you can use the `inference_project_id` method:

```python
project_id = 18635

predictions_list = session.inference_project_id(project_id)
```

## Advanced. Working with raw JSON output

### SessionJSON

There is a `sly.nn.inference.SessionJSON` class which is useful when it needed to work with raw json outputs.

The class has all the same methods as `Session`, it just returns a raw JSONs.

The prediction is a `dict` with the following fields:

* `"annotation"`: contains a predicted annotation, that can be easily converted to `sly.Annotation`.
* `"data"`: additional metadata of the prediction. In most cases you won't need this.

```python
session = sly.nn.inference.SessionJSON(api, task_id=task_id)

prediction_json = session.inference_image_path("img/image_01.jpg")
prediction_json
```

```
{'annotation': {'description': '',
  'size': {'height': 1600, 'width': 1280},
  'tags': [],
  'objects': [{'classTitle': 'sheep',
    'description': '',
    'tags': [{'name': 'confidence', 'value': 0.255615234375}],
    'points': {'exterior': [[308, 1049], [501, 1410]], 'interior': []},
    'geometryType': 'rectangle',
    'shape': 'rectangle'},
  {'classTitle': 'person',
    'description': '',
    'tags': [{'name': 'confidence', 'value': 0.869140625}],
    'points': {'exterior': [[764, 272], [1062, 1002]], 'interior': []},
    'geometryType': 'rectangle',
    'shape': 'rectangle'},
  {'classTitle': 'horse',
    'description': '',
    'tags': [{'name': 'confidence', 'value': 0.87109375}],
    'points': {'exterior': [[393, 412], [1274, 1435]], 'interior': []},
    'geometryType': 'rectangle',
    'shape': 'rectangle'}],
  'customBigData': {}},
'data': {}}
```
