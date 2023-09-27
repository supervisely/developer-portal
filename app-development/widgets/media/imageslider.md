# ImageSlider

## Introduction

**`ImageSlider`** widget in Supervisely is a simple widget that displays images using Slider and is convenient to use when there is no need to add extra functions for displaying annotations or adjusting their settings, but only to display the images passed to it by a list of URLs or local paths.

## Function signature

```python
ImageSlider(
    previews=None,
    examples=None,
    combined_data=None,
    height=200,
    selectable=False,
    preview_idx=None,
    preview_url=None,
    widget_id=None,
)
```

![default](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/79905215/0e09b6a6-e8a4-48a9-b9cf-649ac119a7ac)

## Parameters

|   Parameters    |       Type        |                           Description                            |
| :-------------: | :---------------: | :--------------------------------------------------------------: |
|   `previews`    |    `List[str]`    |          List of previews URLs for the `Slider` widget           |
|   `examples`    | `List[List[str]]` |          List of image examples for the `Slider` widget          |
| `combined_data` |   `List[dict]`    | List of image previews and examples URLs for the `Slider` widget |
|    `height`     |       `int`       |                  Height of the `Slider` widget                   |
|  `selectable`   |      `bool`       |    Determines whether image selection is enabled or disabled     |
|  `preview_idx`  |       `int`       |        Index of the initially selected image in `Slider`         |
|  `preview_url`  |       `str`       |         URL of the initially selected image in `Slider`          |
|   `widget_id`   |       `str`       |                         ID of the widget                         |

### previews

List of previews URLs for the `Slider` widget.

**type:** `List[str]`

**default value:** `None`

```python
image_urls = [
    "https://www.w3schools.com/howto/img_nature.jpg",
    "https://www.quackit.com/pix/samples/18m.jpg",
    "https://i.imgur.com/35pUPD2.jpg",
    "https://i.imgur.com/fB2DBcM.jpeg",
    "https://i.imgur.com/OpSj3JE.jpg",
]

image_slider = ImageSlider(previews=image_urls)
```

![default](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/79905215/0e09b6a6-e8a4-48a9-b9cf-649ac119a7ac)

### examples

List of image examples for the `Slider` widget.

**type:** `List[List[str]]`

**default value:** `None`

```python
image_urls = [
    "https://www.w3schools.com/howto/img_nature.jpg",
    "https://www.quackit.com/pix/samples/18m.jpg",
    "https://i.imgur.com/35pUPD2.jpg",
    "https://i.imgur.com/fB2DBcM.jpeg",
    "https://i.imgur.com/OpSj3JE.jpg",
]

example_urls = [
    [
        "https://www.w3schools.com/howto/img_nature.jpg",
        "https://www.quackit.com/pix/samples/18m.jpg",
        "https://i.imgur.com/35pUPD2.jpg",
        "https://i.imgur.com/fB2DBcM.jpeg",
        "https://i.imgur.com/OpSj3JE.jpg",
    ],
    ["https://www.quackit.com/pix/samples/18m.jpg"],
    ["https://i.imgur.com/35pUPD2.jpg"],
    ["https://i.imgur.com/fB2DBcM.jpeg"],
    ["https://i.imgur.com/OpSj3JE.jpg"],
]

image_slider = ImageSlider(previews=image_urls, examples=example_urls)
```

![examples](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/79905215/26207fef-58c1-4609-9810-466ced589ace)

### combined_data

List of image previews and examples URLs for the `Slider` widget.

**type:** `List[dict]`

**default value:** `None`

```python
combined_data = [
    {
        "preview": "https://i.imgur.com/0XbKOJ3.jpeg",
        "moreExamples": ["https://i.imgur.com/0XbKOJ3.jpeg", "https://i.imgur.com/mIcObyL.jpeg"],
    },
    {
        "preview": "https://i.imgur.com/3rYHhEu.jpeg",
        "moreExamples": [
            "https://i.imgur.com/3rYHhEu.jpeg",
            "https://i.imgur.com/pSafUhg.jpeg",
            "https://i.imgur.com/yKc0xTb.jpeg",
        ],
    },
]

image_slider = ImageSlider(combined_data=combined_data)
```

