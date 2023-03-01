# Notification Box

## Introduction

**`NotificationBox`** is a widget to display messages to the user. It immediately attracts the user's attention and can be used to inform the user about important events, such as the successful completion of a task, errors during execution, or the need to make adjustments to the annotation. NotificationBox supports various levels of messages, such as informational, success, warning, and error.

[Read this tutorial in developer portal.](https://developer.supervise.ly/app-development/apps-with-gui/notification-box)

## Function signature

```python
note_box = NotificationBox(
    title="Notification Box",
    description="Lorem ipsum dolor sit amet...",
    box_type="info",
    widget_id=None
)
```

![notification-box-default](https://user-images.githubusercontent.com/79905215/218077227-b81d577e-e6a7-49d9-b2f5-4ac0429727c0.png)

## Parameters

| Parameters  |                      Type                      |          Description           |
| :---------: | :--------------------------------------------: | :----------------------------: |
|    `title`    |                      `str`                       | Main title of notification box |
| `description` |                      `str`                       |        Description text        |
|  `box_type`   | `Literal["success", "info", "warning", "error"]` |     Notification box style     |
|  `widget_id`  |                      `str`                       |        ID of the widget        |

### title

Main title of notification box

**type:** `str`

**default value:** `None`

```python
note_box = NotificationBox(title="Notification Box")
```

![notification-box-title](https://user-images.githubusercontent.com/79905215/218077496-ef53205d-cb1f-47a6-93f6-9a330493670b.png)

### description

Description text

**type:** `str`

**default value:** `None`

```python
note_box = NotificationBox(description="Lorem ipsum dolor sit amet...")
```

![notification-box-desc](https://user-images.githubusercontent.com/79905215/218077562-40876bd1-68ea-4fab-9a31-b30c74a3b822.png)

### box_type

Parameter to change notification style

**type:** `Literal["success", "info", "warning", "error"]`

**default value:** `"info"`

```python
note_box_success = NotificationBox(
    title="Box type: SUCCESS",
    description="Lorem ipsum dolor sit amet... anim id est laborum.",
    box_type="success"
)
```

```python
note_box_info = NotificationBox(
    title="Box type: INFO",
    description="Lorem ipsum dolor sit amet... anim id est laborum.",
    box_type="info"
)
```

```python
note_box_warning = NotificationBox(
    title="Box type: WARNING",
    description="Lorem ipsum dolor sit amet... anim id est laborum.",
    box_type="warning"
)
```

```python
note_box_error = NotificationBox(
    title="Box type: ERROR",
    description="Lorem ipsum dolor sit amet... anim id est laborum.",
    box_type="error"
)
```

![notification-box-app](https://user-images.githubusercontent.com/79905215/218076686-43d2e536-6231-4d66-8582-4e94376035c0.png)

### widget_id

ID of the widget.

**type:** `str`

**default value:** `None`

## Methods and attributes

|       Attributes and Methods        | Description                                         |
| :---------------------------------: | --------------------------------------------------- |
|               `title`               | Get or set notification box `title` property.       |
|            `description`            | Get or set notification box `description` property. |
| `set(title: str, description: str)` | Set `title` and `description` properties.           |

## Mini App Example

You can find this example in our Github repository:

[supervisely-ecosystem/ui-widgets-demos/018_notification_box/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/018_notification_box/src/main.py)

### Import libraries

```python
import os
from time import sleep

import supervisely as sly
from dotenv import load_dotenv
from supervisely.app.widgets import Button, Card, Container, Flexbox, NotificationBox
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()
```

### Initialize `NotificationBox` widget

In this tutorial we will use 4 types of notification box.

```python
note_box_info = NotificationBox()
note_box_info.hide() # hide widget (you can show it later)

note_box_success = NotificationBox(title="Finished.", box_type="success")
note_box_success.hide()

note_box_error = NotificationBox(title="Error.", box_type="error")
note_box_error.hide()

note_box_warning = NotificationBox(title="Warning.", box_type="warning")
note_box_warning.hide()

```

### Create app layout

For 1 example we will create project and show notication with `info` box type

```python
step_1_btn = Button(text="INFO", button_type="info")
```

For 2 example we will use `Progress` widget and show `success` box type notification.

```python
progress_bar = sly.app.widgets.Progress()

step_2_btn = Button(text="SUCCESS", button_type="success")
step_2 = Container(widgets=[note_box_success, step_2_btn])
```

Create buttons for `error` and `warning` box type notifications.

```python
step_3_btn = Button(text="ERROR", button_type="danger")
step_3 = Container(widgets=[note_box_error, step_3_btn])

step_4_btn = Button(text="WARNING", button_type="warning")
step_4 = Container(widgets=[note_box_warning, step_4_btn])
```

Prepare a layout for app using `Card` widget with the `content` parameter and place widgets that we've just created in the `Container` or `Flexbox` widgets.

```python
step_1_container = Container(widgets=[step_1_btn, note_box_info])

notes_container = Flexbox(widgets=[note_box_success, note_box_error, note_box_warning])
btns_container = Container(
    widgets=[step_2_btn, step_3_btn, step_4_btn],
    direction="horizontal",
)

progress_container = Container(widgets=[btns_container, progress_bar, notes_container])

container = Container(
    widgets=[step_1_container, progress_container],
    direction="horizontal",
)

card = Card(
    title="Notification Box",
    content=container,
)
layout = Container(widgets=[card])
```

### Create app using layout

Create an app object with layout parameter.

```python
app = sly.Application(layout=layout)
```

### Add functions to control widgets from python code

```python

@step_1_btn.click
def create_project_and_dataset():
    project = api.project.create(
        workspace_id=workspace_id,
        name="New project",
        type=sly.ProjectType.IMAGES,
        change_name_if_conflict=True,
    )

    if project is not None:
        note_box_info.set(
            title="New project and dataset created.",
            description=f"Project ID: {project.id}).",
        )
        note_box_info.show()


@step_2_btn.click
def start_progress():
    note_box_warning.hide()
    note_box_error.hide()
    with progress_bar(message="Processing items...", total=5) as pbar:
        for _ in range(5):
            sleep(1)
            pbar.update(1)

    note_box_success.show()


@step_3_btn.click
def show_error():
    note_box_success.hide()
    note_box_warning.hide()
    note_box_error.show()


@step_4_btn.click
def show_warning():
    note_box_success.hide()
    note_box_error.hide()
    note_box_warning.show()
```

![note-gif](https://user-images.githubusercontent.com/79905215/218417029-28c397a9-dab0-4b5f-9ea2-a230c722538b.gif)
