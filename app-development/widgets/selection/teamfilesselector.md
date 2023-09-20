# Team files selector

## Introduction

**`TeamFilesSelector`** is a graphic interface widget in the Supervisely platform that enables users to easily select files and/or folders from their Team files. It allows to customize selecting type (files, folder or both of this types) and displaying additional columns with information about files and folder (size, created date, updated date, type, mime type). The widget features a user-friendly interface and is optimized for performance, making it a valuable tool for teams working.

## Function signature

```python
TeamFilesSelector(
    team_id=435,
    multiple_selection=False,
    max_height=500,
    selection_file_type=None,
    hide_header=True,
    hide_empty_table=True,
    additional_fields=[],
    widget_id=None,
)
```

![default](https://user-images.githubusercontent.com/79905215/224646397-cf970a40-d7f8-4842-a98d-009df109533e.png)

## Parameters

|      Parameters       |                                    Type                                     |                                 Description                                  |
| :-------------------: | :-------------------------------------------------------------------------: | :--------------------------------------------------------------------------: |
|       `team_id`       |                                    `int`                                    |                                   Team ID                                    |
| `multiple_selection`  |                                   `bool`                                    |              Whether available selection multiple files/folders              |
|     `max_height`      |                                    `int`                                    |                    Determine maximum height of the widget                    |
| `selection_file_type` |                    `Literal["folder", "file"]` or `None`                    |               Determine type of items available for selection                |
|     `hide_header`     |                                   `bool`                                    |                      If `True` hide widget table header                      |
|  `hide_empty_table`   |                                   `bool`                                    |     If `True` and Team files directory is empty it will display message      |
|  `additional_fields`  | `List[Literal["id", "createdAt", "updatedAt", "type", "size", "mimeType"]]` | Determine column names to display additional information about files/folders |
|      `widget_id`      |                                    `str`                                    |                               ID of the widget                               |

### team_id

Team ID

**type:** `int`

```python
file_selector = TeamFilesSelector(team_id=435)
```

![team_id](https://user-images.githubusercontent.com/79905215/224650598-747dd279-4ac1-4057-bbdb-1501c40eb30b.png)

### multiple_selection

Whether available selection multiple files/folders

**type:** `bool`

**default value:** `False`

```python
file_selector = TeamFilesSelector(
    team_id=435,
    multiple_selection=True,
)
```

<p align="center">
  <img src="https://user-images.githubusercontent.com/79905215/224651701-f7493dd0-cb7f-4ff5-878f-43514651eb40.gif" alt="multiselect_true" />
</p>

```python
file_selector = TeamFilesSelector(
    team_id=435,
    multiple_selection=True,
)
```

<p align="center">
  <img src="https://user-images.githubusercontent.com/79905215/224651760-df61466e-9ec4-4965-afa7-5c60bc93565b.gif" alt="multiselect_false" />
</p>

### max_height

Determine maximum height of the widget

**type:** `int`

**default value:** `500`

```python
file_selector = TeamFilesSelector(
    team_id=435,
    max_height=200,
)
```

<p align="center">
  <img src="https://user-images.githubusercontent.com/79905215/224652579-364da35d-1a2b-4719-bfe6-ddfc42d20cb7.gif" alt="max_height" />
</p>

### hide_header

If `True` hide widget table header

**type** `bool`

**default value:** `True`

```python
file_selector = TeamFilesSelector(
    team_id=435,
    hide_header=False
)
```

### selection_file_type

Determine type of items available for selection

**type:** `Literal["folder", "file"]` or `None`

**default value:** `None`

```python
file_selector = TeamFilesSelector(
    team_id=435,
    selection_file_type=None,
)
```

<p align="center">
  <img src="https://user-images.githubusercontent.com/79905215/224663695-c1c7d261-95fb-4f39-8a5f-b87526904189.gif" alt="type_none" />
</p>

```python
file_selector = TeamFilesSelector(
    team_id=435,
    selection_file_type="file"
)
```

![type_file](https://user-images.githubusercontent.com/79905215/224664127-234a9f9e-254b-4ead-b02e-c288355f34d3.png)

```python
file_selector = TeamFilesSelector(
    team_id=435,
    selection_file_type="folder"
)
```

![type_folder](https://user-images.githubusercontent.com/79905215/224664204-4f0fb069-6430-404a-bc40-015dd3d44fd5.png)

### hide_empty_table

If `True` and Team files directory is empty it will display message

**type:** `bool`

**default value:** `True`

```python
file_selector = TeamFilesSelector(
    team_id=435,
    hide_empty_table=True,
)
```

![empty_true](https://user-images.githubusercontent.com/79905215/224665005-98483fcb-11f2-403b-bb74-38dd900833ad.png)

```python
file_selector = TeamFilesSelector(
    team_id=435,
    hide_empty_table=False,
)
```

![empty_false](https://user-images.githubusercontent.com/79905215/224665087-0ddbc209-8ee0-4e40-ba32-6f34edb0f8b8.png)

### additional_fields

Determine column names to display additional information about files/folders

**type:** `List[Literal["id", "createdAt", "updatedAt", "type", "size", "mimeType"]]`

**default value:** `[]`

```python
file_selector = TeamFilesSelector(
    team_id=435,
    hide_header=False,
    additional_fields=["id", "createdAt", "updatedAt", "type", "size", "mimeType"],
)
```

![additional_fields](https://user-images.githubusercontent.com/79905215/224665607-714df427-f8bd-43a3-bf28-3e2cc16ab8ed.png)

### widget_id

ID of the widget

**type:** `str`

**default value:** ``

## Methods and attributes

|   Attributes and Methods    | Description                                                   |
| :-------------------------: | ------------------------------------------------------------- |
|   `get_selected_paths()`    | Get list of path for selected files/folders in Team files.      |
|   `get_selected_items()`    | Get list of selected files/folders information in Team files.   |
| `set_team_id(team_id: int)` | Set `team_id` for `TeamFilesSelector` widget.                 |

## Mini App Example

You can find this example in our Github repository:

[supervisely-ecosystem/ui-widgets-demos/selection/012_team_files_selector/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/selection/012_team_files_selector/src/main.py)

### Import libraries

```python
import os

import supervisely as sly
from dotenv import load_dotenv
from supervisely.app.widgets import Button, Card, Container, TeamFilesSelector, Text
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))
api = sly.Api()
```

### Get `team_id` from environment variables

```python
team_id = sly.env.team_id()
```

### Initialize `TeamFilesSelector` widget

```python
file_selector = TeamFilesSelector(
    team_id=team_id,
    max_height=300,
    multiple_selection=True,
    selection_file_type="folder",
    hide_header=False,
    additional_fields=["createdAt", "type", "size"],
)
```

### Create `Text`, `Button` widgets we will use in UI for demo

```python
text = Text()
button = Button()
```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter and place widget that we've just created in the `Container` widget.

```python
card = Card(
    title="Team Files Selector",
    content=Container([file_selector, button, text]),
)
layout = Container(widgets=[card])
```

### Create app using layout

Create an app object with layout parameter.

```python
app = sly.Application(layout=layout)
```

### Add functions to control widgets from code

```python
@button.click
def show_selected():
    selected_paths = file_selector.get_selected_paths()
    text.text = "<br>".join(selected_paths)
```

<p align="center">
  <img src="https://user-images.githubusercontent.com/79905215/224669380-35e5ce6f-b7ee-4d46-bf8e-bfc704e96b40.gif" alt="layout" />
</p>
