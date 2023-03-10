# RadioTabs

## Introduction

**`RadioTabs`** is a graphical user interface tool in Supervisely that allows users to group related widgets into tabs and switch between them using radio buttons. This feature is particularly useful for organizing and presenting complex interfaces with multiple settings and options. `RadioTabs` offers a simple and intuitive interface for users to quickly navigate between different tabs and access the desired widgets.

## Function signature

```python
RadioTabs(
    titles=["Info tab", "Success tab"],
    contents=[
        Text("Info text", status="info"),
        Text("Success text", status="success")
    ],
    descriptions=None,
    widget_id=None,
)
```

<figure><img src="https://user-images.githubusercontent.com/120389559/222171253-af777d91-7c48-4990-b807-9238e2968662.gif" alt=""><figcaption></figcaption></figure>

## Parameters

|   Parameters   |      Type      |        Description        |
| :------------: | :------------: | :-----------------------: |
|    `titles`    |   `List[str]`  |  List of the tabs titles  |
|   `contents`   | `List[Widget]` |    List of tabs content   |
| `descriptions` |   `List[str]`  | List of tabs descriptions |
|   `widget_id`  |      `str`     |      ID of the widget     |

### titles

Determine list of the tabs titles.

**type:** `List[str]`

### contents

Determine list of the tabs content.

**type:** `List[Widget]`

```python
tabs = RadioTabs(
    titles=["Info tab", "Success tab"],
    contents=[
        Text("Info text", status="info"),
        Text("Success text", status="success")
    ]
)
```

<figure><img src="https://user-images.githubusercontent.com/120389559/222171253-af777d91-7c48-4990-b807-9238e2968662.gif" alt=""><figcaption></figcaption></figure>

### descriptions

Determine list of tabs descriptions.

**type:** `List[str]`

**default value:** `None`

```python
tabs = RadioTabs(
    titles=["Info tab", "Success tab"],
    contents=[
        Text("Info text", status="info"),
        Text("Success text", status="success")
    ],
    descriptions=[
        "Tab with info text",
        "Tab with success text"
    ]
)
```

<figure><img src="https://user-images.githubusercontent.com/120389559/222174503-b99916b1-06e8-4845-a2dc-6abef1d1cb1e.gif" alt=""><figcaption></figcaption></figure>

### widget\_id

ID of the widget.

**type:** `str`

**default value:** `None`

## Methods and attributes

|    Attributes and Methods    | Description                                                   |
| :--------------------------: | ------------------------------------------------------------- |
| `set_active_tab(value: str)` | Set active tab by title.                                      |
|      `get_active_tab()`      | Return active tab title.                                      |
|       `@value_changed`       | Decorator function handled when `RadioTabs` value is changed. |

## Mini App Example

You can find this example in our Github repository:

[ui-widgets-demos/layouts and containers/011\_radio\_tabs/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/layouts%20and%20containers/011\_radio\_tabs/src/main.py)

### Import libraries

```python
import os

import supervisely as sly
from dotenv import load_dotenv
from supervisely.app.widgets import Card, Container, RadioTabs, Text
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()
```

### Initialize `RadioTabs` widgets

```python
tabs = RadioTabs(
    titles=["Info tab", "Success tab", "Warning tab", "Error tab"],
    contents=[
        Text("Info text", status="info"),
        Text("Success text", status="success"),
        Text("Warning text", status="warning"),
        Text("Error text", status="error"),
    ],
    descriptions=[
        "Tab with info text",
        "Tab with success text",
        "Tab with warning text",
        "Tab with error text",
    ],
)

many_tabs = RadioTabs(
    titles=[f"{i + 1} tab" for i in range(10)],
    contents=[Text(f"{i + 1} text", status="info") for i in range(10)],
    descriptions=[f"Tab with {i + 1} text" for i in range(10)],
)
```

### Initialize `Text` widget to change text on widget

```python
text = Text("Opened tab: Info tab")
```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter and place widget that we've just created in the `Container` widget.

```python
card = Card(
    title="Radio tabs",
    content=Container([tabs, text]),
)
card_many_tabs = Card(
    "Many radio tabs",
    content=many_tabs,
)

layout = Container(widgets=[card, card_many_tabs])
```

### Create app using layout

Create an app object with layout parameter.

```python
app = sly.Application(layout=layout)
```

### Add functions to control widgets from python code

```python
@tabs.value_changed
def show_opened_tab(value):
    text.text = f"Opened tab: {value}"
```

<figure><img src="https://user-images.githubusercontent.com/120389559/222177913-2c4d26de-f555-4747-8fed-ee8807f7d6ce.gif" alt=""><figcaption></figcaption></figure>
