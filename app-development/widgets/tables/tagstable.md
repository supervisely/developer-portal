# TagsTable

## Introduction

**`TagsTable`** widget in Supervisely is a table that displays all given tags from the project meta. It allows you to select tags, get a list of selected tags, select all tags or clear the selection. For example, you can use this widget in training applications to select tags for the training process or in applications where you need to filter data by tags.

## Function signature

```python
TagsTable(
    project_meta,
    project_id=None,
    project_fs=None,
    allowed_types=[sly.TagValueType.NONE],
    selectable=True,
    disabled=False,
    widget_id=None,
)
```

<figure><img src="https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/79905215/98cb2e85-5d18-4312-b644-7ec20ca244fb" alt=""><figcaption></figcaption></figure>

## Parameters

|   Parameters    |                Type                |             Description             |
| :-------------: | :--------------------------------: | :---------------------------------: |
| `project_meta`  |    `Optional[sly.ProjectMeta]`     | Project meta in Supervisely format  |
|  `project_id`   |          `Optional[int]`           |     Project ID from Supervisely     |
|  `project_fs`   |      `Optional[sly.Project]`       |  Project object from local project  |
| `allowed_types` | `Optional[List[sly.TagValueType]]` |      List of allowed tag types      |
|  `selectable`   |          `Optional[bool]`          | Enable or disable selection of tags |
|   `disabled`    |          `Optional[bool]`          |           Disable widget            |
|   `widget_id`   |          `Optional[str]`           |          ID of the widget           |


### widget_id

ID of the widget.

**type:** `str`

**default value:** `None`

## Methods and attributes

| Attributes and Methods | Description                                                               |
| :--------------------: | ------------------------------------------------------------------------- |
|    `allowed_types`     | Property. Get a list of allowed tag types.                                |
|      `project_id`      | Property. Get project ID.                                                 |
|      `project_fs`      | Property. Get project object from local project.                          |
|       `loading`        | Property. Get loading status.                                             |
|      `read_meta`       | Read project meta from `sly.ProjectMeta` object and set it to the widget. |
| `read_project_from_id` | Read project from Supervisely by `project_id` and set it to the widget.   |
|     `read_project`     | Read project from local project by `project_fs` and set it to the widget. |
|  `get_selected_tags`   | Return a list of selected tags (str).                                     |
|   `set_project_meta`   | Set project meta to the widget.                                           |
|     `select_tags`      | Select tags by given names.                                               |
|      `select_all`      | Select all tags.                                                          |
|   `clear_selection`    | Clear selection of tags.                                                  |

### allowed_types

Property. Get a list of allowed tag types.

**rtype:** `List[sly.TagValueType]`

### project_id

Property. Get project ID.

**rtype:** `int`

### project_fs

Property. Get project object from local project.

**rtype:** `sly.Project`

### loading

Property. Get loading status.

**rtype:** `bool`

### read_meta

Read project meta from `sly.ProjectMeta` object and set it to the widget.

```python
project_meta = sly.ProjectMeta.from_json(api.project.get_meta(project_id))
tags_table = TagsTable(project_meta=project_meta)
```

{% hint style="info" %}
If you initialize the `TagsTable` widget without providing one of the `project_id` or `project_fs` parameters, the "images count" and "labels count" columns in the table will be hidden. If you need to display information about the number of images and labels for each tags, make sure to pass the appropriate `project_id` or `project_fs` object when initializing the widget.
{% endhint %}

<figure><img src="https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/79905215/d4baab9e-6105-4e76-8325-5c0c02ed0b43" alt=""><figcaption></figcaption></figure>

### read_project_from_id

Read data from Supervisely by `project_id` and set it to the widget.

```python
tags_table = TagsTable(project_id=777)
```

<figure><img src="https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/79905215/98cb2e85-5d18-4312-b644-7ec20ca244fb" alt=""><figcaption></figcaption></figure>

### read_project

Read data from a local project by `project_fs` and set it to the widget.

```python
project_fs = sly.Project(directory="/path/to/project")
tags_table = TagsTable(project_fs=project_fs)
```

<figure><img src="https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/79905215/98cb2e85-5d18-4312-b644-7ec20ca244fb" alt=""><figcaption></figcaption></figure>

### get_selected_tags

Return a list of selected tags (str).

**rtype:** `List[str]`

```python
selected_tags = tags_table.get_selected_tags()
print(selected_tags)
```

Output:

```python
["age", "tag2"]
```

### set_project_meta

Set project meta to the widget.

```python
project_meta = sly.ProjectMeta.from_json(api.project.get_meta(project_id))
tags_table.set_project_meta(project_meta)
```

### select_tags

Select tags by given names.

```python
tags_table.select_tags(["tag1", "tag2"])
```

### select_all

Select all tags.

```python
tags_table.select_all()
```

### clear_selection

Clear selection of tags.

```python
tags_table.clear_selection()
```

## Mini App Example

You can find this example in our Github repository:

[ui-widgets-demos/tables/007_tags_table/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/tables/007_tags_table/src/main.py)

### Create `local.env` file

Create a `local.env` file with the following environment variables:
Read more about environment variables [here](../../../getting-started/environment-variables.md)

```
PROJECT_ID=38857
```

### Import libraries

```python
import os

import supervisely as sly
from dotenv import load_dotenv
from supervisely.app.widgets import Button, Card, Container, InputNumber, TagsTable, Text
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
if sly.is_development():
    load_dotenv("local.env")
    load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()
```

### Get project ID from environment variables

```python
project_id = sly.env.project_id()
```

### Initialize `TagsTable`, `Button` and `Text` widgets

```python
tags_table = TagsTable(project_id=project_id)
tags_text = Text("Selected tags: ")
input_number = InputNumber(controls=False)
btn = Button("Set project ID")
```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter and place widget that we've just created in the `Container` widget.

```python
card = Card(
    title="TagsTable",
    content=Container([tags_table, tags_text, input_number, btn]),
)
```

### Create the app using a layout

Create an app object with a layout parameter.

```python
app = sly.Application(layout=card)
```

### Add function to control widget from code

```python
@tags_table.value_changed
def display_selected_tags(value):
    tags_text.text = f"Selected tags: {', '.join(value)}"

@btn.click
def click_btn():
    tags_table.read_project_from_id(input_number.value)
```

<figure><img src="https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/79905215/27f8292c-17ce-4655-b6ca-8277846fbcfd" alt=""><figcaption></figcaption></figure>
