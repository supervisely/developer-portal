# CompareAnnotations

## Introduction

**`CompareAnnotations`** is a simple widget that allows you to compare different annotations for one image. It can be useful for comparing annotations of different applied NN models. Widget doesn't support `Keypoints` shape. This widget is based on [GridGallery](https://developer.supervise.ly/app-development/widgets/media/gridgallery) widget.

## Function signature

```python
CompareAnnotations(
        columns_number=5,
        default_opacity=0.5,
        fill_rectangle=True,
        border_width=3,
        view_height=None,
        empty_message="No image was provided",
        widget_id=None,
)
```

![](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/4af434e6-44e7-481b-8a09-6e046f783c8d)

## Parameters

|     Parameters    |   Type  |                       Description                       |
| :---------------: | :-----: | :-----------------------------------------------------: |
|  `columns_number` |  `int`  |          Determines number of columns in widget         |
| `default_opacity` | `float` |                     Figures opacity                     |
|  `fill_rectangle` |  `bool` | If `False` labels with shape `Rectangle` will be hollow |
|   `border_width`  |  `int`  |                       Border width                      |
|   `view_height`   |  `int`  |             Set fixed gallery height in `px`            |
|  `empty_message`  |  `str`  |  If no image is given, this message will be displayed.  |
|    `widget_id`    |  `str`  |                     Id of the widget                    |

### columns\_number

Determines the number of columns on `CompareAnnotations`.

**type:** `int`

**default value:** `1`

```python
compare_annotations = CompareAnnotations(columns_number=2)
```

![](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/9afbac3c-2c83-4714-b869-bf76f5978e69)

### default\_opacity

Figures opacity.

**type:** `float`

**default value:** `0.5`

```python
compare_annotations = CompareAnnotations(columns_number=3, default_opacity=1)
```

![](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/12709230-e9c2-4159-bbf1-e2241b718778)

### fill\_rectangle

If `False` labels with shape `Rectangle` will be hollow.

**type:** `bool`

**default value:** `true`

```python
compare_annotations = CompareAnnotations(columns_number=3, fill_rectangle=False)
```

![](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/228e7b51-fb94-4bdc-adfd-8fbdf76a804e)

### border\_width

Determines border width to rectangle figures.

**type:** `int`

**default value:** `3`

```python
compare_annotations = CompareAnnotations(columns_number=3, border_width=10)
```

![](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/38810bc0-8736-45a2-b3b0-76b844f2b1f1)

### view\_height

Set fixed gallery height in `px`.

**type:** `int`

**default value:** `None`

```python
compare_annotations = CompareAnnotations(columns_number=3, view_height=300)
```

![](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/e1995878-18ee-4dca-8c1d-b43c72ab9b5c)

### empty\_message

If no image is given, this message will be displayed.

**type:** `str`

**default value:** `"No image was provided"`

```python
compare_annotations = CompareAnnotations(columns_number=3, empty_message="Set image URL and annotations")
```

![](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/b061e027-cf86-4f89-b8aa-b48897287656)

### widget\_id

ID of the widget.

**type:** `str`

**default value:** `None`

## Methods and attributes

| Attributes and Methods | Description                                |
| :--------------------: | ------------------------------------------ |
|      `image_url()`     | Returns current image URL                  |
|    `set_image_url()`   | Set current image by URL                   |
|       `append()`       | Add annotations to gallery                 |
|      `is_empty()`      | Checks whether image is set or not         |
|      `clean_up()`      | Clean up widget from image and annotations |

## Mini App Example

In this example, we will use `CompareAnnotations` widget to display different annotations for one image. We will modify and duplicate an existing image annotation and slightly alter the labels on it, to make them different. All labels will be converted to shape `Rectangle`.

You can find this example in our Github repository:

[supervisely-ecosystem/ui-widgets-demos/compare-data/005\_compare\_annotations/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/compare%20data/005\_compare\_annotations/src/main.py)

### Import libraries

