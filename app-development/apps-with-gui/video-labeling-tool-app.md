# Create a custom app integrated into the Video Labeling Tool

## Introduction

{% hint style="info" %}
Supervisely instance version >= 6.8.--<br>
Supervisely SDK version >= 6.72.---<br>

In the tutorial, Supervisely Python SDK version is not directly defined in the dev_requirements.txt and config.json files. But when developing your app, we recommend defining the SDK version in the dev_requirements.txt and the config.json file.
{% endhint %}

Developing a custom app for the Labeling Tool be useful in the following cases:
1. When you need to combine manual labeling and the algorithmic post-processing of the labels in real-time.
2. When you need to validate created labels for some specific rules in real-time.
3. When you need to disable / enable some actions for the labeler (e.g. Submitting the job, etc.) if the created labels do not meet some specific rules.

In this tutorial, we'll learn how to develop an application that will validate the created labels for specified rules by clicking on the `Validate` button. You can implement your logic for the validation of the labels, but here now we'll use the following rules:
1. First, we create a tag on the video for a range of frames.
2. Then, we assign a value to the tag with the name of an object class, where the object with the same class name must be present at least in one frame of the range.
3. When the application is started it will disable the buttons `Confirm` for the video and the `Submit job` for the labeling job.
4. When the `Validate` button is clicked, the application will check the labels for the conditions above and enable the buttons `Confirm` and `Submit job` if the conditions are met, otherwise, the buttons will be disabled.

We will go through the following steps:

