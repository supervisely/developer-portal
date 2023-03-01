# One of

## Introduction

**`OneOf`**  is a container widget that can hold one of several child widgets, such as `Radio`, `Select`, or `Switch`. Depending on the value selected by the user within these child widgets, the corresponding item will be added to the page markup. It is very convenient for those who are involved in object annotation on images, as it allows them to quickly switch between different types of annotation without losing sight of the overall picture.

[Read this tutorial in developer portal.](https://developer.supervise.ly/app-development/apps-with-gui/one-of)

## Function signature

```python
one_of = OneOf(
    conditional_widget=Select(items=animals),
    widget_id=None
)
```

![sheeps-one-of](https://user-images.githubusercontent.com/79905215/218075609-0428af83-0ef1-492b-8623-fa7a7bd0d3de.png)

## Parameters

|     Parameters     |       Type        |                             Description                             |
| :----------------: | :---------------: | :-----------------------------------------------------------------: |
| `conditional_widget` | `ConditionalWidget` | conditional widget with preset items (`Select`, `RadioButton`, `Switch`) |
|     `widget_id`      |        `str`        |                          ID of the widget                           |

### conditional_widget

Conditional widget with preset items.

**type:** `ConditionalWidget`

```python
# prepare "animals" using RadioGroup.Item
one_of = OneOf(
    conditional_widget=RadioGroup(items=animals)
)
```

![horse-one-of](https://user-images.githubusercontent.com/79905215/218075942-d2754ba6-0b9c-4572-b619-9363a2eecaf3.png)

### widget_id

ID of the widget.

**type:** `str`

**default value:** `None`

## Mini App Example

You can find this example in our Github repository:

[supervisely-ecosystem/ui-widgets-demos/017_one_of/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/017_one_of/src/main.py)

### Import libraries

```python
import os

import supervisely as sly
from dotenv import load_dotenv
from supervisely.app.widgets import Card, Container, OneOf, Select, Image
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()
```

### Prepare conditional widget content

You can use `RadioGroup`, `Select` or `Switch` widgets to set in `conditional_widget` parameter.
In this tutorial we will use `Select` widget. 

Prepare sample images

```python
image_1 = Image(
    url="https://user-images.githubusercontent.com/79905215/218269544-2e126d4a-20eb-4ace-8933-d36732bb0634.jpeg"
)
image_2 = Image(
    url="https://user-images.githubusercontent.com/79905215/218269547-5b5316f9-9ae2-4b0c-aedb-b2238e44f95d.jpeg"
)
image_3 = Image(
    url="https://user-images.githubusercontent.com/79905215/218269550-a5caba65-1f0f-4986-8711-7d36c7911e51.jpeg"
)
```

Prepare items for `Select` widget using `Select.Item`. [Learn more](https://github.com/supervisely-ecosystem/ui-widgets-demos/tree/master/009_select)

```python
items = [
    Select.Item(value="image 1", label="image 1", content=image_1),
    Select.Item(value="image 2", label="image 2", content=image_2),
    Select.Item(value="image 3", label="image 3", content=image_3),
]
```

Create `Select` widget.

```python
select_items = Select(items=items)
```


### Initialize `OneOf` widget

```python
one_of = OneOf(conditional_widget=select_items)
```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter and place widget that we've just created in the `Container` widget.

```python
card = Card(
    title="One of",
    content=Container(widgets=[select_items, one_of]),
)

layout = Container(widgets=[card])
```

### Create app using layout

Create an app object with layout parameter.

```python
app = sly.Application(layout=layout)
```

![one-of-gif](https://user-images.githubusercontent.com/79905215/218269955-86b5bb95-f242-4e05-9bc6-be86e633f2b1.gif)
