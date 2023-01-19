---
description: How to create keypoints annotation in Python using Supervisely
---

# Supervisely Data Labeling Example: Keypoints

## Introduction

In this tutorial we will show you how to use sly.GraphNodes class to create data annotation for pose estimation / keypoints detection task. The tutorial illustrates basic upload-download scenario:

* create project and dataset on server
* upload image
* programmatically create annotation and upload it to image
* download image and annotation

ℹ️ Everything you need to reproduce [this tutorial is on GitHub](https://github.com/supervisely-ecosystem/keypoints-labeling-example): source code, Visual Studio Code configuration, and a shell script for creating virtual env.

## How to debug this tutorial

**Step 1.** Prepare  `~/supervisely.env` file with credentials. [Learn more here.](https://developer.supervise.ly/getting-started/basics-of-authentication)

**Step 2.** Clone [repository](https://github.com/supervisely-ecosystem/keypoints-labeling-example) with source code and demo data and create [Virtual Environment](https://docs.python.org/3/library/venv.html).

```bash
git clone https://github.com/supervisely-ecosystem/keypoints-labeling-example
cd keypoints-labeling-example
./create_venv.sh
```

**Step 3.** Open repository directory in `Visual Studio Code`

```bash
code -r .
```

**Step 4.** Start debugging `src/main.py`
![vscode_screen](https://user-images.githubusercontent.com/91027877/212680680-a09293aa-6885-4a2e-a45b-ab3125be1f51.jpg)

## Python Code

## Importing Necessary Libraries

Import necessary libraries:

```python
import supervisely as sly
from supervisely.geometry.graph import Node, KeypointsTemplate
import os
import json
from dotenv import load_dotenv
```

Before we will start creating our project, let's learn how to create keypoints template - we are going to use it in our project.

## Working With Keypoints Template

We will need an image to create and visualize our keypoints template.

Image for building keypoints template:

![girl](https://user-images.githubusercontent.com/91027877/212680563-4b1ff700-461f-418d-9051-d50719dd404e.jpg)

Create keypoints template:
```python
# initialize template
template = KeypointsTemplate()
# add nodes
template.add_point(label="nose", row=635, col=427)
template.add_point(label="left_eye", row=597, col=404)
template.add_point(label="right_eye", row=685, col=401)
template.add_point(label="left_ear", row=575, col=431)
template.add_point(label="right_ear", row=723, col=425)
template.add_point(label="left_shoulder", row=502, col=614)
template.add_point(label="right_shoulder", row=794, col=621)
template.add_point(label="left_elbow", row=456, col=867)
template.add_point(label="right_elbow", row=837, col=874)
template.add_point(label="left_wrist", row=446, col=1066)
template.add_point(label="right_wrist", row=845, col=1073)
template.add_point(label="left_hip", row=557, col=1035)
template.add_point(label="right_hip", row=743, col=1043)
template.add_point(label="left_knee", row=541, col=1406)
template.add_point(label="right_knee", row=751, col=1421)
template.add_point(label="left_ankle", row=501, col=1760)
template.add_point(label="right_ankle", row=774, col=1765)
# add edges
template.add_edge(src="left_ankle", dst="left_knee")
template.add_edge(src="left_knee", dst="left_hip")
template.add_edge(src="right_ankle", dst="right_knee")
template.add_edge(src="right_knee", dst="right_hip")
template.add_edge(src="left_hip", dst="right_hip")
template.add_edge(src="left_shoulder", dst="left_hip")
template.add_edge(src="right_shoulder", dst="right_hip")
template.add_edge(src="left_shoulder", dst="right_shoulder")
template.add_edge(src="left_shoulder", dst="left_elbow")
template.add_edge(src="right_shoulder", dst="right_elbow")
template.add_edge(src="left_elbow", dst="left_wrist")
template.add_edge(src="right_elbow", dst="right_wrist")
template.add_edge(src="left_eye", dst="right_eye")
template.add_edge(src="nose", dst="left_eye")
template.add_edge(src="nose", dst="right_eye")
template.add_edge(src="left_eye", dst="left_ear")
template.add_edge(src="right_eye", dst="right_ear")
template.add_edge(src="left_ear", dst="left_shoulder")
template.add_edge(src="right_ear", dst="right_shoulder")
```

Visualize your keypoints template:
```python
template_img = sly.image.read("images/girl.jpg")
template.draw(image=template_img, thickness=7)
sly.image.write("images/template.jpg", template_img)
```

![template](https://user-images.githubusercontent.com/91027877/212680582-cb52d214-835d-4cf5-b61c-ba45704af6f1.jpg)

You can also transfer your template to json:
```python
template_json = template.to_json()
```

Now, when we have successfully created keypoints template, we can start creating keypoints annotation for our project.

## Programmatically Create Keypoints Annotation

Authenticate (learn more [here](https://developer.supervise.ly/getting-started/first-steps/basics-of-authentication)):
```python
load_dotenv(os.path.expanduser('~/supervisely.env'))
api = sly.Api.from_env()
my_teams = api.team.get_list()
team = my_teams[0]
workspace = api.workspace.get_list(team.id)[0]
```

Input image:

![person_with_dog](https://user-images.githubusercontent.com/91027877/212680598-8de619e1-ea2a-46d6-9a61-28e7669455dc.jpg)

Create project and dataset:
```python
project = api.project.create(workspace.id, "Human Pose Estimation", change_name_if_conflict=True)
dataset = api.dataset.create(project.id, "Person with dog", change_name_if_conflict=True)
print(f"Project {project.id} with dataset {dataset.id} are created")
```

Now let's create annotation class using our keypoints template as a geometry config (unlike other supervisely geometry classes, sly.GraphNodes requires geometry config to be passed - it is necessary for object class initialization):
```python
person = sly.ObjClass("person", geometry_type=sly.GraphNodes, geometry_config=template)
project_meta = sly.ProjectMeta(obj_classes=[person])
api.project.update_meta(project.id, project_meta.to_json())
```

You can also go to Supervisely platform and check that class with shape "Keypoints" was successfully added to your project:

![class_screen](https://user-images.githubusercontent.com/91027877/212680691-90cb1be2-956c-433b-a5cd-6ec6b5364f13.jpg)


Upload image:
```python
image_info = api.image.upload_path(
    dataset.id, name="person_with_dog.jpg", path="images/person_with_dog.jpg"
)
```

Build keypoints graph:
```python
nodes = [
    sly.Node(label="nose", row=146, col=670),
    sly.Node(label="left_eye", row=130, col=644),
    sly.Node(label="right_eye", row=135, col=701),
    sly.Node(label="left_ear", row=137, col=642),
    sly.Node(label="right_ear", row=142, col=705),
    sly.Node(label="left_shoulder", row=221, col=595),
    sly.Node(label="right_shoulder", row=226, col=738),
    sly.Node(label="left_elbow", row=335, col=564),
    sly.Node(label="right_elbow", row=342, col=765),
    sly.Node(label="left_wrist", row=429, col=555),
    sly.Node(label="right_wrist", row=438, col=784),
    sly.Node(label="left_hip", row=448, col=620),
    sly.Node(label="right_hip", row=451, col=713),
    sly.Node(label="left_knee", row=598, col=591),
    sly.Node(label="right_knee", row=602, col=715),
    sly.Node(label="left_ankle", row=761, col=573),
    sly.Node(label="right_ankle", row=766, col=709),
]
```

Label the image:
```python
input_image = sly.image.read("images/person_with_dog.jpg")
img_height, img_width = input_image.shape[:2]
label = sly.Label(sly.GraphNodes(nodes), person)
ann = sly.Annotation(img_size=[img_height, img_width], labels=[label])
api.annotation.upload_ann(image_info.id, ann)
```

You can check that keypoints annotation was successfully created in Annotation Tool:

![labelled](https://user-images.githubusercontent.com/91027877/212680735-5f356373-ea81-4f66-9898-7872d6573593.gif)

Download data:
```python
image = api.image.download_np(image_info.id)
ann_json = api.annotation.download_json(image_info.id)
```

Draw annotation:
```python
ann = sly.Annotation.from_json(ann_json, project_meta)
output_path = "images/person_with_dog_labelled.jpg"
ann.draw_pretty(image, output_path=output_path, thickness=7)
```

Result:

![person_with_dog_labeled](https://user-images.githubusercontent.com/91027877/212680609-ea1915da-dd8a-4305-9290-272d6b2a32e5.jpg)
