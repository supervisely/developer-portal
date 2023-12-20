# Cloning projects for development


## Introduction

When developing apps or scripts, you may start worrying about deleting or modifying data by mistake in active projects. But there's no need to worry, Supervisely has very simple ways to clone your projects and ensure that your data is safe.
This tutorial can be helpful in the following cases:
1. You're developing an app or script that modifies data in the project, while you want to keep the original data safe.
2. You need to work with the real data, not some dummy data, but it's important to keep the original data safe.

In this tutorial, we'll show you how to clone (or save a backup of) your projects in Supervisely.

And there are several ways you can achieve this:<br>
[**Option 1.**](backing-up-projects.md#option-1.-clone-the-project-in-ui) Clone the project in UI.<br>
[**Option 2.**](backing-up-projects.md#option-2.-export-the-project-in-ui) Export the project in UI.<br>
[**Option 3.**](backing-up-projects.md#option-3.-save-the-project-with-python-sdk) Save the project with Python SDK.<br>
[**Option 4.**](backing-up-projects.md#option-4.-clone-the-project-with-python-sdk) Clone the project with Python SDK.<br>

{% hint style="info" %}
Everything you need to reproduce [this tutorial is on GitHub](https://github.com/supervisely-ecosystem/backing-up-data): source code and additional app files.
{% endhint %}

## Using UI
UI ways are usually fast and easy, but only if we're talking about one or two projects and don't need some post-processing or automation. So, if it's your case, you can use the following ways to clone your projects, otherwise, check the Options 3 and 4.

### Option 1. Clone the project in UI
**Use cases:** when you need to clone a project once or twice.<br>
**Pros:** fast and easy.<br>
**Cons:** not suitable for automation or for cloning many projects.<br>

So it's the easiest and fastest way to clone a project in Supervisely, if you need to do it once or twice. You can do it just in two clicks:
1. Open the list of projects in your workspace.
2. Find the project you want to clone and click on the three dots on the right side of the project name.
3. Click `Clone` in the dropdown menu.

![Clone project in UI](https://github-production-user-asset-6210df.s3.amazonaws.com/118521851/290832511-5ee60457-fba6-48e0-9321-c6535450a724.png)

And that's it! You'll find `Clone` application on the `Workspace tasks` page. When the application is finished, you'll see the cloned project in the list of projects in your workspace. But, of course, if you need to clone many projects, it's not the best way to do it. In this case, you can use the Supervisely Python SDK.

### Option 2. Export the project in UI
**Use cases:** you want to have a backup of your project and store it somewhere outside Supervisely.<br>
**Pros:** fast, easy and can be used as a snapshot of your project.<br>
**Cons:** not suitable for automation or for cloning many projects, not convenient for further work with the project.<br>

It's not a way to clone a project, so it might be a little bit off-topic. But still, we want to mention it, because it's a very simple way to save a backup of your project. You can export your project in the Supervisely format and store it somewhere outside Supervisely. You can do it in the UI:
1. Open the list of projects in your workspace.
2. Find the project you want to clone and click on the three dots on the right side of the project name.
3. Find `Download` section in the dropdown menu and select `Export to Supervisely format`.

![Export project in UI](https://github-production-user-asset-6210df.s3.amazonaws.com/118521851/290832501-f40d6ffe-ff15-4e01-aa3a-53cb43110b03.png)

You'll find `Export to Supervisely format` application on the `Workspace tasks` page. When the application is finished, you'll be able to download the archive with your project. You can store it somewhere outside Supervisely and use it as a snapshot of your project. It's important to mention, that you can use `Import Images in Supervisely format` later to import the project back to Supervisely, but we believe that it's not the best way to save a data, when you need a project copy for further work.

## Using Supervisely Python SDK
Supervisely Python SDK is a powerful tool, which can solve almost any problem in Supervisely. It's a great way to clone your projects, if you need to do it more than once or twice. You can use it for automation and for cloning many projects.

{% hint style="info" %}
Supervisely instance version >= 6.8.54
Supervisely SDK version >= 6.72.200


In the tutorial, Supervisely Python SDK version is not directly defined in the requirements.txt. But when developing your app, we recommend defining the SDK version in the requirements.txt and the config.json file.
{% endhint %}

But first, we need to import the required packages and modules and read .env files with your credentials:

```python
import os
import supervisely as sly
from dotenv import load_dotenv

if sly.is_development():
    load_dotenv(os.path.expanduser("~/supervisely.env"))
    load_dotenv("local.env")
```

Now we'll retrieve your credentials from .env files and initialize the API access point:

```python
# Read environment variables and create an API client.
team_id = sly.env.team_id()
workspace_id = sly.env.workspace_id()
api: sly.Api = sly.Api.from_env()

# Read project ID from the environment variables.
project_id = sly.env.project_id()
```
So now, we're ready to use Python SDK for backing up or cloning your projects.


### Option 3. Save the project with Python SDK
**Use cases:** you need to save a backup of many projects or you want to automate the process.<br>
**Pros:** can be used for automation and for cloning many projects.<br>
**Cons:** can be inconvenient for further work with the project, since the copies are not stored in Supervisely.<br>

So it's not the way to clone a project too, but it's a fast and secure way to save a backup of your project. It can be used for automation and for backing up many projects. It's easy to do with Supervisely Python SDK:

```python

save_dir = "my_saved_data"
sly.Project.download(
    api, project_id, save_dir, save_image_info=True, save_image_meta=True
)

# Now the project is saved locally. We can archive it and then save in another place.
archive_path = f"{project_id}_archive.zip"
sly.fs.archive_directory(save_dir, archive_path)
```

In this example, we've downloaded the project and archived it. This backup can be stored somewhere outside Supervisely and used as a snapshot of your project. Later you can easily import it back to Supervisely with `Import Images in Supervisely format` application or with Supervisely Python SDK using `sly.Project.upload` method.

### Option 4. Clone the project with Python SDK
**Use cases:** you need to clone many projects or you want to automate the process.<br>
**Pros:** can be used for automation and for cloning many projects.<br>
**Cons:** none.<br>

Now we can talk about the most effective way, when you need to keep your data safe, while working with it, and it can be fully automated, when working with many projects. It's easy to do with Supervisely Python SDK:

```python
task_id = api.project.clone(project_id, workspace_id, "my_cloned_project")

# Wait until the task is finished.
api.task.wait(task_id, api.task.Status.FINISHED)

# Now the project is cloned and we can retrieve its ID.
task_info = api.task.get_info_by_id(task_id)
dst_project_id = task_info["meta"]["output"]["project"]["id"]
```

So in this example, we've cloned the project and retrieved the ID of the cloned project. Now you can work with the cloned project and be sure that the original data is safe. 

## Summary

In this tutorial, we've shown you how to clone or save a backup of your projects in Supervisely. We've shown you how to do it in the UI and with Supervisely Python SDK. We've also shown you how to save a backup of your project and how to clone it. We hope that this tutorial was helpful for you and now you can clone your projects and keep your data safe.