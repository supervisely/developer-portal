# Folder Thumbnail

## Introduction

**`FolderThumbnail`** widget in Supervisely is a widget that allows for quickly identifying and navigating to specific folder in Team files. Clicking on it opens the folder in Supervisely Team files interface.


[Read this tutorial in developer portal.](https://developer.supervise.ly/app-development/widgets/thumbnail/folderthumbnail)

## Function signature

```python
FolderThumbnail(info=None, widget_id=None)
```

![default](https://user-images.githubusercontent.com/120389559/218969636-c64db884-4133-4d80-b23e-7375b7b536b2.png)

## Parameters

| Parameters  |    Type    |         Description          |
| :---------: | :--------: | :--------------------------: |
|   `info`    | `FileInfo` | Information about given file |
| `widget_id` |   `str`    |       ID of the widget       |

### info

Determine information about given file.

**type:** `FileInfo`

**default value:** `None`

### widget_id

ID of the widget.

**type:** `str`

**default value:** `None`

## Methods and attributes

| Attributes and Methods | Description                                |
| :--------------------: | ------------------------------------------ |
| `set(info: FileInfo)`  | Set given `FileInfo` in `FolderThumbnail` widget. |

## Mini App Example

You can find this example in our Github repository:

[supervisely-ecosystem/ui-widgets-demos/thumbnail/004_folder_thumbnail/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/thumbnail/004_folder_thumbnail/src/main.py)

### Import libraries

```python
import os

import supervisely as sly
from dotenv import load_dotenv
from supervisely.app.widgets import Card, Container, FolderThumbnail
import supervisely.io.env as env
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()
```

### Prepare file and upload in Team files

```python
team_id = env.team_id()
local_filepath = "/Users/almaz/job/text.txt"
remote_filepath = "/folder_thumbnail_demo/text.txt"

if api.file.exists(team_id, remote_filepath):
    api.file.remove(team_id, remote_filepath)

api.file.upload(team_id, local_filepath, remote_filepath)
# get file info from server
fileinfo = api.file.get_info_by_path(team_id, remote_filepath)
```

### Initialize `FolderThumbnail` widget

```python
folder_thumbnail = FolderThumbnail(info=fileinfo)
```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter and place widget that we've just created in the `Container` widget.

```python
card = Card(
    title="Folder Thumbnail",
    content=Container(widgets=[folder_thumbnail]),
)

layout = Container(widgets=[card])
```

### Create app using layout

Create an app object with layout parameter.

```python
app = sly.Application(layout=layout)
```

![layout](https://user-images.githubusercontent.com/120389559/218971785-137c437e-5a9f-47e3-a292-a0114532dc5c.png)
