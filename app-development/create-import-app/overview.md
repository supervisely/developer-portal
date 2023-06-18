# Custom Import

This tutorial provides guidance on how to create a custom Supervisely import application.

We advise reading our [from script to supervisely app](../basics/from-script-to-supervisely-app.md) guide if you are unfamiliar with the [file structure](../basics/from-script-to-supervisely-app.md#repository-structure) of a Supervisely app repository because it addresses the majority of the potential questions.

We recommend to use import template for creating custom import applications using class `sly.app.Import` from Supervisely SDK. It is the easiest way to create import app with convenient GUI and designed to speed up and simplify the development of import apps.

* [Learn how to create import app from template](./create-import-app-from-template.md)

However, if your use case is not covered by our import template, you can create your own app **from scratch**  without the template using basic methods and [widgets](../widgets/README.md) from Supervisely SDK.

* [Learn how to create import app from scratch](./create-import-app-without-template.md)
* [Learn how to create import app from scratch with GUI](./create-import-app-without-template-gui.md)

## [`sly.app.Import`](https://github.com/supervisely/supervisely/blob/master/supervisely/app/import_template.py) advantages

`sly.app.Import` class will handle boilerplate/routine operations for you:

- ✅ Check that the selected team, workspace, project or dataset exists and that you have access to it
- ⬇️ Download your data from the Supervisely platform to a remote container or local hard drive if you are debugging your app
- 🪄 Automatically detect app context with all required information for creating import app
- ⬆️ Upload result data to new or existing Supervisely project or dataset
- 🧹 Remove source directory from Team Files after successful import

**Simply inherit from `sly.app.Import` and implement the `process` method.**

`sly.app.Import` has a `Context` subclass which contains all required information that you need for importing your data from the Supervisely platform. You can read about `context` in [this section](./create-import-app-from-template.md).

```python
class MyImport(sly.app.Import):
    def process(self, context: sly.app.Import.Context):
        # Implement your import logic here
        # Return result project id
        # That's it!
```

## Set up an environment for the development

**Follow the steps below:**

**Step 1.** Prepare `~/supervisely.env` file with credentials. [Learn more here.](../../getting-started/basics-of-authentication.md#how-to-use-in-python)

**Step 2.** Fork and clone the repository with source code and create [Virtual Environment](https://docs.python.org/3/library/venv.html).

**for [import with template](./create-import-app-from-template.md):**

```bash
git clone https://github.com/supervisely-ecosystem/template-import-app
cd template-import-app
./create_venv.sh
```

**for [import from scratch](./create-import-app-without-template.md)**

```bash
git clone https://github.com/supervisely-ecosystem/import-app-from-scratch
cd import-app-from-scratch
./create_venv.sh
```

**for [import from scratch GUI](./create-import-app-without-template-gui.md)**

```bash
git clone https://github.com/supervisely-ecosystem/import-app-from-scratch-gui
cd import-app-from-scratch-gui
./create_venv.sh
```

**Step 3.** Open the repository directory in Visual Studio Code.

```bash
code -r .
```

**Step 4.** Select created virtual environment as a Python interpreter.

## How to run it in Supervisely

Submitting an app to the Supervisely Ecosystem isn’t as simple as pushing code to the GitHub repository, but it’s not as complicated as you may think of it either.

Please follow this [link](../basics/add-private-app.md) for instructions on adding your app. We have produced a step-by-step guide on how to add your application to the Supervisely Ecosystem.
