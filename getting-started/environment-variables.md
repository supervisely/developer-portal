---
description: >-
  Supervisely sets default environment variables for each Supervisely
  Application run. You can also use these environment variables for your
  development and debugging.
---

# Environment variables

Supervisely has the set default environment variables, Supervisely Application has access to them and we recommend using the same variables during development and debugging.&#x20;

{% hint style="info" %}
Environment variables are case-sensitive.
{% endhint %}

For development and debugging purposes you can manually **copy the ID to a clipboard** for every item you are working on: team, workspace, project, dataset, image, labeling job, team member, etc... Just open the context menu of the item and press **`Copy ID`** button.

![](https://user-images.githubusercontent.com/12828725/180638373-7f0d81c1-9e53-454e-82c6-50abd184bd00.png)

## **.ENV file**

For convenient development and debugging we recommend using `.env` files to avoid hardcoding test variables into your sources and [keep private your passwords, secrets, and other sensitive information](basics-of-authentication.md#use-.env-file-recommended). Also, it allows avoiding accidentally committing (exposing) these values to a git repository. &#x20;

Save `.env` file to `~/supervisely.env` [with your credentials](basics-of-authentication.md). The content of this file will look something like this:

```python
SERVER_ADDRESS="https://app.supervise.ly"
API_TOKEN="4r47N.....blablabla......xaTatb" 
```

Also with every tutorial, guide, and demo application you will find `local.env` file that contains other environment variables used for debugging. For example:

```
# change the Project ID to your value
modal.state.slyProjectId=12208 # ⬅️ change it
```

And then load it in your python code:

```python
import os
from dotenv import load_dotenv
import supervisely as sly

load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api.from_env()
```

## Default environment variables

### **`SERVER_ADDRESS`**

address of your Supervisely instance, for Community Edition the value should be `https://app.supervise.ly`. For the Enterprise Edition the value is custom and depends on your configuration. Learn more [here](basics-of-authentication.md#server\_address-env). This variable is always passed to an App.&#x20;

### **`API_TOKEN`**

Your personal access token for authentication. Learn more [here](basics-of-authentication.md#api\_token-env). This variable is always passed to an App.&#x20;

### **`TASK_ID`**

When you run an app on Supervisely, the platform creates a task for this app to store all relevant information for this task (logs, persistent data, cache, temporary files, ...).  Task ID is needed to access this data (read or write). This variable is always passed to an App. &#x20;

![Aplication task on page "Workspace tasks"](https://user-images.githubusercontent.com/12828725/180637942-73b9b411-8251-48f6-a0bf-3b341346d55e.png)

### **`context.teamId`**

The ID of the currently opened team. This variable is always passed to an App.&#x20;

![Current team](https://user-images.githubusercontent.com/12828725/180637662-83b572ee-c49f-41df-9114-241b92207e82.png)

### **`context.workspaceId`**

The ID of the currently opened workspace. This variable is always passed to an App.&#x20;

![Current workspace](https://user-images.githubusercontent.com/12828725/180637666-c3778b97-f616-4f93-9c8c-e66b82da0257.png)

### `context.userLogin`

Name of the user who run (spawned) current application session (`task_id`).

### **`modal.state.slyProjectId`**

It is set when an app is spawned from the context menu of a project.

### **`modal.state.slyDatasetId`**

It is set when an app is spawned from the context menu of a dataset.

### **`context.slyFolder`**

It is set when an app is spawned from the context menu of a folder in Team Files.
