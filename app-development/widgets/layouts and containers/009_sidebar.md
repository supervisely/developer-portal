# Siderbar

## Introduction

In this tutorial you will learn how to use `Siderbar` widget in Supervisely app.

[Read this tutorial in developer portal.](https://developer.supervise.ly/app-development/apps-with-gui/siderbar)

## Function signature

```python
Siderbar(left_content, right_content, width_percent=25, widget_id=None)
```

![default](https://user-images.githubusercontent.com/120389559/218459213-d0e7e1f3-b073-47c0-a759-b3741cb1df2a.gif)

## Parameters

|   Parameters    |   Type   |                  Description                  |
| :-------------: | :------: | :-------------------------------------------: |
| `left_content`  | `Widget` | Widget to display in left part of `Siderbar`  |
| `right_content` | `Widget` | Widget to display in right part of `Siderbar` |
| `width_percent` |  `int`   |   Width of the left part of `Siderbar` in %   |
|   `widget_id`   |  `str`   |               Id of the widget                |

### left_content

Determine `Widget` to display in left part of `Siderbar`.

**type:** `Widget`

### right_content

Determine `Widget` to display in right part of `Siderbar`.

**type:** `Widget`

```python
l = Button(text="Left Button")
r = Button(text="Right Button")
sidebar = Sidebar(left_content=l, right_content=r)
```

![left_right_content](https://user-images.githubusercontent.com/120389559/218466287-28579783-ceb6-4f50-aea3-87c24b11d968.png)

### width_percent

Determines width of the left part of `Siderbar` in %.

**type:** `int`

**default value:** `25`

```python
l = Button(text="Left Button")
r = Button(text="Right Button")
sidebar = Sidebar(left_content=l, right_content=r, width_percent=75)
```

![width_percent](https://user-images.githubusercontent.com/120389559/218466726-aab7e4d6-319b-4bcc-b7b6-4aa324269ac6.png)

### widget_id

ID of the widget.

**type:** `str`

**default value:** `None`

## Mini App Example

You can find this example in our Github repository:

[supervisely-ecosystem/ui-widgets-demos/037_sidebar/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/037_sidebar/src/main.py)

### Import libraries

```python
import os

import supervisely as sly
from dotenv import load_dotenv
from supervisely.app.widgets import Select, Sidebar, Text
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()
```

### Initialize left and right widgets

```python
l = Text(text="left part", status="success")
items = [
    Select.Item(label="CPU", value="cpu"),
    Select.Item(label="GPU 0", value="cuda:0"),
    Select.Item(value="option3"),
]
r = Select(items=items, filterable=True, placeholder="select me")
```

### Initialize `Sidebar` widget

```python
sidebar = Sidebar(left_content=l, right_content=r)
```

### Create app using layout

Create an app object with layout parameter.

```python
app = sly.Application(layout=sidebar)
```

![default](https://user-images.githubusercontent.com/120389559/218459213-d0e7e1f3-b073-47c0-a759-b3741cb1df2a.gif)
