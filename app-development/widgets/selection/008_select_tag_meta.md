# Select Tag Meta

## Introduction

This widget is a select `TagMeta` input, clicking on it can be processed from python code. In this tutorial you will learn how to use `SelectTagMeta` widget in Supervisely app.


[Read this tutorial in developer portal.](https://developer.supervise.ly/app-development/apps-with-gui/SelectTagMeta)

## Function signature

```python

SelectTagMeta(
    default=None,
    project_id=None,
    project_meta=None,
    allowed_types=None,
    multiselect=False,
    show_label=True,
    size=None,
    widget_id=None,
)

```

![default](https://user-images.githubusercontent.com/120389559/218047319-9e4b681c-ee63-4709-b5c1-24c94f457e7a.png)

## Parameters

|   Parameters    |                   Type                    |                                    Description                                     |
| :-------------: | :---------------------------------------: | :--------------------------------------------------------------------------------: |
|    `default`    |                   `str`                   |                                     `Tag` name                                     |
|  `project_id`   |                   `int`                   |              Determine `Project` from which `Tags` will be selected.               |
| `project_meta`  |               `ProjectMeta`               |             Determine `ProjectMeta` from which `Tags` will be selected             |
| `allowed_types` |                `List[str]`                | Determine `Tags` types witch will be available to select from all `Project` `Tags` |
|  `multiselect`  |                  `bool`                   |                          Allows to select multiple `Tags`                          |
|  `show_label`   |                  `bool`                   |                                     Show label                                     |
|     `size`      | `Literal["large", "small", "mini", None]` |                          Selector size (large/small/mini)                          |
|   `widget_id`   |                   `str`                   |                                  Id of the widget                                  |

### default

Determine `Tag` will be selected by default.

**type:** `str`

**default value:** `None`

```python
select_tag_meta = SelectTagMeta(default="rust", project_id=project_id)
```

![default](https://user-images.githubusercontent.com/120389559/218047689-6236f160-b0f9-43da-b0b8-f44e3a9eacbb.png)

### project_id

Determine `Project` from which `Tags` will be selected.

**type:** `int`

**default value:** `None`

```python
select_tag_meta = SelectTagMeta(project_id=project_id)
```

![project_id](https://user-images.githubusercontent.com/120389559/218047909-e3433e3b-a6b9-42cb-b7d7-3fc37788a481.png)

### project_meta

Determine `ProjectMeta` from which `Tags` will be selected.

**type:** `ProjectMeta`

**default value:** `None`

```python
meta_json = api.project.get_meta(project_id)
project_meta = sly.ProjectMeta.from_json(meta_json)
select_tag_meta = SelectTagMeta(project_meta=project_meta)
```

![project_meta](https://user-images.githubusercontent.com/120389559/218048143-452ccdd9-0573-494c-af19-d427585ee2bd.png)

### allowed_types

Determine `Tags` types witch will be available to select from all `Project` `Tags`. Possible `Tag` types: `any_number`, `any_string`, `one_of_string`, `none`.

**type:** `List[str]`

**default value:** `None`

```python
from supervisely.annotation.tag_meta import TagValueType
allowed_types = [TagValueType.ANY_STRING, TagValueType.NONE]
select_tag_meta = SelectTagMeta(project_id=project_id, allowed_types=allowed_types)
```

![allowed_types](https://user-images.githubusercontent.com/120389559/218048774-60b853c8-a597-4e8b-8eeb-216687b7d416.png)

### multiselect

Allows to select multiple `Tags`.

**type:** `bool`

**default value:** `False`

```python
select_tag_meta = SelectTagMeta(project_id=project_id, multiselect=True)
```

![multiselect](https://user-images.githubusercontent.com/120389559/218099610-e36e6221-74d3-4271-89d4-da7edf088f80.gif)

### show_label

Determine show text `Tag` on widget or not.

**type:** `bool`

**default value:** `True`

```python
select_tag_meta = SelectTagMeta(project_id=project_id, show_label=False)
```

![show_label](https://user-images.githubusercontent.com/120389559/218049460-041e2086-548c-4961-8245-33847900b59c.png)

### size

Size of input.

**type:** `Literal["large", "small", "mini", None]`

**default value:** `None`

```python
select_tag_meta = SelectTagMeta(default="cat", project_id=project_id, show_label=False)
select_mini = SelectTagMeta(default="cat", project_id=project_id, show_label=False, size="mini")
select_small = SelectTagMeta(default="cat", project_id=project_id, show_label=False, size="small")
select_large = SelectTagMeta(default="cat", project_id=project_id, show_label=False, size="large")

card = Card(
    title="Select TagMeta",
    content=Container(widgets=[select_tag_meta, select_mini, select_small, select_large]),
)
```

![size](https://user-images.githubusercontent.com/120389559/218728869-7b0f787e-e09b-4c72-9353-93f6398611b9.png)

### widget_id

ID of the widget.

**type:** `str`

**default value:** `None`

## Methods and attributes

|      Attributes and Methods       | Description                                                                                              |
| :-------------------------------: | -------------------------------------------------------------------------------------------------------- |
|       `get_selected_name()`       | Return selected tag name. If multiselect is `True` raise `RuntimeError`.                                 |
|      `get_selected_names()`       | Return `List` with selected `Tag` names. If multiselect is `False` raise `RuntimeError`.                 |
| `get_tag_meta_by_name(name: str)` | Return `TagMeta` by `Tag` name.                                                                          |
|       `get_selected_item()`       | Return `TagMeta` for selected `Tag`.                                                                     |
|      `get_selected_items()`       | Return `List`, containing `TagMeta` for selected tags.                                                   |
|       `set_name(name: str)`       | Set `Tag` with given name as `selected`.                                                                 |
|   `set_names(names: List[str])`   | Set `Tags` from given list of `Tag` names as `selected`. If multiselect is `False` raise `RuntimeError`. |

## Mini App Example

You can find this example in our Github repository:

[supervisely-ecosystem/ui-widgets-demos/015_select_tag_meta/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/015_select_tag_meta/src/main.py)

### Import libraries

```python
import os

import supervisely as sly
from dotenv import load_dotenv
from supervisely.app.widgets import Card, Container, SelectTagMeta
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()
```

### Prepare `Project` ID

```python
project_id = int(os.environ["modal.state.slyProjectId"])
```

### Initialize `SelectTagMeta` widget

```python
select_tag_meta = SelectTagMeta(default="cat", project_id=project_id)
```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter and place widget that we've just created in the `Container` widget.

```python
card = Card(
    title="Select TagMeta",
    content=select_tag_meta,
)

layout = Container(widgets=[card])
```

### Create app using layout

Create an app object with layout parameter.

```python
app = sly.Application(layout=layout)
```

![layout](https://user-images.githubusercontent.com/120389559/218049706-c4b125c8-bc0d-49e4-b829-cde852f008ef.png)
