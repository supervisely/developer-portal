# Dialog window

## Introduction

In this tutorial you will learn how to use: `DialogWindowBase`, `DialogWindowMessage`, `DialogWindowError`, `DialogWindowWarning` widgets in Supervisely app.

[Read this tutorial in developer portal.](https://developer.supervise.ly/app-development/apps-with-gui/dialog_window)

## Function signature

```python
sly.app.show_dialog(title, description, status="info")
```

![default](https://user-images.githubusercontent.com/120389559/219675028-d17973c6-5342-4721-b86a-9989381928d8.gif)

## Parameters

|  Parameters   |                       Type                       |          Description           |
| :-----------: | :----------------------------------------------: | :----------------------------: |
|    `title`    |                      `str`                       |      dialog window title       |
| `description` |                      `str`                       | dialog window description text |
|   `status`    | `Literal["info", "success", "warning", "error"]` |      dialog window status      |

### title

Dialog window title.

**type:** `str`

```python
sly.app.show_dialog(title="Hello", description="some message")
```

### description

Dialog window description.

**type:** `str`

```python
sly.app.show_dialog(title="Hello", description="some message")
```

![default](https://user-images.githubusercontent.com/120389559/219675028-d17973c6-5342-4721-b86a-9989381928d8.gif)

### status

Dialog window status.

**type:** `Literal["info", "success", "warning", "error"]`

**default value:** `info`

## Mini App Example

You can find this example in our Github repository:

[supervisely-ecosystem/ui-widgets-demos/041_dialog_message/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/041_dialog_message/src/main.py)

### Import libraries

```python
import os
import supervisely as sly
from dotenv import load_dotenv
from supervisely.app.widgets import Card, Button, Flexbox
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()
```

### Get environment variables

```python
project_id = int(os.environ["modal.state.slyProjectId"])
workspace_id = int(os.environ["modal.state.slyWorkspaceId"])
```

### Initialize `Button` widgets we will use

```python
info_btn = Button("Show info", button_type="info")
success_btn = Button("Show success", button_type="success")
warning_btn = Button("Show warning", button_type="warning")
error_btn = Button("Show error", button_type="danger")
```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter.

```python
# create new cards
card = Card(
    title="Dialog message",
    description="click button to show dialog window",
    content=Flexbox([info_btn, success_btn, warning_btn, error_btn]),
)
```

### Create app using card

Create an app object with card parameter.

```python
app = sly.Application(layout=card)
```

### Add functions to control `Button` widgets from python code and use `show_dialog` widget

```python
@info_btn.click
def show_info():
    sly.app.show_dialog(title="Hello", description="Info description", status="info")


@success_btn.click
def show_success():
    sly.app.show_dialog(title="My success", description="Success description", status="success")


@warning_btn.click
def show_waring():
    sly.app.show_dialog(title="My warning", description="Warning description", status="warning")
    # or
    # raise sly.app.DialogWindowWarning(title="My warning", description="Warning description")


@error_btn.click
def show_error():
    sly.app.show_dialog(title="My error", description="Error description", status="error")
```

![layout](https://user-images.githubusercontent.com/120389559/219678145-110212d9-2dfb-4977-acaf-4b4f92b933ba.gif)
