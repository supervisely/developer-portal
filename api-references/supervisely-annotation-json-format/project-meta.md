# Project Meta
Project Meta is a crucial part of any project in Supervisely. It contains the essential information about the project - Classes and Tags. These are defined project-wide and can be used for labeling every dataset inside the current project. Project Meta is stored in the `meta.json` file in the root of the project folder. Each tag or class must be present in the project meta to be used in the project.

## Example

Whenever you're downloading a project from Supervisely, the `meta.json` file will be present in the root of the project folder. No matter how you downloaded the project: with an Export application, Python SDK or CLI - the structure will be the same.

```json
{
    "classes": [
        {
            "id": 8987050,
            "title": "kiwi",
            "description": "",
            "shape": "bitmap",
            "hotkey": "",
            "color": "#FF0000"
        },
        {
            "id": 8987051,
            "title": "lemon",
            "description": "",
            "shape": "bitmap",
            "hotkey": "",
            "color": "#51C6AA"
        }
    ],
    "tags": [
        {
            "id": 466875,
            "name": "fruit",
            "color": "#EF067A",
            "value_type": "none",
            "hotkey": "",
            "applicable_type": "all",
            "classes": []
        },
        {
            "id": 466876,
            "name": "vegetable",
            "color": "#CA1FE0",
            "value_type": "none",
            "hotkey": "",
            "applicable_type": "all",
            "classes": []
        }
    ],
    "projectType": "images",
    "projectSettings": {
        "multiView": {
            "enabled": false,
            "tagName": null,
            "tagId": null,
            "isSynced": false
        }
    }
}
```

Let's break down the structure of the `meta.json` file:
- `classes` - list of classes in the project. Each class has the following fields:
    - `id` - unique identifier of the class
    - `title` - the name of the class
    - `description` - description of the class
    - `shape` - shape of the class (bitmap, polygon, rectangle, etc.)
    - `hotkey` - hotkey for the class
    - `color` - color of the class
- `tags` - list of tags in the project. Each tag has the following fields:
    - `id` - unique identifier of the tag
    - `name` - the name of the tag
    - `color` - color of the tag
    - `value_type` - type of the tag value (none, text, number, one of, etc.)
    - `hotkey` - hotkey for the tag
    - `applicable_type` - the type of the tag application (image, object, all)
    - `classes` - list of classes that can be tagged with this tag
- `projectType` - the type of the project (images, videos, point clouds, etc.)
- `projectSettings` - additional project settings, such as multi-view settings
    - `multiView` - multi-view settings. Read more about multi-view in [this tutorial](!!!!ADD URL HERE!!!!).
        - `enabled` - whether multi-view is enabled
        - `tagName` - the name of the tag with which multi-view is associated
        - `tagId` - id of the tag for multi-view
        - `isSynced` - if set to true, the multi-view will also have synced panning, zooming and labeling

## Working with Project Meta
If you need to add new classes or tags to the project, they must be added not only in specific annotations but also in the project meta. So when working with annotations, you should always remember to update the project meta accordingly. You can do it manually by editing the `meta.json` file or with Supervisely Python SDK.<br>
Check out our [SDK Reference](!!!ADD URL HERE!!!) documentation for the full list of methods available for working with project meta. In this section, we'll cover the most common use cases.

#### Downloading Project Meta or creating a new one
The first step in working with a project meta is to download it from the platform. We'll need to install the Supervisely Python SDK first:

```bash
pip install supervisely
```

This is how we can download the project meta of an existing project:
```python
import supervisely as sly

api: sly.Api = sly.Api(server_address, token)
project_id = 123

# Important note: get_meta method returns a dictionary, not a ProjectMeta object!
meta_json = api.project.get_meta(project_id)

# We strongly recommend to always work with ProjectMeta objects, not with it's JSON representation!
# To convert JSON to ProjectMeta object, use ProjectMeta.from_json method
meta = sly.ProjectMeta.from_json(meta_json)

# So to do it in one line:
meta = sly.ProjectMeta.from_json(api.project.get_meta(project_id))
```

If we want to create a new project meta from scratch, we can do it like this:
```python
import supervisely as sly

# Create a new empty ProjectMeta object
meta = sly.ProjectMeta()
```

That's it! Now you have a ProjectMeta object that you can work with.<br>
NOTE: If you work with an existing project, you should **always download** the project meta first and then modify it. Otherwise, you can lose some important information. Use the new ProjectMeta object only when you create a new project from scratch or when you're fully aware of what you're doing.

