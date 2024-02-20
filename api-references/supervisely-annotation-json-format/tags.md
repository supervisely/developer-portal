# Tags

In Supervisely tags provide an option to associate some additional information with the labeled image or the labels on it. Each individual tag can be attached to a single image or asingle annotation only once, but there's not limit on how many times the same tag can be attached to different parts of the scene. There are different lists of tags for images and figures in the annotation file.

When defining a tag, you assign it a name, possible values for a tag instance and what types of things it can be attached to. We support values of the following types: None (without an assigned value), Text, Number, and One of.<br>

## Adding Tags

As we mentioned earlier in the section about [ProjectMeta](../supervisely-annotation-json-format/project-meta.md), to be able assign tags, it must be defined in the project meta. The tag can be added to the project meta in the following ways:

#### 1. **In the UI**:
It's the simplest way to add a tag. However, it's important to note, that this way is not suitable for automation and/or when it's necessary to add a large number of tags.<br>
To do it:
1. Open the project.
2. Go to `Tags` tab.
3. Click on `New` button.
4. Select the type of the tag and fill in the necessary fields.


![Creating Tag in UI](../../.gitbook/assets/creating_tag_in_ui.png)


#### 2. **Using Python SDK**:
This way is suitable for automation and/or when it's necessary to add a large number of tags. To do it, you can use the following code:

```bash
pip install supervisely
```


```python
import supervisely as sly

# Create an API client using environment variables.
api = sly.Api.from_env()

project_id = 123

# Retrieve project meta from the server.
project_meta = sly.ProjectMeta.from_json(api.project.get_meta(project_id))

# Create a new tag meta, using tag name and value type.
tag_meta = sly.TagMeta('like', sly.TagValueType.NONE)

# Add the new tag meta to the project meta.
new_project_meta = project_meta.add_tag_meta(tag_meta)

# Now, we can update the project meta on the server.
api.project.update_meta(project_id, new_project_meta)
```

In this example both steps (retrieving and updating the project meta) are shown. However, it's optional, you can work with ProjectMeta object even locally and/or update it only when it's necessary. For example, you can add different tags to Project Meta and only after that update the project meta on the server once.


#### 3. **Using CLI**:
Alternatively, you can use the Supervisely CLI to add a tag to the project meta. To do it, you can use the following command:

```bash
pip install supervisely
```

```bash
supervisely add-tag --name "like" --value-type "none" --project-id 123
```

##  Tag Value Types
Tags in Supervisely can have the following value types:
1. **None** - tag without an assigned value. They can be used when the tag should not have any additional information, e.g. the classic scenario with train/val/test tags.
2. **Text** - tag with a string value. They can be used when you need to have one tag, which can store a wide variety of values, e.g. model's confidence, etc.
3. **One Of** - tag with a value from a given list. They can be used when you need to have a tag, which can store only a limited number of values, e.g. car color, etc.

Same with creating tags you can select the type of the tag in the UI, using Python SDK or CLI.
Here's some references when creating tags with Python SDK:

```python
import supervisely as sly

none_tag_meta = sly.TagMeta('like', sly.TagValueType.NONE)
text_tag_meta = sly.TagMeta('car_color', sly.TagValueType.TEXT)
one_of_tag_meta = sly.TagMeta('situated', sly.TagValueType.ONE_OF, values=['inside', 'outside'])
```

And for CLI:

```bash
supervisely add-tag --name "like" --value-type "none" --project-id 123
supervisely add-tag --name "car_color" --value-type "text" --project-id 123
supervisely add-tag --name "situated" --value-type "one_of" --values "inside, outside" --project-id 123
```

Now, let's take a look at the different types of tags and their json format.


#### Tags With 'None' Value

Tags of 'none' type can't be assigned a value. Adding one manually will result in an error. Also it [could not be used](../supervisely-annotation-json-format/project-classes-and-tags.md#fields-definitions) as a group tag for the multi-view mode.

Json format for 'None' tags:

```json
{
    "id": 86334622,
    "tagId": 28256197,
    "labelerLogin": "alexxx",
    "createdAt": "2020-08-23T09:51:06.246Z",
    "updatedAt": "2020-08-23T09:51:06.246Z",
    "name": "like",
    "value": null
}
```

Fields definitions:

