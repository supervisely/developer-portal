# Timeline

## Introduction

**`Timeline`** widget is used to display video timeline. It can be used to get current frame and display information about the frame or it's annotation.

## Function signature

```python
timeline = Timeline(
    frames_count=31,
    intervals=[[0, 6], [6, 11], [12, 15], [16, 17], [18, 19], [20, 31]],
    colors=["#DB7093", "#93db70", "#7093db", "#70dbb8", "#db8370", "#db70c9"],
    height="30px",
)
```

![default](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/e79702bb-9a9a-46bb-87a1-00c0fc786064)

## Parameters

|     Parameters    |        Type       |              Description              |
| :---------------: | :---------------: | :-----------------------------------: |
|   `frames_count`  |       `int`       |         Timeline frames count         |
|    `intervals`    | `List[List[int]]` |            Frame intervals            |
|      `colors`     |    `List[str]`    | Intervals colors in `hex` color codes |
|      `height`     |       `str`       |        Timeline height in `px`        |
|  `pointer_color`  |       `str`       |          Color of the pointer         |
| `tooltip_content` |      `Widget`     |         Content of the tooltip        |
|    `widget_id`    |       `str`       |            ID of the widget           |

### frames\_count

Timeline (video) frames count.

**type:** `int`

```python
timeline = Timeline(
    frames_count=31,
    intervals=[[0, 6], [6, 11], [12, 15], [16, 17], [18, 19], [20, 31]],
    colors=["#DB7093", "#93db70", "#7093db", "#70dbb8", "#db8370", "#db70c9"],
)
```

![frames\_count](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/e79702bb-9a9a-46bb-87a1-00c0fc786064)

### intervals

Set frame intervals. Each interval is a list of two integers: `[start_frame, end_frame]`. Intervals and color lists must be the same length.

**type:** `List[List[int]]`

```python
timeline = Timeline(
    frames_count=31,
    intervals=[[0, 15], [16, 31]],
    colors=["#93db70", "#7093db"],
)
```

![](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/172757aa-3bd9-4516-941f-d528d34c1eee)

### colors

Set interval colors in `hex` color codes. Intervals and color lists must be the same length.

**type:** `List[str]`

```python
timeline = Timeline(
    frames_count=31,
    intervals=[[0, 31]],
    colors=["#4C0013"],
)
```

![](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/ca97c624-bcdd-4a49-9f3a-307298b0b51a)

### height

Set widget height in `px`.

**type:** `str`

**default value:** `30px`

```python
timeline = Timeline(
    frames_count=31,
    intervals=[[0, 6], [6, 11], [12, 15], [16, 17], [18, 19], [20, 31]],
    colors=["#DB7093", "#93db70", "#7093db", "#70dbb8", "#db8370", "#db70c9"],
    height="60px",
)
```

![](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/668e3834-5c6e-4591-a2a8-44500b6fea16)

### pointer\_color

Color of the pointer. Set hex color code or `None` to use the default color.

**type:** `str`

**default value:** `None`

```python
timeline = Timeline(
    frames_count=31,
    intervals=[[0, 6], [6, 11], [12, 15], [16, 17], [18, 19], [20, 31]],
    colors=["#DB7093", "#93db70", "#7093db", "#70dbb8", "#db8370", "#db70c9"],
    pointer_color="#FF0000", # red color
)
```

![](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/867fe213-5c4f-4601-988f-84e85bd16361)

### tooltip\_content

Content of Tooltip.

**type:** `Widget`

**default value:** `None`

```python
timeline = Timeline(
    frames_count=31,
    intervals=[[0, 6], [6, 11], [12, 15], [16, 17], [18, 19], [20, 31]],
    colors=["#DB7093", "#93db70", "#7093db", "#70dbb8", "#db8370", "#db70c9"],
    pointer_color="#FF0000", # red color
    tooltip_content=Text("Clickable Area", "text"),
)
```

![](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/e42d1bd5-5691-4289-b532-c7cc8c8d30e7)

### widget\_id

ID of the widget.

**type:** `str`

**default value:** `None`

## Mini App Example

You can find this example in our Github repository:

[supervisely-ecosystem/ui-widgets-demos/controls/008\_timeline/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/controls/008\_timeline/src/main.py)

### Import libraries

```python
import os
from random import randint
import supervisely as sly
from dotenv import load_dotenv
from supervisely.app.widgets import (
    Card,
    Container,
    Timeline,
    Text,
    InputNumber,
    VideoThumbnail,
)
```

### Write a function for dividing the total number of frames by ranges

```python
def divide_to_ranges(total, n):
    step = total // n
    ranges = []
    for i in range(n):
        start = i * step
        end = start + step - 1 if i < n - 1 else total - 1
        ranges.append([start, end])
    return ranges, len(ranges)
```

### Write function generating random hex colors for each range

```python
def generate_hex_colors(n):
    colors = []
    for _ in range(n):
        color = "#{:06x}".format(randint(0, 0xFFFFFF))
        colors.append(color)
    return colors
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()
```

### Initialize `Video ID` we will use

```python
video_id = 350000  # set your video id here
```

### Get video info from server and prepare VideoThumbnail widget

```python
video = api.video.get_info_by_id(video_id)
video_thumbnail = VideoThumbnail(video)
```

### Get video intervals and colors from video frame count

```python
video_intervals, intervals_n = divide_to_ranges(video.frames_count, 5)
intervals_colors = generate_hex_colors(intervals_n)
```

### Initialize Timeline widget

```python
timeline = Timeline(
    frames_count=video.frames_count,
    intervals=video_intervals,
    colors=intervals_colors,
    height="35px",
)
```

### Create InputNumber widget for setting current frame

```python
timeline_select_frame = InputNumber(value=1, min=0, max=video.frames_count, step=1)
timeline_text = Text("<span>Frame:</span>")
```

### Create Timeline container

```python
timeline_container = Container(
    widgets=[
        timeline,
        timeline_text,
        timeline_select_frame,
    ],
    direction="horizontal",
    fractions=[1, 0, 0],
    style="place-items: center;",
)
```

### Create app layout

```python
main_container = Container(widgets=[video_thumbnail, timeline_container])
layout = Card(title="Timeline", content=main_container)
```

### Create app using layout

Create an app object with layout parameter.

```python
app = sly.Application(layout=layout)
```

![](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/48913536/564094d3-b0b0-4988-a43e-760b26cd0cab)

### Add callbacks for changing the current frame

```python
@timeline.click
def show_current_value(pointer):
    timeline_select_frame.value = pointer


@timeline_select_frame.value_changed
def set_timeline_pointer(frame):
    timeline.set_pointer(frame)
```
