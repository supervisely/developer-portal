# Tabs

## Introduction

**`Tabs`** is a graphical user interface tool in Supervisely that allows users to group related widgets into multiple tabs, providing a more organized and streamlined interface. Each tab contains a set of widgets specific to a particular task or category.

## Function signature

```python
border_tabs = Tabs(
    labels=["Info", "Success", "Warning", "Error"],
    contents=[
        Text("Info text", status="info"),
        Text("Success text", status="success"),
        Text("Warning text", status="warning"),
        Text("Error text", status="error"),
    ],
    type="border-card",
    widget_id=None,
)
```

<figure><img src="https://user-images.githubusercontent.com/79905215/224059607-c87776e6-3988-4c41-b1b0-92ff49d19cd1.gif" alt=""><figcaption></figcaption></figure>

## Parameters

| Parameters  |                    Type                    |       Description       |
| :---------: | :----------------------------------------: | :---------------------: |
|  `labels`   |                `List[str]`                 | List of the tabs labels |
| `contents`  |               `List[Widget]`               |  List of tabs content   |
|   `type`    | `Optional[Literal["card", "border-card"]]` | Style of `Tabs` widget  |
| `widget_id` |                   `str`                    |    ID of the widget     |

### labels

Determine list of the tabs labels.

**type:** `List[str]`

### contents

Determine list of the tabs content.

**type:** `List[Widget]`

```python
tabs = Tabs(
    labels=["Info", "Success", "Warning", "Error"],
    contents=[
        Text("Info text", status="info"),
        Text("Success text", status="success"),
        Text("Warning text", status="warning"),
        Text("Error text", status="error"),
    ],
)
```

<figure><img src="https://user-images.githubusercontent.com/79905215/224059607-c87776e6-3988-4c41-b1b0-92ff49d19cd1.gif" alt=""><figcaption></figcaption></figure>

### type

Determine style of `Tabs` widget.

**type:** `Optional[Literal["card", "border-card"]]`

**default value:** `"border-card"`

```python
tabs = Tabs(
    labels=["Info", "Success", "Warning", "Error"],
    contents=[
        Text("Info text", status="info"),
        Text("Success text", status="success"),
        Text("Warning text", status="warning"),
        Text("Error text", status="error"),
    ],
    type="card",
)
```

<figure><img src="https://user-images.githubusercontent.com/79905215/224063450-5616bc8b-a09b-4d58-8c72-37821ca3f79a.png" alt=""><figcaption></figcaption></figure>

```python
border_tabs = Tabs(
    labels=["Info", "Success", "Warning", "Error"],
    contents=[
        Text("Info text", status="info"),
        Text("Success text", status="success"),
        Text("Warning text", status="warning"),
        Text("Error text", status="error"),
    ],
    type="border-card",
)
```

<figure><img src="https://user-images.githubusercontent.com/79905215/224063401-043be153-d73d-4c81-bb67-2facb892dba2.png" alt=""><figcaption></figcaption></figure>

### widget_id

ID of the widget.

**type:** `str`

**default value:** `None`

## Methods and attributes

|    Attributes and Methods    | Description                                                                                            |
| :--------------------------: | ------------------------------------------------------------------------------------------------------ |
| `set_active_tab(value: str)` | Set active tab by title.                                                                               |
|      `get_active_tab()`      | Return active tab title.                                                                               |
| `enable_tab(tab_name: str)`  | Enables a previously disabled tab, making it interactive and selectable again.                         |
| `disable_tab(tab_name: str)` | Disables a specific tab, making it non-interactive and visually indicating that it cannot be selected. |
|           `@click`           | Decorator for setting a callback function for the `click` event.                                       |

## Mini App Example

You can find this example in our Github repository:

[ui-widgets-demos/layouts and containers/012_tabs/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/layouts%20and%20containers/012_tabs/src/main.py)

### Import libraries

```python
import os

import supervisely as sly
from dotenv import load_dotenv
from supervisely.app.widgets import Card, Container, Tabs, Text, Button
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()
```

### Initialize `Tabs` widgets

```python
tabs = Tabs(
    labels=["Info", "Success", "Warning", "Error"],
    contents=[
        Text("Info text", status="info"),
        Text("Success text", status="success"),
        Text("Warning text", status="warning"),
        Text("Error text", status="error"),
    ],
    type="card",
)

border_tabs = Tabs(
    labels=["Info", "Success", "Warning", "Error"],
    contents=[
        Text("Info text", status="info"),
        Text("Success text", status="success"),
        Text("Warning text", status="warning"),
        Text("Error text", status="error"),
    ],
    type="border-card",
)
```

### Initialize `Text` and `Button` widgets

```python
button = Button("Show opened tab")
text = Text("")
text.hide()
```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter and place widget that we've just created in the `Container` widget.

```python
card = Card(
    title="Tabs (type 'card')",
    content=Container([tabs, button, text]),
)
border_card = Card(
    title="Tabs (type 'border-card')",
    content=border_tabs,
)

layout = Container(widgets=[card, border_card])
```

### Create app using layout

Create an app object with layout parameter.

```python
app = sly.Application(layout=layout)
```

### Add functions to control widgets from python code

```python
@button.click
def show_opened_tab():
    text.show()
    text.text = f"Opened tab: {tabs.get_active_tab()}"
```

<figure><img src="https://user-images.githubusercontent.com/79905215/224062389-4ef12e28-76bd-43da-a5d5-ccfbf0f768db.gif" alt=""><figcaption></figcaption></figure>
