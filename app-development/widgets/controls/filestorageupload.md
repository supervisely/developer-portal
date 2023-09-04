# File storage upload

## Introduction

**`FileStorageUpload`** is a widget in Supervisely's web interface that allows users to upload files directly to Team files by given path. With this widget, users can easily transfer data from their local machine to Team files without the need for any external tools or commands. The widget supports multiple file uploads and progress tracking, making the process of transferring data efficient and streamlined.


## Function signature

```python
FileStorageUpload(
    team_id=435,
    path="folder",
    change_name_if_conflict=False,
    widget_id=None,
)
```

![default](https://user-images.githubusercontent.com/79905215/224288968-b3edf93e-2bd9-4a26-94f8-41165e0e3387.png)

## Parameters

|        Parameters         |  Type  |                        Description                         |
| :-----------------------: | :----: | :--------------------------------------------------------: |
|         `team_id`         | `int`  |                          Team ID                           |
|          `path`           | `str`  | Set destination path in Team files to upload files/folders |
| `change_name_if_conflict` | `bool` |      Whether change destination folder name if exists      |
|        `widget_id`        | `str`  |                      ID of the widget                      |

### team_id

Team ID

**type:** `int`

### path

Set destination path in Team files to upload files/folders

**type:** `str`

**default value:** `"folder"`

```python
file_upload = FileStorageUpload(
    team_id=435,
    path="folder",
)
```

<p align="center">
  <img src="https://user-images.githubusercontent.com/79905215/224320961-52434e4c-4ac2-4d37-a9fb-6ce248dc8760.gif" alt="team_id_and_path" />
</p>

### change_name_if_conflict

Whether change destination folder name if exists.

**type:** `bool`

**default value:** `False`

```python
file_upload = FileStorageUpload(
    team_id=435,
    path="folder",
    change_name_if_conflict=True
)
```

<p align="center">
  <img src="https://user-images.githubusercontent.com/79905215/224321678-bb188f7d-0051-4946-afe3-2db589e63daa.gif" alt="change_name_if_conflict" />
</p>

### widget_id

ID of the widget

**type:** `str`

**default value:** `None`

## Methods and attributes

| Attributes and Methods | Description                                                  |
| :--------------------: | ------------------------------------------------------------ |
|         `path`         | Get or set `path` property to upload files.                  |
| `set_path(path: str)`  | Set `path` to upload files to.                               |
| `get_uploaded_paths()` | Get list of path in Team files where files/folders uploaded. |

## Mini App Example

You can find this example in our Github repository:

[supervisely-ecosystem/ui-widgets-demos/controls/007_file_storage_upload/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/controls/007_file_storage_upload/src/main.py)

### Import libraries

```python
import os

from dotenv import load_dotenv
import supervisely as sly
from supervisely.app.widgets import Button, Card, Container, Field
from supervisely.app.widgets import FileStorageUpload, Flexbox, Input, Text
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

### Initialize `FileStorageUpload` widget

```python
file_upload_1 = FileStorageUpload(
    team_id=team_id,
    path="folder",
)

file_upload_2 = FileStorageUpload(
    team_id=team_id,
    path="folder",
    change_name_if_conflict=True,
)
```

### Create `Field`, `Button`, `Text`, `Input` widgets we will use in UI for demo

```python
upload_1 = Field(
    title="Existing directory",
    description="Upload files/folders to an existing directory in Team files",
    content=file_upload_1,
)


upload_2 = Field(
    title="New path",
    description="Enter a new path and set it by clicking on button",
    content=file_upload_2,
)

input = Input(placeholder="Please enter path")
button_change_path = Button("Change path name")
button_get_paths_1 = Button("Get upoaded paths")
button_get_paths_2 = Button("Get upoaded paths")

text = Text()
```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter and place widget that we've just created in the `Container` widget.

```python
upload_container = Container([upload_1, upload_2])
btns_box = Flexbox([button_change_path, button_get_paths_1, button_get_paths_2])
controls_container = Container([input, btns_box, text])

card = Card(
    title="File Storage Upload",
    content=Container([upload_container, controls_container]),
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
@button_change_path.click
def change_path():
    path = input.get_value()
    file_upload_2.set_path(path)
    file_upload_1.set_path(path)
    text.status = "success"
    text.text = "Upload path has been changed."


@button_get_paths_1.click
def show_uploaded_paths():
    paths = file_upload_1.get_uploaded_paths()
    text.status = "text"
    text.text = "<br>".join(paths)


@button_get_paths_2.click
def show_uploaded_paths():
    paths = file_upload_2.get_uploaded_paths()
    text.status = "text"
    text.text = "<br>".join(paths)
```

<p align="center">
    <img src="https://user-images.githubusercontent.com/79905215/224339790-01af8df0-16fd-4b34-ab81-5ea760a33f46.gif" alt="layout">
</p>