![data](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/79905215/a7288c71-3054-4bb8-81c6-a7e349c02ba6)

### height

Height of the `Slider` widget.

**type:** `int`

**default value:** `200`

```python
image_slider = ImageSlider(data=image_urls, height=100)
```

![height](https://user-images.githubusercontent.com/120389559/224554054-e5ce7521-283c-4a17-aa97-3432e2bb03df.png)

### selectable

Determines whether image selection is enabled or disabled.

**type:** `bool`

**default value:** `False`

```python
image_slider = ImageSlider(data=image_urls, selectable=True)
```

![selectable](https://user-images.githubusercontent.com/120389559/224554387-5cb72289-a185-4a98-86b3-bfc8793b4c48.gif)

### preview_idx

Index of the initially selected image in `Slider`. Use only if `selectable` is `True`.

**type:** `int`

**default value:** `None`

```python
image_slider = ImageSlider(data=image_urls, selectable=True, preview_idx=4)
```

![preview_idx](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/79905215/6a0aac9d-5eff-4383-b8f6-652a7a47c960)

### preview_url

URL of the initially selected image in `Slider`. Use only if `selectable` is `True`.

**type:** `str`

**default value:** `None`

```python
image_slider = ImageSlider(
    data=image_urls, selectable=True, preview_url=image_urls[2]
)
```

![preview_url](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/79905215/cf71feae-e439-4df0-9d05-bc0c6b0d3982)

### widget_id

ID of the widget.

**type:** `str`

**default value:** `None`

|                                  Attributes and Methods                                  | Description                                                                        |
| :--------------------------------------------------------------------------------------: | ---------------------------------------------------------------------------------- |
|                                 `get_selected_preview()`                                 | Get URL of the selected image in the slider.                                       |
|                            `set_selected_preview(value: str)`                            | Sets URL of the image to be displayed as the preview image in the slider.          |
|                                   `get_selected_idx()`                                   | Retrieves the index of the currently selected image in the slider.                 |
|                              `set_selected_idx(value: int)`                              | Sets the index of the image to be displayed as the preview image in the slider.    |
|                                `get_selected_examples()`                                 | Retrieves the list of URLs of the currently selected image examples in the slider. |
|                                     `is_selectable`                                      | Returns a boolean indicating whether image selection is enabled or disabled.       |
|                                   `enable_selection()`                                   | Enables image selection.                                                           |
|                                  `disable_selection()`                                   | Disables image selection.                                                          |
|                                   `get_data_length()`                                    | Retrieves the length of the image URL list.                                        |
|                                       `get_data()`                                       | Retrieves the list of image URLs and examples.                                     |
|  `set_data(previews: List[str], examples: List[List[str]], combined_data: List[dict])`   | Sets the list of image URLs and examples.                                          |
| `append_data(previews: List[str], examples: List[List[str]], combined_data: List[dict])` | Extends the list of image URLs and examples.                                       |

## Mini App Example

You can find this example in our GitHub repository:

[supervisely-ecosystem/ui-widgets-demos/media/011\_image\_slider/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/media/011\_image\_slider/src/main.py)

```python
import os
import json

from dotenv import load_dotenv
import supervisely as sly
from supervisely.app.widgets import Button, Card, Container, Editor, Flexbox, Text, ImageSlider
```

### Init API client

Init API for communicating with Supervisely Instance. First, we load environment variables with credentials:

```python
if sly.is_development():
    load_dotenv("local.env")
    load_dotenv(os.path.expanduser("~/supervisely.env"))
api = sly.Api()
```

### Prepare data for `ImageSlider` widget

```python

image_urls = [
    "https://www.w3schools.com/howto/img_nature.jpg",
    "https://www.quackit.com/pix/samples/18m.jpg",
    "https://i.imgur.com/35pUPD2.jpg",
    "https://i.imgur.com/fB2DBcM.jpeg",
    "https://i.imgur.com/OpSj3JE.jpg",
]

example_urls = [
    [
        "https://www.w3schools.com/howto/img_nature.jpg",
        "https://www.quackit.com/pix/samples/18m.jpg",
        "https://i.imgur.com/35pUPD2.jpg",
        "https://i.imgur.com/fB2DBcM.jpeg",
        "https://i.imgur.com/OpSj3JE.jpg",
    ],
    ["https://www.quackit.com/pix/samples/18m.jpg"],
    ["https://i.imgur.com/35pUPD2.jpg"],
    ["https://i.imgur.com/fB2DBcM.jpeg"],
    ["https://i.imgur.com/OpSj3JE.jpg"],
]


combined_data = [
    {
        "preview": "https://i.imgur.com/0XbKOJ3.jpeg",
        "moreExamples": ["https://i.imgur.com/0XbKOJ3.jpeg", "https://i.imgur.com/mIcObyL.jpeg"],
    },
    {
        "preview": "https://i.imgur.com/3rYHhEu.jpeg",
        "moreExamples": [
            "https://i.imgur.com/3rYHhEu.jpeg",
            "https://i.imgur.com/pSafUhg.jpeg",
            "https://i.imgur.com/yKc0xTb.jpeg",
        ],
    },
]
```

### Initialize `ImageSlider` widget

```python
image_slider = ImageSlider(previews=image_urls, examples=example_urls)
```

### Initialize `Text`, `Editor` and `Button` widgets, we will use

```python
image_url = Text(status="info")
image_index = Text(status="info")
image_examples = Text(status="info")
data_length = Text(status="info")

text_container = Container(widgets=[image_url, image_index, image_examples, data_length])

button_url = Button(text="Get info")
button_set_url = Button(text="Select image by url")
button_set_index = Button(text="Select image by index")
button_extend_data = Button(text="Extend data")
button_set_data = Button(text="Set new data")
button_toggle_selectable = Button(text="Toggle selectable")
button_get_data = Button(text="Get data")

buttons_container = Flexbox(
    widgets=[
        button_url,
        button_set_url,
        button_set_index,
        button_extend_data,
        button_set_data,
        button_toggle_selectable,
        button_get_data,
    ]
)

all_data = Editor(height_lines=20)
all_data.hide()
```

### Create app layout

Prepare a layout for the app using `Card` widget with the `content` parameter and place widgets that we've just created in the `Container` widget. Place order in the `Container` is important, we want buttons to be displayed above the `Text` widget.

```python
card = Card(
    title="Image Slider",
    content=Container([image_slider, text_container, buttons_container, all_data]),
)

layout = Container(widgets=[card])
```

### Create an app using the layout

Create an app object with the layout parameter.

```python
app = sly.Application(layout=layout)
```

Our app layout is ready. It's time to handle button clicks.

### Add functions to control widgets from python code

```python
@button_url.click
def get_info():
    image_url.text = f"Image URL: {image_slider.get_selected_preview()}"
    image_index.text = f"Image index on slider: {image_slider.get_selected_idx()}"
    image_examples.text = f"Image examples: {image_slider.get_selected_examples()}"
    data_length.text = f"Data length: {image_slider.get_data_length()}"


@button_set_url.click
def set_url():
    image_slider.set_selected_preview("https://i.imgur.com/OpSj3JE.jpg")
    get_info()


@button_set_index.click
def set_index():
    image_slider.set_selected_idx(2)
    get_info()


@button_extend_data.click
def extend_data():
    image_slider.append_data(combined_data=combined_data[:1])
    get_info()


@button_set_data.click
def set_data():
    image_slider.set_data(combined_data=combined_data)
    get_info()


@button_toggle_selectable.click
def toggle_selectable():
    if image_slider.is_selectable:
        image_slider.disable_selection()
    else:
        image_slider.enable_selection()


@button_get_data.click
def get_data():
    data = image_slider.get_data()
    all_data.set_text(json.dumps(data, indent=4))
    all_data.show()
```

![layout](https://github.com/supervisely-ecosystem/ui-widgets-demos/assets/79905215/13c6cd60-7b88-4776-96b9-49e4e32887fe)
