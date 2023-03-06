# DialogMessage

## Introduction

**`show_dialog`** widget in Supervisely is a function that is used to display a modal dialog window with customizable content. Users can specify the `title`, `message`, and `status` of the dialog window, such as informational, success, warning, or error.

## Function signature

```python
sly.app.show_dialog(
    title,
    description,
    status="info",
)
```

<figure><img src="https://user-images.githubusercontent.com/120389559/219675028-d17973c6-5342-4721-b86a-9989381928d8.gif" alt=""><figcaption></figcaption></figure>

## Parameters

|   Parameters  |                       Type                       |           Description          |
| :-----------: | :----------------------------------------------: | :----------------------------: |
|    `title`    |                       `str`                      |       Dialog window title      |
| `description` |                       `str`                      | Dialog window description text |
|    `status`   | `Literal["info", "success", "warning", "error"]` |      Dialog window status      |

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

<figure><img src="https://user-images.githubusercontent.com/120389559/219675028-d17973c6-5342-4721-b86a-9989381928d8.gif" alt=""><figcaption></figcaption></figure>

### status

Dialog window status.

**type:** `Literal["info", "success", "warning", "error"]`

**default value:** `info`

## Mini App Example

You can find this example in our Github repository:

[ui-widgets-demos/status elements/004\_dialog\_message/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/status%20elements/004\_dialog\_message/src/main.py)

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
    sly.app.show_dialog(
        title="Hello",
        description="Info description",
        status="info",
    )


@success_btn.click
def show_success():
    sly.app.show_dialog(
        title="My success",
        description="Success description",
        status="success",
    )


@warning_btn.click
def show_waring():
    sly.app.show_dialog(
        title="My warning",
        description="Warning description",
        status="warning",
    )
    # or
    # raise sly.app.DialogWindowWarning(title="My warning", description="Warning description")


@error_btn.click
def show_error():
    sly.app.show_dialog(
        title="My error",
        description="Error description",
        status="error",
    )
```

<figure><img src="https://user-images.githubusercontent.com/120389559/219678145-110212d9-2dfb-4977-acaf-4b4f92b933ba.gif" alt=""><figcaption></figcaption></figure>
