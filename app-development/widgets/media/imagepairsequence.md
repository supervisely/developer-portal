# ImagePairSequence

## Introduction

The **`ImagePairSequence`** widget is a widget in Supervisely designed for displaying pairs of images and annotations. It is useful for comparing. For example, it can be used to compare ground truth and predictions in a grid format. It allows users to navigate through multiple pages of predictions and provides zooming functionality, making it convenient for visualizing annotated image results.


> All images will be saved in the `Team Files` in the `offline-sessions` directory:
> `/offline-sessions/{task_id}/app-template/sly/css/app/widgets/image_pair_sequence/{image_name}`

## Function signature

```python
ImagePairSequence(
    opacity=0.4,
    enable_zoom=False,
    sync_views=True,
    slider_title="pairs",
    widget_id=None,
)
```

<figure><img src="https://github.com/supervisely/developer-portal/assets/79905215/1f1adfcd-4fce-489c-bf4f-d8acbbc5a1be" alt=""><figcaption></figcaption></figure>

## Parameters

|   Parameters   |       Type        |                   Description                   |
| :------------: | :---------------: | :---------------------------------------------: |
|   `opacity`    | `Optional[float]` |                 Objects opacity                 |
| `enable_zoom`  | `Optional[bool]`  |       Enable zoom on `ImagePairSequence`       |
|  `sync_views`  | `Optional[bool]`  | Enable sync zoom on `ImagePairSequence` images |
| `slider_title` |  `Optional[str]`  |    Measurement units in the widget controls     |
|  `widget_id`   |  `Optional[str]`  |                Id of the widget                 |

### slider_title

Measurement units in the widget controls.

**type:** `Optional[str]`

**default value:** `epochs`

```python
image_pair_sequence = ImagePairSequence(slider_title="predictions")
```

<figure><img src="https://github.com/supervisely/developer-portal/assets/79905215/88ec3b86-4916-4904-96e2-44f38822fc0b" alt=""><figcaption></figcaption></figure>


### enable_zoom

Enable zoom on `ImagePairSequence`.

**type:** `Optional[bool]`

**default value:** `False`

```python
image_pair_sequence = ImagePairSequence(enable_zoom=True)
```

<figure><img src="https://github.com/supervisely/developer-portal/assets/79905215/6c7ea31c-c9cd-4931-a80c-d246f620923d" alt=""><figcaption></figcaption></figure>

### opacity

Objects opacity.

**type:** `Optional[float]`

**default value:** `0.4`

```python
image_pair_sequence = ImagePairSequence(opacity=0.8)
```

