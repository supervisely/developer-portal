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

![image example](<../../.gitbook/assets/image (1) (1) (1) (1).png>)

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
