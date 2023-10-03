# Carousel

## Introduction

**`Carousel`** is a widget in Supervisely that allows loop a series of images or texts in a limited space on the UI.

## Function signature

```python
Carousel(
    items: List[Carousel.Item],
    height=150,
    initial_index=0,
    trigger="hover",
    autoplay=True,
    interval=3000,
    indicator_position="none",
    arrow="hover",
    type=None,
    widget_id=None,
)
```

Example of input data we will use.

```python
# Example of local image
image_id = 22683828
static = os.path.join(sly.app.get_data_dir(), "static")
api.image.download_path(image_id, os.path.join(static, "image.jpg"))

items = [
    Carousel.Item(name="Slide 1", label="https://www.w3schools.com/howto/img_nature.jpg"), # image by URL
    Carousel.Item(name="Slide 2", label=f"{os.path.join('static', 'image.jpg')}"), # image from local directory
    Carousel.Item(name="Slide 3", label="Lorem ipsum dolor sit amet", is_link=False), # text
    Carousel.Item(name="Slide 4", label="https://www.quackit.com/pix/samples/18m.jpg"),
    Carousel.Item(name="Slide 5", label="https://i.imgur.com/OpSj3JE.jpg"),
]

carousel = Carousel(items=items)
```

![carousel_default](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/79905215/c3b786e2-5cf9-4278-8df5-b394dd660df0)

## Parameters

|      Parameters      |                   Type                    |                      Description                      |
| :------------------: | :---------------------------------------: | :---------------------------------------------------: |
|       `items`        |           `List[Carousel.Item]`           |                 Input `Carousel` data                 |
|       `height`       |                   `int`                   |               Height of the `Carousel`                |
|   `initial_index`    |                   `int`                   | Index of the initially active slide (starting from 0) |
|      `trigger`       |        `Literal["hover", "click"]`        |             How indicators are triggered              |
|      `autoplay`      |                  `bool`                   |         Whether automatically loop the slides         |
|      `interval`      |                   `int`                   |      Interval of the auto loop, in milliseconds       |
| `indicator_position` | `Union[Literal["outside", "none"], None]` |              Position of the indicators               |
|       `arrow`        |   `Literal["always", "hover", "never"]`   |                 When arrows are shown                 |
|        `type`        |      `Union[Literal["card"], None]`       |                Type of the `Carousel`                 |
|     `widget_id`      |                   `str`                   |                   ID of the widget                    |

### items

Determine input `Carousel` widget data.

**type:** `List[Carousel.Item]`

### height

Determine height of the `Carousel` widget.

**type:** `int`

**default value:** `350`

```python
carousel = Carousel(items=items, height=500)
```

![height](https://user-images.githubusercontent.com/120389559/227726162-3e00a9fb-2c74-4288-bb48-97d49c5355f9.gif)

### initial_index

Index of the initially active slide (starting from 0).

**type:** `int`

**default value:** `0`

### trigger

Determine how indicators are triggered.

**type:** `Literal["hover", "click"]`

**default value:** `"click"`

### autoplay

Determine whether automatically loop the slides.

**type:** `bool`

**default value:** `False`

### interval

Determine interval of the auto loop, in milliseconds.

**type:** `int`

**default value:** `3000`

```python
carousel = Carousel(items=items, interval=1000)
```

![interval](https://user-images.githubusercontent.com/120389559/227726892-627eb7b2-d3a3-4928-96b9-ef208acbc85b.gif)

### indicator_position

Determine position of the indicators.

**type:** `Literal["outside", "none"]`

**default value:** `"none"`

```python
carousel = Carousel(items=items, indicator_position="outside")
```

![indicator_position](https://user-images.githubusercontent.com/120389559/227727002-25c0bbe5-4786-493f-9426-05b05ce157e6.gif)

### arrow

Determine when arrows are shown.

**type:** `Literal["always", "hover", "never"]`

**default value:** `"hover"`

```python
carousel = Carousel(items=items, arrow="always")
```

![arrow](https://user-images.githubusercontent.com/120389559/227727145-68b5b62e-88e3-436f-bd63-7f8c016af887.gif)

### type

Determine type of the `Carousel`.

**type:** `Union[Literal["card"], None]`

**default value:** `None`

```python
carousel = Carousel(items=items, type="card")
```

![type](https://user-images.githubusercontent.com/120389559/227727229-0dd1bc3a-7312-4f8c-b589-6930d734ed21.gif)

## Methods and attributes

|         Attributes and Methods          | Description                                         |
| :-------------------------------------: | --------------------------------------------------- |
|           `get_active_item()`           | Return `Carousel` selected index.                   |
|              `get_items()`              | Return `Carousel` items.                            |
| `set_items(value: List[Carousel.Item])` | Set `Carousel` items.                               |
| `add_items(value: List[Carousel.Item])` | Add items in `Carousel`.                            |
|             `set_height()`              | Set `Carousel` height.                              |
|          `get_initial_index()`          | Return `Carousel` `initial_index` value.            |
|          `set_initial_index()`          | Set `Carousel` `initial_index` value.               |
|            `@value_changed`             | Decorator function to handle selected value change. |

## Mini App Example

You can find this example in our Github repository:

[supervisely-ecosystem/ui-widgets-demos/media/012\_carousel/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/media/012\_carousel/src/main.py)

### Import libraries

```python
import os

import supervisely as sly
from dotenv import load_dotenv
from supervisely.app.widgets import Card, Container, Carousel, Text
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()
```

### Prepare items for cascader

You can use images from local directory or image URL.

```python
image_id = 22683828
static = os.path.join(sly.app.get_data_dir(), "static")
api.image.download_path(image_id, os.path.join(static, "image.jpg"))

items = [
    Carousel.Item(name="Slide 1", label="https://www.w3schools.com/howto/img_nature.jpg"),
    Carousel.Item(name="Slide 2", label=f"{os.path.join('static', 'image.jpg')}"), # local images
    Carousel.Item(name="Slide 3", label="Lorem ipsum dolor sit amet", is_link=False), # text
    Carousel.Item(name="Slide 4", label="https://www.quackit.com/pix/samples/18m.jpg"),
    Carousel.Item(name="Slide 5", label="https://i.imgur.com/OpSj3JE.jpg"),
]
```

### Initialize `Carousel` and `Text` widgets

```python
carousel = Carousel(items=items, autoplay=False, trigger="click", height=500)

text = Text()
```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter.

```python
card = Card(
    "Carousel",
    content=Container([carousel, text]),
)
layout = Container(widgets=[card])
```

### Create app using layout

Create an app object with layout parameter.

```python
app = sly.Application(layout=layout, static_dir=static)
```

### Add functions to control widgets from python code

```python
@carousel.value_changed
def test(res):
    info = f"Current slide index: {res}"
    text.set(text=info, status="info")
```

<p align="center">
  <img src="https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/79905215/d1482d63-325b-4479-be61-e112f2f7fbac" alt="layout" />
</p>
