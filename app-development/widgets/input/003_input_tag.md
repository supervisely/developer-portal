# Input Tag

## Introduction

In this tutorial you will learn how to use `InputTag` widget in Supervisely app.

[Read this tutorial in developer portal.](https://developer.supervise.ly/app-development/apps-with-gui/inputtag)

## Function signature

```python
InputTag(tag_meta, max_width=300, widget_id=None)
```

![default](https://user-images.githubusercontent.com/120389559/218993249-8d449098-3efa-4c60-92d5-3019c76a1106.gif)

## Parameters

| Parameters  |   Type    |                 Description                  |
| :---------: | :-------: | :------------------------------------------: |
| `tag_meta`  | `TagMeta` | `TagMeta` from which `Tags` will be selected |
| `max_width` |   `int`   |             Max tag field width              |
| `widget_id` |   `str`   |               Id of the widget               |

### tag_meta

Determine `TagMeta` from which `Tags` will be selected. Possible `Tag` types: `any_number`, `any_string`, `one_of_string`, `none`.

**type:** `TagMeta`

### max_width

Determine max tag field width.

**type:** `int`

**default value:** `300`

```python
tag_inputs = [InputTag(t, max_width=800) for t in tag_metas]
```

![max_width](https://user-images.githubusercontent.com/120389559/219026202-ec7ebafe-215a-4672-b833-4c826bc6fd0e.png)

### widget_id

ID of the widget.

**type:** `str`

**default value:** `None`

## Methods and attributes

|       Attributes and Methods       | Description                                   |
| :--------------------------------: | --------------------------------------------- |
|          `get_tag_meta()`          | Return current `TagMeta`.                     |
|            `activate()`            | Activate `InputTag` switch.                   |
|           `deactivate()`           | Deactivate `InputTag` switch.                 |
|           `is_active()`            | Check `InputTag` switch is active.            |
| `is_valid_value(value: tag.value)` | Check `InputTag` current value is valid.      |
|    `set(tag: Union[Tag, None])`    | Set given value in `InputTag`.                |
|            `get_tag()`             | Get current `Tag` from `InputTag`.            |
|         `value_changed()`          | Handled when `InputTag` value is changed.     |
|       `selection_changed()`        | Handled when `InputTag` selection is changed. |
|     `value(value: tag.value)`      | Handled when input value is setting.          |

## Mini App Example

You can find this example in our Github repository:

[supervisely-ecosystem/ui-widgets-demos/047_input_tag/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/047_input_tag/src/main.py)

### Import libraries

```python
import os

import supervisely as sly
from dotenv import load_dotenv
from supervisely.app.widgets import Button, Card, Container, InputTag, Text

```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()
```

### Initialize `Project` ID and `TagMeta` we will use

```python
project_id = sly.env.project_id()

meta_json = api.project.get_meta(project_id)
project_meta = sly.ProjectMeta.from_json(meta_json)
tag_metas = project_meta.tag_metas
```

### Initialize list of `InputTag` widgets for each `TagMeta` in project

```python
tag_inputs = [InputTag(t) for t in tag_metas]
```

### Initialize `Button` and `Text` widgets we will use

```python
toggle_btn = Button("Toggle")
value_text = Text()
```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter and place widget that we've just created in the `Container` widget.

```python
tag_inputs_container = Container(widgets=tag_inputs)
container = Container(widgets=[tag_inputs_container, toggle_btn, value_text])
card = Card(title="Tag inputs", content=container)
layout = Container(widgets=[card])
```

### Create app using layout

Create an app object with layout parameter.

```python
app = sly.Application(layout=layout)
```

## Handle button clicks

Use the decorators to handle button click and tag values changing. We have button to change tags switch status and text field to show tags values changing.

```python
for tag_input in tag_inputs:

    @tag_input.value_changed
    def show_message(value):
        value_text.text = value


@toggle_btn.click
def toggle_tag_inputs():
    for tag_input in tag_inputs:
        if tag_input.is_active():
            tag_input.deactivate()
        else:
            tag_input.activate()
```

![layout](https://user-images.githubusercontent.com/120389559/219036626-79af7718-3e93-4528-8a11-642c8798e154.gif)
