---
description: This guide explains how to manage application sessions using API
---

# Start and stop app

## Introduction

[Supervisely Ecosystem](https://ecosystem.supervise.ly/) has hundreds of different apps from different categories: from import and data manipulation to neural network training. The huge value for end users is in applications.&#x20;

However, sometimes it is needed to run multiple apps in a specific order to solve custom task and it may be a bit inconvenient or boring for users. Supervisely SDK allows automate this procedure and build custom pipelines of apps where multiple apps can be run one by one or in parallel in a specific order.

This tutorial demonstrates basic functionality for managing app sessions from Python code: how to start and stop an app, how to wait for a task to end, handle exceptions and iterate over application sessions in your team.

{% hint style="info" %}
📗 Everything you need to reproduce [this tutorial is on GitHub](https://github.com/supervisely-ecosystem/automation-with-python-sdk-and-api/tree/master/examples/start-stop-app): source code and demo data.
{% endhint %}

## How to debug this tutorial

**Step 1.** Prepare `~/supervisely.env` file with credentials. [Learn more here.](https://developer.supervise.ly/getting-started/basics-of-authentication#how-to-use-in-python)

**Step 2.** Clone [repository](https://github.com/supervisely-ecosystem/automation-with-python-sdk-and-api) with source code and demo data and create [Virtual Environment](https://docs.python.org/3/library/venv.html).

```
git clone https://github.com/supervisely-ecosystem/automation-with-python-sdk-and-api
cd automation-with-python-sdk-and-api
./create_venv.sh
```

**Step 3.** Open repository directory in Visual Studio Code.

```
code -r .
```

**Step 4.** Go into the folder with the source code for the current tutorial and prepare values in`examples/start-stop-app/local.env` file before debug&#x20;

**Step 5.** Start debugging `examples/start-stop-app/main.py`

## Python code

### Import libraries

```python
import os
import json
from dotenv import load_dotenv
import supervisely as sly
```

### Init API client

Init API for communicating with Supervisely Instance. First, we load environment variables with credentials and additional demo values from `local.env` :

```python
load_dotenv("examples/start-stop-app/local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))
api = sly.Api()
```

### Init variables with demo data

We are going to run `export-to-pascal-voc` application. You can change it to any other app and slightly change the script.

```python
team_id = sly.env.team_id()
workspace_id = sly.env.workspace_id()
agent_id = sly.env.agent_id()
project_id = sly.env.project_id()
app_slug = "supervisely-ecosystem/export-to-pascal-voc"
```

### Load information about app from Ecosystem

To work with app sessions we need `module_id` - identifier of application in Ecosystem.&#x20;

```python
module_id = api.app.get_ecosystem_module_id(app_slug)
# module_id = 83  # or copy module ID of application in ecosystem
module_info = api.app.get_ecosystem_module_info(module_id)
print("Start app: ", module_info.name)
```

Output:

```
Start app:  Export to Pascal VOC
```

It can be obtained by using application slug (text identifier in Ecosystem) value in the following format`supervisely-ecosystem/<repository-name>`)

<figure><img src="https://user-images.githubusercontent.com/12828725/194827617-7cac57fb-0bee-4868-8da0-5c3530b3b988.png" alt=""><figcaption><p>Application slug - text identifier in Ecosystem</p></figcaption></figure>

Or alternatively, you can copy `module ID` from the application page in Ecosystem on **your Supervisely Instance.**

<figure><img src="https://user-images.githubusercontent.com/12828725/194828456-414ab519-d334-4beb-b80e-312c9322642f.png" alt=""><figcaption><p>Copy module ID from application page in Ecosystem</p></figcaption></figure>

### App input arguments

Every app has its own input arguments. They are defined by optional configurable parameters in the modal dialog window before the app starts.&#x20;

And also in most cases there is a valiable for the context menu of some Supervisely entity. For example, if the application starts from the context menu of images project - id of this project is the required input argument.&#x20;

Supervisely SDK alows to get the list of needed arguments for every app and also validates missing ones automatically.

```python
print("List of available app arguments for developers (like --help in terminal):")
module_info.arguments_help()
```

Output:

```
List of available app arguments for developers (like --help in terminal):
App 'Export to Pascal VOC' has additional options that can be configured manually 
in modal dialog window before running app. You can change them or keep defaults: 
{
    "pascalContourThickness": 3,
    "trainSplitCoef": 0.8
}
App has to be started from the context menus:
images_project : Context menu of images project. Target value is project id.
It is needed to call get_arguments method with defined target 
argument (pass one of the values above).
```

As you can see from the output, the application has two optional values with already defined default values (we can change them) and that app starts from the context menu of the images project. It means that one argument `images_project` is required.

Python SDK allows to generate needed arguments and validate them:

```python
params = module_info.get_arguments(images_project=project_id)
print("App input arguments with predefined default values:")
print(json.dumps(params, indent=4))

# Let's modify some optional input arguments for this app:
params["trainSplitCoef"] = 0.7
params["pascalContourThickness"] = 2
```

Here is the correspondence between input arguments and the modal window. It is worth mentioning that now all apps have paramaters in modal window. So that `arguments_help` method helps to identify existing parameters automatically.

<figure><img src="https://user-images.githubusercontent.com/12828725/194833164-5343145a-000c-4304-bed4-64bc1d800ed4.png" alt=""><figcaption><p>App input arguments and modal dialog window</p></figcaption></figure>

### Start application

```python
session = api.app.start(
    agent_id=agent_id,
    module_id=module_id,
    workspace_id=workspace_id,
    task_name="custom session name",
    params=params,
)
print("App is started, task_id = ", session.task_id)
print(session)
```

Output:

```
App is started, task_id =  21043
SessionInfo(task_id=21043, user_id=6, module_id=83, app_id=578, details={})
```

### Wait until app is finished

Option 1 - wait task finish or specific task status

```python
api.app.wait(session.task_id, target_status=api.task.Status.FINISHED)
```

Option 2 - wait until task end without any limitations (can work infinite in the worst case)

```python
api.task.wait(session.task_id)
```

Option 3 - set timeout and handle corresponding exceptions:

```python
try:
    # it is also possible to limit maximum waiting time
    # in the example below maximum waiting time will be 20*3=60 seconds
    api.app.wait(
        session.task_id,
        target_status=api.task.Status.FINISHED,
        attempts=20,
        attempt_delay_sec=3,
    )

except sly.WaitingTimeExceeded as e:
    print(e)
    # we don't want to wait more, let's stop our long-lived or "zombie" task
    api.app.stop(session.task_id)
except sly.TaskFinishedWithError as e:
    print(e)

print("Task status: ", api.app.get_status(session.task_id))
```

As you can see there is a method for getting the app session status `api.app.get_status`. You can use it to implement your custom logic for waitings or timeouts.

Method `api.app.stop` is used to send stop signal to the app.

### List existing app sessions in Team

Let's list all sessions of a specific app in our team with additional optional filtering by statuses `[finished]`.

```python
sessions = api.app.get_sessions(
    team_id=team_id, module_id=module_id, statuses=[api.task.Status.FINISHED]
)
for session_info in sessions:
    print(session_info)
```

Output:

```
SessionInfo(task_id=21043, user_id=6, module_id=83, app_id=578, details={"...": "... dict with lots of technical information"})
SessionInfo(task_id=21042, user_id=6, module_id=83, app_id=578, details={"...": "... dict with lots of technical information"})
SessionInfo(task_id=21041, user_id=6, module_id=83, app_id=578, details={"...": "... dict with lots of technical information"})
```

## Recap

In this tutorial we learned how to use python to

* get information from Ecosystem about certain application&#x20;
* how to work and prepare input arguments for application
* how to start application &#x20;
* how to wait session of application, monitor its status (task status) and handle exceptions
* how to stop application
