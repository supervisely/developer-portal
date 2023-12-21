# CircleProgress

## Introduction

**Circle Progress** widget is a wrapper for [Progress](https://developer.supervisely.com/app-development/widgets/status-elements/progressbar) widget to display it in a circular form. This widget display progress only in percentage.

## Function signature

```python
CircleProgress(
    progress=progress,
    widget_id=None,
)
```

![default](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/4c8ae731-2e2a-4cd3-b2f9-e78b7f2a0615)

## Parameters

| Parameters |   Type   |         Description         |
| :--------: | :------: | :-------------------------: |
|  progress  | Progress | Supervisely Progress widget |
| widget\_id |    str   |       ID of the widget      |

### progress

Supervisely Progress widget. You can read more about it [here](https://developer.supervisely.com/app-development/widgets/status-elements/circle-progress).

**type:** `Progress`

```python
progress = Progress()
```

### widget\_id

ID of the widget.

**type:** `str`

**default value:** `None`

| Attributes and Methods | Description                                                       |
| :--------------------: | ----------------------------------------------------------------- |
|     `set_status()`     | Set one of the available statuses: `none`, `exception`, `success` |

## Mini App Example

You can find this example in our Github repository:

[supervisely-ecosystem/ui-widgets-demos/status-elements/009\_circle\_progress/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/status%20elements/009\_circle\_progress/src/main.py)

### Import libraries

```python
import os
from time import sleep
import supervisely as sly
from supervisely.app.widgets import Card, CircleProgress, Flexbox, Button, Progress
from dotenv import load_dotenv
```

### Init API client

Init API for communicating with Supervisely Instance. First, we load environment variables with credentials:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))
api = sly.Api()
```

### Initialize widgets: `Progress`, `Circle Progress` and `Button`

```python
progress = Progress()
circle_progress = CircleProgress(progress=progress)
button = Button("Start", button_size="mini")
```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter and place variables `circle_progress` and `button` that we've just created to the `container` variable, which uses `Container` widget.

```python
container = Container([circle_progress, button])
card = Card(
    title="Circle Progress",
    content=container,
)
layout = card
```

### Create app using layout

Create an app object with layout parameter.

```python
app = sly.Application(layout=layout)
```

Our app layout is ready. Progress bar will appear after pressing the `Start` button.

### Start progress with button click

Use the decorator as shown below to handle button click. `Progress` will be updating itself (`pbar.update(1)`) every half second by 1 point as specified in `sleep` function until it reaches `total`.

```python
@button.click
def start_progress():
    circle_progress.set_status("none")
    with progress(message="Processing items ...", total=100) as pbar:
        for _ in range(10):
            sleep(0.5)
            pbar.update(10)
        circle_progress.set_status("success")
```

![miniapp](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/70e25ebf-8840-4bdc-a343-25f26ece1697)
