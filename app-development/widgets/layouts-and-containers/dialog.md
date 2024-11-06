# Dialog

## Introduction

**`Dialog`** is a widget that allows to show a dialog window that contain any other widgets. It can be used to show a message to the user or to ask for confirmation.

[Read this tutorial in developer portal.](https://developer.supervisely.com/app-development/apps-with-gui/dialog)

## Function signature

```python
dialog = Dialog(
    title = "Dialog Window",
    content = None,
    size = "small",
)
```

![](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/51306ae3-3bcd-4138-abde-1360ee9b7965)

## Parameters

|  Parameters |                          Type                         |             Description            |
| :---------: | :---------------------------------------------------: | :--------------------------------: |
|   `title`   |                         `str`                         |     Title of the dialog window     |
|  `content`  |                        `Widget`                       | Widget to display in dialog window |
|    `size`   | `Literal["tiny", "small", "large", "full"] = "small"` |      Size of the dialog window     |
| `widget_id` |                         `str`                         |          ID of the widget          |

### title

Title of the dialog window that will be displayed in the header.

**type:** `str`

**default value:** `""`

```python
dialog = Dialog(title="Dialog Window")
```

![](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/51306ae3-3bcd-4138-abde-1360ee9b7965)

### content

Widget to display in the dialog window. Use `Container` widget to display multiple widgets.

**type:** `Widget`

**default value:** `None`

```python
dialog_container = Container(
    [Text(text="Hello world!", status="text", font_size=32), Button("Click me!")]
)

dialog = Dialog(
    title = "Dialog Window",
    content = dialog_container,
    size = "small",
)
```

![](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/f727d25c-c2a9-4ccf-8357-6cd6790cfd07)

### size

Width of the annotation border (contour) line. Set to 0 to hide a line.

**type:** `Literal["tiny", "small", "large", "full"]`

**default value:** `"small"`

```python
dialog_container = Container(
    [Text(text="Hello world!", status="text", font_size=32), Button("Click me!")]
)

dialog = Dialog(
    title = "Dialog Window",
    content = dialog_container,
    size = "large",
)
```

![](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/e4dd96b0-24d0-4021-8b27-8a660fd1e23f)

### widget\_id

ID of the widget.

**type:** `str`

**default value:** `None`

## Methods and attributes

| Attributes and Methods | Description                 |
| :--------------------: | --------------------------- |
|        `show()`        | Open (show) dialog window.  |
|        `hide()`        | Close (hide) dialog window. |
|         `title`        | Set title for dialog window |

## Mini App Example

You can find this example in our Github repository: [supervisely-ecosystem/ui-widgets-demos/layouts and containers/016\_dialog/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/layouts%20and%20containers/016\_dialog/src/main.py)

### Import libraries

```python
import os
import supervisely as sly
from dotenv import load_dotenv
from supervisely.app.widgets import Button, Card, Container, Dialog, Text
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()
```

### Initialize content widgets for Dialog

```python
dialog_text = Text(text="Hello world!", status="text", font_size=32)
close_dialog_btn = Button("Click me!")
dialog_container = Container([dialog_text, close_dialog_btn])
```

### Initialize Dialog widget

```python
dialog = Dialog(title="Dialog window", content=dialog_container, size="small")
```

### Create button to open Dialog

```python
open_dialog_btn = Button("Open dialog")
```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter and place Button for opening dialog to the content.

```python
card = Card(title="Dialog", content=open_dialog_btn)
```

### Create app using layout

Add Container widget with card and dialog widgets to the app layout.

```python
app = sly.Application(layout=Container([card, dialog]))
```

![](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/f727d25c-c2a9-4ccf-8357-6cd6790cfd07)

### Add button click event to update open and close Dialog

```python
@open_dialog_btn.click
def show_dialog():
    dialog.show()


@close_dialog_btn.click
def hide_dialog():
    dialog.hide()
```
