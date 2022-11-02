---
description: Configuration that connects Python application with Supervisely
---

# App JSON config

## Introduction

The app config (**config.json**) is used for configuring how a project loads in Supervisely. All data is stored in app configuration as key-values, where keys are the string type and values must be in valid JSON format for Supervisely to process it correctly. Otherwise, app might fail. Configuration file must be located at the root of your project, next to the `.env` file.

**Here is a bare-minimum** [**example**](https://github.com/supervisely-ecosystem/hello-world-app/blob/master/config.json)**:**

```json
{
  "main_script": "src/main.py",
  "headless": true,
  "name": "Hello World!",
  "description": "Demonstrates how to turn your python script into Supervisely App",
  "categories": ["development"],
  "icon": "https://user-images.githubusercontent.com/12828725/182186256-5ee663ad-25c7-4a62-9af1-fbfdca715b57.png",
  "poster": "https://user-images.githubusercontent.com/12828725/182181033-d0d1a690-8388-472e-8862-e0cacbd4f082.png"
}
```

## Properties

The Supervisely app config configures many things such as app name, category, icon, poster, docker image and so on. A complete list of available properties with example values is described below. Don't worry, you don't need all of them.

### `name`

Name of the app

```json
"name": "Hello World"
```

### `type`

Specifies type of the Ecosystem entity. Available types: `app`, `project`, `collection`

```json
"type": "app"
```

### `version`

App engine version

```json
"version": "2.0.0"
```

### `categories`

App category in Ecosystem

```json
"categories": ["development"]
```

### `description`

App description in Ecosystem

```json
"description": "Working demo, use it as a template for your custom apps"
```

### `docker_image`

Docker image used to run the app

```json
"docker_image": "supervisely/base-py-sdk:6.68.6"
```

### `main_script`

Relative path to main script from project root

```json
"main_script": "src/main.py"
```

### `gui_template`

Relative path to GUI template from project root

```json
"gui_template": "src/gui.html"
```

### `modal_template`

Relative path to modal window template from project root

```json
"modal_template": "src/modal.html"
```

### `modal_template_data`

Initializes default values for data variables in modal window

```json
  "modal_template_data": {
    "test_1": "modalDataVal 1",
    "files": null
  }
```

### `modal_template_state`

Initializes default values for state variables in modal window

```json
"modal_template_state": {
    "checkbox_1": false,
    "checkbox_2": true,
    "input": ""
  }
```

### `headless`

Specifies if app uses frontend. Set true for the apps without GUI.

```json
"headless": true
```

### `isolate`

Runs app in the isolated container

```json
"isolate": true
```

### hotkeys

Specifies hotkeys that can be used in app

```json
"hotkeys": [
      {"hotkey": "ctrl+m", "command": "inference"}
  ]
```

### `task_location`

Defines where the task will be displayed on app launch

```json
"task_location": "workspace_tasks"
```

### `icon`

Link to the application icon. If blank the first letter of the app name will be displayed as an icon

```json
"icon": "https://user-images.githubusercontent.com/12828725/182186256-5ee663ad-25c7-4a62-9af1-fbfdca715b57.png"
```

### `icon_cover`

Stretches the icon to full width

```json
"icon_cover": false
```

### `icon_background`

Icon background color in hex color code format

```json
"icon_background": "#FFFFFF"
```

### `poster`

Link to the application poster

```json
"poster": "https://user-images.githubusercontent.com/12828725/182181033-d0d1a690-8388-472e-8862-e0cacbd4f082.png"
```

### `context_menu`

Context menu configuartion options

`context_category` creates a sub section in context menu

`target` determines where the application can be launched from

```json
"context_menu": {
    "context_category": "my apps",
    "target": ["team", "workspace", "images_project", "videos_project", "point_cloud_project", "volumes_project", "images_dataset", "videos_dataset", "point_cloud_dataset", "volumes_dataset", "labeling_job", "files_folder", "files_file", "team_member"]
  }
```

### `instance_version`

Minimum instance version to launch app. Current instance version can be found at the bottom right corner at the Supervisely

```json
"instance_version": "6.5.51"
```

### `min_instance_version`

Same as [**`instance_version`**](./#instance\_version)

```json
"min_instance_version": "6.5.51"
```

### `min_agent_version`

Minimum agent version to launch app. Current agent version can be found at the **`Team Cluster`** page

```json
"min_agent_version": "6.7.4"
```

### `entrypoint`

Instruction for executing app scripts (v2.0.0 app engine only)

```json
"entrypoint": "python -m uvicorn src.main:app --host 0.0.0.0 --port 8000"
```

### `port`

Use this key if you want to specify certain port (v2.0.0 app engine only)

```json
"port": 8000
```

### `restart_policy`

Restarts app when certain condition occurs

```json
"restart_policy": "on_error"
```

### `community_agent`

Determines if app can be launched from community agent

```json
"community_agent": false
```

### `session_tags`

Makes app session available in another apps, e.g [`serve YOLOV5`](https://ecosystem.supervise.ly/apps/yolov5/supervisely/serve) app is available in [`Apply NN to Images Project`](https://ecosystem.supervise.ly/apps/nn-image-labeling/project-dataset) app session

```json
"session_tags": [
    "sly_video_tracking",
    "sly_smart_annotation"
  ]
```

### `integrated_into`

Integrates app into selected tool. E.g [smart tool app](https://ecosystem.supervise.ly/apps/ritm-interactive-segmentation/supervisely) can be used in image annotation tool.

```json
"integrated_into": ["panel", "files", "standalone", "data_commander", "image_annotation_tool", "video_annotation_tool", "dicom_annotation_tool", "pointcloud_annotation_tool"]
```

## Configuration examples

Configurations will not vary that much depending on type of the project, whether it's a small headless app or complicated app with UI and a lot of widgets.

We'll consider a few examples of app configs:

1. ****[**Headless**](example-1.-headless.md)****
2. ****[**Modal window**](example-2.-modal-window.md)****
3. ****[**GUI**](example-3.-gui.md)****
