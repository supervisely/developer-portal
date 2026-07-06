---
description: How to create, upload, download, draw, and split Multipolygon labels in Python SDK
---

# Multipolygons

## Introduction

In this tutorial, you will learn how to work with `sly.Multipolygon` in Supervisely Python SDK. Multipolygon is useful when one label must contain several separate polygon parts, for example one building group, one road island object, or one instance split into disconnected visible regions.

You will learn how to:

1. Create a `Multipolygon` from several `Polygon` objects.
2. Add a Multipolygon class to project meta.
3. Upload an image annotation with a Multipolygon label.
4. Download and deserialize the annotation.
5. Draw the label locally.
6. Convert a Multipolygon label into separate Polygon labels.

{% hint style="info" %}
Supervisely Python SDK version `6.74.10` or newer is required.
{% endhint %}

## Prepare credentials

Prepare `~/supervisely.env` with credentials and load it before creating the API client.

```python
import os

from dotenv import load_dotenv

import supervisely as sly

load_dotenv(os.path.expanduser("~/supervisely.env"))
api = sly.Api.from_env()
```

## Create a project and class

```python
workspace_id = sly.env.workspace_id()

project = api.project.create(
    workspace_id,
    name="Multipolygon tutorial",
    change_name_if_conflict=True,
)
dataset = api.dataset.create(project.id, name="ds0")

buildings = sly.ObjClass(
    name="building_group",
    geometry_type=sly.Multipolygon,
    color=[255, 0, 121],
)

meta = sly.ProjectMeta(obj_classes=[buildings])
api.project.update_meta(project.id, meta.to_json())
```

## Create a Multipolygon

A Multipolygon is created from two or more polygon parts. Each part can have its own holes.

```python
part_1 = sly.Polygon(
    exterior=[
        [80, 80],
        [80, 220],
        [220, 220],
        [220, 80],
    ]
)

part_2 = sly.Polygon(
    exterior=[
        [280, 100],
        [280, 250],
        [450, 250],
        [450, 100],
    ],
    interior=[
        [
            [330, 145],
            [330, 195],
            [390, 195],
            [390, 145],
        ]
    ],
)

multipolygon = sly.Multipolygon([part_1, part_2])
label = sly.Label(multipolygon, buildings)
```

## Upload image and annotation

```python
image_path = "data/city.jpg"
image_info = api.image.upload_path(dataset.id, "city.jpg", image_path)

image_np = sly.image.read(image_path)
annotation = sly.Annotation(img_size=image_np.shape[:2], labels=[label])

api.annotation.upload_ann(image_info.id, annotation)
```

## Download and deserialize annotation

```python
meta_json = api.project.get_meta(project.id)
meta = sly.ProjectMeta.from_json(meta_json)

ann_info = api.annotation.download(image_info.id)
ann = sly.Annotation.from_json(ann_info.annotation, meta)

downloaded_label = ann.labels[0]
downloaded_geometry = downloaded_label.geometry

print(type(downloaded_geometry).__name__)
print(len(downloaded_geometry.parts))
```

## Draw Multipolygon locally

```python
canvas = image_np.copy()
ann.draw_pretty(canvas, thickness=3)

sly.image.write("multipolygon_preview.jpg", canvas)
```

## Convert Multipolygon to polygons

Use `Label.convert()` when you need one Polygon label per Multipolygon part.

```python
polygon_class = sly.ObjClass("building_part", sly.Polygon, color=[0, 255, 0])

polygon_labels = downloaded_label.convert(polygon_class)

print(len(polygon_labels))
print([type(label.geometry).__name__ for label in polygon_labels])
```

You can also get polygon geometries directly:

```python
polygons = downloaded_geometry.to_polygons()
```

## JSON structure

In Supervisely annotation JSON, Multipolygon uses `geometryType: "multipolygon"` and stores parts in the `parts` field.

```json
{
    "classTitle": "building_group",
    "description": "",
    "geometryType": "multipolygon",
    "tags": [],
    "parts": [
        {
            "exterior": [[80, 80], [220, 80], [220, 220], [80, 220]],
            "interior": []
        },
        {
            "exterior": [[280, 100], [450, 100], [450, 250], [280, 250]],
            "interior": [
                [[330, 145], [390, 145], [390, 195], [330, 195]]
            ]
        }
    ]
}
```

## Recap

In this tutorial, you created a Multipolygon class, uploaded and downloaded a Multipolygon label, rendered it locally, and converted it into separate Polygon labels.
