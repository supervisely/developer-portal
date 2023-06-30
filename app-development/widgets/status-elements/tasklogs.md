# TaskLogs

## Introduction

The**`TaskLogs`** widget in Supervisely is designed to display task logs for a given task ID.

> ℹ️ This widget will display logs only when the application with the widget is released **in production**. In development mode, a message will be shown: `"You are currently in development mode. Task logs will be displayed only in production mode."`

## Function signature

```python
TaskLogs(task_id=None, widget_id=None)
```

<figure><img src="https://user-images.githubusercontent.com/79905215/249870664-619dfbce-8ac6-44ec-b52a-15f675e74dfa.png" alt=""><figcaption></figcaption></figure>

## Parameters

|  Parameters |  Type |           Description           |
| :---------: | :---: | :-----------------------------: |
|  `task_id`  | `int` | The task ID to display logs for |
| `widget_id` | `str` |         ID of the widget        |

### task\_id

The task ID to display logs for.

**type:** `str`

**default value:** `None`

```python
task_id = 36926
logs = TaskLogs(task_id)
```

<figure><img src="https://user-images.githubusercontent.com/79905215/250025703-2a616b26-af64-4608-8fc3-6ff5476cda7c.png" alt=""><figcaption></figcaption></figure>

### widget\_id

ID of the widget.

**type:** `str`

**default value:** `None`

## Methods and attributes

|    Attributes and Methods   | Description      |
| :-------------------------: | ---------------- |
|       `get_task_id()`       | Get the task ID. |
| `set_task_id(task_id: int)` | Set the task ID. |

## Mini App Example

You can find this example in our Github repository:

[supervisely-ecosystem/ui-widgets-demos/status-elements/005\_task\_logs/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/status%20elements/005\_task\_logs/src/main.py)

### Import libraries

```python
import supervisely as sly
from supervisely.app.widgets import Button, Card, Container, InputNumber, TaskLogs
```

> ℹ️ This widget will display logs only when the application with the widget is released **in production**. In development mode, a message will be shown: `"You are currently in development mode. Task logs will be displayed only in production mode."`

### Initialize `TaskLogs` widget we will use in UI

```python
task_id = 36925

logs = TaskLogs(task_id)
```

or

```python
logs = TaskLogs()
logs.set_task_id(task_id)
```

### Add `Button` and `InputNumber` widgets to use in demo

```python
btn = Button("Set task id", button_size="small")
input_task_id = InputNumber(size="small", controls=False)
```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter and place widget that we've just created in the `Container` widget.

```python
container = Container(widgets=[input_task_id, btn, logs])
card = Card(content=container, title="Logs")
layout = Container(widgets=[card])
```

### Create app using layout

Create an app object with `layout` parameter.

```python
app = sly.Application(layout=layout)
```

### Add function to controls widgets from python code

```python
@btn.click
def set_task_id():
    task_id = int(input_task_id.get_value())
    logs.set_task_id(task_id)
```

<figure><img src="https://user-images.githubusercontent.com/79905215/250033161-d3b88741-736d-458c-8c0a-cdd3613bc10a.gif" alt=""><figcaption></figcaption></figure>
