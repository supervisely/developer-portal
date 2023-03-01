# Progress Bar

## Introduction

This Supervisely widget allows you to display the progress of a task or operation. It is a useful widget for applications that involve long-running processes or tasks that take time to complete.

With the `Progress Bar` widget, you can easily create a progress bar that updates in real-time or update the progress bar dynamically as a task progresses, giving users a clear indication of how much time is left until the task is complete.

The Progress Bar Widget can be customized to show progress as percents or values.

[Read this tutorial in developer portal.](https://developer.supervise.ly/app-development/apps-with-gui/progress-bar)

## Function signature

```python
Progress(message=None, show_percents=False, hide_on_finish=True, widget_id=None)
```

![default](https://user-images.githubusercontent.com/48913536/202434648-cda78cff-0796-498b-b77e-8eb6e8909e9c.gif)

## Parameters

|   Parameters   | Type  |                Description                 |
| :------------: | :---: | :----------------------------------------: |
|    message     |  str  |            progress bar message            |
| show_percents  | bool  |         show progress in percents          |
| hide_on_finish | bool  |        hide progress bar on finish         |
|   widget_id    |  str  | determine whether button is a plain button |

### message

Progress bar message.

**type:** `str`

**default value:** `None`

```python
progress = Progress(message="My progress message")
```

![message](https://user-images.githubusercontent.com/48913536/202438044-1b805dec-7e29-4969-867e-b9fc1d28cea4.gif)

### show_percents

Show progress in percents.

**type:** `bool`

**default value:** `False`

```python
progress = Progress(show_percents=True)

```

![show_percent](https://user-images.githubusercontent.com/48913536/202434656-3785abb8-b05b-46c1-a57f-e88349670300.gif)

### hide_on_finish

Hide progress bar on finish

**type:** `bool`

**default value:** `True`

```python
progress = Progress(hide_on_finish=True)
```

![hide_on_finish](https://user-images.githubusercontent.com/48913536/202434654-f2846a23-4bfd-4319-9cdd-3e047281a663.gif)

### widget_id

ID of the widget.

**type:** `str`

**default value:** `None`

## Mini App Example

You can find this example in our Github repository:

[supervisely-ecosystem/ui-widgets-demos/002_progress_bar/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/002_progress_bar/src/main.py)

### Import libraries

```python
import os
from time import sleep

import supervisely as sly
from dotenv import load_dotenv
from supervisely.app.widgets import Card, Container
```

### Init API client

Init API for communicating with Supervisely Instance. First, we load environment variables with credentials:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))
api = sly.Api()
```

### Initialize `Progress Bar`, `Text` and `Button` widgets

```python
progress_bar = sly.app.widgets.Progress()
start_btn = sly.app.widgets.Button(text="Start", icon="zmdi zmdi-play")
finish_msg = sly.app.widgets.Text(status="success")
```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter and place 3 widgets that we've just created in the `Container` widget. Place order in the `Container` is important, we want the finish message text to be displayed below the progress bar button.

```python
card = Card(
    title="Progress Bar",
    description="👉 Click on the button to start",
    content=Container([progress_bar, start_btn, finish_msg]),
)
layout = Container(widgets=[card], direction="vertical")
```

### Create app using layout

Create an app object with layout parameter.

```python
app = sly.Application(layout=layout)
```

Our app layout is ready. Progress bar will appear after pressing the `Start` button.

![layout](https://user-images.githubusercontent.com/48913536/202438081-552d2ba1-c682-42aa-9010-064b460f3ce4.png)

### Start progress with button click

Use the decorator as shown below to handle button click.
`Progress Bar` will be updating itself (`pbar.update(1)`) every half second by 1 point as specified in `sleep` function until it reaches `total` 10.

```python
@start_btn.click
def start_progress():
    with progress_bar(message=f"Processing items...", total=10) as pbar:
        for _ in range(10):
            sleep(0.5)
            pbar.update(1)

    finish_msg.text = "Finished"
```

![demo](https://user-images.githubusercontent.com/48913536/202436155-e9721f44-916d-48c2-9c30-f43f41f4c9ba.gif)
