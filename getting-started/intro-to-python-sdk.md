---
description: >-
  Let's try Supervisely SDK for Python and create your first python script for
  Supervisely automation.
---

# Intro to Python SDK

In this example we will show you how it is easy to communicate with your Supervisely instance (Community or your private Enterprise installation) from python code. The tutorial illustrates basic upload-download scenario:

* create project and dataset on server
* upload image
* programmatically create annotation (two bounding boxes and tag) and upload it to image
* download image and annotation

{% hint style="info" %}
You can try this example for yourself: VSCode project config, original image, and python script for this tutorial are ready on [GitHub](https://github.com/supervisely-ecosystem/supervisely-python-sdk-example).
{% endhint %}

Watch the video tutorial here:

{% embed url="https://youtu.be/Mp0BnWEujhA" %}
Video tutorial for beginners - introduction to Supervisely SDK for python developers
{% endembed %}

## Installation

Run the following command (learn more [here](installation.md))

```bash
pip install supervisely
```

## Input data

![Input image preview](https://user-images.githubusercontent.com/12828725/179228335-93ac7ec5-31e1-46da-b8fa-86d3bfe3b769.jpg)

## Python code

### Import and authentication

Import Supervisely, initialize API with your credentials and test authentication ([learn the basics of authentication here](basics-of-authentication.md)). In this example, we use the server address of Community Edition. Change it if you have a private instance of Supervisely.

```python
import json
import supervisely as sly

api = sly.Api(server_address="https://app.supervise.ly", token="4r47N...xaTatb")

my_teams = api.team.get_list()
print(f"I'm a member of {len(my_teams)} teams")

# get first team and workspace
team = my_teams[0]
workspace = api.workspace.get_list(team.id)[0]
```

### Create project on server

Let's create an empty project `animals` with one dataset `cats`, then one class `cat` of shape Rectangle and one tag `scene` with string value and upload them into the project. Now we can use created classes and tags for labeling.

```python
project = api.project.create(workspace.id, "animals", change_name_if_conflict=True)
dataset = api.dataset.create(project.id, "cats", change_name_if_conflict=True)
print(f"Project {project.id} with dataset {dataset.id} are created")

cat_class = sly.ObjClass("cat", sly.Rectangle, color=[0, 255, 0])
scene_tag = sly.TagMeta("scene", sly.TagValueType.ANY_STRING)
project_meta = sly.ProjectMeta(obj_classes=[cat_class], tag_metas=[scene_tag])

api.project.update_meta(project.id, project_meta.to_json())
```

### Upload image

Let's upload local image `images/my-cats.jpg` to dataset.

```python
image_info = api.image.upload_path(dataset.id, name="my-cats.jpg", path="images/my-cats.jpg")
```

### Create annotation and upload to image

```python
cat1 = sly.Label(sly.Rectangle(top=875, left=127, bottom=1410, right=581), cat_class)
cat2 = sly.Label(sly.Rectangle(top=549, left=266, bottom=1500, right=1199), cat_class) 
tag = sly.Tag(scene_tag, value="indoor")

ann = sly.Annotation(img_size=[1600, 1200], labels=[cat1, cat2], img_tags=[tag])
api.annotation.upload_ann(image_info.id, ann)
```

### Download data

```python
img = api.image.download_np(image_info.id)  # RGB ndarray
print("image shape (height, width, channels)", img.shape)

ann_json = api.annotation.download_json(image_info.id) 
print("annotaiton:\n", json.dumps(ann_json, indent=4))
```

{% hint style="info" %}
You can download the whole script using this [link](https://github.com/supervisely-ecosystem/supervisely-python-sdk-example/blob/master/main.py)
{% endhint %}

## Result

In less than 50 lines of code (including lots of comments) you can easily automate Supervisely using Python and integrate it with your software stack.

That’s just a taste of what you can do with the Supervisely SDK for Python. For more, take a look [at the reference](https://supervisely.readthedocs.io/en/latest/sdk\_packages.html) and [Supervisely Annotation JSON format](broken-reference).

![Result in labeling tool](https://user-images.githubusercontent.com/12828725/179226131-cd7f7058-ebca-4aa1-8660-951bf88a42af.png)
