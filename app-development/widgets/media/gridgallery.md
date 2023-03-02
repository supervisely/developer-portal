# GridGallery

## Introduction

**`GridGallery`** is a widget in Supervisely used for displaying multiple images in a grid format. Allowing for annotations to be displayed on the images, the ability to hide annotations of specific classes, adjust their transparency and zoom level make it a convenient widget for visualizing annotated image results.

## Function signature

```python
GridGallery(
    columns_number,
    annotations_opacity=0.5,
    show_opacity_slider=True,
    enable_zoom=False,
    esize_on_zoom=False,
    sync_views=False,
    fill_rectangle=True,
    border_width=3,
    widget_id=None,
)
```

<figure><img src="https://user-images.githubusercontent.com/79905215/221854917-45d58570-a2f3-4c2b-a107-023a4346ed8f.png" alt=""><figcaption></figcaption></figure>

## Parameters

|       Parameters      |   Type  |                         Description                        |
| :-------------------: | :-----: | :--------------------------------------------------------: |
|    `columns_number`   |  `int`  |        Determines number of columns on `GridGallery`       |
| `annotations_opacity` | `float` |                       Figures opacity                      |
| `show_opacity_slider` |  `bool` | Determines the presence of opacity slider on `GridGallery` |
|     `enable_zoom`     |  `bool` |                Enable zoom on `GridGallery`                |
|    `resize_on_zoom`   |  `bool` |                  Resize card to fit figure                 |
|      `sync_views`     |  `bool` |                Sync pan & zoom between views               |
|    `fill_rectangle`   |  `bool` |                        Fill rectange                       |
|     `border_width`    |  `int`  |                        Border width                        |
|      `widget_id`      |  `str`  |                      Id of the widget                      |

### columns\_number

Determines number of columns on `GridGallery`.

**type:** `int`

**default value:** `1`

```python
grid_gallery = GridGallery(columns_number=4)
```

<figure><img src="https://user-images.githubusercontent.com/79905215/221855119-b605aaa9-82ea-4e6d-aef1-6c9f4d5f7041.png" alt=""><figcaption></figcaption></figure>

### annotations\_opacity

Figures opacity.

**type:** `float`

**default value:** `0.5`

```python
grid_gallery = GridGallery(columns_number=3, annotations_opacity=1)
```

<figure><img src="https://user-images.githubusercontent.com/79905215/221855404-56e3d197-200c-41ce-bab2-78bdfddeeaef.png" alt=""><figcaption></figcaption></figure>

### show\_opacity\_slider

Determines the presence of opacity slider on `GridGallery`.

**type:** `bool`

**default value:** `true`

```python
grid_gallery = GridGallery(columns_number=3, show_opacity_slider=False)
```

<figure><img src="https://user-images.githubusercontent.com/79905215/221855593-99aeda6d-68bf-4cfa-87cc-1f78a2e46c18.png" alt=""><figcaption></figcaption></figure>

### enable\_zoom

Enable zoom on `GridGallery`.

**type:** `bool`

**default value:** `false`

```python
grid_gallery = GridGallery(columns_number=3, enable_zoom=True)
```

<figure><img src="https://user-images.githubusercontent.com/79905215/221856143-7eb079d0-38ea-460d-9ca5-f38d1146bbb3.gif" alt=""><figcaption></figcaption></figure>

### resize\_on\_zoom

Resize card to fit figure.

**type:** `bool`

**default value:** `false`

### sync\_views

Sync pan & zoom between views.

**type:** `bool`

**default value:** `false`

```python
grid_gallery = GridGallery(columns_number=3, enable_zoom=True, sync_views=True)
```

<figure><img src="https://user-images.githubusercontent.com/79905215/221856489-2aa2d8ad-4930-47ae-a907-98f47764e176.gif" alt=""><figcaption></figcaption></figure>

### fill\_rectangle

Determines to fill rectange figure or not.

**type:** `bool`

**default value:** `true`

```python
grid_gallery = GridGallery(columns_number=3, fill_rectangle=False)
```

<figure><img src="https://user-images.githubusercontent.com/120389559/221578556-76d7bc99-c74d-4b0e-a64f-af7371b15e43.gif" alt=""><figcaption></figcaption></figure>

```python
grid_gallery = GridGallery(columns_number=3)
```

<figure><img src="https://user-images.githubusercontent.com/120389559/221579195-e66458a0-b79c-43f6-9310-819150833b1a.gif" alt=""><figcaption></figcaption></figure>

### border\_width

Determines border width to rectange figures.

**type:** `int`

**default value:** `3`

```python
grid_gallery = GridGallery(columns_number=3, border_width=12)
```

<figure><img src="https://user-images.githubusercontent.com/120389559/221580110-d98abb1b-bc91-4928-a419-b2841fcd7eea.png" alt=""><figcaption></figcaption></figure>

### widget\_id

ID of the widget.

**type:** `str`

**default value:** `None`

## Methods and attributes

|                                 Attributes and Methods                                | Description                         |
| :-----------------------------------------------------------------------------------: | ----------------------------------- |
|                                       `loading`                                       | Get or set `loading` property.      |
|                           `get_column_index(incoming_value)`                          | Return column index by given value. |
| `append(image_url, annotation, title, column_index, zoom_to, zoom_factor, title_url)` | Add item in `GridGallery`.          |
|                                      `clean_up()`                                     | Clean `GridGallery` from all items. |

## Mini App Example

You can find this example in our Github repository:

[ui-widgets-demos/media/003\_grid\_gallery/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/media/003\_grid\_gallery/src/main.py)

### Import libraries

```python
import os

import supervisely as sly
from dotenv import load_dotenv
from supervisely.app.widgets import Card, Container, GridGallery
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()
```

### Get project ID and dataset ID from environment variables

```python
project_id = sly.env.project_id()
dataset_id = sly.env.dataset_id()
project_meta = sly.ProjectMeta.from_json(data=api.project.get_meta(id=project_id))
```

### Initialize `GridGallery` widget we will use in UI

```python
grid_gallery = GridGallery(columns_number=3, enable_zoom=False)
```

### Set data to `GridGallery` widget

```python
images_infos = api.image.get_list(dataset_id=dataset_id)[: grid_gallery.columns_number]
anns_infos = api.annotation.get_list(dataset_id=dataset_id)[: grid_gallery.columns_number]

for idx, (image_info, ann_info) in enumerate(zip(images_infos, anns_infos)):
    image_name = image_info.name
    image_url = image_info.full_storage_url
    image_ann = sly.Annotation.from_json(data=ann_info.annotation, project_meta=project_meta)

    grid_gallery.append(
        title=image_name, image_url=image_url, annotation=image_ann, column_index=idx
    )
```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter and place widget that we've just created in the `Container` widget.

```python
card = Card(
    title="Grid Gallery",
    content=grid_gallery,
)

layout = Container(widgets=[card])
```

### Create app using layout

Create an app object with layout parameter.

```python
app = sly.Application(layout=layout)
```

<figure><img src="https://user-images.githubusercontent.com/79905215/221858652-bc342549-d16b-44bb-8452-b4c0e18ec8a7.gif" alt=""><figcaption></figcaption></figure>
