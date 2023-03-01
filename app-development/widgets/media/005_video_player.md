# Video Player

## Introduction

**`VideoPlayer`** widget in Supervised is a media player that utilizes the HTML `<video>` tag and provides additional functionalities to controll it through Python code such as seeking and controlling the current playback time in seconds.It uses the built-in browser player, which is supported by most devices.

In addition, Video Player has a feature for overlaying an image overlay on top of the video without modifying the video itself.

[Read this tutorial in developer portal.](https://developer.supervise.ly/app-development/widgets/media/videoplayer)

## Function signature

```python
VideoPlayer(url=None, mime_type="video/mp4", widget_id=None)
```

![videoplayer-default](https://user-images.githubusercontent.com/79905215/221818450-262abd3d-a883-4f1d-bddd-d0aba35d5ff3.gif)

## Parameters

| Parameters  | Type  |                 Description                  |
| :---------: | :---: | :------------------------------------------: |
|    `url`    | `str` | Video url or local path to video on the host |
| `mime_type` | `str` |                  Video type                  |
| `widget_id` | `str` |               ID of the widget               |

### url

Video url or local path to video on the host. Determines the video to be displayed on widget.

**type:** `str`

**default value:** `None`

### mime_type

Determines video type.

**type:** `str`

**default value:** `video/mp4`

ID of the widget.

**type:** `str`

**default value:** `None`

## Methods and attributes

|               Attributes and Methods                | Description                                                   |
| :-------------------------------------------------: | ------------------------------------------------------------- |
|                        `url`                        | Get `url` property.                                           |
|                     `mime_type`                     | Get `mime_type` property.                                     |
| `set_video(url: str, mime_type: str = "video/mp4")` | Set video in `VideoPlayer` widget by url.                     |
|                      `play()`                       | Start video to play.                                          |
|                      `pause()`                      | Stop video playback.                                          |
|              `get_current_timestamp()`              | Return current video playing step.                            |
|         `set_current_timestamp(value: int)`         | Set current video playing step.                               |
|               `draw_mask(path: str)`                | Draw a mask image on top of the video without changing video. |
|                    `hide_mask()`                    | Hide the drawn mask from the video.                           |

## Mini App Example

You can find this example in our Github repository:

[supervisely-ecosystem/ui-widgets-demos/media/005_video_player/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/media/005_video_player/src/main.py)

### Import libraries

```python
import os
from pathlib import Path

import supervisely as sly
from dotenv import load_dotenv
from supervisely.app.widgets import Button, Card, Container, Checkbox
from supervisely.app.widgets import Flexbox, InputNumber, VideoPlayer
from supervisely._utils import abs_url
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()
```

### Prepare videos url and type

```python
video_id = int(os.environ["modal.state.slyVideoId"])  # get video ID from environment
video_info = api.video.get_info_by_id(id=video_id)  # get VideoInfo from server
video_url = abs_url(video_info.path_original)
video_mime_type = video_info.file_meta["mime"]

local_video_url = "/static/video-cam2.mp4" # local video url
local_video_type = "video/mp4" # local video url
```

### Initialize `VideoPlayer` widgets we will use in UI

```python
video1 = VideoPlayer(url=local_video_url, mime_type=local_video_type)

video2 = VideoPlayer()
video2.set_video(url=video_url, mime_type=video_mime_type)
```

### Create a video control form

```python
play_btn = Button(text="Play", button_size="mini", icon="zmdi zmdi-play")
pause_btn = Button(text="Pause", button_size="mini", icon="zmdi zmdi-pause")

get_time_btn = Button(text="Get timestamp", button_size="mini")
input_time = InputNumber(
    value=0,
    min=0,
    max=round(video_info.file_meta["duration"], 1),
    step=0.1,
)
set_time_btn = Button(text="Set timestamp", button_size="mini")


input_sec = InputNumber(min=0.1, max=1000, step=0.1, size="small", controls=True)
forward_btn = Button(text="", button_size="mini", icon="zmdi zmdi-fast-forward", icon_gap=0)
rewind_btn = Button(text="", button_size="mini", icon="zmdi zmdi-fast-rewind", icon_gap=0)


controls_container = Flexbox(widgets=[play_btn, pause_btn])
change_timestamp_container = Flexbox(
    widgets=[rewind_btn, input_sec, forward_btn, get_time_btn, input_time, set_time_btn]
)

draw_mask_checkbox = Checkbox("draw mask on video")
mask_url = "https://user-images.githubusercontent.com/79905215/221801327-7be20a37-d4c0-4f7d-aa35-a072c4839985.png"

```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter and place widget that we've just created in the `Container` widget.

```python
card1 = Card(
    title="Video player",
    content=video1,
)
card2 = Card(
    title="Controls - operations from python code",
    content=Container(
        widgets=[video2, draw_mask_checkbox, controls_container, change_timestamp_container]
    ),
)

layout = Container(widgets=[card1, card2], direction="horizontal", fractions=[1, 1])
```

### Create app using layout

Define `static_dir` parameter for using files from local directory.
Create an app object with `layout` parameter.

```python
static_dir = Path("media/005_video_player/videos")

app = sly.Application(layout=layout, static_dir=static_dir)
```

### Handle button clicks

Use the decorator as shown below to handle button click. We have 4 buttons: to play video, to stop video, to get timestamp, to set timestamp.

```python
@play_btn.click
def play():
    video2.play()


@pause_btn.click
def pause():
    video2.pause()


@get_time_btn.click
def get_current_timestamp():
    input_time.value = video2.get_current_timestamp()


@set_time_btn.click
def set_current_timestamp():
    time_to_set = input_time.get_value()
    video2.set_current_timestamp(time_to_set)


@draw_mask_checkbox.value_changed
def draw_mask(value):
    global g
    if value is False:
        video2.hide_mask()
        return
    video2.draw_mask(mask_url)


@forward_btn.click
def fast_forward_video():
    step = input_sec.get_value()
    if video2.url is None:
        return
    currrent_time = video2.get_current_timestamp()
    video2.set_current_timestamp(currrent_time + step)


@rewind_btn.click
def fast_rewind_video():
    step = input_sec.get_value()
    if video2.url is None:
        return
    currrent_time = video2.get_current_timestamp()
    video2.set_current_timestamp(currrent_time - step)
```

![videoplayer-app](https://user-images.githubusercontent.com/79905215/221830670-81099679-2627-4b6e-9a98-9b409345c7dc.gif)