#### Updating Project Meta on the platform
When you've made changes to the project meta, you can upload it back to the platform.<br>
NOTE: this operation will **OVERWRITE** the existing project meta, not merge it. Only use it when you're sure that you want to replace the existing project meta with the new one and don't forget to download the existing project meta first if you're working with an existing project.<br>
We also recommend always working with cloned copies of projects, not the original ones, to avoid accidental data loss. Please, read the [Cloning projects for development](ADD URL HERE) tutorial to learn more about cloning projects, when developing with Supervisely Python SDK.<br>

```python
import supervisely as sly

# Let's imagine that we've already prepared a new project meta and now we want to upload it to the platform.
meta: sly.ProjectMeta

api.project.update_meta(project_id, meta)
```
That's it! Now you've updated the project meta on the platform. And you can upload the annotations with new classes and tags to the project.

#### Iterating over classes and tags
It's a common use case to iterate over classes and tags in the project meta and it can be done easily with Supervisely Python SDK. Here's how you can do it:

```python
import supervisely as sly

# Let's imagine that we've already downloaded the project meta and now we want to iterate over classes and tags.
meta: sly.ProjectMeta

# Iterating over classes.
for obj_class in meta.obj_classes:
    # Each obj_class is an instance of sly.ObjClass, with a lot of useful fields and methods.
    # Check out [this section]() in the SDK Reference to learn more about ObjClass.
    obj_class: sly.ObjClass
    print(obj_class.name)

# We can iterate over tags in a similar way.
for tag_meta in meta.tag_metas:
    # Each tag_meta is an instance of sly.TagMeta, it also has a lot of useful fields and methods.
    # Check out [this section]() in the SDK Reference to learn more about TagMeta.
    tag_meta: sly.TagMeta
    print(tag_meta.name)
```

So it's easy to iterate over classes and tags in the project meta with Supervisely Python SDK, and it can be very helpful in a lot of use cases.

#### Adding new classes and tags
So, now we know how to download and upload the project meta. It's time to make a real case: we'll download the project meta, add a new class and a new tag, and upload the updated project meta back to the platform.

```python
import supervisely as sly

api: sly.Api = sly.Api(server_address, token)
project_id = 123

# Download the project meta, remember to crate a new ProjectMeta object from the JSON representation.
meta = sly.ProjectMeta.from_json(api.project.get_meta(project_id))

# Now, we want to add a new class to the project meta.
new_class_name = "cat"

# First, we need to check if the class already exists in the project meta.
if not meta.obj_classes.get(new_class_name):
    # If it doesn't exist, we can create a new class and add it to the project meta.
    new_class = sly.ObjClass(name=new_class_name, geometry_type=sly.Bitmap)
    meta = meta.add_obj_class(new_class)

# So, we've added a new class to the project meta. But we need to remember that this class
# only exist in our local ProjectMeta object, not on the platform yet!
# Before we upload a new annotation with this class, we need to upload the updated project meta to the platform.
# But we'll do it later after we add a new tag.

# Now, let's add a new tag to the project meta.
# The process is similar to adding a new class, but we'll add a tag to the project meta.

new_tag_name = "animal"
if not meta.tag_metas.get(new_tag_name):
    new_tag = sly.TagMeta(name=new_tag_name, value_type=sly.TagValueType.NONE)
    meta = meta.add_tag_meta(new_tag)

# Now we've added a new tag to the project meta. And we can upload the updated project meta to the platform.
api.project.update_meta(project_id, meta)
```

In this case, we've added a new class and a new tag to the project meta and updated it on the platform. Now we can upload new annotations with these classes and tags to the project.

## Summary
Now you know what is Project Meta and how to work with it. Let's make a quick summary:
1. Project Meta is a crucial part of any project in Supervisely. It contains the essential information about the project - Classes and Tags.
2. It's stored in the `meta.json` file in the root of the project folder.
3. You can download the project meta from the platform, create a new one from scratch, update it and upload it back to the platform.
4. When working with annotations, you should always remember to update the project meta accordingly.
5. You can do it manually by editing the `meta.json` file or with Supervisely Python SDK.
6. When working with an existing project, you should always download the project meta first and then modify it, or you may lose some important information.
7. Remember that updating the project meta will overwrite the existing project meta, not merge it.