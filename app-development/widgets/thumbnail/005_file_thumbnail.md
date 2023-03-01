# File Thumbnail

## Introduction

**`FileThumbnail`** widget in Supervisely displays an icon, link, and path to the file in Team Files. Clicking on the widget opens Supervisely Team Files interface, where you can view and edit the file.

[Read this tutorial in developer portal.](https://developer.supervise.ly/app-development/widgets/thumbnail/filethumbnail)

## Function signature

```python
FileThumbnail(info=None, widget_id=None)
```

![default](https://user-images.githubusercontent.com/120389559/218972939-2bbfb453-a841-45a6-8a2d-c4aec487095a.png)

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

| Attributes and Methods | Description                              |
| :--------------------: | ---------------------------------------- |
| `set(info: FileInfo)`  | Set given `FileInfo` in `FileThumbnail`. |

## Mini App Example

You can find this example in our Github repository:

[supervisely-ecosystem/ui-widgets-demos/thumbnail/005_file_thumbnail/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/thumbnail/005_file_thumbnail/src/main.py)

### Import libraries

```python
import os

import supervisely as sly
from dotenv import load_dotenv
from supervisely.app.widgets import Card, Container, FileThumbnail
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

### Initialize `FileThumbnail` widget

```python
file_thumbnail = FileThumbnail(info=fileinfo)
```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter and place widget that we've just created in the `Container` widget.

```python
card = Card(
    title="File Thumbnail",
    content=Container(widgets=[file_thumbnail]),
)

layout = Container(widgets=[card])
```

### Create app using layout

Create an app object with layout parameter.

```python
app = sly.Application(layout=layout)
```

![layout](https://user-images.githubusercontent.com/120389559/218973475-708c8587-f82b-4ffa-a21c-298791025ba8.png)
