# Video Thumbnail

## Introduction

**`VideoThumbnail`** widget in Supervisely is a widget that allows to display a thumbnail image that represents specific video. It is a useful widget for applications that work with videos, allowing users to have quick access to this video, so that when the user clicks on the thumbnail, the link will take him to this video.

[Read this tutorial in developer portal.](https://developer.supervise.ly/app-development/widgets/thumbnail/videothumbnail)

## Function signature

```python
VideoThumbnail(info=None, widget_id=None)
```

![default](https://user-images.githubusercontent.com/120389559/217834448-44807559-f3b8-482b-a9e8-c3f95b1c0f3f.png)

## Parameters

| Parameters  |    Type     |                       Description                        |
| :---------: | :---------: | :------------------------------------------------------: |
|   `info`    | `VideoInfo` | `NamedTuple`, containing information about video project |
| `widget_id` |    `str`    |                     ID of the widget                     |

### info

`NamedTuple`, containing information about video project.

**type:** `VideoInfo`

**default value:** `None`

```python
video_info = api.video.get_info_by_id(video_id)
video_thumbnail = VideoThumbnail(info=video_info)
```

### widget_id

ID of the widget.

**type:** `str`

**default value:** `None`

## Methods and attributes

|    Attributes and Methods    | Description                                         |
| :--------------------------: | --------------------------------------------------- |
| `set_video(info: VideoInfo)` | Set input video project data by video project info. |
|   `set_video_id(id: int)`    | Set input video project data by video project id.   |

## Mini App Example

You can find this example in our Github repository:

[supervisely-ecosystem/ui-widgets-demos/thumbnail/003_video_thumbnail/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/thumbnail/003_video_thumbnail/src/main.py)

### Import libraries

```python
import os

import supervisely as sly
from dotenv import load_dotenv
from supervisely.app.widgets import Card, Container, VideoThumbnail
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()
```

### Initialize `VideoThumbnail` widget

```python
video_id = int(os.environ["modal.state.slyVideoId"])
video_info = api.video.get_info_by_id(id=video_id)

video_thumbnail = VideoThumbnail(info=video_info)
```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter and place widget that we've just created in the `Container` widget.

```python
card = Card(
    title="Video Thumbnail",
    content=Container(widgets=[video_thumbnail]),
)
layout = Container(widgets=[card])
```

### Create app using layout

Create an app object with layout parameter.

```python
app = sly.Application(layout=layout)
```
