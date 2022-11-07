---
description: config.json for v1 app with GUI explained
---

# Example 2. v1 app with GUI

## Introduction

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

<figure><img src="../../../.gitbook/assets/open-app.png" alt=""><figcaption></figcaption></figure>

## Properties

### **`name`**

Name of the app in Supervisely

```json
"name": "Convert Class Shape"
```

### **`type`**

Entity type in Supervisely Ecosystem

```json
"type": "app"
```

### **`categories`**

Сategories under which the app will be displayed in Ecosystem

```json
"categories": [
    "images",
    "annotation transformation",
    "data operations"
  ]
```

### **`description`**

Description of the app in Supervisely

```json
"description": "Converts shapes of classes (e.g. polygon to bitmap) and all corresponding objects"
```

### **`docker_image`**

Docker image used to launch the app with all pre-installed requirements

```json
"docker_image": "supervisely/base-py-sdk:6.4.57"
```

### **`instance_version`**

Minimum instance version to launch app. Same as **`min_instance_version`.** Current instance version can be found at the bottom right corner of the Supervisely page.

![](<../../../.gitbook/assets/instance\_ver (1).png>)

```json
"instance_version": "6.4.57"
```

### **`main_script`**

Relative path to the main script of the application from the root of the project

<pre class="language-json"><code class="lang-json"><strong>"main_script": "src/convert_class_shape.py"</strong></code></pre>

### **`gui_template`**

Relative path to the GUI template from the root of the project

```json
"gui_template": "src/gui.html"
```

### **`modal_template`**

Relative path to the modal window template from the root of the project. GUI apps can use modal window functionality too. In case of this app modal window only contain text information hence **`modal_template_state`** is not needed

```json
"modal_template": "src/modal.html"
```

### **`task_location`**

Specifies where to display task

<figure><img src="../../../.gitbook/assets/workspace_task_gui.png" alt=""><figcaption><p>workspace task</p></figcaption></figure>

```json
"task_location": "workspace_tasks"
```

### `isolate`

Runs app in isolated container&#x20;

```json
"isolate": true
```

### **`icon`**

Link to the app icon

```json
"icon": "https://i.imgur.com/TxR0dfX.png"
```

### **`icon_background`**

Background of app icon in hex color code

```json
"icon_background": "#FFFFFF"
```

### **`context_menu`**

App context menu configuration

<figure><img src="../../../.gitbook/assets/context_menu_gui.png" alt=""><figcaption></figcaption></figure>

```json
"context_menu": {
    "target": ["images_project"],
    "context_category": "Transform"
  }
```

### **`poster`**

Link to app poster

```json
"poster": "https://github.com/supervisely-ecosystem/import-images/releases/download/v1.0.0/poster.png"
```
