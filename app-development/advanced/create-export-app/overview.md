# Overview

## Introduction

There are many different applications in the Supervisely ecosystem for exporting data to various popular formats. However, companies often need to implement custom data export in their specific format to meet their particular requirements. 

In the upcoming tutorial series, you will learn **2 ways to create a custom export app** for exporting data from the Supervisely platform.

### Option 1  Create an app without using a prepared export template

It is more recommended way to use SDK export template class (`sly.app.Export`) to create custom export app.
However, if your use case is not covered by our export template, you can create your own app without the template. 
We will also learn this way in the upcoming tutorial series.

### Option 2. Use our SDK export template (class `sly.app.Export`)

👍 This way is more convenient and can handle most of the routine tasks and cover most required use cases. All you need to do is create your own class (inherit from `sly.app.Export`), and override the `process` method. 
Method `process` should return the path to the folder/archive that needs to be downloaded.

#### More details about `sly.app.Export`

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
