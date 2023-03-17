# Flexbox

## Introduction

**`Flexbox`** widget in Supervisely is a widget that enables users to arrange other widgets in a flexible and responsive layout. Users can customize the layout by setting the gap between widgets and aligning them to the center. With the `Flexbox` widget, users can easily create dynamic and adaptable layouts that can be optimized for different devices and screen sizes

## Function signature

```python
Flexbox(
    widgets=[Input(), Input()],
    gap=10,
    center_content=False,
    widget_id=None,
)
```

<figure><img src="https://user-images.githubusercontent.com/79905215/223953933-2d096050-3449-4e68-9ac9-71e50248e454.png" alt=""><figcaption></figcaption></figure>

## Parameters

|    Parameters    |      Type      |                           Description                           |
| :--------------: | :------------: | :-------------------------------------------------------------: |
|     `widgets`    | `List[Widget]` |             List if widgets to display on `Flexbox`             |
|       `gap`      |      `int`     |                 Gap between widgets on `Flexbox`                |
| `center_content` |     `bool`     | Determines whether to place widgets in the center of the window |
|    `widget_id`   |      `str`     |                         ID of the widget                        |

### widgets

Determine list of `Widgets` to display in `Flexbox` widget.

**type:** `List[Widget]`

```python
flexbox = Flexbox(widgets=[Input(), Input()])
```

<figure><img src="https://user-images.githubusercontent.com/79905215/223953933-2d096050-3449-4e68-9ac9-71e50248e454.png" alt=""><figcaption></figcaption></figure>

### gap

Determine gap between `Widgets` on `Flexbox`.

**type:** `int`

**default value:** `10`

```python
flexbox = Flexbox(
    widgets=[Input(), Input()],
    gap=25,
)
```

<figure><img src="https://user-images.githubusercontent.com/79905215/223955398-37eedd00-a26e-4118-b566-3863ffa7a983.png" alt=""><figcaption></figcaption></figure>

### center\_content

Determines whether to place widgets in the center of the window.

**type:** `bool`

**default value:** `False`

```python
flexbox = Flexbox(
    widgets=[Button(), Button()],
    center_content=True,
)
```

<figure><img src="https://user-images.githubusercontent.com/79905215/223955717-e37d1d3c-b94d-4e92-a570-8cef5b1133f0.png" alt=""><figcaption></figcaption></figure>

### widget\_id

ID of the widget.

**type:** `str`

**default value:** `None`

## Mini App Example

You can find this example in our Github repository:

[ui-widgets-demos/layouts and containers/005\_flexbox/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/layouts%20and%20containers/005\_flexbox/src/main.py)

#### Import libraries

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

<figure><img src="https://user-images.githubusercontent.com/120389559/218082231-76e037ec-095f-42f9-8f89-e387aed00360.png" alt=""><figcaption></figcaption></figure>
