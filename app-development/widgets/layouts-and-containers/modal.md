# Modal

## Introduction

**`Modal`** widget in Supervisely displays content in a modal overlay window with customizable size and content. It can contain any other widgets and provides programmatic control to show/hide the modal. The modal also supports callbacks to track when it opens or closes.

[Read this tutorial in developer portal.](https://developer.supervisely.com/app-development/widgets/layouts-and-containers/modal)

## Function signature

```python
Modal(
    title="My Modal",
    widgets=[text_widget, input_widget],
    size="small",
    widget_id=None,
)
```

![modal](https://github.com/supervisely-ecosystem/ui-widgets-demos/releases/download/v0.0.8/screenshot-localhost-8000-1765988335167.png)

## Parameters

| Parameters  |                    Type                     |         Description         |
| :---------: | :-----------------------------------------: | :-------------------------: |
|   `title`   |                    `str`                    |     Modal window title      |
|  `widgets`  |               `List[Widget]`                | Widgets to display in modal |
|   `size`    | `Literal["tiny", "small", "large", "full"]` |         Modal size          |
| `widget_id` |                    `str`                    |      ID of the widget       |

### title

Modal window title displayed at the top of the modal.

**type:** `str`

**default value:** `""`

### widgets

List of widgets to be displayed inside the modal. Can contain any Supervisely widgets.

**type:** `List[Widget]`

**default value:** `[]`

```python
input_widget = Input("Enter value")
text_widget = Text("This is modal content")

modal = Modal(
    title="My Modal",
    widgets=[text_widget, input_widget]
)
```

### size

Size of the modal window. Available options: `"tiny"`, `"small"`, `"large"`, `"full"`.

**type:** `Literal["tiny", "small", "large", "full"]`

**default value:** `"small"`

### widget_id

ID of the widget.

**type:** `str`

**default value:** `None`

## Methods and attributes

|      Methods       |                        Description                         |
| :----------------: | :--------------------------------------------------------: |
|      `show()`      |                  Shows the modal window.                   |
|      `hide()`      |                  Hides the modal window.                   |
|   `show_modal()`   |             Shows the modal (alias for show).              |
|  `close_modal()`   |             Closes the modal (alias for hide).             |
|   `is_opened()`    |          Returns True if modal is currently open.          |
| `@value_changed()` | Decorator to handle modal visibility changes (open/close). |
|      `title`       |            Property to get/set the modal title.            |
|     `widgets`      |          Property to get/set the list of widgets.          |

## Mini App Example

You can find this example in our Github repository:

[supervisely-ecosystem/ui-widgets-demos/layouts-and-containers/018_modal/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/layouts-and-containers/018_modal/src/main.py)

### Import libraries

```python
import os
import supervisely as sly
from supervisely.app.widgets import Card, Container, Input, Text, Modal, Button
from dotenv import load_dotenv
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()
```

### Initialize widgets to display inside modal

```python
input_widget = Input("Enter value")
text_widget = Text("This is modal content")
button = Button("Open modal")
```

### Initialize `Modal` widget

```python
modal = Modal(title="My Modal Window", widgets=[text_widget, input_widget], size="small")
```

![Show modal](https://github.com/supervisely-ecosystem/ui-widgets-demos/releases/download/v0.0.8/screenshot-localhost-8000-1765988381426.png)

````python

### Create app layout

Prepare a layout for the app using `Card` and `Container` widgets.

```python
container = Container(widgets=[modal, button])

layout = Card(
    title="Modal Example",
    content=container,
)
````

### Create app using layout

Create an app object with layout parameter.

```python
app = sly.Application(layout=layout)
```

### Add button click handler to open modal

```python
@button.click
def on_button_click():
    modal.show()
```

### Add value changed callback to track modal state

```python
@modal.value_changed
def on_value_changed(value):
    print(f"Modal value changed: {value}")
    print("Modal closed")
```
