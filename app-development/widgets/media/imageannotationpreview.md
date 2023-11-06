# ImageAnnotationPreview

## Introduction

**`ImageAnnotationPreview`** is a widget for displaying image annotation.

## Function signature

```python
image_preview = ImageAnnotationPreview(
    annotations_opacity = 0.5,
    enable_zoom = False,
    line_width = 1,
)
```

![](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/6f29084d-c71e-47f0-a962-12ac2f472f23)

## Parameters

|       Parameters      |   Type  |                  Description                  |
| :-------------------: | :-----: | :-------------------------------------------: |
| `annotations_opacity` | `float` |           Opacity of the annotation           |
|     `enable_zoom`     |  `bool` |         If `True` allows to zoom image        |
|      `line_width`     |  `int`  | Width of the annotation border (contour) line |
|      `widget_id`      |  `str`  |                ID of the widget               |

### annotations\_opacity

Annotation opacity. Value must be between 0 and 1. Set to 0 to hide annotations.

**type:** `float`

**default value:** `0.5`

```python
image_preview = ImageAnnotationPreview(
    annotations_opacity = 1,
)
```

![](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/87d6b835-b6cd-4e66-9479-6114d2c86d7c)

### enable\_zoom

If `True`, allows to zoom image with the mouse wheel.

**type:** `bool`

**default value:** `False`

```python
image_preview = ImageAnnotationPreview(
    enable_zoom = True,
)
```

### line\_width

Width of the annotation border (contour) line. Set to 0 to hide line.

**type:** `int`

**default value:** `1`

```python
image_preview = ImageAnnotationPreview(
    line_width = 0,
)
```

![](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/88af91d5-45a2-4a5c-93fd-1adef1b3721e)

### widget\_id

ID of the widget.

**type:** `str`

**default value:** `None`

## Methods and attributes

| Attributes and Methods | Description                                     |
| :--------------------: | ----------------------------------------------- |
|         `set()`        | Set image and annotation to widget.             |
|      `clean_up()`      | Clean up widget from image.                     |
|      `is_empty()`      | Return bool value, whether image is set or not. |

## Mini App Example

You can find this example in our Github repository:

[supervisely-ecosystem/ui-widgets-demos/media/015\_image\_annotation\_preview/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/media/015\_image\_annotation\_preview/src/main.py)

### Import libraries

```python
import os
from random import choice
import supervisely as sly
from dotenv import load_dotenv
from supervisely.app.widgets import Button, Card, Container, ImageAnnotationPreview
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()
```

### Initialize `Project ID`

```python
project_id = sly.env.project_id()
```

### Get Project ID and meta

```python
project_id = sly.env.project_id()
project_meta_json = api.project.get_meta(project_id)
project_meta = sly.ProjectMeta.from_json(project_meta_json)
```

### Get images from dataset

```python
dataset = api.dataset.get_list(project_id)[0]
images = api.image.get_list(dataset.id)
image = images[0]
```

### Get annotation for image

```python
ann_json = api.annotation.download(image.id).annotation
ann = sly.Annotation.from_json(ann_json, project_meta)
```

### Initialize ImageAnnotationPreview widget and set image with annotation and project meta

```python
image_preview = ImageAnnotationPreview(
    annotations_opacity=0.5,
    enable_zoom=False,
    line_width=1,
)
image_preview.set(
    image_url=image.preview_url,
    ann=ann,
    project_meta=project_meta
)
```

### Add button widget to show random image, we will use it later

```python
random_button = Button("Random image")
```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter and place widgets that we've just created in the `Container` widget.

```python
card = Card(
    title="ImageAnnotationPreview",
    content=Container([image_preview, random_button])
)
```

### Create app using layout

Create an app object with layout parameter.

```python
app = sly.Application(layout=layout)
```

![](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/990461f1-8f5c-4e12-9257-7fe36952e897)

### Add button click event to update preview

```python
@random_button.click
def set_random_image():
    random_image = choice(images)
    random_ann_json = api.annotation.download(random_image.id).annotation
    random_ann = sly.Annotation.from_json(
        random_ann_json,
        project_meta
    )

    image_preview.set(
        image_url=random_image.preview_url,
        ann=random_ann,
        project_meta=project_meta
    )
```
