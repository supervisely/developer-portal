# Flexbox

## Introduction

In this tutorial you will learn how to use `Flexbox` widget in Supervisely app.

[Read this tutorial in developer portal.](https://developer.supervise.ly/app-development/apps-with-gui/flexbox)

## Function signature

```python
Flexbox(widgets, gap=10, center_content=False, widget_id=None)
```

![default](https://user-images.githubusercontent.com/120389559/218078423-ef63be35-8c0e-4674-8071-4ed8a1f66a1c.png)

## Parameters

|    Parameters    |      Type      |                           Description                           |
| :--------------: | :------------: | :-------------------------------------------------------------: |
|    `widgets`     | `List[Widget]` |             List if widgets to display on `Flexbox`             |
|      `gap`       |     `int`      |                Gap between widgets on `Flexbox`                 |
| `center_content` |     `bool`     | Determines whether to place widgets in the center of the window |
|   `widget_id`    |     `str`      |                        Id of the widget                         |

### widgets

Determine list of `Widgets` to display on `Flexbox`.

**type:** `List[Widget]`

### gap

Determine gap between `Widgets` on `Flexbox`.

**type:** `int`

**default value:** `10`

```python
flexbox = Flexbox(widgets=obj_class_view_widgets, gap=25)
```

![gap](https://user-images.githubusercontent.com/120389559/218081572-1f7f6fd6-e518-4651-8373-d107304275f7.png)

### center_content

Determines whether to place widgets in the center of the window.

**type:** `bool`

**default value:** `false`

### widget_id

ID of the widget.

**type:** `str`

**default value:** `None`

## Mini App Example

You can find this example in our Github repository:

[supervisely-ecosystem/ui-widgets-demos/027_flexbox/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/027_flexbox/src/main.py)

### Import libraries

```python
import os

import supervisely as sly
from dotenv import load_dotenv
from supervisely.app.widgets import Card, Container, Flexbox, ObjectClassView
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()
```

### Initialize widgets we will use in UI

```python
obj_class_cat = sly.ObjClass(name="cat", geometry_type=sly.Bitmap, color=[255, 0, 0])
obj_class_dog = sly.ObjClass(name="dog", geometry_type=sly.Bitmap, color=[0, 255, 0])
obj_class_sheep = sly.ObjClass(name="sheep", geometry_type=sly.Bitmap, color=[0, 0, 255])
obj_class_horse = sly.ObjClass(name="horse", geometry_type=sly.Bitmap, color=[255, 255, 0])
obj_class_squirrel = sly.ObjClass(name="squirrel", geometry_type=sly.Bitmap, color=[255, 0, 255])

obj_classes = [
obj_class_cat,
obj_class_dog,
obj_class_sheep,
obj_class_horse,
obj_class_squirrel,
]

obj_class_view_widgets = [ObjectClassView(obj_class=obj_class) for obj_class in obj_classes]
```

### Initialize `Flexbox` widget

```python
flexbox = Flexbox(
    widgets=obj_class_view_widgets,
    gap=100,
    center_content=True,
)
```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter and place widget that we've just created in the `Container` widget.

```python
card = Card(
    title="Flexbox",
    content=flexbox,
)

layout = Container(widgets=[card])
```

### Create app using layout

Create an app object with layout parameter.

```python
app = sly.Application(layout=layout)
```

![layout](https://user-images.githubusercontent.com/120389559/218082231-76e037ec-095f-42f9-8f89-e387aed00360.png)
