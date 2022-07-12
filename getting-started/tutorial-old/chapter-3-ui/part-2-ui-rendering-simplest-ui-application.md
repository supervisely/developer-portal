---
description: In this part, you will learn to create a basic UI application.
---

# Part 2 — UI Rendering \[Simplest UI Application]

### Table of contents

1. [HTML file](part-2-ui-rendering-simplest-ui-application.md#step-1-html-file)
2. [Python script](part-2-ui-rendering-simplest-ui-application.md#step-2-python-script)
3. [Updating debug.env && config.json](part-2-ui-rendering-simplest-ui-application.md#step-3-updating-debug.env-and-and-config.json)
4. [Results](part-2-ui-rendering-simplest-ui-application.md#step-4-results)

### Step 1 — HTML file

This is very similar to a modal window application.\
But now instead of a modal window — a **whole browser page**!

![memo](https://github.githubassets.com/images/icons/emoji/unicode/1f4dd.png) you can preview your HTML in [our Application Designer](https://app.supervise.ly/apps/designer)

Let's create some simple HTML:

**src/gui.html**

```
<div>
	<sly-field title="INFO: Test massage"
			   description="this is a common template for displaying information"
			   style="padding-top: 0; padding-bottom: 0">

		<sly-icon slot="icon" :options="{ color: '#13ce66', bgColor: '#e1f7eb', rounded: false }">
			<i class="zmdi zmdi-info"></i>
		</sly-icon>
	</sly-field>
	<sly-field title="WARNING: Test message"
			   description="this is a common template for displaying warnings">
		<sly-icon slot="icon" :options="{ color: '#fba607', bgColor: '#ffeee3', rounded: false }">
			<i class="zmdi zmdi-alert-triangle"></i>
		</sly-icon>
	</sly-field>
	<sly-field title="ERROR: Test message"
			   description="this is a common template for displaying errors">
		<sly-icon slot="icon" :options="{ color: '#de1212', bgColor: '#ffebeb', rounded: false }">
			<i class="zmdi zmdi-alert-circle"></i>
		</sly-icon>
	</sly-field>
</div>
```

### Step 2 — Python Script

To keep the application running all the time, we will use the `app.run` method.

**src/main.py**

```
import supervisely_lib as sly
from dotenv import load_dotenv  # pip install python-dotenv
                                # don't forget to add to requirements.txt!

load_dotenv("../debug.env")
load_dotenv("../secret_debug.env", override=True)

app = sly.AppService()


def main():
    sly.logger.info('Application starting...')
    app.run()


if __name__ == "__main__":
    sly.main_wrapper("main", main)
```

### Step 3 — Updating debug.env && config.json

1. Write `TASK_ID` While True Script from [Part 7](https://github.com/supervisely-ecosystem/how-to-create-app/tree/master/chapter-03-ui/part-07-while-true-script#step-3--results) to `debug.env`\
   We also need to specify the path along which our application can cache debugging data `DEBUG_APP_DIR` and `DEBUG_CACHE_DIR`

**debug.env**

```
PYTHONUNBUFFERED=1

TASK_ID=7715

context.teamId=238
context.workspaceId=333

DEBUG_APP_DIR="/home/user/app_debug_data/"
DEBUG_CACHE_DIR="/home/user/app_debug_cache/"

LOG_LEVEL="debug"
```

1. Write path to GUI file to `config.json`

**config.json**

```
{
  "name": "UI Rendering",
  "type": "app",
  "categories": [
    "quickstart"
  ],
  "description": "There will be some description",
  "docker_image": "supervisely/base-py-sdk:6.1.93",
  "main_script": "chapter-03-headless/part-02-ui-rendering/src/main.py",
  "gui_template": "chapter-03-headless/part-02-ui-rendering/src/gui.html",
  "task_location": "workspace_tasks",
  "icon": "https://img.icons8.com/color/100/000000/2.png",
  "icon_background": "#FFFFFF"
}
```

### Step 4 — Results

{% embed url="https://www.youtube.com/watch?v=bUaAbjJbRh4" %}
UI Rendering
{% endembed %}