![opacity](https://github.com/supervisely/developer-portal/assets/79905215/27e1ad16-2073-40bf-abf7-eed6ee7bbc41)

### sync_views

Enable sync zoom on `ImagePairSequence` images.

**type:** `Optional[bool]`

**default value:** `True`

```python
image_pair_sequence = ImagePairSequence(sync_views=False)
```

<figure><img src="https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/79905215/0b3cb50b-c817-4609-9f72-1e573baf19c3" alt=""><figcaption></figcaption></figure>

### widget_id

ID of the widget.

**type:** `str`

**default value:** `None`

## Methods and attributes

By using the `ImagePairSequence` widget, you can set the images, annotations and titles to display on the left and right sides of the widget. You can also set a batch of images, annotations and titles to display on the left and right sides of the widget. The widget also provides methods for clearing the widget.

All the images will be saved in the `Team Files` in the `offline-sessions` directory:

`/offline-sessions/{task_id}/app-template/sly/css/app/widgets/image_pair_sequence/{image_name}`

|                                        Attributes and Methods                                         | Description                                                                                          |
| :---------------------------------------------------------------------------------------------------: | ---------------------------------------------------------------------------------------------------- |
|                         `append_left(url: str, ann: sly.Annotation, title: str)`                         | Sets the image, annotation and title to display on the left side of `ImagePairSequence`.            |
|                        `append_right(url: str, ann: sly.Annotation, title: str)`                         | Sets the image, annotation aand title to display on the right side of `ImagePairSequence`.          |
| `extend_left(urls: List[str], anns: Optional[List[sly.Annotation]], titles: Optional[List[str]])`  | Sets a batch of images, annotations and titles to display on the left side of `ImagePairSequence`.  |
| `extend_right(urls: List[str], anns: Optional[List[sly.Annotation]], titles: Optional[List[str]])` | Sets a batch of images, annotations and titles to display on the right side of `ImagePairSequence`. |
|       `append_pair(left: Tuple[str, sly.Annotation, str], right: Tuple[str, sly.Annotation, str])`       | Sets a pair of images, annotations and titles to display on the `ImagePairSequence`.                |
|                       `extend_pairs(left: List[Tuple], right: List[Tuple])`                        | Sets a batch of pairs of images, annotations and titles to display on the `ImagePairSequence`.      |
|                                             `clean_up()`                                              | Clears the `ImagePairSequence` widget.                                                              |
|                                              `disable()`                                              | Disables the `ImagePairSequence` widget controls.                                                   |
|                                              `enable()`                                               | Enables the `ImagePairSequence` widget controls.                                                    |

## Mini App Example

You can find this example in our Github repository:

[supervisely-ecosystem/ui-widgets-demos/media/007\_image\_pair\_sequence/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/media/007\_image\_pair\_sequence/src/main.py)


### Import libraries

```python
import os

import supervisely as sly
from dotenv import load_dotenv
from supervisely.app.widgets import Button, Card, Container, Flexbox, ImagePairSequence, Text
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

### Create static directory (optional)

```python
data_dir = sly.app.get_data_dir()
static_dir = os.path.join(data_dir, "static")
if not os.path.exists(static_dir):
    os.makedirs(static_dir)
```

### Create `Button` widgets for controlling `ImagePairSequence`

```python
left_btn = Button(text="add 1 prediction to left")
right_btn = Button(text="add 1 prediction to right")
left_three_btn = Button(text="add 3 predictions to left")
right_three_btn = Button(text="add 3 predictions to right")
pair_btn = Button(text="add 1 pair")
pairs_batch_btn = Button(text="add 3 pairs")
clean_btn = Button(text="clean up")

btn_container = Flexbox(
    [left_btn, right_btn, left_three_btn, right_three_btn, pair_btn, pairs_batch_btn, clean_btn]
)
```

### Prepare images and annotations for `ImagePairSequence` widget

```python
# get all image infos in dataset
used_names = set()
images_infos = api.image.get_list(dataset_id=dataset_id)
images_infos = sorted(images_infos, key=lambda image_info: image_info.name)
image_ids = [image_info.id for image_info in images_infos]
urls = [img.full_storage_url for img in images_infos]
paths = [
    os.path.join(static_dir, sly.generate_free_name(used_names, image_info.name, with_ext=True))
    for image_info in images_infos
]
static_paths = [os.path.join("static", os.path.basename(path)) for path in paths]

api.image.download_paths(dataset_id, image_ids, paths)

# get annotations for all images
anns_json = api.annotation.download_json_batch(dataset_id=dataset_id, image_ids=image_ids)
anns = [sly.Annotation.from_json(ann_json, project_meta) for ann_json in anns_json]
```

### Initialize `ImagePairSequence` widget we will use in UI

```python
image_pair_sequence = ImagePairSequence()
```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter and place widget that we've just created in the `Container` widget.

```python
text = Text()

card = Card(
    title="Image Pair Sequence",
    content=Container([image_pair_sequence]),
)
layout = Container(widgets=[btn_container, card, text])
```

### Create app using layout

Create an app object with layout parameter. We also pass the `static_dir` parameter to the app object so that the app can serve static files (optional).

```python
app = sly.Application(layout=layout, static_dir=static_dir)
```

### Add function to add predictions

```python
left_urls_generator = (url for url in static_paths)
right_urls_generator = (url for url in static_paths)
left_anns_generator = (ann for ann in anns)
right_anns_generator = (ann for ann in anns)


left_num = 0
right_num = 0


def get_next_prediction(side):
    global left_num, right_num
    url, ann, title = None, None, None
    url_gen = left_urls_generator if side == "left" else right_urls_generator
    ann_gen = left_anns_generator if side == "left" else right_anns_generator
    try:
        url = next(url_gen)
        ann = next(ann_gen)
        if side == "left":
            title = f"Predictions {left_num}"
            left_num += 1
        else:
            title = f"Predictions {right_num}"
            right_num += 1
    except StopIteration:
        sly.logger.info("No more predictions.")
        text.set(text="No more predictions.", status="info")
    finally:
        return url, ann, title


@pair_btn.click
def pair_btn_click_handler():
    left = get_next_prediction("left")
    right = get_next_prediction("right")
    if left[0] is not None and right[0] is not None:
        image_pair_sequence.append_pair(left=left, right=right)


@pairs_batch_btn.click
def pairs_batch_btn_click_handler():
    lefts = []
    rights = []

    for _ in range(3):
        left = get_next_prediction("left")
        right = get_next_prediction("right")

        if left[0] is not None and right[0] is not None:
            lefts.append(left)
            rights.append(right)

    if len(lefts) > 0 and len(rights) > 0:
        image_pair_sequence.set_pairs_batch(lefts, rights)


@left_btn.click
def left_btn_click_handler():
    url, ann, title = get_next_prediction("left")
    if url is not None:
        image_pair_sequence.append_left(url, ann, title)


@right_btn.click
def right_btn_click_handler():
    url, ann, title = get_next_prediction("right")
    if url is not None:
        image_pair_sequence.append_right(url, ann, title)


@left_three_btn.click
def left_three_btn_click_handler():
    data = []

    for _ in range(3):
        left = get_next_prediction("left")

        if left[0] is not None:
            data.append(left)

    if len(data) > 0:
        urls, anns, titles = zip(*data)
        image_pair_sequence.extend_left(urls=urls, anns=anns, titles=titles)


@right_three_btn.click
def right_three_btn_click_handler():
    data = []

    for _ in range(3):
        right = get_next_prediction("right")

        if right[0] is not None:
            data.append(right)

    if len(data) > 0:
        urls, anns, titles = zip(*data)
        image_pair_sequence.extend_right(urls=urls, anns=anns, titles=titles)


@clean_btn.click
def clean_btn_click_handler():
    image_pair_sequence.clean_up()
```

### Run the app

<figure><img src="https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/79905215/ebd8cfcb-3444-454c-b8eb-30a25f8d0f37" alt=""><figcaption></figcaption></figure>
