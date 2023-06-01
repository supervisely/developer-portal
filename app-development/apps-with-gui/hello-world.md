---
description: Create simple supervisely app with GUI
---

# Hello World!

## Introduction

In this tutorial you will learn how to create Supervisely apps with GUI on pure python using Supervisely app engine and widgets. We will create a simple "Hello, World!" app that will generate names using `Text` and `Button` widgets.

[`main.py`](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/hello_world/src/main.py) is just 26 lines of code.

## Requirements

Install latest [`supervisely` ](https://pypi.org/project/supervisely/)version to have access to all [available widgets](https://ecosystem.supervise.ly/docs/table) and [`names` ](https://pypi.org/project/names/)library for names generation

```python
names # requires for names generation
supervisely
```

## How to debug this tutorial

**Step 1.** Prepare `~/supervisely.env` file with credentials. [Learn more here.](../../getting-started/basics-of-authentication.md#how-to-use-in-python)

**Step 2.** Clone [repository](https://github.com/supervisely-ecosystem/ui-widgets-demos) with source code and create [Virtual Environment](https://docs.python.org/3/library/venv.html).

```bash
git clone https://github.com/supervisely-ecosystem/ui-widgets-demos
cd ui-widgets-demos
./create_venv.sh
```

**Step 3.** Open the `.vscode/launch.json` file in the project and specify the path to your script in launching configuration arguments.

```python
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Uvicorn",
            "type": "python",
            "request": "launch",
            "module": "uvicorn",
            "args": [
                "000_hello_world.src.main:app", # ⬅️ Path to your script
                "--host",
                "0.0.0.0",
                "--port",
                "8000",
                "--ws",
                "websockets",
                "--reload"
            ],
            "jinja": true,
            "justMyCode": true,
            "env": {
                "PYTHONPATH": "${workspaceFolder}:${PYTHONPATH}",
                "LOG_LEVEL": "DEBUG",
            }
        }
    ]
}
```

**Step 4** Open repository directory in Visual Studio Code.

```bash
code -r .
```
                                                            
**Step 5.** Start debugging [`hello_world/src/main.py`](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/hello_world/src/main.py)``

## Hello, World! app

### Import libraries

```python
import os

import names  # requires
import supervisely as sly
from dotenv import load_dotenv
from supervisely.app.widgets import Button, Card, Container, Text
```

### Init API client

Init API for communicating with Supervisely instance. First, we load environment variables with credentials:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))
api = sly.Api()
```

### Initialize `Text` and `Button` widgets.

```python
hello_msg = Text(text="Hello, World!", status="text")
start_btn = Button(text="Generate Name", icon="zmdi zmdi-play")
```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter and place 2 widgets that we've just created in the `Container` widget. Place order in the `Container` is also important, we want the "hello text" to be above the name generation button.

```python
layout = Card(
    title="Hello, World!", 
    content=Container([hello_msg, start_btn])
    )
```

### Create app using layout

Create an app object with layout parameter.

```python
app = sly.Application(layout=layout)
```

<figure><img src="https://user-images.githubusercontent.com/48913536/194583142-06d801c8-fe97-4429-9d9a-6bac720eefda.png" alt=""><figcaption><p>Layout</p></figcaption></figure>

### Handle button clicks

Use the decorator as shown below to handle button click. When we change `hello_msg.text` value, data will be pushed to web browser via web sockets.

```python
@start_btn.click
def generate_name():
    hello_msg.text = f"Hello, {names.get_first_name()}!"
```

<figure><img src="https://user-images.githubusercontent.com/48913536/194533336-6983fbd9-c6dc-4f44-867d-aec8526d9a64.gif" alt=""><figcaption><p>Result</p></figcaption></figure>
