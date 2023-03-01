# Video

## Introduction

**`Video`** widget in Supervisely is a custom player designed for displaying videos and annotations frame-by-frame. It allows to play, pause, rewind, and fast-forward videos, as well as move through them frame by frame. It is a powerful tool for visualizing and analyzing video data with annotations.

[Read this tutorial in developer portal.](https://developer.supervise.ly/app-development/widgets/media/video)

## Function signature

```python
Video(video_id=None, widget_id=None)
```

![default](https://user-images.githubusercontent.com/120389559/218671402-bd79b3a6-171c-439a-a012-ed58098e1c4e.gif)

## Parameters

| Parameters  | Type  |   Description    |
| :---------: | :---: | :--------------: |
| `video_id`  | `int` |    `Video` ID    |
| `widget_id` | `str` | ID of the widget |

### video_id

Determines `Video` ID, used in `Video` widget.

**type:** `int`

**default value:** `None`

### widget_id

ID of the widget.

**type:** `str`

**default value:** `None`

## Methods and attributes

|     Attributes and Methods      | Description                                                                                |
| :-----------------------------: | ------------------------------------------------------------------------------------------ |
|       `video_id(id: int)`       | Get `video_id` property.                                                                   |
|     `loading(value: bool)`      | Get or set `loading` property to `Video` widget.                                           |
|      `set_video(id: int)`       | Set video in `Video` widget by ID.                                                         |
| `set_current_frame(value: int)` | Set video player to given frame.                                                           |
|      `get_current_frame()`      | Get current video player frame.                                                            |
|      `get_frames_count()`       | Return number of frames in video.                                                          |
|         `@play_clicked`         | Decodator function is handled when video start to play.                                    |
|        `@pause_clicked`         | Decodator function is handled when video stopped.                                          |
|     `@frame_change_started`     | Decodator function is handled when a frame index is being changed.                         |
|    `@frame_change_finished`     | Decodator function is handled when there was no change in frame index for the last second. |

## Mini App Example

You can find this example in our Github repository:

[supervisely-ecosystem/ui-widgets-demos/media/004_video/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/media/004_video/src/main.py)

### Import libraries

```python
import os
from random import randint

import supervisely as sly
from dotenv import load_dotenv
from supervisely.app.widgets import Button, Card, Container, Video
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()
```

### Get Video ID from environment variables

```python
video_id = int(os.environ["modal.state.slyVideoId"])
```

### Get `VideoInfo` from server

```python
video_info = api.video.get_info_by_id(id=video_id)
```

### Initialize `Video` widget we will use in UI

```python
video = Video(video_id=video_id)
```

### Initialize buttons to control widget

```python
button_random_frame = Button(text="Random", icon="zmdi zmdi-tv")
button_loading = Button(text="Loading", icon="zmdi zmdi-refresh")

buttons_container = Container(
    widgets=[button_random_frame, button_loading], direction="horizontal",)
```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter and place widget that we've just created in the `Container` widget.

```python
card = Card(
    title="Video",
    content=Container(widgets=[video, buttons_container]),
)

layout = Container(widgets=[card])
```

### Create app using layout

Create an app object with layout parameter.

```python
app = sly.Application(layout=layout)
```

### Handle button clicks

Use the decorator as shown below to handle button click. We have 6 buttons: to play video, to stop video, to change video frame, to stop change video frame, to set random frame, to download video.

```python
@button_random_frame.click
def set_random_frame():
    video.set_current_frame(randint(0, video_info.frames_count - 1))


@button_loading.click
def video_loading():
    if video.loading:
        video.loading = False
    else:
        video.loading = True
    print(f"Loading: {video.loading}")
```

![default](https://user-images.githubusercontent.com/120389559/218671402-bd79b3a6-171c-439a-a012-ed58098e1c4e.gif)
