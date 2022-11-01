---
description: config.json for app with GUI explained
---

# Example 3. GUI

Configuration for apps with graphical user interface are pretty much the same like any other Supervisely apps. In this section we'll look into [`Convert Class Shape`](https://ecosystem.supervise.ly/apps/convert-class-shape) app. This app converts labeled objects from one geometry to another and creates a new project from original with converted class shapes.

[supervisely-ecosystem/convert-class-shape/config.json](https://github.com/supervisely-ecosystem/convert-class-shape/blob/master/config.json)

```json
{
  "name": "Convert Class Shape",
  "type": "app",
  "categories": [
    "images",
    "annotation transformation",
    "data operations"
  ],
  "description": "Converts shapes of classes (e.g. polygon to bitmap) and all corresponding objects",
  "docker_image": "supervisely/base-py-sdk:6.35.0",
  "instance_version": "6.4.57",
  "main_script": "src/convert_class_shape.py",
  "gui_template": "src/gui.html",
  "modal_template": "src/modal.html",
  "task_location": "workspace_tasks",
  "isolate": true,
  "icon": "https://i.imgur.com/TxR0dfX.png",
  "icon_background": "#FFFFFF",
  "context_menu": {
    "target": [
      "images_project"
    ],
    "context_category": "Transform"
  },
  "poster": "https://user-images.githubusercontent.com/106374579/186599439-6b6848e6-48cb-4fdc-912e-1a4493c79f41.png"
}
```
