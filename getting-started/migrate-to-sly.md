---
description: Migrate to Supervisely from other platforms with your data and annotations.
---

# Migration to Supervisely

Supervisely offers a variety of tools to import data in different formats such as COCO, Pascal VOC, etc. But if you have an important data on other platforms we also can offer you a much more convenient way to migrate to Supervisely - fully automated migration tools, which can copy your data on the fly or simple import apps, which can import data in platform-specific formats.

Currently, automated migration tools are available for the following platforms:
- [CVAT](https://ecosystem.supervisely.com/apps/cvat-to-sly/migration_tool)
- [Roboflow](https://ecosystem.supervisely.com/apps/roboflow-to-sly)
- [Labelbox](https://ecosystem.supervisely.com/apps/labelbox-to-sly)
- [V7](https://ecosystem.supervisely.com/apps/v7-to-supervisely/migration_tool)

For the following platforms, we also can offer simple import apps in specific formats:
- [CVAT](https://ecosystem.supervisely.com/apps/cvat-to-sly/import_cvat)
- [V7](https://ecosystem.supervisely.com/apps/v7-to-supervisely/import_v7)

## Migration Tools

Migratiol Tools are fully automated UI apps, which can copy all your data from the specified platform to Supervisely using the platform API, where you only need to enter your credentials and select the data you want to copy.

How to use migration tools:

1. Open the Migration Tool app.
2. Enter your credentials to the platform and press the `Connect` button.
3. Select the projects you want to copy.
4. Click on the `Start copying` button.

That's all! The application will export your data from the platform, converts it and imports it into Supervisely. You'll see projects with correspodning URLs in the app's table (from the platform and in Supervisely) and the application will bne stopped automatically, when all projects will be processed.

## Import Apps

Import apps are simple headless apps, which can import data in platform-specific formats (e.g. CVAT format, V7 format, etc.). In this case you need to manually export your data in the specified format and upload it to Supervisely, but you don't need to obtain any API keys or credentials.

How to use import apps:

1. Export your data in the platform-specific format.
2. Open the import app.
3. Drag and drop the exported data (archive or folder) to the app.
4. Press the `Run` button.

It's simple too, isn't it? This way is good for one-time imports, but if you have a lot of data, we recommend you to use Migration Tools since they are much more convenient and can do all the work for you.