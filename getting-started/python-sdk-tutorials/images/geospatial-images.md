# Geospatial Data: Satellite Imagery, DTM, and OSM Annotations

## Introduction

This tutorial walks through a complete geospatial data workflow in Supervisely: downloading satellite imagery, DTM elevation tiles, and OpenStreetMap vector annotations; storing geographic context in image metadata and dataset custom data; labeling with multi-layer interfaces and AI assistance; and exporting annotated data back to OSM XML format.

Everything in this workflow is built on standard Supervisely primitives — images with metadata, datasets with custom data, and object annotations. No special project types or external geodata infrastructure are required. The result is a fully reproducible pipeline where geographic context travels with the data at every step and any Supervisely platform feature can be applied without modification.

The [Satellite, DTM & OSM Downloader](https://ecosystem.supervisely.com/apps/slyosm/import_osm) and [Export to OSM Format](https://ecosystem.supervisely.com/apps/slyosm/export_to_osm) apps implement this pipeline end to end. This article explains the underlying mechanics so you can reproduce, extend, or integrate any part of it using the Python SDK.

{% hint style="info" %}
Source code for both apps is available at [github.com/supervisely-ecosystem/slyosm](https://github.com/supervisely-ecosystem/slyosm).
{% endhint %}

## Prerequisites

**Step 1.** Prepare `~/supervisely.env` with your credentials. [Learn more here.](../../getting-started/basics-of-authentication.md)

**Step 2.** Install dependencies.

```bash
pip install supervisely pyproj numpy
```

**Step 3.** Initialize the API client.

```python
import os
from dotenv import load_dotenv
import supervisely as sly

if sly.is_development():
    load_dotenv("local.env")
    load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api.from_env()
```

## Architecture Overview

The geospatial pipeline relies on two storage mechanisms built into the Supervisely API:

| Storage | What is stored | API |
|---|---|---|
| **Image metadata** (`meta` field) | Per-image geographic context: center coordinates, homography matrix, CRS, bounding box | `api.image.upload_path(..., meta={...})` |
| **Dataset custom data** | OSM class mapping shared across all images in the dataset | `api.dataset.update_custom_data(id, data)` |

This separation is deliberate. Geographic context is per-image because each tile covers a different location. The OSM class mapping is per-dataset because it is the same for every image downloaded in one session — changing it between images would make annotations inconsistent.

## OSM Class Mapping

The OSM class mapping defines which OpenStreetMap features are downloaded, how they map to Supervisely object classes, and which OSM tags are written when annotations are exported back to `.osm` files.

### Structure

The mapping is a list of class specification objects stored as a JSON array. Each entry describes one Supervisely object class:

```python
osm_class_specs = [
    {
        "name": "building",
        "geometry": "polygon",
        "tags": {"building": True},
        "default_tag": {"building": "yes"},
        "color": [180, 180, 180],
    },
    {
        "name": "road_main",
        "geometry": "line",
        "tags": {"highway": ["motorway", "trunk", "primary", "secondary", "tertiary"]},
        "default_tag": {"highway": "secondary"},
        "buffer_m": 5.0,
        "color": [240, 93, 66],
    },
    {
        "name": "water",
        "geometry": "polygon",
        "tags": {"natural": "water"},
        "default_tag": {"natural": "water"},
        "color": [0, 130, 200],
    },
    {
        "name": "forest",
        "geometry": "polygon",
        "tags": {"natural": "wood", "landuse": "forest", "landcover": "trees"},
        "default_tag": {"natural": "wood"},
        "color": [34, 139, 34],
    },
]
```

| Field | Required | Description |
|---|---|---|
| `name` | Yes | Supervisely object class name |
| `geometry` | Yes | `"polygon"`, `"line"`, or `"point"` |
| `tags` | Yes | OSM tag filters. Value can be a string, boolean, or list of strings |
| `default_tag` | Yes | Single OSM tag written when exporting annotations to `.osm` files |
| `buffer_m` | No | Expand lines and points by this many meters to produce a polygon footprint in Supervisely |
| `color` | No | RGB color `[R, G, B]` for the Supervisely object class |

### Saving the Mapping to a Dataset

```python
dataset_info = api.dataset.get_info_by_id(dataset_id)
custom_data = dict(dataset_info.custom_data or {})

custom_data["osm_class_specs"] = osm_class_specs
custom_data["slyosm_schema_version"] = 1

api.dataset.update_custom_data(dataset_id, custom_data)
```

### Reading the Mapping Back

```python
dataset_info = api.dataset.get_info_by_id(dataset_id)
custom_data = dataset_info.custom_data or {}

specs = custom_data.get("osm_class_specs", [])
schema_version = custom_data.get("slyosm_schema_version", 0)

print(f"Found {len(specs)} class specs (schema v{schema_version})")
for spec in specs:
    print(f"  {spec['name']} — geometry={spec['geometry']}, tags={spec['tags']}")
```

The export app reads from this key automatically. If the key is absent it falls back to the built-in default mapping, so existing datasets without custom data continue to work.

## Image Geo Metadata

Every georeferenced image uploaded by the downloader app carries a `geo` object inside its `meta` field. This payload contains everything needed to project pixel coordinates back to longitude/latitude at export time.

### Geo Payload Structure

```python
geo = {
    # Scene center in WGS84
    "center_lat": 48.8566,
    "center_lon": 2.3522,

    # 3×3 homography matrix: pixel (col, row) → local CRS (x, y) in meters
    # Applied as: [x, y, 1]^T = H @ [col, row, 1]^T
    "pixel_to_local_h": [
        [0.489, -0.002, -250.3],
        [0.001,  0.489, -249.8],
        [0.0,    0.0,    1.0  ],
    ],

    # Azimuthal equidistant CRS centered on the scene, as WKT or ProjJSON
    "local_crs_wkt": "PROJCRS[\"...\"]",

    # Image dimensions
    "image_size_px": {"width": 1024, "height": 1024},

    # Approximate bounding box: [min_lon, min_lat, max_lon, max_lat]
    "bbox_left_bottom_right_top": [2.317, 48.833, 2.396, 48.880],

    # Rotation applied to the tile bounding box, in degrees
    "rotation_deg": 0.0,
}
```

{% hint style="info" %}
The local CRS is an azimuthal equidistant projection centered on the scene center. It maps the scene to a flat plane in meters, which is valid for tiles up to a few kilometers across. The homography matrix maps directly from pixel space to this local metric space. To reach WGS84 longitude/latitude, apply the homography then transform with pyproj.
{% endhint %}

### Uploading an Image with Geo Metadata

```python
import json
import numpy as np
from pyproj import CRS, Transformer

# Build the local CRS centered on the scene
center_lat, center_lon = 48.8566, 2.3522
local_crs = CRS.from_proj4(
    f"+proj=aeqd +lat_0={center_lat} +lon_0={center_lon} +units=m"
)

# Compute the pixel→local homography from the four corner points
# (src: pixel corners, dst: local metric corners — omitted for brevity)
pixel_to_local_h = np.eye(3)  # replace with actual computed homography

geo_meta = {
    "center_lat": center_lat,
    "center_lon": center_lon,
    "pixel_to_local_h": pixel_to_local_h.tolist(),
    "local_crs_wkt": local_crs.to_wkt(),
    "image_size_px": {"width": 1024, "height": 1024},
    "bbox_left_bottom_right_top": [2.317, 48.833, 2.396, 48.880],
    "rotation_deg": 0.0,
}

image_info = api.image.upload_path(
    dataset_id=dataset_id,
    name="paris_48_8566_2_3522.png",
    path="/path/to/tile.png",
    meta={"geo": geo_meta},
)
print(f"Uploaded image id={image_info.id}")
```

### Reading Geo Metadata from an Existing Image

```python
image_info = api.image.get_info_by_id(image_id)
image_meta = image_info.meta or {}
geo = image_meta.get("geo")

if geo is None:
    print("Image has no geo metadata — not a georeferenced tile.")
else:
    print(f"Center: {geo['center_lat']:.6f}, {geo['center_lon']:.6f}")
    print(f"Bbox: {geo['bbox_left_bottom_right_top']}")
    h = np.asarray(geo["pixel_to_local_h"])
    print(f"Homography:\n{h}")
```

### Projecting Pixel Coordinates to WGS84

```python
from pyproj import CRS, Transformer

geo = image_info.meta["geo"]

# Reconstruct the transform
local_crs = CRS.from_wkt(geo["local_crs_wkt"])
to_wgs84 = Transformer.from_crs(local_crs, "EPSG:4326", always_xy=True)
h = np.asarray(geo["pixel_to_local_h"])

def pixel_to_lonlat(col: float, row: float):
    point_h = np.array([col, row, 1.0])
    local = h @ point_h
    local_x, local_y = local[0] / local[2], local[1] / local[2]
    lon, lat = to_wgs84.transform(local_x, local_y)
    return lon, lat

# Example: center pixel of a 1024×1024 image
lon, lat = pixel_to_lonlat(512, 512)
print(f"Center pixel → {lat:.6f}°N, {lon:.6f}°E")
```

## Creating a Georeferenced Project

Here is a minimal end-to-end example that creates a project, adds object classes from the OSM mapping, saves the class mapping to the dataset, and uploads a georeferenced image.

```python
import supervisely as sly

api = sly.Api.from_env()

# 1. Create project and dataset
project = api.project.create(
    workspace_id, "geo_dataset", change_name_if_conflict=True
)
dataset = api.dataset.create(project.id, "berlin_grid")

# 2. Build project meta from OSM class specs
obj_classes = [
    sly.ObjClass("building", sly.Polygon, color=[180, 180, 180]),
    sly.ObjClass("road_main", sly.Polygon, color=[240, 93, 66]),
    sly.ObjClass("water",     sly.Polygon, color=[0, 130, 200]),
    sly.ObjClass("forest",    sly.Polygon, color=[34, 139, 34]),
]
meta = sly.ProjectMeta(obj_classes=sly.ObjClassCollection(obj_classes))
api.project.update_meta(project.id, meta.to_json())

# 3. Save OSM class mapping to dataset custom data
api.dataset.update_custom_data(dataset.id, {
    "osm_class_specs": osm_class_specs,  # defined earlier
    "slyosm_schema_version": 1,
})

# 4. Upload image with geo metadata
image_info = api.image.upload_path(
    dataset_id=dataset.id,
    name="berlin_52_5200_13_4050.png",
    path="/path/to/tile.png",
    meta={"geo": geo_meta},  # defined earlier
)
print(f"Project: {project.id}, Dataset: {dataset.id}, Image: {image_info.id}")
```

## Labeling Interfaces

Because each downloaded tile is a standard Supervisely image with object annotations, all labeling toolbox modes are available without any additional setup.

### Multiview

When the downloader app runs in **Multiview** mode, the satellite image and the DTM elevation image are uploaded as a linked pair using `api.image.upload_multiview_images`. Both images appear side by side in the labeling toolbox. Annotations are shared — a polygon drawn on the satellite view is visible on the DTM view and vice versa.

This is useful when elevation context is needed to make annotation decisions, for example distinguishing embankments from roads, identifying vegetation by canopy height, or locating buildings in shadow.

[Learn more about Multiview labeling →](https://docs.supervisely.com/labeling/labeling-toolbox/multi-view-images)

### Overlay

In **Overlay** mode, a single composited image is uploaded with the DTM blended on top of the satellite layer. The annotator can adjust DTM transparency on the fly in the labeling interface.

[Learn more about Overlay mode →](https://docs.supervisely.com/labeling/labeling-toolbox/overlay)

### Choosing a Mode Programmatically

The downloader app exposes interface mode as a configurable parameter. To replicate this in your own pipeline, upload either a single image (satellite-only or DTM-only) or a multiview pair:

```python
# Satellite only — standard upload
api.image.upload_path(dataset.id, "tile_sat.png", path_sat, meta={"geo": geo_meta})

# Multiview — satellite + DTM as a linked pair
api.project.set_multiview_settings(project.id)
api.image.upload_multiview_images(
    dataset_id=dataset.id,
    group_name="tile_001",
    paths=[path_sat, path_dtm],
    metas=[{"geo": geo_meta}, {"geo": geo_meta}],
)
```

## AI-Assisted Annotation

Georeferenced datasets are standard Supervisely image datasets, so every AI feature on the platform works without modification.

**Smart Tool** — interactive segmentation model that produces polygon or mask output from a few clicks. Effective for consistently shaped features like building footprints, water bodies, and agricultural fields that repeat across many tiles.

**Auto Labeling** — run any model from the Supervisely Ecosystem on your geospatial dataset to pre-annotate tiles in batch. Use the resulting annotations as a starting point for human review rather than annotating from scratch.

**NN training** — train or fine-tune a model directly on your geospatial dataset within the platform, then apply it to newly downloaded tiles. The geographic metadata in image `meta` is preserved through the training pipeline and remains available for export afterward.

## Exporting to OSM Format

The [Export to OSM Format](https://ecosystem.supervisely.com/apps/slyosm/export_to_osm) app reads geo metadata from each image and the OSM class mapping from the dataset, then projects polygon and line annotations back to WGS84 coordinates and writes standard OSM XML files.

### Output Archive Structure

```text
img/
    tile_001.png
    tile_002.png
ann/
    tile_001.png.json       ← Supervisely annotation JSON
    tile_002.png.json
osm/
    tile_001.png.osm        ← OSM XML, ready for JOSM or osmium
    tile_002.png.osm
```

### Running the Export via SDK

The export logic is importable directly. To run it programmatically:

```python
from import_osm.src.slyosm.osm_export import export_dataset_to_supervisely_dir
from pathlib import Path

result = export_dataset_to_supervisely_dir(
    api=api,
    dataset_id=dataset_id,
    output_dir=Path("/tmp/my_export"),
)

print(f"Exported {len(result.images)} image(s), {len(result.failures)} failure(s)")
for img in result.images:
    print(f"  {img.image_name}: osm={'yes' if img.osm_path else 'skipped (no geo)'}")
```

### OSM XML Output Format

Each `.osm` file is a standard [OSM XML](https://wiki.openstreetmap.org/wiki/OSM_XML) file compatible with [JOSM](https://josm.openstreetmap.de/), osmium, Overpass API, and any other OSM tooling. Polygon annotations become closed ways or multipolygon relations (when holes are present). Line annotations become open ways. All IDs are negative, following OSM convention for locally generated data.

```xml
<?xml version='1.0' encoding='utf-8'?>
<osm version="0.6" generator="slyosm-export">
  <bounds minlat="48.833000" minlon="2.317000" maxlat="48.880000" maxlon="2.396000"/>
  <node id="-1" action="modify" visible="true" lat="48.856600" lon="2.352200"/>
  <node id="-2" action="modify" visible="true" lat="48.857100" lon="2.353800"/>
  ...
  <way id="-101" action="modify" visible="true">
    <nd ref="-1"/>
    <nd ref="-2"/>
    ...
    <nd ref="-1"/>
    <tag k="building" v="yes"/>
  </way>
</osm>
```

## Dataset Custom Data Reference

The following keys are written and read by the slyosm apps. Use the same schema if you are integrating your own pipeline.

| Key | Type | Description |
|---|---|---|
| `osm_class_specs` | `list[dict]` | OSM class mapping array (see structure above) |
| `slyosm_schema_version` | `int` | Schema version, currently `1` |

```python
# Full read-write example
dataset_info = api.dataset.get_info_by_id(dataset_id)
custom_data = dict(dataset_info.custom_data or {})

# Read
specs = custom_data.get("osm_class_specs", [])

# Modify — add a new class
specs.append({
    "name": "solar_panel",
    "geometry": "polygon",
    "tags": {"generator:source": "solar"},
    "default_tag": {"generator:source": "solar"},
    "color": [255, 215, 0],
})

# Write back
custom_data["osm_class_specs"] = specs
api.dataset.update_custom_data(dataset_id, custom_data)
```

## Summary

This tutorial covered the complete geospatial data workflow in Supervisely using standard SDK primitives:

1. **OSM class mapping** is stored in dataset custom data under `osm_class_specs` and read automatically by the export app — change it once per dataset, not per image.
2. **Geographic context** is stored in image metadata under the `geo` key as a homography matrix, local CRS, and bounding box — enough to round-trip any pixel coordinate to WGS84 and back.
3. **Labeling** works in Multiview, Overlay, or standard single-image mode — all Supervisely tools and AI features apply without modification.
4. **Export** produces standard OSM XML files alongside original images and Supervisely annotation JSONs, ready to open in [JOSM](https://josm.openstreetmap.de/) or any other OSM tool.

The full source code is available in the [slyosm repository](https://github.com/supervisely-ecosystem/slyosm).
