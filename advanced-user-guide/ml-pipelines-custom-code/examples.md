---
description: Short Custom Code scripts for each step of a video clip pipeline
---

# Examples

Each example is a complete script. Save it in Team Files under `/ml-pipelines/custom-code/`, select it in the **Custom Code** node, and put its parameters in **Parameters**. Read [ML Pipelines: Custom Code node](README.md) first for the contract.

The pipeline these examples build:

`Videos Project` → `Filter Videos by Tags` → `Custom Code` → `Create New Project`

1. Select videos that have a tag.
2. Take a subset of them.
3. Go over all frames, resize them and apply an OpenCV function.
4. Select the frame ranges you are interested in.
5. Cut a clip for each range and save the clips to a new project.

## 1. Select videos by tag

The simplest way is the `Filter Videos by Tags` node: choose the tag and connect its `Output True` to the next node. To decide in code instead, for example on a tag value:

```python
def process(video_path, video_info, ann, params):
    """Keep the whole video only if it has the video tag params["tag"]."""
    tag_names = [tag.name for tag in ann.tags if tag.frame_range is None]
    if params.get("tag") in tag_names:
        return [(0, video_info.frames_count - 1)]
    return []
```

Parameters: `{"tag": "lane"}`

## 2. Take a subset

```python
import random


def process(video_path, video_info, ann, params):
    """Keep about params["ratio"] of the videos. The choice depends only on the seed and the video
    id, so it is the same on every run and in every worker process."""
    rng = random.Random(f'{params.get("seed", 42)}-{video_info.id}')
    if rng.random() < params.get("ratio", 0.5):
        return [(0, video_info.frames_count - 1)]
    return []
```

Parameters: `{"ratio": 0.5, "seed": 42}`

Videos are processed in parallel and independently, so a script cannot count how many videos it has kept. Use a ratio with a fixed seed, as above, rather than "the first N videos".

## 3. Go over all frames, resize and apply an OpenCV function

```python
import cv2


def frame_scores(video_path, width=320):
    """Go over all frames, resize each one and apply an OpenCV function. Here: how much the
    picture changed since the previous frame (mean absolute difference, 0-255)."""
    scores = []
    previous = None
    capture = cv2.VideoCapture(video_path)
    while True:
        ok, frame = capture.read()
        if not ok:
            break
        height = max(1, round(frame.shape[0] * width / frame.shape[1]))
        gray = cv2.cvtColor(cv2.resize(frame, (width, height)), cv2.COLOR_BGR2GRAY)
        scores.append(0.0 if previous is None else float(cv2.absdiff(gray, previous).mean()))
        previous = gray
    capture.release()
    return scores


def process(video_path, video_info, ann, params):
    scores = frame_scores(video_path, params.get("width", 320))
    print(f"{video_info.name}: {len(scores)} frames, max change {max(scores, default=0):.1f}")
    return [(0, video_info.frames_count - 1)]
```

Parameters: `{"width": 320}`

This one passes every video through unchanged and prints its scores to the log: a quick way to choose a threshold for the next step. Replace `frame_scores` with your own function, for example one imported from your package ([Your own packages](own-packages.md)).

## 4. Select frame ranges

```python
def select_ranges(scores, threshold, min_length, last_frame):
    """Turn per-frame scores into the (start, end) frame ranges where score > threshold for at
    least min_length frames in a row."""
    ranges = []
    start = None
    for index, score in enumerate(scores[: last_frame + 1]):
        if score > threshold and start is None:
            start = index
        elif score <= threshold and start is not None:
            if index - start >= min_length:
                ranges.append((start, index - 1))
            start = None
    end = min(len(scores), last_frame + 1)
    if start is not None and end - start >= min_length:
        ranges.append((start, end - 1))
    return ranges
```

Pass `last_frame=video_info.frames_count - 1`: the number of frames OpenCV decodes can differ slightly from the video's metadata, and every range must stay inside `0 .. frames_count - 1`.

## 5. Cut clips and save them to a new project

Nothing to write: return the ranges from `process`, and the node cuts a clip for each one, at exact frames, with its annotations. Connect `Create New Project` after the node, enter the project name and run the pipeline. The clips are named `<video name>_frames_<start>-<end>.mp4`.

## The full pipeline

Steps 2 to 4 in one script. Tag selection (step 1) is done by `Filter Videos by Tags` before it.

```python
"""Custom Code example: subset, frame function and frame ranges in one script.

Pipeline: Videos Project -> Filter Videos by Tags -> Custom Code (this script) -> Create New Project.
Parameters: {"ratio": 0.75, "seed": 42, "width": 320, "threshold": 5.0, "min_length": 25}
"""

import random

import cv2


def frame_scores(video_path, width):
    scores = []
    previous = None
    capture = cv2.VideoCapture(video_path)
    while True:
        ok, frame = capture.read()
        if not ok:
            break
        height = max(1, round(frame.shape[0] * width / frame.shape[1]))
        gray = cv2.cvtColor(cv2.resize(frame, (width, height)), cv2.COLOR_BGR2GRAY)
        scores.append(0.0 if previous is None else float(cv2.absdiff(gray, previous).mean()))
        previous = gray
    capture.release()
    return scores


def select_ranges(scores, threshold, min_length, last_frame):
    ranges = []
    start = None
    for index, score in enumerate(scores[: last_frame + 1]):
        if score > threshold and start is None:
            start = index
        elif score <= threshold and start is not None:
            if index - start >= min_length:
                ranges.append((start, index - 1))
            start = None
    end = min(len(scores), last_frame + 1)
    if start is not None and end - start >= min_length:
        ranges.append((start, end - 1))
    return ranges


def process(video_path, video_info, ann, params):
    # 1. Subset: keep about `ratio` of the videos, the same ones on every run.
    rng = random.Random(f'{params.get("seed", 42)}-{video_info.id}')
    if rng.random() >= params.get("ratio", 1.0):
        return []

    # 2. Go over all frames, resize them and apply an OpenCV function.
    scores = frame_scores(video_path, params.get("width", 320))

    # 3. Keep the frame ranges we are interested in. Each one becomes a clip.
    return select_ranges(
        scores,
        threshold=params.get("threshold", 5.0),
        min_length=params.get("min_length", 25),
        last_frame=video_info.frames_count - 1,
    )
```

Set up the pipeline:

1. Launch **ML Pipelines** on your video project. The `Videos Project` node is added for you.
2. Add `Filter Videos by Tags`, select your tag, and keep the condition **Video has tags**.
3. Add `Custom Code`, connect it to `Output True` of the filter. In **EDIT**, open the script, set **Parameters**, and **Save settings**.
4. Add `Create New Project`, connect it to `Custom Code`, and enter the project name.
5. Click **Run**. The log shows the script's path and hash, and at the end how many videos were processed, how many clips were made and how many videos were dropped or failed.

{% hint style="info" %}
Use **Save** at the top of the app to keep the whole pipeline as a preset. The preset keeps the path of the script, so the next run uses the latest version of the script in Team Files.
{% endhint %}
