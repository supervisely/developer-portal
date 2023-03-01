# Text

## Introduction

This Supervisely widget allows you to display text. You can customize the displayed text and it's status which defines text color.

[Read this tutorial in developer portal.](https://developer.supervise.ly/app-development/apps-with-gui/text)

## Function signature

```python
Text(text=None, status="text", widget_id=None)
```

![default](https://user-images.githubusercontent.com/120389559/217817133-5d90a597-64c8-4555-ab77-4aca594e39e6.png)

## Parameters

| Parameters  |                           Type                           |       Description        |
| :---------: | :------------------------------------------------------: | :----------------------: |
|   `text`    |                          `str`                           | Determine displayed text |
|  `status`   | `Literal["text", "info", "success", "warning", "error"]` |      `Text` status       |
| `widget_id` |                          `str`                           |     Id of the widget     |

### text

Determine displayed text.

**type:** `str`

**default value:** `None`

```python
text = Text(text="My text")
```

### status

Text status.

**type:** `str`

**default value:** `text`

```python
text = Text(text="My text", status="text")
text_info = Text(text="My info text", status="info")
text_success = Text(text="My success text", status="success")
text_warning = Text(text="My warning text", status="warning")
text_error = Text(text="My error text", status="error")
```

![status](https://user-images.githubusercontent.com/120389559/218086875-c6990527-44f2-44a3-9367-c79d3c6717e9.png)

### widget_id

ID of the widget.

**type:** `str`

**default value:** `None`

## Mini App Example

You can find this example in our Github repository:

[supervisely-ecosystem/ui-widgets-demos/003_text/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/003_text/src/main.py)

### Import libraries

```python
import os

import supervisely as sly
from dotenv import load_dotenv
from supervisely.app.widgets import Button, Card, Container, Text
```

### Init API client

Init API for communicating with Supervisely Instance. First, we load environment variables with credentials:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))
api = sly.Api()
```

### Initialize five `Button` widgets for each type of text message: text, info, success, warning, error

```python
button_show_text = Button(
    text="Text",
    button_type="primary",
    button_size="small",
    icon="zmdi zmdi-comment-text",
)
button_show_info = Button(
    text="Info", button_type="info", button_size="small", icon="zmdi zmdi-info-outline"
)
button_show_success = Button(
    text="Success", button_type="success", button_size="small", icon="zmdi zmdi-check"
)
button_show_warning = Button(
    text="Warning",
    button_type="warning",
    button_size="small",
    icon="zmdi zmdi-alert-circle-o",
)
button_show_error = Button(
    text="Error",
    button_type="danger",
    button_size="small",
    icon="zmdi zmdi-minus-circle-outline",
)
```

### Initialize `Container` widget

Create `Container` widget for all buttons.

```python
buttons_container = Container(
    widgets=[
        button_show_text,
        button_show_info,
        button_show_success,
        button_show_warning,
        button_show_error,
    ],
    direction="horizontal",
)
```

### Initialize `Text` widget with text and status

```python
text = Text(text="Lorem ipsum dolor sit amet... anim id est laborum.", status="text")
```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter and place 2 widgets that we've just created in the `Container` widget. Place order in the `Container` is important, we want the message text to be displayed below the buttons.

```python
card = Card(
    title="Text",
    description="👉 Click on the button to change text highlighting",
    content=Container(widgets=[buttons_container, text]),
)
layout = Container(widgets=[card], direction="vertical")
```

### Create app using layout

Create an app object with layout parameter.

```python
app = sly.Application(layout=layout)
```

Our app layout is ready.

### Start text status change with button click

Use the decorator as shown below to handle button click.
`Text` will be updating it status after pressing corresponding button.

```python
@button_show_text.click
def show_text():
    text.status = "text"
```

```python
@button_show_info.click
def show_info():
    text.status = "info"
```

```python
@button_show_success.click
def show_success():
    text.status = "success"
```

```python
@button_show_warning.click
def show_warning():
    text.status = "warning"
```

```python
@button_show_error.click
def show_error():
    text.status = "error"
```

![layout](https://user-images.githubusercontent.com/120389559/218090541-30f6b8f6-90e0-4785-8491-455a7d4b6635.gif)
