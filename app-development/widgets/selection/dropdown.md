# Dropdown

## Introduction

**`Dropdown`** is a widget in Supervisely that allows for selecting action from dropdown menu on the UI.

## Function signature

```python
Dropdown(
    items=None,
    trigger="hover",
    menu_align="end",
    hide_on_click=True,
    header="Dropdown List",
    widget_id=None,
)
```

Example of input data we will use.

```python
items = [
    Dropdown.Item(text="1"),
    Dropdown.Item(text="2"),
    Dropdown.Item(text="3"),
    Dropdown.Item(text="4"),
    Dropdown.Item(text="5"),
    Dropdown.Item(text="6"),
]

dropdown = Dropdown(items=items)
```

![dropdown_default](https://user-images.githubusercontent.com/120389559/227707948-0a29cf46-50f2-4198-8659-2c32892e8e23.gif)

## Parameters

|   Parameters    |            Type             |                  Description                  |
| :-------------: | :-------------------------: | :-------------------------------------------: |
|     `items`     |    `List[Dropdown.Item]`    |             Input `Dropdown` data             |
|    `trigger`    | `Literal["hover", "click"]` |        How to trigger `Dropdown` items        |
|  `menu_align`   |  `Literal["start", "end"]`  |             Horizontal alignment              |
| `hide_on_click` |           `bool`            | Whether to hide menu after clicking menu-item |
|    `header`     |            `str`            |               `Dropdown` header               |
|   `widget_id`   |            `str`            |               ID of the widget                |

### items

Determine input `Dropdown` data.

**type:** `List[Dropdown.Item]`

**default value:** `None`

### trigger

Determine how to trigger `Dropdown` items.

**type:** `Literal["hover", "click"]`

**default value:** `hover`

### menu_align

Determine horizontal alignment.

**type:** `Literal["start", "end"]`

**default value:** `end`

### hide_on_click

Determine whether to hide menu after clicking menu-item.

**type:** `bool`

**default value:** `True`

```python
dropdown = Dropdown(items=items, hide_on_click=False)
```

![hide_on_click](https://user-images.githubusercontent.com/120389559/227708228-465f44a7-8885-45e6-9595-2368fa2f5b97.gif)

### header

Determine `Dropdown` header.

**type:** `str`

**default value:** `"Dropdown List"`

```python
dropdown = Dropdown(items=items, header="Your text here")
```

![header](https://user-images.githubusercontent.com/120389559/227708344-d35ac75e-a732-426a-8fae-53487a114091.png)

## Methods and attributes

|         Attributes and Methods          | Description                                         |
| :-------------------------------------: | --------------------------------------------------- |
|              `get_value()`              | Return `Dropdown` selected values `command`.        |
|         `set_value(value: str)`         | Set `Dropdown` selected value `command`.            |
|              `get_items()`              | Return `Dropdown` items.                            |
| `set_items(value: List[Dropdown.Item])` | Set `Dropdown` items.                               |
| `add_items(value: List[Dropdown.Item])` | Add items in `Dropdown`.                            |
|           `get_header_text()`           | Return `Dropdown` header text.                      |
|      `set_header_text(value: str)`      | Set `Dropdown` header text.                         |
|            `@value_changed`             | Decorator function to handle selected value change. |

## Mini App Example

You can find this example in our Github repository:

[supervisely-ecosystem/ui-widgets-demos/selection/014_dropdown/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/selection/014_dropdown/src/main.py)

### Import libraries

```python
import os

import supervisely as sly
from dotenv import load_dotenv
from supervisely.app.widgets import Card, Container, Dropdown, Text
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()
```

### Prepare items for cascader

```python
items = [
    Dropdown.Item(text="1", command="A"),
    Dropdown.Item(text="2", divided=True, command="B"),
    Dropdown.Item(text="3", disabled=True, command="C"),
    Dropdown.Item(text="4", command="D"),
    Dropdown.Item(text="5", divided=True, command="E"),
    Dropdown.Item(text="6", command="F"),
]
```

### Initialize `Dropdown` and `Text` widgets

```python
dropdown = Dropdown(items=items, header="Example dropdown")

text = Text()
```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter.

```python
card = Card(
    "Dropdown",
    content=Container([dropdown, text]),
)
layout = Container(widgets=[card])
```

### Create app using layout

Create an app object with layout parameter.

```python
app = sly.Application(layout=card)
```

### Add functions to control widgets from python code

```python
@dropdown.value_changed
def show_item(res):
    info = f"Command {res} will be executed"
    text.set(text=info, status="info")
```

<p align="center">
  <img src="https://user-images.githubusercontent.com/120389559/227708677-c79e9c18-3496-484a-a181-a9c53fc5c1a8.gif" alt="layout" />
</p>
