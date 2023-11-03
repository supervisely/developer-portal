# Sidebar

## Introduction

**`Sidebar`** widget from Supervisely is a tool that provides quick access to important information and features in Supervisely apps. `Sidebar` is a vertical panel that appears on the left side of the app interface and includes widget user will placed in. `Sidebar` widget is a useful tool for streamlining workflows and improving user productivity in the apps.

## Function signature

```python
Sidebar(
    left_content=Button(),
    right_content=Card("Input", content=Input()),
    width_percent=25,
    widget_id=None,
)
```

<figure><img src="https://user-images.githubusercontent.com/79905215/224029407-3e2d1e59-2210-4069-b106-1ccbd112b5b5.png" alt=""><figcaption></figcaption></figure>

## Parameters

|   Parameters    |   Type   |                 Description                  |
|:---------------:|:--------:|:--------------------------------------------:|
| `left_content`  | `Widget` | Widget to display in left part of `Sidebar`  |
| `right_content` | `Widget` | Widget to display in right part of `Sidebar` |
| `width_percent` |  `int`   |   Width of the left part of `Sidebar` in %   |
|    `height`     |  `str`   | Height of sidebar, can be set in px, % or hv |
|  `standalone`   |  `bool`  |      Add paddings for full screen apps       |
|   `widget_id`   |  `str`   |               Id of the widget               |

### left\_content

Determine `Widget` to display in left part of `Sidebar`.

**type:** `Widget`

### right\_content

Determine `Widget` to display in right part of `Sidebar`.

**type:** `Widget`

```python
left = Button(text="Left Button")
right = Button(text="Right Button")
sidebar = Sidebar(left_content=left, right_content=right)
```

<figure><img src="https://user-images.githubusercontent.com/120389559/218466287-28579783-ceb6-4f50-aea3-87c24b11d968.png" alt=""><figcaption></figcaption></figure>

### width\_percent

Determines width of the left part of `Sidebar` in %.

**type:** `int`

**default value:** `25`

```python
left = Button(text="Left Button")
right = Button(text="Right Button")
sidebar = Sidebar(left_content=left, right_content=right, width_percent=75)
```

<figure><img src="https://user-images.githubusercontent.com/120389559/218466726-aab7e4d6-319b-4bcc-b7b6-4aa324269ac6.png" alt=""><figcaption></figcaption></figure>

### height

Height of sidebar, can be set in `px`,`%` or `hv`.

**type:** `str`

```python
left = Button(text="Left Button")
right = Button(text="Right Button")
sidebar = Sidebar(left_content=left, right_content=right, height="250px")
```

<figure><img src="https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/b25faee3-69df-48d7-ac68-1c955dbb0f0f" alt=""><figcaption></figcaption></figure>

### standalone

Add paddings for full screen apps.

**type:** `bool`

**default value:** `False`

```python
left = Button(text="Left Button")
right = Button(text="Right Button")
sidebar = Sidebar(left_content=left, right_content=right, height="350px", standalone=True)
```

<figure><img src="https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/e78afe2f-3c82-4aae-8818-46ee4aa280a6" alt=""><figcaption></figcaption></figure>

### widget\_id

ID of the widget.

**type:** `str`

**default value:** `None`

## Mini App Example

You can find this example in our Github repository:

[ui-widgets-demos/layouts and containers/009\_sidebar/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/layouts%20and%20containers/009\_sidebar/src/main.py)

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
left = Text(text="left part", status="success")
items = [
    Select.Item(label="CPU", value="cpu"),
    Select.Item(label="GPU 0", value="cuda:0"),
    Select.Item(value="option3"),
]
right = Select(items=items, filterable=True, placeholder="select me")
```

### Initialize `Sidebar` widget

```python
sidebar = Sidebar(left_content=left, right_content=right)
```

### Create app using layout

Create an app object with layout parameter.

```python
app = sly.Application(layout=sidebar)
```

<figure><img src="https://user-images.githubusercontent.com/120389559/218459213-d0e7e1f3-b073-47c0-a759-b3741cb1df2a.gif" alt=""><figcaption></figcaption></figure>