* `name` - string - name of the tag
* `value` - value of the current tag (always null for any tag of type 'none')
* Optional fields `id`, `tagId`, `labelerLogin`, `createdAt`, `updatedAt` will be described [below](tags.md#optional-fields)

## Tag with String('Text') Value

Tags of type 'string' can only take a string value. Adding a different type of value during manual annotation will result in an error.

Json format for 'text' tags:

```json
{
    "id": 95462538,
    "tagId": 28256201,
    "labelerLogin": "alexxx",
    "createdAt": "2020-07-24T07:30:39.202Z",
    "updatedAt": "2020-07-24T07:30:39.202Z",
    "name": "car_color",
    "value": "red"
}
```

Fields definitions:

* `name` - string - name of the tag
* `value` - value of current tag
* Optional fields `id`, `tagId`, `labelerLogin`, `createdAt`, `updatedAt` will be described [below](tags.md#optional-fields)

## Tag with value from a given list ('One Of')

Tag of type 'One Of' can only take a value from the list of possible values for this tag. List of possible values is set when creating the tag. Adding a value not from the list during manual annotation will result in an error.

Json format for 'one of' tags:

```json
 {
    "id": 86334621,
    "tagId": 28256198,
    "labelerLogin": "alexxx",
    "createdAt": "2020-08-23T09:51:02.843Z",
    "updatedAt": "2020-08-23T09:51:02.843Z",
    "name": "situated",
    "value": "outside"
}
```

Fields definitions:

* `name` - string - name of the tag
* `value` - value of current tag
* Optional fields `id`, `tagId`, `labelerLogin`, `createdAt`, `updatedAt` will be described [below](tags.md#optional-fields)

## Optional fields

The following fields are created and assigned automatically by the system when the tags are first created in it (or the data is uploaded). This means these fields are optional and you don't have to assign them during manual annotation.

Optional fields:

```json
"id": 503051990,
"tagId": 1693352,
"labelerLogin": "alexxx",
"createdAt": "2020-08-22T09:32:48.010Z",
"updatedAt": "2020-08-22T09:33:08.926Z".
```

Fields definitions:

* `id` - unique identifier of the current object
* `tagId` - unique tag identifier of the current object
* `labelerLogin` - string - the name of user who created the current figure
* `createdAt` - string - date and time of figure creation
* `updatedAt` - string - date and time of the last figure update

## Examples

**Image tags:**

![](../../.gitbook/assets/image\_tags.png)

Json format for image tags:

```json
"tags": [
    {
        "id": 86334622,
        "tagId": 28256197,
        "labelerLogin": "alexxx",
        "createdAt": "2020-08-23T09:51:06.246Z",
        "updatedAt": "2020-08-23T09:51:06.246Z",
        "name": "like",
        "value": null
    },
    {
        "id": 86334621,
        "tagId": 28256198,
        "labelerLogin": "alexxx",
        "createdAt": "2020-08-23T09:51:02.843Z",
        "updatedAt": "2020-08-23T09:51:02.843Z",
        "name": "situated",
        "value": "outside"
    }
]
```

Fields definitions:

* `name` - string - name of the tag
* `value` - value of current tag
* Optional fields `id`, `tagId`, `labelerLogin`, `createdAt`, `updatedAt` are described [above](tags.md#optional-fields)

#### **Object tags:**

![](../../.gitbook/assets/object\_tags.png)

Json format for object tags:

```json
"tags": [
    {
        "id": 95462539,
        "tagId": 28256199,
        "labelerLogin": "alexxx",
        "createdAt": "2020-07-24T07:30:39.202Z",
        "updatedAt": "2020-07-24T07:30:39.202Z",
        "name": "vehicle_age",
        "value": "vintage"
    },
    {
        "id": 95462538,
        "tagId": 28256201,
        "labelerLogin": "alexxx",
        "createdAt": "2020-07-24T07:30:39.202Z",
        "updatedAt": "2020-07-24T07:30:39.202Z",
        "name": "car_color",
        "value": "red"
    }
]
```

Fields definitions:

* `name` - string - name of the tag
* `value` - value of current tag
* Optional fields `id`, `tagId`, `labelerLogin`, `createdAt`, `updatedAt` are described [above](tags.md#optional-fields)
