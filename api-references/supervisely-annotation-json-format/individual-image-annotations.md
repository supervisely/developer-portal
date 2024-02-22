# Individual Image Annotations

For each image, we store the annotations in a separate json file named `image_name.image_format.json` with the following file structure:

```json
{
    "description": "food",
    "name": "tomatoes-eggs-dish.jpg",
    "size": {
        "width": 2100,
        "height": 1500
    },
    "tags": [],
    "objects": []
}
```



Fields definitions:

* `name` - string - image name
* `description` - string - (optional) - This field is used to store the text we want to assign to the image. In the labeling interface it corresponds to the 'data' filed.&#x20;
* `size` - stores image size. Mostly, it is used to get the image size without the actual image reading to speed up some data processing steps.
  * `width` - image width in pixels
  * `height` - image height in pixels
* `tags` - list of strings that will be interpreted as image [tags](broken-reference)
* `objects` - list of [objects on the image](broken-reference)

## Full image annotation example with objects and tags

![image example](<../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1).png>)

Example:

```json
{
    "description": "",
    "tags": [
        {
            "id": 86458971,
            "tagId": 28283797,
            "name": "like",
            "value": null,
            "labelerLogin": "alexxx",
            "createdAt": "2020-08-26T09:12:51.155Z",
            "updatedAt": "2020-08-26T09:12:51.155Z"
        },
        {
            "id": 86458968,
            "tagId": 28283798,
            "name": "situated",
            "value": "outside",
            "labelerLogin": "alexxx",
            "createdAt": "2020-08-26T09:07:26.408Z",
            "updatedAt": "2020-08-26T09:07:26.408Z"
        }
    ],
    "size": {
        "height": 952,
        "width": 1200
    },
    "objects": [
        {
            "id": 497521359,
            "classId": 1661571,
            "description": "",
            "geometryType": "bitmap",
            "labelerLogin": "alexxx",
            "createdAt": "2020-08-07T11:09:51.054Z",
            "updatedAt": "2020-08-07T11:09:51.054Z",
            "tags": [],
            "classTitle": "person",
            "bitmap": {
                "data": "eJwBgQd++IlQTkcNChoKAAAADUlIRF",
                "origin": [
                    535,
                    66
                ]
            }
        },
        {
            "id": 497521358,
            "classId": 1661574,
            "description": "",
            "geometryType": "rectangle",
            "labelerLogin": "alexxx",
            "createdAt": "2020-08-07T11:09:51.054Z",
            "updatedAt": "2020-08-07T11:09:51.054Z",
            "tags": [],
            "classTitle": "bike",
            "points": {
                "exterior": [
                    [
                        0,
                        236
                    ],
                    [
                        582,
                        872
                    ]
                ],
                "interior": []
            }
        }
    ]
}
```

## Common tasks

In this article, we will cover the most common tasks that you can perform with individual image annotations using Supervisely Python SDK.

#### Retrieve image annotation from the API

Don't forget to install the package first:

```bash
pip install supervisely
```

```python
import supervisely as sly

api = sly.Api.from_env()

# Download image annotatation in JSON format.
image_id = 123
ann_json = api.annotation.download_json(image_id)

# We recommend to work with sly.Annotation object, not with raw JSON.
# To retrieve sly.Annotation object from JSON, you'll need the sly.ProjectMeta.

project_id = 456
project_meta = sly.ProjectMeta.from_json(api.project.get_meta(project_id))

# Now you can create sly.Annotation object from JSON and work with it.
ann = sly.Annotation.from_json(ann_json, project_meta)
```

Now you have sly.Annotation object and you can work with it.

#### Upload image annotation to the API

```python
# Let's assume that you have sly.Annotation object and image_id.
# And now you want to upload the annotation to the API.

ann: sly.Annotation

api.annotation.upload_ann(image_id, ann)
```

That's it! The annotation was replaced on the server with the new one.


#### Add label (figure) to the image annotation

It's a common task to add a new label to the image annotation. But let's imagine that we want to create a new empty annotation and add a new label to it.

```python
import supervisely as sly
import cv2

# When creating a new annotation, you need to know the image size.
image_size = (300, 600)

# If you don't know the image size, you can read the image and get its size.
image_path = "path/to/image.jpg"
image_np = cv2.imread(image_path)
image_size = (image_np.shape[1], image_np.shape[0])

# Now, when you know the image size, you can create a new empty annotation.
ann = sly.Annotation(image_size)

# Let's now create a label, but of couese first we need to create a new object class.
class_kiwi = sly.ObjClass('kiwi', sly.Rectangle)

# So, we created a new Rectangle object class with the name 'kiwi'.
# Now, let's create a new label with this class and add it to the annotation.
label_kiwi = sly.Label(sly.Rectangle(0, 0, 300, 300), class_kiwi)

# Annotation object is immutable, so we'll need to create a new one, using 
# add_label() method:
new_ann = ann.add_label(label_kiwi)

# Our annotation is ready, but before upload we must add this new object class to the ProjectMeta.
new_project_meta = project_meta.add_obj_class(class_kiwi)

# And now, we need to update the project meta on the server.
api.project.update_meta(project_id, new_project_meta)

# And finally, we can upload our new annotation to the server.
api.annotation.upload_ann(image_id, new_ann)
```

So, let's take a short summary of what we did:
1. We created a new empty annotation with the image size.
2. We created a new object class.
3. We created a new label with the new object class and added it to the annotation.
4. We added the new object class to the ProjectMeta and updated the project meta on the server.
5. We uploaded the new annotation to the server.

When working with annotations, pay attention to make the project meta up-to-date.