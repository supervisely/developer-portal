# Creating a custom app for the Image Labeling Tool

## Introduction

{% hint style="info" %}
Supervisely instance version >= 6.8.52
Supervisely SDK version >= 6.72.197

In the tutorial Python SDK version of Supervisely is not fixed in the dev_requirements.txt and in config.json, because we'll be using the latest version of the SDK. But when developing your app, we recommend fixing the SDK version in the dev_requirements.txt and the config.json file.
{% endhint %}

In this tutorial, we'll learn how to develop an application that will process masks in real-time while working with the Image Labeling Tool. The processing will be triggered automatically after the mask is created with the Brush tool (after releasing the left mouse button). 
The app will also have settings for enabling/disabling the processing and adjusting the processing strength.

We will go through the following steps:

[**Step 1.**](labeling-tool-app.md#step-1.-preparing-ui-widgets) Prepare UI widgets and the application's layout.<br>
[**Step 2.**](labeling-tool-app.md#step-2.-enabling-advanced-debug-mode) Enable advanced debug mode.<br>
[**Step 3.**](labeling-tool-app.md#step-3.-handling-the-events) Handle the events.<br>
[**Step 4.**](labeling-tool-app.md#step-4.-preparing-config.json-file) Prepare the config.json file.<br>
[**Step 5.**](labeling-tool-app.md#step-5.-using-cache-optional) Use cache (optional).<br>
[**Step 6.**](labeling-tool-app.md#step-6.-processing-the-mask) Process the mask.<br>
[**Step 7.**](labeling-tool-app.md#step-7.-implementing-the-processing-function) Implement the processing function.<br>
[**Step 8.**](labeling-tool-app.md#step-8.-running-the-app-locally) Run the app locally.<br>
[**Step 9.**](labeling-tool-app.md#step-9.-releasing-the-app-and-running-it-in-supervisely) Release the app and run it in Supervisely.<br>


{% hint style="info" %}
Everything you need to reproduce [this tutorial is on GitHub](https://github.com/supervisely-ecosystem/labeling-tool-template): source code and additional app files.
{% endhint %}

## Step 1. Preparing UI widgets

But first, we need to import required packages and modules:

```python
import cv2
import os
import numpy as np
import supervisely as sly
from dotenv import load_dotenv
```

To be able to change the app's settings we need to add UI widgets to the app's layout.
So, we'll need two widgets:
- Switch widget for enabling/disabling the processing
- Slider widget for adjusting the processing strength

```python
import supervisely.app.development as sly_app_development
from supervisely.app.widgets import Container, Switch, Field, Slider


# Creating widget to turn on/off the processing of labels.
need_processing = Switch(switched=True)
processing_field = Field(
    title="Process masks",
    description="If turned on, then the mask will be processed after every change on left mouse release after drawing",
    content=need_processing,
)

# Creating widget to set the strength of the processing.
dilation_strength = Slider(value=10, min=1, max=50, step=1)
dilation_strength_field = Field(
    title="Dilation",
    description="Select the strength of the dilation operation",
    content=dilation_strength,
)
```

Now, our widgets are ready and we can create the app's layout:

```python
layout = Container(widgets=[processing_field, dilation_strength_field])
app = sly.Application(layout=layout)
```

## Step 2. Enabling advanced debug mode

In this tutorial, we'll be using advanced debug mode. It allows you to run your code locally from VSCode, while the application will be linked to the LabelingTool and you'll be able to see the results of your actions in the LabelingTool in real-time.

```python
if sly.is_development():
    load_dotenv("local.env")
    team_id = sly.env.team_id()
    load_dotenv(os.path.expanduser("~/supervisely.env"))
    sly_app_development.supervisely_vpn_network(action="up")
    sly_app_development.create_debug_task(team_id, port="8000")
```

To use the advanced debug mode, you'll need to prepare two .env files. Learn more about them [here]().

## Step 3. Handling the events

Now, we need to handle the events that will be triggered by the LabelingTool. In this tutorial, we'll be using only one event, when the left mouse button is released after drawing a mask. 
So, catching the event will is pretty simple:

```python
@app.event(sly.Event.Brush.DrawLeftMouseReleased)
def brush_left_mouse_released(api: sly.Api, event: sly.Event.Brush.DrawLeftMouseReleased):
    sly.logger.info("Left mouse button released after drawing mask with brush")
```

That's it! Our function will receive the API object and the Event object and that's all we need to process the mask.

## Step 4. Preparing config.json file

Now, when we're ready to start testing our app, we first need to prepare the config.json file, so our app can be launched directly in the Labeling Tool. You can find a lot of information using the config.json file [here](https://developer.supervisely.com/app-development/basics/app-json-config/config.json). In this tutorial, we will pay attention to the specific key in the file:

```json
"integrated_into": ["image_annotation_tool"],
```
So, it will allow us to run the application directly in the Image Labeling Tool.


## Step 5. Using cache (optional)

While working in the LabelingTool, we are waiting for the results of our actions in real-time. So, we need to process the mask as fast as possible.
That's why it's better to use a cache to avoid unnecessary API calls each time the function is triggered.
In this tutorial, we will use a very simple caching just as a reference. In the real app, you can implement more advanced caching.
So, we'll need to cache images (as NumPy arrays) and [Supervisely Project Meta](https://docs.supervisely.com/customization-and-integration/00_ann_format_navi/02_project_classes_and_tags) objects.

```python
project_metas = {}
images_cache = {}

def get_project_meta(api: sly.Api, project_id: int) -> sly.ProjectMeta:
    if project_id not in project_metas:
        project_meta = sly.ProjectMeta.from_json(api.project.get_meta(project_id))
        project_metas[project_id] = project_meta
    else:
        project_meta = project_metas[project_id]
    return project_meta


def get_image_np(api: sly.Api, image_id: int) -> np.ndarray:
    if len(images_cache) > 100:
        # Clearing the cache if it's too big.
        images_cache.clear()
    if image_id not in images_cache:
        image_np = api.image.download_np(image_id)
        images_cache[image_id] = image_np
    else:
        image_np = images_cache[image_id]
    return image_np
```

So, if we already have the image or project meta in the cache, we'll use it. Otherwise, we'll get it from the API and save it to the cache.
And it will save us some time when processing the mask.

## Step 6. Processing the mask
And now we're ready to implement the mask processing. But first, let's do some checks to make sure that we need to use the processing of the mask.

```python
@app.event(sly.Event.Brush.DrawLeftMouseReleased)
def brush_left_mouse_released(api: sly.Api, event: sly.Event.Brush.DrawLeftMouseReleased):
    sly.logger.info("Left mouse button released after drawing mask with brush")
    if not need_processing.is_on():
        # Checking if the processing is turned on in the UI.
        return

    if event.is_erase:
        # If the eraser was used, then we don't need to process the label in this tutorial.
        return
```

So, if the processing is turned off or the eraser was used, then we don't need to process the mask and we'll just exit the function. And now, let's finally process the mask!

```python
    project_meta = get_project_meta(api, event.project_id)

    image_np = get_image_np(api, event.image_id)

    label = api.annotation.get_label_by_id(event.label_id, project_meta)

    new_label = process(label, image_np)
    api.annotation.update_label(event.label_id, new_label)
```

Let's take a closer look at the process function:
1. We're retrieving the ProjectMeta object from the cache function.
2. We're retrieving the image NumPy array from the cache function.
3. We're creating the Label object using its ID and ProjectMeta object.
4. We're processing the label in the process function and creating a new label.
5. We're uploading the new label to the Supervisely platform.

## Step 7. Implementing the processing function

So, we already have the code for all the application's logic. But we still don't have the code for the processing function.
In this tutorial, we'll be using a simple mask transformation just for demonstration purposes. But you can implement any logic you want, just remember that this function will receive the Label object and return the Label object. It may also receive additional parameters if you need them.

```python
def process(label: sly.Label, image_np: np.ndarray) -> sly.Label:
    image_size = image_np.shape[:2]
    mask = label.get_mask(image_size)
    dilation = cv2.dilate(mask.astype(np.uint8), None, iterations=dilation_strength.get_value())
    return label.clone(geometry=sly.Bitmap(data=dilation.astype(bool)))
```

Let's take a closer look at the process function:
1. We're retrieving image size from the NumPy array to create a full image size mask.
2. We're creating a full image-size mask from the label's object mask using the get_mask method.
3. We're reading the strength of the dilation operation from the UI and applying it to the mask using the cv2.dilate function.
4. We're returning a new label using the clone method with the processed data.

## Step 8. Running the app locally

Now, when everything is ready let's run the app and test it in the LabelingTool. After launching the app from your VSCode you'll need to enter your root password to run the VPN connection. If everything works as it should, you'll see the following message in the terminal:
```bash
INFO:     Application startup complete.
```
Now follow the steps below to test the app while running it locally:
1. Open Image Labeling Tool in Supervisely.
2. Select the `Apps` tab.
3. Find the `Develop and Debug` application with a running marker and click `Open`.<br>
![opening-develop-and-debug]()
4. The app's UI will be opened in the right sidebar.

Now we're ready to test our app. Let's try to draw a mask with the Brush tool and release the left mouse button. After that mask will be processed and become a little bit bigger as you will see in the LabelingTool.
Let's also check that our widgets are working properly:
- The Switch widget disables and enables the processing
- The Slider widget changes the strength of the processing

![working-with-app-running-locally]()

## Step 9. Releasing the app and running it in Supervisely
When we test the application, we can release it and run it in Supervisely.
You can find a detailed guide on how to release the app [here](https://developer.supervisely.com/app-development/basics/add-private-app#step-2.-release), but in this tutorial, we'll just use the following command:

```bash
supervisely release
```

After it's done, you can find your app in the Apps section of the platform and run it in the LabelingTool without running the code locally. Let's check it. The steps are the same as in the previous step, but this time we'll be launching the actual application. In this tutorial the app's name in config.json is `Labeling Tool template`, so we'll find it in the list and click `Run`.

## Summary

In this tutorial, we learned how to develop an application for the Image Labeling Tool. We learned how to use UI widgets, how to handle the events and how to process the mask. We also learned how to use advanced debug mode and how to release the app and run it in Supervisely.
We hope that this tutorial was helpful for you and you'll be able to use it as a reference for your application.