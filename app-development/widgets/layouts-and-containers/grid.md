# Grid

## Introduction

**`Grid`** widget in Supervisely is a widget that enables users to arrange other widgets in a flexible and responsive grid layout. Users can customize the layout by setting the gap between widgets and number of columns. With the `Grid` widget, users can easily create dynamic and adaptable layouts that can be optimized for different devices and screen sizes

## Function signature

```python
Grid(
    widgets=[Input(), Input(), Input(), Input()],
    columns=2,
    gap=10,
    widget_id=None,
)
```

<figure><img src="https://user-images.githubusercontent.com/79905215/224012052-20054f6a-716c-4eff-8c75-bce3a0974eaf.png" alt=""><figcaption></figcaption></figure>

## Parameters

|  Parameters |      Type      |              Description             |
| :---------: | :------------: | :----------------------------------: |
|  `widgets`  | `List[Widget]` | List if widgets to display on `Grid` |
|  `columns`  |      `int`     |      Number of columns on `Grid`     |
|    `gap`    |      `int`     |     Gap between widgets on `Grid`    |
| `widget_id` |      `str`     |           ID of the widget           |

### widgets

Determine list of `Widgets` to display on `Grid`.

**type:** `List[Widget]`

```python
Grid(
    widgets=[Input()],
)
```

<figure><img src="https://user-images.githubusercontent.com/79905215/224015238-ede7e112-d1dd-4cfa-922d-547b43a2a311.png" alt=""><figcaption></figcaption></figure>

### columns

Number of columns on `Grid`.

**type:** `int`

**default value:** `1`

```python
grid = Grid(
    widgets=[Input(), Input(), Input(), Input()],
    columns=2,
)
```

<figure><img src="https://user-images.githubusercontent.com/79905215/224012052-20054f6a-716c-4eff-8c75-bce3a0974eaf.png" alt=""><figcaption></figcaption></figure>

### gap

Determine gap between `Widgets` on `Grid`.

**type:** `int`

**default value:** `10`

```python
grid = Grid(
    widgets=[Input(), Input(), Input(), Input()],
    gap=50,
)
```

<figure><img src="https://user-images.githubusercontent.com/79905215/224015899-c9bc0e72-f912-42f1-851b-aaddaf43a9e7.png" alt=""><figcaption></figcaption></figure>

```python
grid = Grid(
    widgets=[Input(), Input(), Input(), Input()],
    columns=3,
    gap=50,
)
```

<figure><img src="https://user-images.githubusercontent.com/79905215/224016117-99d1a1d9-934b-48f5-b6d8-05ec4515526b.png" alt=""><figcaption></figcaption></figure>

### widget\_id

ID of the widget.

**type:** `str`

**default value:** `None`

## Mini App Example

You can find this example in our Github repository:

[ui-widgets-demos/layouts and containers/006\_grid/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/layouts%20and%20containers/006\_grid/src/main.py)

### Import libraries

```python
import os

import supervisely as sly
from dotenv import load_dotenv
from supervisely.app.widgets import Card, Container, Grid, ObjectClassView
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

### Initialize `Grid` widget

```python
grid = Grid(
    widgets=obj_class_view_widgets,
    columns=3,
    gap=50,
)
```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter and place widget that we've just created in the `Container` widget.

```python
card = Card(
    title="Grid",
    content=grid,
)

layout = Container(widgets=[card])
```

### Create app using layout

Create an app object with layout parameter.

```python
app = sly.Application(layout=layout)
```

<figure><img src="https://user-images.githubusercontent.com/120389559/218116106-778e8c3e-8663-4b9a-96d7-3aef82190be8.png" alt=""><figcaption></figcaption></figure>
