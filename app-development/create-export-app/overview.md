# Overview

## Introduction

There are many different applications in the Supervisely ecosystem for exporting data to various popular formats. However, companies often need to implement custom data export in their specific format to meet their particular requirements. 

In the upcoming tutorial series, you will learn **2 ways to create a custom export app** for exporting data from the Supervisely platform.

### Option 1. Use our SDK export template class `sly.app.Export` (simple)

👍 This way is more convenient and can handle most of the routine tasks and cover most required use cases. All you need to do is create your own class (inherit from `sly.app.Export`), and override the `process` method. 
Method `process` should return the path to the result data (folder or archive).

[✅ Learn step-by-step tutorial here](./create-export-app-from-template.md).

[💻 Source code](https://github.com/supervisely-ecosystem/template-export-app/tree/master).

### Option 2. Create an app from scratch (advanced)

It is more recommended way to use SDK export template class (`sly.app.Export`) to create custom export app.
However, if your use case is not covered by our export template, you can create your own app without the template. 
We will also learn this way in the upcoming tutorial series.

[✅ Learn step-by-step tutorial here](./create-export-app-without-template.md).

[💻 Source code](https://github.com/supervisely-ecosystem/export-custom-format/tree/master).

### More details about sly.app.Export. [See source code](https://github.com/supervisely/supervisely/blob/master/supervisely/app/export_template.py)

`sly.app.Export` class will handle export routines for you:

- it will check that selected project or dataset exist and that you have access to work with it,
- it will upload your result data to Team Files and clean temporary folder, containing result archive in remote container or local hard drive if you are debugging your app. 
- Your application must return string, containing path to result archive or folder. If you return path to folder - this folder will be automatically archived.

`sly.app.Export` has a `Context` subclass which contains all required information that you need for exporting your data from Supervisely platform:

- `Team ID` - shows team id where exporting project or dataset is located
- `Workspace ID` - shows workspace id where exporting project or dataset is located
- `Project ID` - id of exporting project
- `Dataset ID` - id of exporting dataset (detected only if the export is performed from dataset context menu)

`context` variable is passed as an argument to `process` method of class `MyExport` and `context` object will be created automatically when you execute export script.

```python
class MyExport(sly.app.Export):
    def process(self, context: sly.app.Export.Context):
        print(context)
```

Output:

```text
Team ID: 435
Workspace ID: 680
Project ID: 15623
Dataset ID: 53491
```

## Set up an environment for development

> The following pages of the guide about creating a custom export app will refer to this section, which describes the preparation of the working environment.

We advise reading our [from script to supervisely app](../basics/from-script-to-supervisely-app.md) guide if you are unfamiliar with the [file structure](../basics/from-script-to-supervisely-app.md#repository-structure) of a Supervisely app repository because it addresses the majority of the potential questions.

**For both options, you need to prepare a development environment. Follow the steps below:**

**Step 1.** Prepare `~/supervisely.env` file with credentials. [Learn more here.](../../getting-started/basics-of-authentication.md#how-to-use-in-python)

**Step 2.** Fork and clone repository with source code and create [Virtual Environment](https://docs.python.org/3/library/venv.html).

```bash
git clone https://github.com/<my_team>/<my_repo>
cd <my_repo>
./create_venv.sh
```

**Step 3.** Open repository directory in Visual Studio Code.

```bash
code -r .
```

**Step 4.** Select created virtual environment as python interpreter.

**Step 5.** Open `local.env` and insert your values here. Learn more about environment variables in our [guide](../../getting-started/environment-variables.md)

```python
TASK_ID=10                    # ⬅️ requires to use advanced debugging
TEAM_ID=447                   # ⬅️ change it
WORKSPACE_ID=680              # ⬅️ change it
PROJECT_ID=20934              # ⬅️ ID of the project that you want to export
DATASET_ID=64985              # ⬅️ ID of the dataset that you want to export (leave empty if you want to export whole project)
SLY_APP_DATA_DIR="results/"   # ⬅️ path to directory for local debugging
```

Please note that the path you specify in the `SLY_APP_DATA_DIR` variable will be used for saving application results and temporary files (temporary files will be removed at the end).

For example:
- path on your local computer could be `/Users/maxim/my_data/`
- path in the current project folder on your local computer could be `results/`

> Don't forget to add this path to `.gitignore` to exclude it from the list of files tracked by Git.

![Change variables in local.env](https://user-images.githubusercontent.com/79905215/236182190-3438d72e-919f-4a8f-9544-a105e8441a5a.gif)

When running the app from Supervisely platform: Project and Dataset IDs will be automatically detected depending on how you run your application.
