---
description: VideoPlayer widget tutorial
---

# Video Player

In this tutorial you will learn:

1. [about `Video Player` widget methods and attributes.](#overview)
2. [how to debug this tutorial](#how-to-debug-this-tutorial)
3. [how to use `Video Player` widget in your app](#video-player-demo-without-controls)
4. [how to use `Video Player` widget in your app with controls](#video-player-demo-with-controls)

It is a simple player familiar to everyone and supported in most versions of popular browsers:

<figure><img src="https://user-images.githubusercontent.com/79905215/212617508-522dac90-f5a0-4080-834d-bd43a23cf99c.png" alt=""></figure>

You can control video from you app:

<figure><img src="https://user-images.githubusercontent.com/79905215/212617178-2781f7a8-f380-480e-a07b-2f2d4c95724f.png" alt=""></figure>

## Overview

When developing applications for working with video, a simple player is often need to play the source or any results video.
If working with frames is not required, then using a more powerful `Video` widget will be unnecessary. For this case, the `Video Player` widget is suitable, which includes a built-in player supported in most versions of popular browsers.
In this tutorial, we will introduce you into how to use **`Video Player`** widget in Supervisely app.

**Function signature**

```python
# test data
video_url = "https://user-images.githubusercontent.com/79905215/210067166-e5531dae-d090-436e-bb3b-f053e2e831eb.mp4"
video_type = "video/mp4"
```
Initialize widget with video source:
```python
video1 = sly.app.widgets.VideoPlayer(
    url=video_url,
    mime_type=video_type
)
```

or you can initialize widget without source video and set it later.

```python
video2 = sly.app.widgets.VideoPlayer()
video2.set_video(
    url=video_url,
    mime_type=video_type
)
```

**Methods and attributes**

|         Attributes and Methods         | Description                                                                                                                                          |
| :---------------------: | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
|          `url`          |  Video source url attribute. `Default = None`                                                                                                  |
|       `mime_type`       |  Video source mime type attribute. `Default = None`                                                                                            |
|       `set_video(url: str = None, mime_type: str = None)`       | `Method` Set video source to widget.                                                                                                                 |
|         `play()`          |  Start playing from current timestamp video. [See more](https://developer.mozilla.org/en-US/docs/Web/API/HTMLMediaElement/play_event)        |
|         `pause()`         |  Stop playing videp [See more](https://developer.mozilla.org/en-US/docs/Web/API/HTMLMediaElement/pause_event).                               |
| `get_current_timestamp()` |  Get value indicating the current playback time in seconds. [See more](https://developer.mozilla.org/en-US/docs/Web/API/HTMLMediaElement/currentTime) |
| `set_current_timestamp(time: float)` |  Seek video to the given time, if the media is available. [See more](https://developer.mozilla.org/en-US/docs/Web/API/HTMLMediaElement/currentTime)   |

# Example

## How to debug this tutorial

<details><summary>Start debugging this tutorial in 5 steps.</summary>
<p>

**Step 1.** Prepare `~/supervisely.env` file with credentials. [Learn more here.](https://developer.supervise.ly/getting-started/basics-of-authentication#how-to-use-in-python)

**Step 2.** Clone [repository](https://github.com/supervisely-ecosystem/ui-widgets-demos) with source code and create [Virtual Environment](https://docs.python.org/3/library/venv.html).

```bash
git clone https://github.com/supervisely-ecosystem/ui-widgets-demos

cd ui-widgets-demos

./create_venv.sh
```

**Step 3.** Open repository directory in Visual Studio Code

```bash
code -r .
```

**Step 4.** Check contents of file `launch.json`:

```python
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Uvicorn",
      "type": "python",
      "request": "launch",
      "module": "uvicorn",
      "args": [
        "043_video_player.src.main:app", # path demo file
        "--host",
        "0.0.0.0",
        "--port",
        "8000",
        "--ws",
        "websockets",
        "--reload"
      ],
      "jinja": true,
      "justMyCode": true,
      "env": {
        "PYTHONPATH": "${workspaceFolder}:${PYTHONPATH}",
        "LOG_LEVEL": "DEBUG"
      }
    }
  ]
}

```

**Step 5.** Start debugging [`043_video_player/src/main.py`](https://github.com/supervisely-ecosystem/ui-widgets-demos/tree/master/043_video_player)

</p>
</details>

## Video Player (demo without controls)

<details><summary>Create simple app using widget without controls buttons.</summary>
<p>

### Import libraries

```python
import os

import supervisely as sly
from dotenv import load_dotenv
from supervisely.app.widgets import Card, Container, VideoPlayer
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()
```

### Initialize `Video Player`

```python
video_url = "https://user-images.githubusercontent.com/79905215/210067166-e5531dae-d090-436e-bb3b-f053e2e831eb.mp4"
video_type = "video/mp4"

video1 = VideoPlayer(url=video_url, mime_type=video_type)
```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter and place widget that we've just created in the `Container` widget.

```python
# create new cards
card1 = Card(
    title="Video player",
    content=video1,
)
layout = Container(widgets=[card])
```

### Create app using layout

Create an app object with layout parameter.

```python
app = sly.Application(layout=layout)
```

</p>
</details>

Press `F5` to start debugging and see Video Player widget.

<figure><img src="https://user-images.githubusercontent.com/79905215/212630902-1456c4e4-7370-4872-9079-e6de776f0bb7.gif" alt=""></figure>

## Video Player (demo with controls)

<details><summary>Create app with Video Player widget and control video to get or set video timestamp and to navigate.</summary>
<p>

### Import libraries

```python
import os

import supervisely as sly
from dotenv import load_dotenv
from supervisely.app.widgets import Button, Card, Container, Flexbox, InputNumber, VideoPlayer
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()
```

### Initialize `Video Player`

```python
video_url = "https://user-images.githubusercontent.com/79905215/210067166-e5531dae-d090-436e-bb3b-f053e2e831eb.mp4"
video_type = "video/mp4"

video2 = VideoPlayer(url=video_url, mime_type=video_type)
```

### Create controls

Create play/pause buttons

```python
play_btn = Button(text="Play", button_size="mini", icon="zmdi zmdi-play")
pause_btn = Button(text="Pause", button_size="mini", icon="zmdi zmdi-pause")
```

Create new from to control current timestamp

```python
get_time_btn = Button(text="Get timestamp", button_size="mini")
input_time = InputNumber(
    value=0,
    min=0,
    max=round(video_meta["duration"], 1),
    step=0.1,
)
set_time_btn = Button(text="Set timestamp", button_size="mini")
```

Place control form in the Flexbox widget

```python
controls_container = Flexbox(
    widgets=[play_btn, pause_btn, get_time_btn, input_time, set_time_btn],
    center_content=True,
)
```

### Define functions when buttons are pressed

```python

# start playing video
@play_btn.click
def play():
    video2.play()


# pause video
@pause_btn.click
def pause():
    video2.pause()


# get current timestamp
@get_time_btn.click
def get_current_timestamp():
    input_time.value = video2.get_current_timestamp()


# set current timestamp
@set_time_btn.click
def set_current_timestamp():
    time_to_set = input_time.get_value()
    video2.set_current_timestamp(time_to_set)

```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter and place widget that we've just created in the `Container` widget.

```python
# create new cards
card = Card(
    title="Controls - operations from python code",
    content=Container(widgets=[video2, controls_container]),
)
layout = Container(widgets=[card])
```

### Create app using layout

Create an app object with layout parameter.

```python
app = sly.Application(layout=layout)
```

</p>
</details>

Press `F5` to start debugging and see Video Player widget.

<figure><img src="https://user-images.githubusercontent.com/79905215/212659277-ded863b0-4a78-4f50-ada4-8c6eabb24782.gif" alt=""></figure>