```python
import os
from random import choice
import supervisely as sly
from dotenv import load_dotenv
from supervisely.app.widgets import (
    Button,
    Card,
    Container,
    CompareAnnotations,
)

from supervisely import Annotation, Rectangle
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
if sly.is_development():
    load_dotenv("local.env")
    load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()
```

### Get project ID and dataset ID

This app requires that you project have at least 2 images in it. If you don't have any projects, you can get one from [Supervisely Ecosystem](https://ecosystem.supervisely.com/projects).

```python
project_id = sly.env.project_id()
dataset_id = sly.env.dataset_id()
```

### Get images and annotations infos

```python
project_meta = sly.ProjectMeta.from_json(data=api.project.get_meta(id=project_id))
images_infos = api.image.get_list(dataset_id=dataset_id)
anns_infos = api.annotation.get_list(dataset_id=dataset_id)
```

### Option 1. Collect image names, URLs, annotations and generate annotation names from server

```python
image_names = []
image_urls = []
image_anns = []
for idx in range(len(images_infos)):
    image_names.append(images_infos[idx].name)
    image_urls.append(images_infos[idx].full_storage_url)
    image_anns.append(
        sly.Annotation.from_json(data=anns_infos[idx].annotation, project_meta=project_meta)
    )

ann_names = [f"Model inference {idx+1}" for idx in range(len(image_anns))]
```

### Option 2. You can also serve images from your local machine, using a static directory

Sort lists of images and annotations to make sure they are in the correct order.

```python
static_dir = Path("compare data/005_compare_annotations/images")
ann_dir = Path("compare data/005_compare_annotations/annotations")

image_urls = sorted([
    f"/static/{get_file_name_with_ext(path)}"
    for path in static_dir.iterdir()
    if path.is_file()
])

local_annotations = sorted([str(path) for path in ann_dir.iterdir() if path.is_file()])
```

Read annotations from local .json files

```python
image_anns = []
for local_ann in local_annotations:
    ann = sly.Annotation.from_json(
        data=sly.json.load_json_file(local_ann), project_meta=project_meta
    )
    image_anns.append(ann)
```

### Create a function to modify the annotation

```python
def ann_to_bbox(annotation: Annotation):
    labels = []
    mapping = {}
    for label in annotation.labels:
        if label.obj_class.name not in mapping:
            new_obj_class = sly.ObjClass(label.obj_class.name, Rectangle)
            mapping[label.obj_class] = new_obj_class
            label = label.scale(choice(range(90, 100)) / 100)
            labels.append(label)
    annotation = annotation.clone(labels=labels)
    annotation = annotation.to_detection_task(mapping)
    return annotation
```

### Iterate over column numbers and append modified annotations to widget

We iterate over column numbers for demo purposes to show all annotations in one line.

```python
for i in range(compare_annotations.columns_number):
    ann = ann_to_bbox(image_anns[0])
    compare_annotations.append(
        annotation=ann,
        title=ann_names[i],
        column_index=i,
    )
```

### Create buttons for changing current image and cleaning up widget

```python
change_image_btn = Button("Change image")
clean_up_btn = Button("Clean up")
```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter and place widget that we've just created in the `Container` widget.

```python
card = Card(
    "Compare Annotations",
    content=Container([compare_annotations, change_image_btn, clean_up_btn]),
)

layout = card
```

### Create app using layout

Create an app object with layout parameter.

```python
app = sly.Application(layout=layout)
```

In case you want to use a static directory, you need to pass it to the Application.

```python
app = sly.Application(layout=layout, static_dir=static_dir)
```

### Create a callback functions for buttons

```python
@clean_up_btn.click
def clean_up():
    compare_annotations.clean_up()


@change_image_btn.click
def set_image():
    compare_annotations.clean_up()
    rnd_idx = choice(range(len(images_infos)))
    compare_annotations.set_image_url(image_urls[rnd_idx])
    for i in range(compare_annotations.columns_number):
        ann = ann_to_bbox(image_anns[rnd_idx])
        compare_annotations.append(
            annotation=ann,
            title=f"{ann_names[i]}",
            column_index=i,
        )
```

![](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/2c5ac4dd-fb16-4084-bf2d-28208666a082)
