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

The Supervisely app config configures many things such as app name, category, icon, poster, docker image and so on. A complete list of available properties is described below. Don't worry, you don't need all of them.

|           Key          |                                                                                                                         Value example                                                                                                                        |                          Description                          |
| :--------------------: | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------: | :-----------------------------------------------------------: |
|          name          |                                                                                                                         "Hello World"                                                                                                                        |                            App name                           |
|          type          |                                                                                                                             "app"                                                                                                                            |                                                               |
|         version        |                                                                                                                            "2.0.0"                                                                                                                           |                       App engine version                      |
|     restart\_policy    |                                                                                                                          "on\_error"                                                                                                                         |           Restarts app when certain condition occurs          |
|       categories       |                                                                                                                       \["development"]                                                                                                                       |                          App category                         |
|       description      |                                                                                          "Demo app for tutorial purposes, use it as a template for your custom apps"                                                                                         |                        App description                        |
|      docker\_image     |                                                                                                                  "supervisely/base-py-sdk:6"                                                                                                                 |                Docker image used to run the app               |
|      main\_script      |                                                                                                                   "src/my\_main\_script.py"                                                                                                                  |                      Path to main script                      |
|     modal\_template    |                                                                                                                       "src/modal.html"                                                                                                                       |               Path to modal window html template              |
| modal\_template\_state |                                                                                                                         {"lenght": 7}                                                                                                                        | Initialize default values for state variables in modal window |
|  modal\_template\_data |                                                                                                           {"test1": "modalDataVal 1, "files": null}                                                                                                          |  Initialize default values for data variables in modal window |
|    integrated\_into    |                                             \["panel", "files", "standalone", "data\_commander", "image\_annotation\_tool", "video\_annotation\_tool", "dicom\_annotation\_tool", "pointcloud\_annotation\_tool"]                                            |                                                               |
|         hotkeys        |                                                                                                         {"hotkey": "ctrl+m", "command": "inference"}                                                                                                         |                                                               |
|      gui\_template     |                                                                                                                        "src/gui.html"                                                                                                                        |                   Path to gui html template                   |
|     task\_location     |                                                                                                                    "application\_sessions"                                                                                                                   |                                                               |
|         isolate        |                                                                                                                             true                                                                                                                             |               Runs app in the isolated container              |
|          icon          |                                                                                                   "https://img.icons8.com/fluent/96/000000/source-code.png"                                                                                                  |                            App icon                           |
|       icon\_cover      |                                                                                                                             false                                                                                                                            |                                                               |
|    icon\_background    |                                                                                                                           "#FFFFFF"                                                                                                                          |                   App icon background color                   |
|      session\_tags     |                                                                                                      \["sly\_video\_tracking", "sly\_smart\_annotation"]                                                                                                     |                                                               |
|      context\_menu     | `{"target": ["team", "workspace", "images_project", "videos_project", "point_cloud_project", "volumes_project", "images_dataset", "videos_dataset", "point_cloud_dataset", "volumes_dataset", "labeling_job", "files_folder", "files_file", "team_member"]}` |     Determines where the application can be launched from     |
|       entrypoint       |                                                                                                  "python -m uvicorn src.main:app --host 0.0.0.0 --port 8000"                                                                                                 |                                                               |
|          port          |                                                                                                                             8000                                                                                                                             |                                                               |
|    community\_agent    |                                                                                                                             false                                                                                                                            |     Determines if app can be launched from community agent    |

Configurations will not vary that much depending on type of the project, whether it's a small headless app or complicated app with UI and a lot of widgets.

We'll consider a few examples of app configs:

1. Headless
2. Modal window
3. Single page app

### Example 1. Headless

We will take [`Hello World`](https://ecosystem.supervise.ly/apps/hello-world-app) app as an example of a simple headless app that can be launched from Ecosystem, it uses minimum properties.

<figure><img src="../.gitbook/assets/ecosystem.supervise.ly_search_q=hello.png" alt=""><figcaption><p>Hello World! app</p></figcaption></figure>

[supervisely-ecosystem/hello-world-app/config.json](https://github.com/supervisely-ecosystem/hello-world-app/blob/master/config.json)

```
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