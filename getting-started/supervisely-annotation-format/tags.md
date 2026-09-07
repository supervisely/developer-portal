# Tags

In Supervisely tags provide an option to associate some additional information with the labeled image or the labels on it. Each individual tag can be attached to a single image or a single annotation only once, but there's no limit on how many times the same tag can be attached to different parts of the scene. There are different lists of tags for images and figures in the annotation file.

When defining a tag, you assign it a name, possible values for a tag instance and what types of things it can be attached to. We support values of the following types: None (without an assigned value), Text, Number, Date, and One of.

## Tags With 'None' Value

Tags of 'none' type can't be assigned a value. Adding one manually will result in an error. Also, it [could not be used](../supervisely-annotation-json-format/project-classes-and-tags.md#fields-definitions) as a group tag for the multiview mode.

JSON format for 'None' tags:

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

JSON format for 'text' tags:

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

JSON format for 'one of' tags:

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

## Tag with Date Value

Tags of type 'date' store a date-time value as an ISO 8601 string. In the Labeling Tool, a date picker is provided for input. The `possible_values` field cannot be used with this type.

Accepted formats:
- `2026-04-23T15:15:48`
- `2026-05-12T21:14:12.000Z`
- `2026-04-27 11:00:46`
- `2026-05-12T21:14:12+00:00`

JSON format for 'date' tags:

```json
{
    "name": "reviewed_at",
    "value": "2026-04-23T15:15:48"
}
```

Fields definitions:

* `name` - string - name of the tag
* `value` - ISO 8601 date-time string
* Optional fields `id`, `tagId`, `labelerLogin`, `createdAt`, `updatedAt` will be described [below](tags.md#optional-fields)

## Optional fields

The following fields are created and assigned automatically by the system when the tags are first created in it (or the data is uploaded). This means these fields are optional, and you don't have to assign them during manual annotation.

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

## Custom data

A tag assignment can carry arbitrary user JSON in the optional `customData` field. It is
independent of the tag definition and of the tag value, so the same tag can carry different
custom data on every entity it is attached to.

```json
{
    "id": 503051990,
    "tagId": 1693352,
    "name": "cat",
    "value": "fluffy",
    "customData": {
        "confidence": 0.92,
        "source": "model_v3",
        "reviewed": false,
        "bbox_hint": [10, 20, 30, 40],
        "meta": { "title": "kept as-is", "groupId": 7 }
    }
}
```

Field definition:

* `customData` - object - arbitrary JSON. Nested objects and arrays are allowed, and keys are
  never renamed or stripped, so reserved-looking names such as `title` or `groupId` come back
  exactly as they were stored.

The key is emitted only when it is not empty. Annotations of projects that never use the
feature are unchanged, so existing exports stay byte-identical.

Image, object (figure) and annotation object tags all support it.

### Reading and writing it from the Python SDK

`custom_data` is a property of `Tag`, `VideoTag`, `VolumeTag` and `PointcloudTag`, and it
survives annotation download and upload:

```python
import supervisely as sly

tag = sly.Tag(
    meta=project_meta.get_tag_meta("cat"),
    value="fluffy",
    custom_data={"confidence": 0.92, "source": "model_v3"},
)
print(tag.custom_data)
# Output: {'confidence': 0.92, 'source': 'model_v3'}
```

To change the custom data of a tag that is already attached, use the update methods. They take
the ID of the **tag assignment**, not of the project tag meta, and return the stored object
after the update:

```python
from supervisely.api.entity_annotation.tag_api import TagCustomDataUpdateStrategy

# image, video, volume or point cloud tag
api.image.tag.update_custom_data(
    tag_id=1024,
    custom_data={"confidence": 0.92},
    update_strategy=TagCustomDataUpdateStrategy.MERGE,
)

# figure (object) tag
api.image.tag.update_figure_custom_data(
    tag_id=5077,
    custom_data={"occluded": True},
)

# annotation object tag
api.video.tag.update_annotation_object_custom_data(
    tag_id=311,
    custom_data={"track_quality": "good"},
)
```

`update_strategy` is `merge` by default, which deep-merges the given object into the stored one.
Pass `replace` to overwrite it wholesale - that is what removes a key, since a merge can only
ever add or overwrite one. Under `merge`, an array never merges into an object or the other way
round: it replaces it, and two arrays merge by index.

## Examples

**Image tags:**

![](../../.gitbook/assets/image\_tags.png)

JSON format for image tags:

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

JSON format for object tags:

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