[**Step 1.**](video-labeling-tool-app.md#step-1.-preparing-ui-widgets) Prepare UI widgets and the application's layout.<br>
[**Step 2.**](video-tool-app.md#step-2.-enabling-advanced-debug-mode) Enable advanced debug mode.<br>

{% hint style="info" %}
Everything you need to reproduce [this tutorial is on GitHub](https://github.com/supervisely-ecosystem/video-labeling-tool-template): source code and additional app files.
{% endhint %}

## Step 1. Preparing UI widgets

But first, we need to import required packages and modules:

```python
import os
from dotenv import load_dotenv
import supervisely as sly
```

To be able to change the app's settings we need to add UI widgets to the app's layout.
So, we'll need the following widgets:
- Button to start the validation of the labels.
- Table to display the validation results.
- Checkbox to enable / disable showing all results (including the correct ones).
- Text to show the overall validation result. (optional, just for better UI)
- Field widget to add title and description to the UI section. (optional, just for better UI)

```python
import supervisely.app.development as sly_app_development
from supervisely.app.widgets import (
    Container,
    Button,
    Field,
    Table,
    Text,
    Checkbox,
)


# Preparing a list of columns for the results table.
columns = [
    "Status",
    "Go to Frame",
    "Object Class",
    "Frame Range",
]

# Preparing the status icons, you can change them and use your own.
ok_status = "✅"
error_status = "❌"

# This is the main button that starts the validation.
validate_button = Button("Validate")
validate_field = Field(
    title="Validate current video",
    description="Press the button to check if the video was annotated correctly",
    content=validate_button,
)

# Widget for displaying a result of the check.
validate_text = Text()
validate_text.hide()

# This checkbox allows you to choose which results to show in the table.
# By default, only incorrect results are shown, but you can also show all results.
show_all_checkbox = Checkbox("Show all results")
show_all_field = Field(
    title="Which results to show",
    description="If checked, will be shown both correct and incorrect results",
    content=show_all_checkbox,
)

# This is the table where the results will be displayed.
results_table = Table(columns=columns, fixed_cols=1, sort_direction="desc")
results_table.hide()
```

As you can see, we're using the `hide()` method for the widgets that we don't want to show at the start of the application. Since there are no results yet, we don't need to show the table and the text with the overall result. Later, we'll use the `show()` method to show the widgets when we need them.

Now, our widgets are ready and we can create the app's layout:

```python
# Preparing the layout of the application and creating the application itself.
layout = Container(
    widgets=[
        check_field,
        show_all_field,
        check_text,
        results_table,
    ]
)
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

## Step 3. Handling the events

Now, we need to handle the events that will be triggered by the Labeling Tool. In this tutorial, we'll be using only one event, when the current video has changed. It also will be triggered when the application is launched for the first time. This event is needed for the app, so it will be able to get the current video and its labels.

```python
def video_changed(event_api: sly.Api, event: sly.Event.ManualSelected.VideoChanged):
    sly.logger.info("Current video was changed")
```

That's it! Our function will receive the API object and the Event object and that's all we need to process the mask.
The API object contains credentials for the user, which is currently working in the Labeling Tool and triggered the event.
The Event object contains a lot of context information, such as:

```python
    team_id: int,
    workspace_id: int,
    project_id: int,
    dataset_id: int,
    figure_id: int,
    video_id: int,
    frame: int,
    tool_class_id: int,
    session_id: str,
    tool: str,
    user_id: int,
    job_id: int,
```

So it will be easy to get any required information from the Event object like this:
```python
session_id = event.session_id
dataset_id = event.dataset_id
video_id = event.video_id
project_id = event.project_id
```
and so on.

As you can see, the `event` object contains all the required information, so validation in the tutorial is just one example of what you can do with your application. That means that you can also do some other stuff, for example, process the labels or copy the video to another project, etc. So, your app can do anything you want.

## Step 4. Preparing config.json file

Now, when we're ready to start testing our app, we first need to prepare the config.json file, so our app can be launched directly in the Video Labeling Tool. You can find a lot of information using the config.json file [here](https://developer.supervisely.com/app-development/basics/app-json-config/config.json). In this tutorial, we will pay attention to the specific key in the file:

```json
"integrated_into": ["video_annotation_tool"],
```
So, it will allow us to run the application directly in the Video Labeling Tool.

## Step 5. Using cache (optional)

To avoid unnecessary requests to the Supervisely API, we can use the cache for [Supervisely Project Meta](https://docs.supervisely.com/customization-and-integration/00_ann_format_navi/02_project_classes_and_tags)(list of classes and tags in the project) objects. In this tutorial, we will use a very simple caching just as a reference. In the real app, you can implement more advanced caching.

```python
# We will store the project meta in a dictionary so that we do not have to download it every time.
project_metas = {}

@app.event(sly.Event.ManualSelected.VideoChanged)
def video_changed(event_api: sly.Api, event: sly.Event.ManualSelected.VideoChanged):
    sly.logger.info("Current video was changed")

    # some code here...

    # Using a simple caching mechanism to avoid downloading the project meta every time.
    if event.project_id not in project_metas:
        project_meta = sly.ProjectMeta.from_json(api.project.get_meta(event.project_id))
        project_metas[event.project_id] = project_meta
```

So, if we already have the project meta in the cache, we don't need to download it again. But if we don't have it, we need to download it and add it to the cache.

## Step 6. Reading information from the event

Now, we need to get the required information from the `event` object and save it in global variables, so we can use it later, when the `Validate` button is clicked.

```python
# Initializing global variables.
api = None
session_id = None
dataset_id = None
video_id = None
```

And now the full code of the `video_changed` function:

```python
@app.event(sly.Event.ManualSelected.VideoChanged)
def video_changed(event_api: sly.Api, event: sly.Event.ManualSelected.VideoChanged):
    sly.logger.info("Current video was changed")
    global api, session_id, dataset_id, video_id, project_id
    # Saving the event parameters to global variables.
    api = event_api
    session_id = event.session_id
    dataset_id = event.dataset_id
    video_id = event.video_id
    project_id = event.project_id

    # Using a simple caching mechanism to avoid downloading the project meta every time.
    if event.project_id not in project_metas:
        project_meta = sly.ProjectMeta.from_json(api.project.get_meta(event.project_id))
        project_metas[event.project_id] = project_meta

    api.vid_ann_tool.disable_job_controls(session_id)
```

As you can see in the code above, we're also disabling the buttons `Confirm` for the video and the `Submit job` for the labeling job using the `disable_job_controls` method. We need to do it because we don't want the user to be able to submit the job or confirm the video until the validation is completed. So buttons will be disabled any time the video is changed or when the application is launched for the first time.


## Step 7. Preparing the validation function

Ok, we've got the required information from the event, now we need to prepare the function that will validate the labels. But this function should be called when the `Validate` button is clicked. And we'll use a convenient decorator for this.

```python
# We will store the results in a list of lists, where each list is a row in the table.
table_rows = []

@validate_button.click
def validate_video():
    # If the button is pressed, we clear the table and hide it,
    # because we will fill the table with new results.
    # We also hide the error message from the previous validation
    # and will show it again if there are incorrect results.
    table_rows.clear()
    results_table.hide()
    validate_text.hide()

    # Retrieving project meta from the cache.
    project_meta = project_metas[project_id]

    # Downloading the annotation in JSON format and converting it to VideoAnnotation object.
    ann_json = api.video.annotation.download(video_id)
    ann = sly.VideoAnnotation.from_json(ann_json, project_meta, key_id_map=sly.KeyIdMap())

    # Validating the annotation for the current video.
    validate_annotation(dataset_id, video_id, ann)

    # Filling the table with the results and showing it.
    if len(table_rows) > 0:
        results_table.read_json({"columns": columns, "data": table_rows})
        results_table.show()

    # Checking if there are incorrect results.
    if any([result[0] == error_status for result in table_rows]):
        # If there are incorrect results, we show the error message
        # and block the job buttons.
        api.vid_ann_tool.disable_job_controls(session_id)
        validate_text.text = "The video was not annotated correctly"
        validate_text.status = "error"
    else:
        # If there are no incorrect results, we show the success message
        # and unlock the job buttons.
        api.vid_ann_tool.enable_job_controls(session_id)
        validate_text.text = "The video is annotated correctly"
        validate_text.status = "success"

    # Showing the validation result.
    validate_text.show()
```

Let's take a closer look at the `validate_video` function:

1. We clear the table and hide it because we will fill the table with new results. We also hide the error message from the previous validation and will show it again if there are incorrect results.
2. Then we retrieve the project meta from the cache.
3. Downloading the annotation in JSON format and converting it to the `VideoAnnotation` object.
4. Then we validate the annotation for the current video using the `validate_annotation` function, which is described below.
5. Then we fill the table with the results and show it if there are any results.
6. Then we check if there are incorrect results. If there are, we show the error message and block the job buttons. Otherwise, we show the success message and unlock the job buttons.
7. And finally, we show the validation result.

So now it's time to implement the `validate_annotation` function.

## Step 8. Implementing the annotation validation function

Just to remind you how we will validate the annotation:
1. We check the annotation for the presence of the tags.
2. We read the tag value, which is supposed to be the name of the object class and the frame range.
3. We check if the object with the same class name is present in at least one frame of the range.

After that, we'll need to fill the table with the results. Let's now implement the `validate_annotation` function:

```python
ok_status = "✅"
error_status = "❌"

def validate_annotation(dataset_id: int, video_id: int, ann: sly.VideoAnnotation) -> None:
    # Iterating over all tags in the current annotation.
    for tag in ann.tags:
        # Checking if there's an object with the same ObjClass name
        # as value of the current tag in the tag's frame range.
        status = ok_status if is_in_range(tag, ann) else error_status

        # Preparing an entry for the results table.
        result = [
            status,
            sly.app.widgets.Table.create_button("Open"),
            tag.value,
            tag.frame_range,
        ]

        # If we're showing all results than we'll add ok results too,
        # otherwise we'll add only incorrect results.
        if show_all_checkbox.is_checked() or status == error_status:
            table_rows.append(result)


def is_in_range(tag: sly.VideoTag, ann: sly.VideoAnnotation) -> bool:
    # Retrieving the frame range for the current tag.
    range_start, range_end = tag.frame_range
    for figure in ann.figures:
        if figure.video_object.obj_class.name == tag.value:
            if figure.frame_index in range(range_start, range_end + 1):
                return True
    return False
```

Since we will check the annotation for each tag, we will iterate over all tags in the current annotation and it's better to use another function for this. That's why we've created the `is_in_range()` function. It will check if there's an object with the same ObjClass name as a value of the current tag in the tag's frame range. If there is, it will return `True`, otherwise `False`.

Then we'll use this function in the `validate_annotation()` function. It will iterate over all tags in the current annotation and check them using the `is_in_range()` function. But let's check what the `validate_annotation()` function does in more detail:

1. Iterating over all tags in the current annotation.
2. Calling the `is_in_range()` function to check if there's an object with the same ObjClass name as a value of the current tag in the tag's frame range.
3. Depending on the result of the `is_in_range()` function, we'll set the status of the result to values of the `ok_status` or `error_status` variables.
4. Then we'll prepare an entry for the results table, which is just a list of values for each column in the table.
5. We also check if we need to show all results or only incorrect ones. If we need to show all results, we'll add the result to the table, otherwise, we'll add only incorrect results.

And that's it. Now table rows are saved in the `table_rows` variable and we can fill the table with them. And it will be done in the `validate_video()` function.
And our application is ready, let's test it!

## Step 9. Running the app locally

Now, when everything is ready let's run the app and test it in the Labeling Tool. After launching the app from your VSCode you'll need to enter your root password to run the VPN connection. If everything works as it should, you'll see the following message in the terminal:
```bash
INFO:     Application startup complete.
```
Now follow the steps below to test the app while running it locally:
1. Open Video Labeling Tool in Supervisely.
2. Select the `Apps` tab.
3. Find the `Develop and Debug` application with a running marker and click `Open`.
4. The app's UI will be opened in the right sidebar.

![Opening Develop and Debug]()

The application UI including the widgets we created earlier is displayed in the right sidebar. The buttons `Confirm` for the video and the `Submit job` for the labeling job are disabled already. Just a reminder: you need to open the actual labeling job to see those buttons, if you will open a video through the `Datasets` tab, you won't see those buttons.

![Application UI]()

So, now we can start testing the app. But before ensure that you've already created some tags in range of frames for the current video with values that are the same as the object class names. Otherwise, the app will have nothing to validate and the result will always be `The video is annotated correctly`.

Let's also check that our Checkbox widget is working correctly:
1. Check the `Show all results` checkbox.
2. Click the `Validate` button.
3. You should see all the results in the table (both correct and incorrect).

![Working with app running locally]()

## Step 10. Releasing the app and running it in Supervisely
When we test the application, we can release it and run it in Supervisely.
You can find a detailed guide on how to release the app [here](https://developer.supervisely.com/app-development/basics/add-private-app#step-2.-release), but in this tutorial, we'll just use the following command:

```bash
supervisely release
```

After it's done, you can find your app in the Apps section of the platform and run it in the Labeling Tool without running the code locally. The steps are the same as in the previous step, but this time we'll be launching the actual application. In this tutorial the app's name in config.json is `Video Labeling Tool template`, so we'll find it in the list and click `Run`.

## Summary
In this tutorial, we've learned how to develop a custom app for the Video Labeling Tool. We've learned how to use the UI widgets, how to handle the events, how to read information from the event, how to validate the annotation and show the results in the table. We also learned how to use advanced debug mode and how to release the app and run it in Supervisely.
We hope that this tutorial was helpful for you and you'll be able to use it as a reference for your application.