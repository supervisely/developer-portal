---
description: Progress bar widget tutorial
---

# Progress Bar

<figure><img src="https://user-images.githubusercontent.com/48913536/184925928-c035b6bd-6716-4080-9fac-d01967b01126.png" alt=""><figcaption><p>Progress Bar</p></figcaption></figure>

## Introduction

When code is running a job and it’s going to take a while we want to track it's progress. Especially when dealing with large datasets, even the simplest operations can take hours. Supervisely SDK can provide this functionality using only a couple of lines. In this tutorial, we will introduce you into how to use **`Progress Bar`** widget in Supervisely app.

**Function signature**

```python
sly.app.widgets.Progress(
    message=None, 
    show_percents=False, 
    hide_on_finish=True, 
    widget_id=None
)
```

## How to debug this tutorial

**Step 1.** Prepare `~/supervisely.env` file with credentials. [Learn more here.](https://developer.supervise.ly/getting-started/basics-of-authentication#how-to-use-in-python)

**Step 2.** Clone [repository](https://github.com/supervisely-ecosystem/ui-widgets-demos) with source code and create [Virtual Environment](https://docs.python.org/3/library/venv.html).

```bash
git clone https://github.com/supervisely-ecosystem/ui-widgets-demos
cd ui-widgets-demos
./create_venv.sh
```

**Step 3.** Open repository directory in Visual Studio Code

```bash
code -r .
```

**Step 4.** Start debugging [`001_progress_bar/src/main.py`](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/001\_progress\_bar/src/main.py)

## Progress Bar

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
progress_bar = sly.app.widgets.Progress(hide_on_finish=False)
start_btn = sly.app.widgets.Button(text="Start", icon="zmdi zmdi-play")
finish_msg = sly.app.widgets.Text(status="success")
```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter and place 3 widgets that we've just created in the `Container` widget. Place order in the `Container` is important, we want the finish message text to be displayed below the progress bar button.

```python
card = Card(title="Progress Bar", content=Container([progress_bar, start_btn, finish_msg]))
layout = Container(widgets=[card], direction="vertical")
```

### Create app using layout

Create an app object with layout parameter.

```python
app = sly.Application(layout=layout)
```

Our app layout is ready. Progress bar will appear after pressing the `Start` button.

<figure><img src="https://user-images.githubusercontent.com/48913536/200022344-6af73cce-3960-45ee-bc44-9bd3145e3f3d.png" alt=""><figcaption><p>App layout</p></figcaption></figure>

#### Handle button clicks

Use the decorator as shown below to handle button click. `Progress Bar` will be updating itself (`pbar.update(1)`) every second by 1 point as specified in `sleep` function until it reaches `total` (20).

```python
@start_btn.click
def start_progress():
    with progress_bar(message=f"Processing items...", total=20) as pbar:
        for _ in range(20):
            sleep(1)
            pbar.update(1)

    finish_msg.text = "Finished"
```

<figure><img src="https://user-images.githubusercontent.com/48913536/200021813-27816552-fa3c-4ba0-bffb-14f7948ac3cf.gif" alt=""><figcaption><p>App demo</p></figcaption></figure>
