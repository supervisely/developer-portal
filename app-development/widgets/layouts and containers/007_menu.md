# Menu

## Introduction

In this tutorial you will learn how to use `Menu`, `Menu.Item`, `Menu.Group` widgets in Supervisely app.

[Read this tutorial in developer portal.](https://developer.supervise.ly/app-development/apps-with-gui/menu)

## Function signature

```python
Menu(items=None, groups=None, index=None, width_percent=25, widget_id=None)
```

![default](https://user-images.githubusercontent.com/120389559/220916979-223e32ad-ed9a-4382-be74-148b2f792012.gif)

## Parameters

|   Parameters    |        Type        |                Description                |
| :-------------: | :----------------: | :---------------------------------------: |
|     `items`     | `List[Menu.Item]`  |          List of `Menu` `Items`           |
|    `groups`     | `List[Menu.Group]` |          List of `Menu` `Groups`          |
|     `index`     |       `str`        | Sets the active `Menu` at the given index |
| `width_percent` |       `int`        |   Width of the left part of `Menu` in %   |
|   `widget_id`   |       `str`        |             Id of the widget              |

### items

Determine list of menu `Items`.

**type:** `List[Menu.Item]`

**default value:** `None`

```python
l = Text(text="left part", status="success")
items = [
    Select.Item(label="CPU", value="cpu"),
    Select.Item(label="GPU 0", value="cuda:0"),
    Select.Item(value="option3"),
]
r = Select(items=items, filterable=True)

items = [
    Menu.Item(title="m1", content=r),
    Menu.Item(title="m2", content=l),
]
menu = Menu(items=items)
```

![items](https://user-images.githubusercontent.com/120389559/220916979-223e32ad-ed9a-4382-be74-148b2f792012.gif)

### groups

Determine list of menu `Groups`.

**type:** `List[Menu.Group]`

**default value:** `None`

```python
l = Text(text="left part", status="success")
items = [
    Select.Item(label="CPU", value="cpu"),
    Select.Item(label="GPU 0", value="cuda:0"),
    Select.Item(value="option3"),
]
r = Select(items=items, filterable=True)

g1_items = [
    Menu.Item(title="m1", content=r),
    Menu.Item(title="m2", content=l),
]
g2_items = [
    Menu.Item(title="m3", content=Empty()),
    Menu.Item(title="m4"),
]
g1 = Menu.Group("group_1", g1_items)
g2 = Menu.Group("group_2", g2_items)
menu = Menu(groups=[g1, g2])
```

![groups](https://user-images.githubusercontent.com/120389559/218475188-7c5189bf-68d6-4e3c-a640-2132809c77ff.gif)

### index

Sets the active `Menu` at the given index.

**type:** `str`

**default value:** `None`

```python
l = Text(text="left part", status="success")
items = [
    Select.Item(label="CPU", value="cpu"),
    Select.Item(label="GPU 0", value="cuda:0"),
    Select.Item(value="option3"),
]
r = Select(items=items, filterable=True)

items = [
    Menu.Item(title="m1", content=r),
    Menu.Item(title="m2", content=l, index="m2"),
]
menu = Menu(items=items, index="m2")
```

![index](https://user-images.githubusercontent.com/120389559/218475782-59445b09-bc56-4a90-8825-1a79aa6f8ccb.png)

### width_percent

Determines width of the left part of `Menu` in %.

**type:** `int`

**default value:** `25`

```python
l = Text(text="left part", status="success")
items = [
    Select.Item(label="CPU", value="cpu"),
    Select.Item(label="GPU 0", value="cuda:0"),
    Select.Item(value="option3"),
]
r = Select(items=items, filterable=True)

items = [
    Menu.Item(title="m1", content=r),
    Menu.Item(title="m2", content=l),
]
menu = Menu(items=items, width_percent=75)
```

![width_percent](https://user-images.githubusercontent.com/120389559/218476194-55a6196c-5a8d-40e1-9461-1a2f38dfa1b0.png)

### widget_id

ID of the widget.

**type:** `str`

**default value:** `None`

## Mini App Example

You can find this example in our Github repository:

[supervisely-ecosystem/ui-widgets-demos/038_menu/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/038_menu/src/main.py)

### Import libraries

```python
import os

import supervisely as sly
from dotenv import load_dotenv
from supervisely.app.widgets import Empty, Menu, Select, Text
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()
```

### Initialize `Menu` items

```python
l = Text(text="left part", status="success")
items = [
    Select.Item(label="CPU", value="cpu"),
    Select.Item(label="GPU 0", value="cuda:0"),
    Select.Item(value="option3"),
]
r = Select(items=items, filterable=True, placeholder="select me")

g1_items = [
    Menu.Item(title="m1", content=r),
    Menu.Item(title="m2", content=l),
]
g2_items = [
    Menu.Item(title="m3", content=Empty()),
    Menu.Item(title="m4"),
]
```

### Initialize `Menu` groups

```python
g1 = Menu.Group("g1", g1_items)
g2 = Menu.Group("g2", g2_items)
```

### Initialize `Menu` widget

```python
menu = Menu(groups=[g1, g2])
```

### Create app using layout

Create an app object with layout parameter.

```python
app = sly.Application(layout=menu)
```

![layout](https://user-images.githubusercontent.com/120389559/218477186-12e43454-b8e4-4cf6-bef8-0a78ce0eb320.png)
