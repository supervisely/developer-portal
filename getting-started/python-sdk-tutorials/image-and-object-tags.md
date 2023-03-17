---
description: How to tag image and object using Supervisely SDK
---

# Image and object tags

## Introduction

In this tutorial we will be focusing on working with tags using Supervisely SDK. We'll go through complete cycle from creating TagMeta in project to assigning Tags to images and objects directly.

You will learn:

1. how to create tags for different tasks and scenarios with various parameters.
2. how to create tags (`sly.TagMeta`) in project
3. how to assign tags (`sly.Tag`) to images and objects

📗 Everything you need to reproduce [this tutorial is on GitHub](https://github.com/supervisely-ecosystem/tutorial-working-with-tags): source code and demo data.

## How to debug this tutorial

**Step 1.** Prepare `~/supervisely.env` file with credentials. [Learn more here.](https://developer.supervise.ly/getting-started/basics-of-authentication#how-to-use-in-python)

**Step 2.** Clone [repository](https://github.com/supervisely-ecosystem/tutorial-working-with-tags) with source code and demo data and create [Virtual Environment](https://docs.python.org/3/library/venv.html).

```
git clone https://github.com/supervisely-ecosystem/tutorial-working-with-tags
cd tutorial-working-with-tags
./create_venv.sh
```

**Step 3.** Open repository directory in Visual Studio Code.

```
code -r .
```

**Step 4.** Get [**Lemons (Annotated)**](https://ecosystem.supervise.ly/projects/lemons-annotated) project from ecosystem. Lemons (Annotated) is an example project with annotated lemons and kiwi fruits, with 6 images in it.

<figure><img src="https://user-images.githubusercontent.com/48913536/193692418-731fa985-4958-4a42-893a-411e558faa04.png" alt=""><figcaption></figcaption></figure>

**Step 5.** change ✅ project ID ✅ in `local.env` file by copying the ID from the context menu of the project.

```
PROJECT_ID=111 # ⬅️ change value
```

<figure><img src="https://user-images.githubusercontent.com/48913536/193692408-6a1ba506-751b-4634-937e-3f2cebc2b22c.png" alt=""><figcaption></figcaption></figure>

**Step 6.** Start debugging `src/main.py`

## **Part 1.** Tag Meta

### Import libraries

```python
import os
from dotenv import load_dotenv
import supervisely as sly
```

### Init API client

Init API for communicating with Supervisely Instance. First, we load environment variables with credentials and project ID:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))
api = sly.Api()
```

### Create Tag Meta

TagMeta object contains general information about Tag. In order to create Tag itself you must create TagMeta object (information about tags that we will create and assign to images or objects) with parameters such as:

* name (required) - name of the Tag.
* value\_type (required)- restricts Tag to have a certain value type. Available value types:
  * NONE = "none"
  * ANY\_STRING = "any\_string"
  * ANY\_NUMBER = "any\_number"
  * ONEOF\_STRING = "oneof\_string"
* possible\_values (required if value type is "oneof\_string") - list of possible Tag values.
* color (optional) - color of the Tag, must be an RGB value, if not specified, random color will be generated.
* applicable\_to (optional) - defines if Tag can be assigned to only images, to only objects or both. By default tag can be assigned to both images and objects.
* applicable\_classes (optional) - defines applicability of Tag only to certain classes. List of strings (class names).

Let's start with creating a simple TagMeta for showcasing, it can be applied to both images and objects, and also to any class. We won't use it for our project later. Tags with value type "none" can be used as "train" and "val" tags for example.

```python
tag_meta = sly.TagMeta(
    name="simple_tag", 
    value_type=sly.TagValueType.NONE
    )
print(tag_meta)
# Name: simple_tag
# Value type: none
# Possible values: None
# Hotkey
# Applicable to all
# Applicable classes []
```

Let's make this TagMeta applicable only to images. We can recreate TagMeta with additional parameters. Most supervisely classes are immutable, so you have to assign or reassign them to variables.

```python
tag_meta = sly.TagMeta(
    name="simple_tag", 
    value_type=sly.TagValueType.NONE,
    applicable_to=sly.TagApplicableTo.IMAGES_ONLY
    )

print(tag_meta)
# Name: simple_tag
# Value type: none
# Possible values: None
# Hotkey
# Applicable imagesOnly
# Applicable classes []
```

Now let's create a few TagMetas with different value types that we will apply to our project.

We can start with creating fruit name TagMeta with "any\_string" value type. This Tag can be assigned only to objects of classes "lemon" and "kiwi".

```python
fruit_name_tag_meta = sly.TagMeta(
    name="name",
    applicable_to=sly.TagApplicableTo.OBJECTS_ONLY,
    value_type=sly.TagValueType.ANY_STRING,
    applicable_classes=["lemon", "kiwi"]
)
print(fruit_name_tag_meta)
# Name: name
# Value type: any_string
# Possible values: None
# Hotkey
# Applicable to objectsOnly
# Applicable classes ["lemon", "kiwi"]
```

Create fruit size TagMeta with "oneof\_string" value type. This Tag can be assigned only to objects of any classes and has possible values.

```python
fruit_size_tag_meta = sly.TagMeta(
    name="size",
    applicable_to=sly.TagApplicableTo.OBJECTS_ONLY,
    value_type=sly.TagValueType.ONEOF_STRING,
    possible_values=["small", "medium", "big"]
)
print(fruit_size_tag_meta)
# Name: size
# Value type: oneof_string
# Possible values: ["small", "medium", "big"]
# Hotkey
# Applicable to objectsOnly
# Applicable classes []
```

Now we create a TagMeta with "any\_string" value type to enter the origin of the fruit into it. This Tag can be assigned only to objects of classes "lemon" and "kiwi".

```python
fruit_origin_tag_meta = sly.TagMeta(
    name="imported_from", 
    value_type=sly.TagValueType.ANY_STRING, 
    applicable_to=sly.TagApplicableTo.OBJECTS_ONLY,
    applicable_classes=["lemon", "kiwi"]
    )
print(fruit_origin_tag_meta)
# Name: imported_from
# Value type: any_string
# Possible values: None
# Hotkey
# Applicable to objectsOnly
# Applicable classes ["lemon", "kiwi"]
```

And one more TagMeta with "any\_number" value type for counting total fruits on image. This Tag is applicable only to images.

```python
fruits_count_tag_meta = sly.TagMeta(
    name="fruits_count",
    value_type=sly.TagValueType.ANY_NUMBER,
    applicable_to=sly.TagApplicableTo.IMAGES_ONLY
)
print(fruits_count_tag_meta)
# Name: fruits_count
# Value type: any_number
# Possible values: None
# Hotkey
# Applicable to imagesOnly
# Applicable classes []
```

Bring all created TagMetas together in a list

```python
tag_metas = [fruit_name_tag_meta, fruit_size_tag_meta, fruit_origin_tag_meta, fruits_count_tag_meta]
```

## **Part 2.** Add TagMetas to project

Get project meta from server

```python
project_id = sly.env.project_id()
project_meta_json = api.project.get_meta(id=project_id)
project_meta = sly.ProjectMeta.from_json(data=project_meta_json)
```

<figure><img src="https://user-images.githubusercontent.com/48913536/193692433-806cb981-cc12-4d60-af25-9777b2bfe3f5.png" alt=""><figcaption></figcaption></figure>

Check that created Tag Metas don't already exist in project meta, and if not, add them to project meta.

```python
for tag_meta in tag_metas:
    if tag_meta not in project_meta.tag_metas:
        project_meta = project_meta.add_tag_meta(new_tag_meta=tag_meta)
```

Update project meta on Supervisely instance after adding Tag Metas to project meta.

```python
api.project.update_meta(id=project_id, meta=project_meta)
```

<figure><img src="https://user-images.githubusercontent.com/48913536/194032896-109661c5-ebf4-4b5d-9f66-60669c3d338d.png" alt=""><figcaption></figcaption></figure>

## **Part 3.** Create Tags and update annotation on server

```python
# get list of datasets in our project
datasets = api.dataset.get_list(project_id)
dataset_ids = [dataset.id for dataset in datasets]
# iterate over all images in project datasets
for dataset_id in dataset_ids:
    # get list of images in dataset
    images_infos = api.image.get_list(dataset_id=dataset_id)
    for image_info in images_infos:
        # get image id from image info
        image_id = image_info.id

        # download annotation
        ann_json = api.annotation.download_json(image_id=image_id)
        ann = sly.Annotation.from_json(data=ann_json, project_meta=project_meta)
        
        # create and assign Tag to image
        fruits_count_tag = sly.Tag(meta=fruits_count_tag_meta, value=len(ann.labels))
        ann = ann.add_tag(fruits_count_tag)

        # iterate over objects in annotation and assign appropriate tag
        new_labels = []
        for label in ann.labels:
            new_label = None
            if label.obj_class.name == "lemon":
                name_tag = sly.Tag(meta=fruit_name_tag_meta, value="lemon")
                size_tag = sly.Tag(meta=fruit_size_tag_meta, value="medium")
                origin_tag = sly.Tag(meta=fruit_origin_tag_meta, value="Spain")
                new_label = label.add_tags([name_tag, size_tag, origin_tag])
            elif label.obj_class.name == "kiwi":
                name_tag = sly.Tag(meta=fruit_name_tag_meta, value="kiwi")
                size_tag = sly.Tag(meta=fruit_size_tag_meta, value="small")
                origin_tag = sly.Tag(meta=fruit_origin_tag_meta, value="Italy")
                new_label = label.add_tags([name_tag, size_tag, origin_tag])
            if new_label:
                new_labels.append(new_label)

        # update and upload ann to Supervisely instance
        ann = ann.clone(labels=new_labels)
        api.annotation.upload_ann(img_id=image_id, ann=ann)
```

<figure><img src="https://user-images.githubusercontent.com/48913536/194032163-23103100-81fd-45f0-819f-4abc87256ccb.png" alt=""><figcaption></figcaption></figure>

## Advanced API

Advanced API allows user to manage tags directly on images or objects without downloading annotation data from server.

Get project meta again after updating it with new tags.

```python
project_meta_json = api.project.get_meta(id=project_id)
project_meta = sly.ProjectMeta.from_json(data=project_meta_json)
```

### Create TagCollection from image Tags without downloading annotation

Get image id from image info (see [part 3](image-and-object-tags.md#part-3.-create-tags-and-update-annotation-on-server))

```python
image_id = image_info.id
```

Get image tags

```python
image_tags = image_info.tags
print(f"{image_info.name} tags: {image_tags}")
# IMG_1836.jpeg tags: 
# [
#   {
#       'entityId': 3315606, 
#       'tagId': 369190,
#       'id': 2298259,
#       'value': 3,
#       'labelerLogin': 'cxnt',
#       'createdAt': '2022-10-04T15:43:12.155Z',
#       'updatedAt': '2022-10-04T15:43:12.155Z'
#   }
# ]
```

Create TagCollection from image tags

```python
tag_collection = sly.TagCollection().from_api_response(
    data=image_tags, tag_meta_collection=project_meta.tag_metas
)
print(tag_collection)
# +--------------+------------+-------+
# |     Name     | Value type | Value |
# +--------------+------------+-------+
# | fruits_count | any_number |   3   |
# +--------------+------------+-------+
```

### Add Tag directly to image

Get image tag ID from project meta

```python
fruits_count_tag_meta = project_meta.get_tag_meta("fruits_count")
```

Add Tag to image using Tag supervisely ID from project meta

```python
api.image.add_tag(image_id=image_id, tag_id=fruits_count_tag_meta.sly_id, value=3)
```

### Add Tags to objects directly

```python
ann_json = api.annotation.download_json(image_id=image_id)
ann = sly.Annotation.from_json(data=ann_json, project_meta=project_meta)
# iterate over objects in annotation and add appropriate tag
for label in ann.labels:
    # get figure sly id
    figure_id = label.geometry.sly_id
    # get tag sly id
    tag_meta = project_meta.get_tag_meta("imported_from")
    if label.obj_class.name == "lemon":
        api.advanced.add_tag_to_object(tag_meta_id=tag_meta.sly_id, figure_id=figure_id, value="Spain")
    elif label.obj_class.name == "kiwi":
        api.advanced.add_tag_to_object(tag_meta_id=tag_meta.sly_id, figure_id=figure_id, value="Italy")
```

### Add Tag to set of images
With api.image.add_tag_batch() method you can add a tag to a list of images without need to update annotation of each image one by one.

```python
# get tag meta from project meta
tag_meta = project_meta.get_tag_meta("fruits")

# create a list of images ids from images infos
image_ids = [image_info.id for image_info in images_infos]

# get tag meta id
tag_meta_id = tag_meta.sly_id

# update tags in batches.
api.image.add_tag_batch(image_ids, tag_meta_id, value=None, tag_meta=tag_meta)
```