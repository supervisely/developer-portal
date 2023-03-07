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

```python
# change the Project ID to your value
PROJECT_ID=12208 # ⬅️ change it
```

And then load it in your python code using **`is_development`** method:

```python
import os
from dotenv import load_dotenv
import supervisely as sly

if sly.is_development():
  load_dotenv("local.env")
  load_dotenv(os.path.expanduser("~/supervisely.env"))
  api = sly.Api.from_env()
```
{% hint style="info" %}
**`is_development`** and **`is_production`** methods are used to check the environment variable that tells if the app is is still in development or production.
{% endhint %}

## Basic usage principle

Environment variables could be read both with a pure python, and with our SDK. *`TEAM_ID`* variable will be used down below for an example, this applies to every variable.

Read environment variable with pure python:

```python
import os

team_id = int(os.environ["TEAM_ID"])
```
Read environment variable with our SDK:

```python
import supervisely as sly

team_id = sly.env.team_id()
```
{% hint style="info" %}
Keep in mind that every example in our portal features reading environment variables with SDK methods.
{% endhint %}

It is important to note that every variable reading method in our SDK has an optional **`raise_if_not_found`** flag. 

It comes in handy when you, for example, wait for either **`PROJECT_ID`** or **`DATASET_ID`** variable, and you dont want an exeption to be raised every time either of those are not found.

It is also worth mentioning that it is possible to use legacy variable names, like the **`context.teamId`** for compatibility purposes.



## Environment variables

Here is a list of environment variables you could use in app development.

### **`SERVER_ADDRESS`**

address of your Supervisely instance, for Community Edition the value should be `https://app.supervise.ly`. For the Enterprise Edition the value is custom and depends on your configuration. Learn more [here](basics-of-authentication.md#server\_address-env). This variable is always passed to an App.&#x20;

### **`API_TOKEN`**

Your personal access token for authentication. Learn more [here](basics-of-authentication.md#api\_token-env). This variable is always passed to an App.&#x20;

### **`TASK_ID`**

When you run an app on Supervisely, the platform creates a task for this app to store all relevant information for this task (logs, persistent data, cache, temporary files, ...).  Task ID is needed to access this data (read or write). This variable is always passed to an App. &#x20;

![Aplication task on page "Workspace tasks"](https://user-images.githubusercontent.com/12828725/180637942-73b9b411-8251-48f6-a0bf-3b341346d55e.png)

### **`TEAM_ID`**

Basic usage principle for environment variables
The ID of the currently opened team. This variable is always passed to an App.&#x20;

![Current team](https://user-images.githubusercontent.com/12828725/180637662-83b572ee-c49f-41df-9114-241b92207e82.png)

Alternative env is duplicated for compatibility: **`context.teamId`**

Some Docker images do not support env names with dot `.` symbols. For such cases, the alternative variable **`CONTEXT_TEAMID`** is available starting from  Agent version `>=6.7.0`.

### **`WORKSPACE_ID`**

The ID of the currently opened workspace. This variable is always passed to an App.&#x20;

![Current workspace](https://user-images.githubusercontent.com/12828725/180637666-c3778b97-f616-4f93-9c8c-e66b82da0257.png)

Alternative env is duplicated for compatibility: **`context.workspaceId`**

Some Docker images do not support env names with dot `.` symbols. For such cases, the alternative variable **`CONTEXT_WORKSPACEID`** is available starting from  Agent version `>=6.7.0`.

### `USER_LOGIN`

Name of the user who run (spawned) current application session (`task_id`).

Alternative env is duplicated for compatibility: **`context.userLogin`**

Some Docker images do not support env names with dot `.` symbols. For such cases, the alternative variable **`CONTEXT_USERLOGIN`** is available starting from  Agent version `>=6.7.0`.

### **`PROJECT_ID`**

It is set when an app is spawned from the context menu of a project. For apps, that are running on agents with version <= `6.6.6`, please use **`modal.state.slyProjectId` ** or be sure that you are using the latest version of the Supervisely Agent (recommended).

Alternative env is duplicated for compatibility: **`context.projectId`**

Some Docker images do not support env names with dot `.` symbols. For such cases, the alternative variable **`CONTEXT_PROJECTID`** is available starting from  Agent version `>=6.7.0`.

### **`DATASET_ID`**

It is set when an app is spawned from the context menu of a dataset. For apps, that are running on agents with version <= `6.6.6`, please use **`modal.state.slyDatasetId` ** or be sure that you are using the latest version of the Supervisely Agent (recommended).

Alternative env is duplicated for compatibility: **`context.datasetId`**

Some Docker images do not support env names with dot `.` symbols. For such cases, the alternative variable **`CONTEXT_DATASETID`** is available starting from  Agent version `>=6.7.0`.

### **`FOLDER`**

It is set when an app is spawned from the context menu of a folder in Team Files. For apps, that are running on agents with version <= `6.6.9`, please use **`modal.state.slyFolder` ** or be sure that you are using the latest version of the Supervisely Agent (recommended).

Alternative env is duplicated for compatibility: **`context.slyFolder`**

Some Docker images do not support env names with dot `.` symbols. For such cases, the alternative variable **`CONTEXT_SLYFOLDER`** is available starting from  Agent version `>=6.7.0`.

### **`FILE`**

It is set when an app is spawned from the context menu of a file in Team Files. For apps, that are running on agents with version <= `6.6.9`, please use **`modal.state.slyFile` ** or be sure that you are using the latest version of the Supervisely Agent (recommended).

Alternative env is duplicated for compatibility: **`context.slyFile`**

Some Docker images do not support env names with dot `.` symbols. For such cases, the alternative variable **`CONTEXT_SLYFILE`** is available starting from  Agent version `>=6.7.0`.

### **`APP_NAME`**