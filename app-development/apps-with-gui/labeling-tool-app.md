# Create a custom app integrated into the Image Labeling Tool

## Introduction

{% hint style="info" %}
Supervisely instance version >= 6.8.54<br>
Supervisely SDK version >= 6.72.200<br>

In the tutorial, Supervisely Python SDK version is not directly defined in the dev_requirements.txt and config.json files. But when developing your app, we recommend defining the SDK version in the dev_requirements.txt and the config.json file.
{% endhint %}

Developing a custom app for the Labeling Tool be useful in the following cases:
1. When you need to combine manual labeling and the algorithmic post-processing of the labels in real-time.
2. When you need to validate created labels for some specific rules in real-time.

In this tutorial, we'll learn how to develop an application that will process masks in real-time while working with the Image Labeling Tool. The processing will be triggered automatically after the mask is created with the Brush tool (after releasing the left mouse button). 
The demo app will also have settings for enabling / disabling the processing and adjusting the mask processing settings. We will focus on processing the mask in this tutorial, but it's possible to work with different geometries (points, polygons, rectangles, etc.) in the same way.

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
[**Step 10.**](labeling-tool-app.md#step-10.-optimizations-optional) Optimizations (optional).<br>


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
- Switch widget for enabling / disabling the processing
- Slider widget for adjusting the mask processing settings

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

In this tutorial, we'll be using advanced debug mode. It allows you to run your code locally from VSCode, while the application will be linked to the Labeling Tool and you'll be able to see the results of your actions in the Labeling Tool in real-time.<br>
Ensure that you have installed the required software from step 3 in [this](https://developer.supervisely.com/app-development/advanced/advanced-debugging#prepare-environment) tutorial. Otherwise, you won't be able to debug it in the Video Labeling Tool.

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

Now, we need to handle the events that will be triggered by the Labeling Tool. In this tutorial, we'll be using only one event, when the left mouse button is released after drawing a mask. 
So, catching the event will is pretty simple:

```python
@app.event(sly.Event.Brush.DrawLeftMouseReleased)
def brush_left_mouse_released(api: sly.Api, event: sly.Event.Brush.DrawLeftMouseReleased):
    sly.logger.info("Left mouse button released after drawing mask with brush")
```

That's it! Our function will receive the API object and the Event object and that's all we need to process the mask.
The API object contains credentials for the user, which is currently working in the Labeling Tool and triggered the event.
The Event object contains a lot of context information, such as:
```python
    team_id: int,
    workspace_id: int,
    project_id: int,
    dataset_id: int,
    image_id: int,
    label_id: int,
    object_class_id: int,
    object_class_title: str,
    user_id: int,
    is_fill: bool,
    is_erase: bool,
    mask: np.ndarray,
```

So it will be easy to get any required information from the Event object like this:
```python
    session_id = event.session_id
    dataset_id = event.dataset_id
    video_id = event.video_id
    project_id = event.project_id
```
and so on.

## Step 4. Preparing config.json file

Now, when we're ready to start testing our app, we first need to prepare the config.json file, so our app can be launched directly in the Labeling Tool. You can find a lot of information using the config.json file [here](https://developer.supervisely.com/app-development/basics/app-json-config/config.json). In this tutorial, we will pay attention to the specific key in the file:

```json
"integrated_into": ["image_annotation_tool"],
```
So, it will allow us to run the application directly in the Image Labeling Tool.


## Step 5. Using cache (optional)

While working in the Labeling Tool, we are waiting for the results of our actions in real-time. So, we need to process the mask as fast as possible.
That's why it's better to use a cache to avoid unnecessary API calls each time the function is triggered (e.g. for the same project meta).
In this tutorial, we will use a very simple caching just as a reference. In the real app, you can implement more advanced caching.
So, we'll need to cache [Supervisely Project Meta](https://docs.supervisely.com/customization-and-integration/00_ann_format_navi/02_project_classes_and_tags)(list of classes and tags in the project) objects.

```python
project_metas = {}

def get_project_meta(api: sly.Api, project_id: int) -> sly.ProjectMeta:
    if project_id not in project_metas:
        project_meta = sly.ProjectMeta.from_json(api.project.get_meta(project_id))
        project_metas[project_id] = project_meta
    else:
        project_meta = project_metas[project_id]
    return project_meta
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

    obj_class = project_meta.get_obj_class_by_id(event.object_class_id)

    new_mask = process(event.mask)

    label = sly.Label(geometry=sly.Bitmap(data=new_mask.astype(bool)), obj_class=obj_class)

    api.annotation.update_label(event.label_id, new_label)
```

Let's take a closer look at the process function:
1. We're retrieving the ProjectMeta object from the cache function.
2. We're retrieving the label's object class from the ProjectMeta object.
3. We're processing the mask in the process function.
4. We're creating a new label object with the processed mask and the same object class as the original label.
5. We're uploading the new label to the Supervisely platform.

## Step 7. Implementing the processing function

So, we already have the code for all the application's logic. But we still don't have the code for the processing function.
In this tutorial, we'll be using a simple mask transformation just for demonstration purposes. But you can implement any logic you want.

```python
def process(mask: np.ndarray) -> np.ndarray:
    dilation = cv2.dilate(mask.astype(np.uint8), None, iterations=dilation_strength.get_value())
    return dilation
```

Let's take a closer look at the process function:
1. We're reading the dilation strength from the Slider widget.
2. We're converting the mask to the uint8 type since it cames as a boolean 2D array from the Event object.
3. We're returning a new mask.

## Step 8. Running the app locally

Now, when everything is ready let's run the app and test it in the Labeling Tool. After launching the app from your VSCode you'll need to enter your root password to run the VPN connection. If everything works as it should, you'll see the following message in the terminal:
```bash
INFO:     Application startup complete.
```
Now follow the steps below to test the app while running it locally:
1. Open Image Labeling Tool in Supervisely.
2. Select the `Apps` tab.
3. Find the `Develop and Debug` application with a running marker and click `Open`.
4. The app's UI will be opened in the right sidebar.

![Opening Develop and Debug](https://github-production-user-asset-6210df.s3.amazonaws.com/118521851/284578468-d327f7b5-2bce-40d4-be58-b7917bb88117.png)

The application UI including the widgets we created earlier is displayed in the right sidebar. We can turn the processing on and off using the Switch widget and adjust the strength of the processing using the Slider widget. Both events will be visible in the terminal, so it will be easy and convenient to debug the app.

![Application UI](https://github-production-user-asset-6210df.s3.amazonaws.com/118521851/284578478-e162b62e-5a8c-4920-89b1-2f0381aea43c.png)

Now we're ready to test our app. Let's try to draw a mask with the Brush tool and release the left mouse button. After that mask will be processed and become a little bit bigger as you will see in the Labeling Tool.
Let's also check that our widgets are working properly:
- The Switch widget disables and enables the processing
- The Slider widget changes the strength of the processing

![Working with app running locally](https://github-production-user-asset-6210df.s3.amazonaws.com/118521851/284576492-0d66610d-804b-4f7c-8d85-abf2ee8cb731.gif)

## Step 9. Releasing the app and running it in Supervisely
When we test the application, we can release it and run it in Supervisely.
You can find a detailed guide on how to release the app [here](https://developer.supervisely.com/app-development/basics/add-private-app#step-2.-release), but in this tutorial, we'll just use the following command:

```bash
supervisely release
```

After it's done, you can find your app in the Apps section of the platform and run it in the Labeling Tool without running the code locally. The steps are the same as in the previous step, but this time we'll be launching the actual application. In this tutorial the app's name in config.json is `Labeling Tool template`, so we'll find it in the list and click `Run`.

## Step 10. Optimizations (optional)

It's important to mention that the app we developed in this tutorial is not optimized for production use. The main reason is the delays: when the application receives the mask from the event and starts processing it, the user can already draw a new mask. So, the app will be processing the old mask while the user is already working with a new one. And it can lead to unexpected results.
In this tutorial, we've implemented a very simple throttling mechanism to avoid this issue. Let's take a closer look at it:

```python
timestamp = None

@app.event(sly.Event.Brush.DrawLeftMouseReleased)
def brush_left_mouse_released(api: sly.Api, event: sly.Event.Brush.DrawLeftMouseReleased):
    sly.logger.info("Left mouse button released after drawing mask with brush")
    if not need_processing.is_on():
        # Checking if the processing is turned on in the UI.
        return

    if event.is_erase:
        # If the eraser was used, then we don't need to process the label in this tutorial.
        return

    t = datetime.now().timestamp()
    global timestamp
    timestamp = t

    # Here goes the processing code...

    if t == timestamp:
        api.annotation.update_label(event.label_id, label)
```

So, what's happening here:
1. When the event is triggered, we get the current timestamp at the beginning of the function.
2. We do some processing with our mask.
3. When we're ready to upload the new label to the platform, we're checking if the current timestamp is the same as the one we got at the beginning of the function. If it's not the same, then it means that the user has already drawn a new mask and we don't need to upload the label to the platform.

And this simple solution will do its job in most cases. But it's only implemented for demonstration purposes and it's not optimized for production use since it only handles the cases inside of the application code. That means when the application calls the API to upload the label, there's still a delay before a request reaches the platform and updates the mask in the Labeling Tool. So if you draw new masks fast enough, you can still get outdated results.
For those cases, we recommend implementing a more advanced mechanism for handling queues and delays. But it's out of the scope of this tutorial.

## Summary

In this tutorial, we learned how to develop an application for the Image Labeling Tool. We learned how to use UI widgets, how to handle the events and how to process the mask. We also learned how to use advanced debug mode and how to release the app and run it in Supervisely.
We hope that this tutorial was helpful for you and you'll be able to use it as a reference for your application.