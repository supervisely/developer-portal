# In browser app for the Labeling Tool

## Introduction

{% hint style="info" %}
Supervisely instance version >= 6.12.13\
Supervisely SDK version >= 6.73.272\

Supervisely SDK is used only for releasing the application. The application will be using the `sly_sdk` module as `supervisely` in the app runtime, so it must be present in the repository for application to work.
{% endhint %}

Developing a in-browser custom app for the Labeling Tool can be useful in the following cases:

1. When you need to combine manual labeling and the algorithmic post-processing of the labels in real-time.
2. When you need to validate created labels for some specific rules in real-time.

In this tutorial, we'll learn how to develop an application that will process masks in real-time while working with the Image Labeling Tool. The processing will be triggered automatically after the mask is created with the Brush tool (after releasing the left mouse button). The demo app will also have settings for enabling / disabling the processing and adjusting the mask processing settings.

We will go through the following steps:

[**Step 1.**](labeling-tool-web-app.md#step-1.-preparing-ui-widgets) Prepare UI widgets and the application's layout.\
[**Step 2.**](labeling-tool-web-app.md#step-2.-handling-the-events) Handle the events.\
[**Step 3.**](labeling-tool-web-app.md#step-3.-preparing-config.json-file) Prepare the config.json file.\
[**Step 4.**](labeling-tool-web-app.md#step-4.-processing-the-mask) Process the mask.\
[**Step 5.**](labeling-tool-web-app.md#step-5.-implementing-the-processing-function) Implement the processing function.\
[**Step 6.**](labeling-tool-web-app.md#step-6.-releasing-the-app-and-running-it-in-supervisely) Release the app and run it in Supervisely.\


{% hint style="info" %}
Everything you need to reproduce [this tutorial is on GitHub](https://github.com/supervisely-ecosystem/client_side_app_template): source code and additional app files.
Another example of the app that processes the masks in real-time can be found [here](https://github.com/supervisely-ecosystem/masks-intersections-web-py)
{% endhint %}

## Step 1. Preparing UI widgets

To be able to change the app's settings we need to add UI widgets to the app's layout. So, we'll need two widgets:

* Switch widget for enabling / disabling the processing
* Slider widget for adjusting the mask processing settings

{% hint style="info" %}
Only widgets that are present in `sly_sdk.app.widgets` can be used for such applications. But you should import them from `supervisely.app.widgets` in the app's code.
{% endhint %}

{% hint style="info" %}
The `widget_id` argument is required. It should be unique for each widget in the app.
{% endhint %}

```python
# src/gui.py
from supervisely.app.widgets import Container, Slider, Switch, Field
from supervisely.sly_logger import logger


# Creating widget to turn on/off the processing of labels.
need_processing = Switch(switched=True, widget_id="need_processing_widget")
processing_field = Field(
    title="Process masks",
    description="If turned on, then the mask will be processed after every change on left mouse release after drawing",
    content=need_processing,
    widget_id="processing_field_widget",
)

# Creating widget to set the strength of the processing.
dilation_strength = Slider(value=10, min=1, max=50, step=1, widget_id="dilation_strength_widget")
dilation_strength_field = Field(
    title="Dilation",
    description="Select the strength of the dilation operation",
    content=dilation_strength,
    widget_id="dilation_strength_field_widget",
)
```

Now, our widgets are ready and we can create the app's layout:

```python
layout = Container(widgets=[processing_field, dilation_strength_field], widget_id="layout_widget")
```

## Step 2. Handling the events

Now, we need to handle the events that will be triggered by the Labeling Tool. In this tutorial, we'll be using only one event, when the left mouse button is released after drawing a mask. So, catching the event will is pretty simple:

```python
# src/main.py
from datetime import datetime
import cv2
import numpy as np
from sly_sdk.webpy import WebPyApplication
from sly_sdk.sly_logger import logger

from src.gui import layout, dilation_strength, need_processing


app = WebPyApplication(layout)

@app.event(app.Event.figure_geometry_saved)
def on_figure_geometry_saved(data):
    logger.info("Left mouse button released after drawing mask with brush")
```

That's it! Our function will receive the dict object with event data and that's all we need to process the mask. In our case the data will contain a single key:
```python
{
    "figureId": 1
}
```

## Step 3. Preparing config.json file

Now, when we're ready to start testing our app, we first need to prepare the config.json file, so our app can be launched directly in the Labeling Tool. You can find a lot of information using the config.json file [here](https://developer.supervisely.com/app-development/basics/app-json-config/config.json). In this tutorial, we will pay attention to the specific keys in the file:

```json
"type": "client_side_app",
"integrated_into": ["image_annotation_tool"],
"gui_folder_path": "app",
"main_script": "src/main.py",
"src_dir": "src"
```

- `type`: `client_side_app` - it's a key that tells the platform that this app is an in-browser app.
- `integrated_into`: [`image_annotation_tool`] - it's a key that tells the platform that this app should be integrated into the Image Labeling Tool.
- `gui_folder_path`: `app` - it is a key that tells the platform where the app's layout is located. It can be any non-conflicting path. It will only be used when releasing the application.
- `main_script`: `src/main.py` - it's a key that tells the platform where the main script of the app is located. This fhile should contain the `app` variable of the `WebPyApplication` type.
- `src_dir`: `src` - it's a key that tells the platform where the source code of the app is located. All the modules that you import in the main script should be located in this directory.

So, it will allow us to run the application directly in the Image Labeling Tool.

## Step 4. Processing the mask

And now we're ready to implement the mask processing. But first, let's do some checks to make sure that we need to use the processing of the mask.

```python
# src/main.py
# Creating geometry version dictionary to avoid recursion.
last_geometry_version = {}

@app.event(app.Event.figure_geometry_saved)
def on_figure_geometry_saved(data):
    logger.info("Left mouse button released after drawing mask with brush")
    if not need_processing.is_on():
        # Checking if the processing is turned on in the UI.
        return
    # Get figure
    figure_id = data["figureId"]
    figure = app.get_figure_by_id(figure_id)

    # app.update_figure_geometry will trigger the same event, so we need to avoid infinite recursion.
    current_geom_version = figure.geometry_version
    last_geom_version = last_geometry_version.get(figure_id, None)
    last_geometry_version[figure_id] = current_geom_version + 2
    if last_geom_version is not None and last_geom_version >= current_geom_version:
        return
```

So, if the processing is turned off or the current geometry version of the figure less or equal to the last version we set, then we don't need to process the mask and we'll just exit the function. And now, let's finally process the mask!

```python
    # Get mask from the figure
    figure_geometry = figure.geometry
    mask = figure_geometry["data"]

    # Processing the mask. You need to implement your own logic in the process function.
    new_mask = process(mask)

    # Update the mask in the figure
    app.update_figure_geometry(figure, new_mask)
```

Let's take a closer look at the process function:

1. We're retrieving the geometry from the figure object.
2. We're processing the mask in the process function.
3. We're updating the figure geometry directly in the labeling tool.

## Step 5. Implementing the processing function

So, we already have the code for all the application's logic. But we still don't have the code for the processing function. In this tutorial, we'll be using a simple mask transformation just for demonstration purposes. But you can implement any logic you want.

```python
def process(mask: np.ndarray) -> np.ndarray:
    dilation = cv2.dilate(mask.astype(np.uint8), None, iterations=dilation_strength.get_value())
    return dilation
```

Let's take a closer look at the process function:

1. We're reading the dilation strength from the Slider widget.
2. We're converting the mask to the uint8 type since it cames as a boolean 2D array from the Event object.
3. We're returning a new mask.


## Step 6. Releasing the app and running it in Supervisely

Now we can release it and run it in Supervisely. You can find a detailed guide on how to release the app [here](https://developer.supervisely.com/app-development/basics/add-private-app#step-2.-release), but in this tutorial, we'll just use the following command:

```bash
supervisely release
```

After it's done, you can find your app in the Apps section of the platform and run it in the Labeling Tool. Follow the steps below to run the app in Supervisely:
1. Open Image Labeling Tool in Supervisely.
2. Select the Apps tab.
3. Find the application and click Run.
4. The app's UI will be opened in the right sidebar.


## Summary

In this tutorial, we learned how to develop an in-browser application for the Image Labeling Tool. We learned how to use UI widgets, how to handle the events and how to process the mask. We also learned how to release the app and run it in Supervisely. We hope that this tutorial was helpful for you and you'll be able to use it as a reference for your application.
