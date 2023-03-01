# Done Label

## Introduction

**`DoneLabel`** is a widget in Supervisely that is used to display messages about completed tasks or operations. 

It is a minimalist text element that displays a green "Done" checkmark with the message next to it. `DoneLabel` is usually used to inform the user that a task has been successfully completed or that a process is finished.

[Read this tutorial in developer portal.](https://developer.supervise.ly/app-development/apps-with-gui/done-label)

## Function signature

```python
done_label = DoneLabel(
    text="Task has been successfully finished",
    widget_id=None
)
```

![label-default](https://user-images.githubusercontent.com/79905215/218078545-53840478-4f2d-4b74-a4c7-2838efba93b9.png)

## Parameters

| Parameters | Type |        Description         |
| :--------: | :--: | :------------------------: |
|    `text`    | `str`  | Description text |
| `widget_id`  | `str`  |      ID of the widget      |

### text

Description text

**type:** `str`

**default value:** `None`

```python
done_label = DoneLabel(text="Some text")
```

![label-text](https://user-images.githubusercontent.com/79905215/218078983-94449c90-3436-4da8-8107-cbdc29c416c0.png)

### widget_id

ID of the widget.

**type:** `str`

**default value:** `None`

## Methods and attributes

| Attributes and Methods | Description                 |
| :--------------------: | --------------------------- |
|         `text`         | Get or set `text` property. |

## Mini App Example

You can find this example in our Github repository:

[supervisely-ecosystem/ui-widgets-demos/019_done_label/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/019_done_label/src/main.py)

### Import libraries

```python
import os
from time import sleep

import supervisely as sly
from dotenv import load_dotenv
from supervisely.app.widgets import Button, Card, Container, DoneLabel
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()
```

### Initialize `DoneLabel` widget

```python
done_label = DoneLabel(
    text="Task has been successfully finished",
)
```

### Create app layout

In this tutorial we will use `Button`, `Progress` widgets for demo.

```python
done_label.hide()

progress = sly.app.widgets.Progress()
start_btn = Button(text="START")
```

Prepare a layout for app using `Card` widget with the `content` parameter and place widgets that we've just created in the `Container` widget.

```python
container = Container(widgets=[start_btn, progress, done_label])

card = Card(
    title="Done Label",
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
@start_btn.click
def start_progress():
    done_label.hide()

    with progress(message="Processing...", total=5) as pbar:
        for _ in range(5):
            sleep(1)
            pbar.update(1)

    done_label.show()
```

![done-gif](https://user-images.githubusercontent.com/79905215/218423940-5b178198-06e2-4d4e-8d99-1154f5c3889b.gif)
