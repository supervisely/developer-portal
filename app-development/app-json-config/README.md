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

```
// Some code
```

### `type`

Specifies type of the Ecosystem entity. Available types: app, project, collection

```
// Some code
```

### `version`

App engine version

```
// Some code
```

### `categories`

App category in Ecosystem

```
// Some code
```

### `description`

App description in Ecosystem

```
// Some code
```

### `docker_image`

Docker image used to run the app

```
// Some code
```

### `main_script`

Relative path to main script from project root

```
// Some code
```

### `gui_template`

Relative path to GUI template from project root

```
// Some code
```

### `modal_template`

Relative path to modal window template from project root

```
// Some code
```

### `modal_template_data`

Initializes default values for data variables in modal window

```
// Some code
```

### `modal_template_state`

Initializes default values for state variables in modal window

```
// Some code
```

### `headless`

Specifies if app uses frontend. Set true for the apps without GUI.

```
// Some code
```

### `isolate`

Runs app in the isolated container

```
// Some code
```

### `task_location`

Defines where the task will be displayed on app launch

```
// Some code
```

### `icon`

Link to the application icon

```
// Some code
```

### `icon_cover`

Stretches the icon to full width

```
// Some code
```

### `icon_background`

Icon background color in hex color code format

```
// Some code
```

### `poster`

Link to the application poster

```
// Some code
```

### `context_menu`

Context menu configuartion options

`context_category` creates a sub section in context menu

`target` determines where the application can be launched from

```
// Some code
```

### `instance_version`

Minimum instance version to launch app. Current instance version can be found at the bottom right corner at the Supervisely

```
// Some code
```

### `min_instance_version`

Same as [**`instance_version`**](./#instance\_version)

```
// Some code
```

### `min_agent_version`

Minimum agent version to launch app. Current agent version can be found at the **`Team Cluster`** page

```
// Some code
```

### `entrypoint`

Instruction for executing app scripts (v2.0.0 app engine only)

```
// Some code
```

### `port`

Use this key if you want to specify certain port (v2.0.0 app engine only)

```
// Some code
```

### `restart_policy`

Restarts app when certain condition occurs

```
// Some code
```

### `community_agent`

Determines if app can be launched from community agent

```
// Some code
```

### `session_tags`

Makes app session available in another apps, e.g [`serve YOLOV5`](https://ecosystem.supervise.ly/apps/yolov5/supervisely/serve) app is available in [`Apply NN to Images Project`](https://ecosystem.supervise.ly/apps/nn-image-labeling/project-dataset) app session

```
// Some code
```

### `integrated_into`

Integrates app into selected tool. E.g [smart tool app](https://ecosystem.supervise.ly/apps/ritm-interactive-segmentation/supervisely) can be used in image annotation tool.

```
// Some code
```

## Configuration examples

Configurations will not vary that much depending on type of the project, whether it's a small headless app or complicated app with UI and a lot of widgets.

We'll consider a few examples of app configs:

1. ****[**Headless**](example-1.-headless.md)****
2. ****[**Modal window**](example-2.-modal-window.md)****
3. ****[**GUI**](example-3.-gui.md)****
