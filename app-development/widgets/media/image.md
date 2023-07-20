# Image

## Introduction

**`Image`** widget in Supervisely is a simple widget that displays images using the HTML `<img>` tag, and is convenient to use when there is no need to add extra functions for displaying annotations or adjusting their settings, but only to display the image passed to it by url or local path.

## Function signature

```python
Image(url="", widget_id=None)
```

<figure><img src="https://user-images.githubusercontent.com/120389559/218754874-8b1da0a5-10cb-4384-93ac-8202206a1f56.png" alt=""><figcaption></figcaption></figure>

## Parameters

| Parameters  |            Type             |                 Description                  |
| :---------: | :-------------------------: | :------------------------------------------: |
|    `url`    |            `str`            | Image url or local path to image on the host |
|   `width`   | `Optional[Union[int, str]]` |       Image width in pixels or percent       |
|  `height`   | `Optional[Union[int, str]]` |      Image height in pixels or percent       |
| `widget_id` |            `str`            |               ID of the widget               |

### url

Image url or local path to image on the host. Determines image to be displayed on widget.

**type:** `str`

**default value:** `""`

```python
image_1 = Image(url="https://i.imgur.com/remote_image.png") # set image by url
image_2 = Image(url="/static/local_image.jpg") # set image by local path
image_3 = Image(url="/static/local_image.jpg", width=300, height=300) # set image with image sizes

image_4 = Image()
image_4.set("/static/local_image.jpg")
```

### width

Image width in pixels or percent.

**type:** `Optional[Union[int, str]]`

**default value:** `None`

```python
image_1 = Image(url="https://i.imgur.com/2Yj2QjQ.png", width=300)
image_2 = Image(url="https://i.imgur.com/2Yj2QjQ.png", width="300px")
image_3 = Image(url="https://i.imgur.com/2Yj2QjQ.png", width="20%")
```

### height

Image height in pixels or percent.

**type:** `Optional[Union[int, str]]`

**default value:** `None`

```python
image_1 = Image(url="https://i.imgur.com/2Yj2QjQ.png", height=300)
image_2 = Image(url="https://i.imgur.com/2Yj2QjQ.png", height="300px")
image_3 = Image(url="https://i.imgur.com/2Yj2QjQ.png", height="20%")
```

### widget_id

ID of the widget.

**type:** `str`

**default value:** `None`

## Methods and attributes

|                                Attributes and Methods                                 | Description                         |
| :-----------------------------------------------------------------------------------: | ----------------------------------- |
|                                    `set(url: str)`                                    | Set image by url in `Image` widget. |
|                                     `clean_up()`                                      | Clean `Image` widget from item.     |
| `set_image_size(height: Optional[Union[int, str]], width: Optional[Union[int, str]])` | Set image size in `Image` widget.   |
|                                       `show()`                                        | Show `Image` widget.                |
|                                       `hide()`                                        | Hide `Image` widget.                |

## Mini App Example

You can find this example in our Github repository:

[ui-widgets-demos/media/001_image/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/media/001_image/src/main.py)

### Import libraries

```
import os
from pathlib import Path

from dotenv import load_dotenv
import supervisely as sly
from supervisely.app.widgets import Card, Container, Image
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()
```

### Get images infos from server

```python
IMAGE_ID1 = 11073637
IMAGE_ID2 = 17526002
IMAGE_ID3 = 17526001

image_info_1 = api.image.get_info_by_id(IMAGE_ID1)
image_info_2 = api.image.get_info_by_id(IMAGE_ID2)
image_info_3 = api.image.get_info_by_id(IMAGE_ID3)
```

### Get images urls

```python
image_url_1 = image_info_1.preview_url
image_url_2 = image_info_2.preview_url
image_url_3 = image_info_3.preview_url
```

### Initialize `Image` widgets we will use in UI

```python
image_1 = Image(image_url_1)
image_2 = Image(image_url_2)
image_3 = Image(image_url_3, width="25%")
```

### Declare static files directory path and initialize `Image` using image from local directory

```python
local_image_url = "/static/my-cats.jpg"
local_image = Image(local_image_url)
local_image.set_image_size(width="25%")
```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter and place widget that we've just created in the `Container` widget.

```python
card = Card(
    title="Image Preview",
    content=Container([image_1, image_2, image_3, local_image], direction="horizontal"),
)

layout = Container(widgets=[card])
```

### Create app using layout

Define `static_dir` parameter for using files from local directory. Create an app object with `layout` parameter.

```python
static_dir = Path("media/001_image/images")

app = sly.Application(layout=layout, static_dir=static_dir)
```

<figure><img src="https://user-images.githubusercontent.com/120389559/218758403-0fca4434-207f-4d1a-b5f4-5e65268efcee.png" alt=""><figcaption></figcaption></figure>
