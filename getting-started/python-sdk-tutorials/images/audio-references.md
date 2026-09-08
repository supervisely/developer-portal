# Audio References on Images

## Introduction

An image can carry one or more **audio files as references** — a dictated description, a voice note taken at capture time, the call a screenshot came from. Opening the image in the Image Labeling Toolbox shows an Audio panel with a player, so the annotator can listen while labeling.

Audio references are **reference only**. They are never annotated: they carry no figures, no tags and no geometry, and they are not part of the annotation.

The audio file itself is not stored inside the project — only its URL is. The file has to live somewhere the instance can serve it, and Team Files is the usual place.

{% hint style="info" %}
There is no way to attach audio from inside the labeling tool. Audio references are attached programmatically, at import time or afterwards, which is what this tutorial covers.
{% endhint %}

## Prerequisites

**Step 1.** Prepare `~/supervisely.env` file with credentials. [Learn more here.](../../basics-of-authentication.md)

**Step 2.** Install the SDK.

```bash
pip install supervisely
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

## How audio references are stored

References live in the image's `meta` field, under the `audio` key, as a list of objects:

```json
{
  "audio": [
    {
      "url": "https://app.supervisely.com/h5un6l2bnaz1vj8a9qgms4/operator-note.mp3",
      "name": "Operator note",
      "mimeType": "audio/mpeg"
    },
    {
      "url": "https://app.supervisely.com/h5un6l2bnaz1vj8a9qgms4/second-pass.mp3",
      "name": "Second pass"
    }
  ]
}
```

| Field | Required | What it is |
|---|---|---|
| `url` | yes | Direct URL of the audio file. |
| `name` | no | Label shown next to the player. |
| `mimeType` | no | MIME type, e.g. `audio/mpeg`. |

This is the only shape the SDK reads and writes. `sly.AudioReference` is the typed form of one entry, so you do not have to build these dictionaries by hand:

```python
reference = sly.AudioReference(
    url="https://app.supervisely.com/h5un6l2bnaz1vj8a9qgms4/operator-note.mp3",
    name="Operator note",
    mime_type="audio/mpeg",
)
```

## Upload a local audio file and attach it

`upload_audio_reference` does both steps at once: it uploads the file to Team Files and attaches the resulting URL to the image.

```python
reference = api.image.upload_audio_reference(
    id=3212008,
    team_id=8,
    path="/home/admin/audio/operator-note.mp3",
    name="Operator note",
)

print(reference.url)
# https://app.supervisely.com/.../audio-references/3212008/operator-note.mp3
```

By default the file lands in `/audio-references/<image id>/<file name>`, and `name` defaults to the file name without its extension. Pass `remote_path` to choose a different location in Team Files.

{% hint style="warning" %}
The project keeps only the URL, so the file has to stay in Team Files. Delete it and the player stops working.
{% endhint %}

## Attach a file that is already hosted

If the audio is already reachable by URL, attach it directly.

`set_audio_references` replaces the whole list; other keys in the image meta are left alone:

```python
api.image.set_audio_references(
    id=3212008,
    references=[
        sly.AudioReference(url=first_url, name="Operator note"),
        sly.AudioReference(url=second_url, name="Second pass"),
    ],
)
```

`add_audio_reference` appends to what is already attached:

```python
api.image.add_audio_reference(
    id=3212008,
    reference=sly.AudioReference(url=third_url, name="Third pass"),
)
```

Pass an empty list to remove every reference:

```python
api.image.set_audio_references(id=3212008, references=[])
```

## Read them back

```python
for reference in api.image.get_audio_references(id=3212008):
    print(reference.name, reference.mime_type, reference.url)
# Operator note audio/mpeg https://app.supervisely.com/.../operator-note.mp3
# Second pass None https://app.supervisely.com/.../second-pass.mp3
```

Entries that are not in the canonical form — anything without a `url` — are skipped with a warning instead of raising, so one hand-written entry cannot make the whole image unreadable.

## Attach audio while uploading images

When you upload an image you pass `meta` rather than an image ID. `update_audio_references` returns a copy of a meta dict with the references written into it:

```python
meta = api.image.update_audio_references(
    {"Camera Make": "Canon"},
    sly.AudioReference(url=audio_url, name="Operator note"),
)

image_info = api.image.upload_path(
    dataset_id=452984,
    name="IMG_2084.jpeg",
    path="/home/admin/images/IMG_2084.jpeg",
    meta=meta,
)
```

## Import and export

Audio references travel with the project as part of the image metadata, so **you have to ask for image metadata explicitly**. `save_image_meta` defaults to `False`:

```python
sly.download_project(
    api,
    project_id=17427,
    dest_dir="/home/admin/audio-project",
    save_image_meta=True,  # without this, audio references are not exported
)
```

This writes one JSON file per image under `<dataset>/meta/`, which `upload_project` reads back:

```python
project_id, project_name = sly.upload_project(
    dir="/home/admin/audio-project",
    api=api,
    workspace_id=349,
    project_name="Audio project copy",
)
```

{% hint style="warning" %}
Only the URLs are exported, not the audio files. Importing into a **different** instance leaves references pointing at the instance they came from — reachable only while that instance serves them. To make a copy self-contained, re-upload the audio with `upload_audio_reference` after the import.
{% endhint %}
