---
description: Fork ML Pipelines, add nodes with your own Python code and packages, and run them as a private app
---

# Add your own node to ML Pipelines

## Introduction

[ML Pipelines](https://ecosystem.supervisely.com/apps/data-nodes) builds data pipelines from nodes. When the nodes it ships with are not enough, for example because a step needs your own algorithm or your company's private Python packages, you can add your own nodes:

1. Fork the app's repository: [supervisely-ecosystem/data-nodes](https://github.com/supervisely-ecosystem/data-nodes).
2. Put your nodes in the `src/custom_nodes/` folder of the fork. They appear in the **Custom** group of the app.
3. Put your packages into the fork's Docker image.
4. Release the fork as a [private app](../basics/add-private-app.md) and run it on any agent.

Your code lives in your fork and runs in your app's container. Nothing is typed into the UI.

This tutorial builds one complete video pipeline, one node per step:

| Node | What it does |
| --- | --- |
| **Select Videos by Tag** | keeps the videos that have a tag |
| **Take Subset** | keeps a random share of them |
| **Scan Frames** | goes over all frames, resizes them and scores each one with OpenCV, several videos at a time |
| **Pick Frame Range** | picks the frame ranges with high scores |
| **Cut Clips** | cuts a clip for every range, with its annotation |

The full pipeline is `Videos Project → Select Videos by Tag → Take Subset → Scan Frames → Pick Frame Range → Cut Clips → Create New Project`. Copy the nodes and replace their logic with yours.

## How a node is made

A node is two classes:

* a **compute** class, a subclass of `Layer` from `src/compute/Layer.py`. It processes the data;
* a **UI** class, a subclass of `Action` from `src/ui/dtl/Action.py`. It draws the node and its settings.

Both are registered automatically when they are in `src/custom_nodes/`: you do not edit any file of the app. That is what keeps the fork easy to update (see [Keep the fork up to date](#keep-the-fork-up-to-date)).

Give every node its own folder. A folder needs an empty `__init__.py`; the file names inside are up to you:

```
src/custom_nodes/
├── __init__.py
├── select_videos_by_tag/
│   ├── __init__.py
│   ├── compute.py      # SelectVideosByTagLayer(Layer)
│   ├── ui.py           # SelectVideosByTagAction(VideoAction)
│   └── README.md       # shown when the user opens the node's documentation
├── take_subset/
├── scan_frames/
├── pick_frame_range/
└── cut_clips/
```

The two classes are linked by name: `Action.name` must be the same as `Layer.action`. Nodes are listed in the **Custom** group in the alphabetical order of their folders.

### The compute class

| Member | Meaning |
| --- | --- |
| `action` | the node's id, the same as `name` of the UI class |
| `layer_settings` | JSON schema of the node's settings; the app validates the settings against it before the run |
| `process(data_el)` | handles one item. Yields what goes on, see below |
| `process_batch(data_els)` + `has_batch_processing()` returning `True` | handles a list of items at once. Yields one list with everything that goes on |
| `requires_item()` | `True` if the node reads the media file. Default `False` |
| `modifies_data()` | `True` if the node changes the media or creates new files. Default `False` |
| `video_batch_size()` | how many videos the node wants per `process_batch` call. Default `1`, see [Run videos in parallel](#run-videos-in-parallel) |
| `self.settings` | the node's settings, as returned by the UI class |

An item is a pair `(vid_desc, ann)`:

* `vid_desc.item_data` is the local path of the video file. The video is downloaded the first time a node reads `item_data`, so videos that never reach such a node are never downloaded;
* `vid_desc.info.item_info` is the video's `VideoInfo`: `id`, `name`, `frames_count`, `frames_to_timecodes` and so on;
* `ann` is the video's `VideoAnnotation`.

What the node yields decides where the item goes:

* `(vid_desc, ann)` sends it to the node's only output;
* `(vid_desc, ann, i)` sends it to output number `i` (from 0) of a node with several outputs. To drop items, give the node a second output and leave it unconnected.

Every item must go to some output: a node must not yield nothing at all.

To pass a value to the next nodes, set it as an attribute of the descriptor: `vid_desc.frame_scores = scores`. The descriptor travels with the video through the pipeline.

### The UI class

| Member | Meaning |
| --- | --- |
| `name` | the node's id, the same as `action` of the compute class |
| `title`, `description` | shown in the sidebar and on the node |
| `md_description` | documentation of the node, usually read from the `README.md` next to it |
| `modalities` | `["videos"]`, `["images"]` or both. Without it, a `VideoAction` subclass is a video node and anything else is an image node |
| `create_new_layer()` | creates the node's settings widgets and returns a UI `Layer` |
| `create_outputs()` | the node's outputs. One output by default |

The parent class sets the node's color and icon: `VideoAction`, `FilterAndConditionAction`, `OtherAction` and so on. The settings are ordinary [widgets](../widgets/README.md). In `create_new_layer()` you write two functions:

* `get_settings(options_json)` returns the settings as a dict: this is `self.settings` of the compute class;
* `create_options(src, dst, settings)` puts the saved `settings` into the widgets (when a pipeline is loaded) and returns the widgets to show.

## Step 1. Fork and run locally

Fork [supervisely-ecosystem/data-nodes](https://github.com/supervisely-ecosystem/data-nodes) on GitHub, or copy it to your own git server, then:

```bash
git clone https://github.com/<your-account>/data-nodes.git
cd data-nodes
python3.12 -m venv .venv
source .venv/bin/activate
pip install -r dev_requirements.txt
```

Create `local.env` in the root of the repository:

```python
TEAM_ID=8                       # ⬅️ your team
WORKSPACE_ID=349                # ⬅️ your workspace
USER_ID=7                       # ⬅️ your user id
modal.state.modalityType=videos # the nodes in this tutorial are for videos
```

The app takes the server address and your token from `~/supervisely.env`, see [Basics of authentication](../../getting-started/basics-of-authentication.md). Start the app and open [http://localhost:8000](http://localhost:8000):

```bash
uvicorn src.main:app --host 0.0.0.0 --port 8000 --ws-ping-interval 30 --ws-ping-timeout 120
```

The app runs on your computer and works with the data on your instance. Restart it after you change a node. The repository also has a VS Code launch configuration, `Uvicorn`, if you prefer to debug.

{% hint style="info" %}
Clips are cut with `ffmpeg`, so install it locally to run the **Cut Clips** node outside the app's container.
{% endhint %}

## Step 2. Select Videos by Tag

The simplest node: no media, one setting, two outputs. Videos with the tag go to **Selected**, the rest to **Other**.

{% hint style="info" %}
ML Pipelines already has **Filter Videos by Tags**, with tag values and more options. This node is here as the smallest complete example.
{% endhint %}

`src/custom_nodes/select_videos_by_tag/compute.py`:

```python
from src.compute.Layer import Layer


class SelectVideosByTagLayer(Layer):
    action = "select_videos_by_tag"

    layer_settings = {
        "required": ["settings"],
        "properties": {
            "settings": {
                "type": "object",
                "required": ["tag_name"],
                "properties": {"tag_name": {"type": "string", "minLength": 1}},
            }
        },
    }

    def __init__(self, config, net):
        Layer.__init__(self, config, net=net)

    def process(self, data_el):
        vid_desc, ann = data_el
        tag_names = {tag.name for tag in ann.tags if tag.frame_range is None}
        if self.settings["tag_name"] in tag_names:
            yield vid_desc, ann, 0  # first output: "Selected"
        else:
            yield vid_desc, ann, 1  # second output: "Other"
```

`src/custom_nodes/select_videos_by_tag/ui.py`:

```python
from os.path import dirname, realpath
from typing import Optional

from supervisely.app.widgets import Field, Input, NodesFlow

from src.ui.dtl.Action import VideoAction
from src.ui.dtl.Layer import Layer
from src.ui.dtl.utils import get_layer_docs


class SelectVideosByTagAction(VideoAction):
    name = "select_videos_by_tag"  # the same as `action` of the compute class
    title = "Select Videos by Tag"
    docs_url = ""
    description = "Videos with the tag go to 'Selected', the rest to 'Other'."
    md_description = get_layer_docs(dirname(realpath(__file__)))

    @classmethod
    def create_new_layer(cls, layer_id: Optional[str] = None):
        tag_input = Input(placeholder="for example: daytime", size="small")
        tag_field = Field(
            title="Tag name",
            description="A video tag. Its value does not matter.",
            content=tag_input,
        )

        def get_settings(options_json: dict) -> dict:
            return {"tag_name": tag_input.get_value().strip()}

        def create_options(src: list, dst: list, settings: dict) -> dict:
            tag_input.set_value(settings.get("tag_name", ""))
            return {
                "src": [],
                "dst": [],
                "settings": [
                    NodesFlow.Node.Option(
                        name="tag_name",
                        option_component=NodesFlow.WidgetOptionComponent(tag_field),
                    )
                ],
            }

        return Layer(
            action=cls,
            id=layer_id,
            create_options=create_options,
            get_settings=get_settings,
            need_preview=False,
        )

    @classmethod
    def create_outputs(cls):
        return [
            NodesFlow.Node.Output("destination_selected", "Selected"),
            NodesFlow.Node.Output("destination_other", "Other"),
        ]
```

Add an empty `src/custom_nodes/select_videos_by_tag/__init__.py` and, optionally, a `README.md`. Restart the app: **Select Videos by Tag** is in the **Custom** group.

## Step 3. Take Subset

Keeps about the given percent of the videos. Which videos are kept depends only on the video and the seed, not on the order of the videos, so a rerun keeps the same ones.

`src/custom_nodes/take_subset/compute.py`:

```python
import hashlib

from src.compute.Layer import Layer


class TakeSubsetLayer(Layer):
    action = "take_subset"

    layer_settings = {
        "required": ["settings"],
        "properties": {
            "settings": {
                "type": "object",
                "required": ["percent", "seed"],
                "properties": {
                    "percent": {"type": "number", "minimum": 0, "maximum": 100},
                    "seed": {"type": "integer"},
                },
            }
        },
    }

    def __init__(self, config, net):
        Layer.__init__(self, config, net=net)

    def process(self, data_el):
        vid_desc, ann = data_el
        # A stable random number in [0, 100) per video: the result does not depend on batches,
        # on the order of the videos or on the run.
        key = f"{self.settings['seed']}:{vid_desc.info.item_info.id}".encode()
        value = int(hashlib.sha256(key).hexdigest(), 16) % 10000 / 100
        if value < self.settings["percent"]:
            yield vid_desc, ann, 0  # "Subset"
        else:
            yield vid_desc, ann, 1  # "Rest"
```

`src/custom_nodes/take_subset/ui.py`:

```python
from os.path import dirname, realpath
from typing import Optional

from supervisely.app.widgets import Container, Field, InputNumber, NodesFlow

from src.ui.dtl.Action import VideoAction
from src.ui.dtl.Layer import Layer
from src.ui.dtl.utils import get_layer_docs


class TakeSubsetAction(VideoAction):
    name = "take_subset"
    title = "Take Subset"
    docs_url = ""
    description = "Keep a random share of the videos. The same seed keeps the same videos."
    md_description = get_layer_docs(dirname(realpath(__file__)))

    @classmethod
    def create_new_layer(cls, layer_id: Optional[str] = None):
        percent_input = InputNumber(value=50, min=0, max=100, step=1, size="small")
        seed_input = InputNumber(value=0, min=0, step=1, size="small")
        settings_container = Container(
            widgets=[
                Field(title="Percent of videos", content=percent_input),
                Field(title="Seed", content=seed_input),
            ]
        )

        def get_settings(options_json: dict) -> dict:
            return {"percent": percent_input.get_value(), "seed": int(seed_input.get_value())}

        def create_options(src: list, dst: list, settings: dict) -> dict:
            percent_input.value = settings.get("percent", 50)
            seed_input.value = settings.get("seed", 0)
            return {
                "src": [],
                "dst": [],
                "settings": [
                    NodesFlow.Node.Option(
                        name="settings",
                        option_component=NodesFlow.WidgetOptionComponent(settings_container),
                    )
                ],
            }

        return Layer(
            action=cls,
            id=layer_id,
            create_options=create_options,
            get_settings=get_settings,
            need_preview=False,
        )

    @classmethod
    def create_outputs(cls):
        return [
            NodesFlow.Node.Output("destination_subset", "Subset"),
            NodesFlow.Node.Output("destination_rest", "Rest"),
        ]
```

## Step 4. Scan Frames: go over all frames, in parallel

This node reads every frame of every video, resizes it and scores it with OpenCV. The score here is motion: the mean difference from the previous frame. The scores are passed to the next node as `vid_desc.frame_scores`.

Reading frames is slow, so the node processes several videos at the same time, one per worker process. Keep the per-video function in its own module that imports only what it needs: every worker process imports it.

`src/custom_nodes/scan_frames/scoring.py`:

```python
"""The per-video work. It runs in worker processes, so keep its imports to what it needs."""

import cv2


def score_frames(video_path: str, width: int) -> list:
    """Go over all frames, resize each to `width` and compute its motion score: the mean absolute
    difference from the previous frame (0 for the first frame)."""
    cap = cv2.VideoCapture(video_path)
    if not cap.isOpened():
        raise RuntimeError(f"Cannot open video {video_path}")
    scores = []
    prev = None
    try:
        while True:
            ok, frame = cap.read()
            if not ok:
                break
            h, w = frame.shape[:2]
            frame = cv2.resize(frame, (width, max(1, round(h * width / w))), interpolation=cv2.INTER_AREA)
            gray = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)
            scores.append(0.0 if prev is None else float(cv2.absdiff(gray, prev).mean()))
            prev = gray
    finally:
        cap.release()
    return scores
```

`src/custom_nodes/scan_frames/compute.py`:

```python
import multiprocessing
import os
from concurrent.futures import ProcessPoolExecutor, ThreadPoolExecutor

from supervisely import logger

from src.compute.Layer import Layer
from src.custom_nodes.scan_frames.scoring import score_frames


class ScanFramesLayer(Layer):
    action = "scan_frames"

    layer_settings = {
        "required": ["settings"],
        "properties": {
            "settings": {
                "type": "object",
                "required": ["width", "workers"],
                "properties": {
                    "width": {"type": "integer", "minimum": 16},
                    "workers": {"type": "integer", "minimum": 0},
                },
            }
        },
    }

    def __init__(self, config, net):
        Layer.__init__(self, config, net=net)

    def requires_item(self):
        return True  # the node reads the video files

    def workers(self) -> int:
        workers = self.settings["workers"]
        if workers == 0:
            workers = min(os.cpu_count() or 1, 8)
        return workers

    def video_batch_size(self) -> int:
        return self.workers()  # one video per worker process

    def has_batch_processing(self):
        return True

    def process_batch(self, data_els):
        # Reading item_data downloads the video, so download the whole batch in parallel first.
        with ThreadPoolExecutor(len(data_els)) as pool:
            paths = list(pool.map(lambda data_el: data_el[0].item_data, data_els))

        width = self.settings["width"]
        outputs = []
        ctx = multiprocessing.get_context("spawn")
        with ProcessPoolExecutor(min(self.workers(), len(data_els)), mp_context=ctx) as pool:
            futures = [pool.submit(score_frames, path, width) for path in paths]
            for (vid_desc, ann), future in zip(data_els, futures):
                try:
                    # Any attribute set on the descriptor travels with the video to the next nodes.
                    vid_desc.frame_scores = future.result()
                    outputs.append((vid_desc, ann, 0))  # "Scanned"
                except Exception as e:
                    logger.warning(f"Scan Frames failed on {vid_desc.get_item_name()}: {e}")
                    outputs.append((vid_desc, ann, 1))  # "Failed"
        yield outputs
```

What matters here:

* `requires_item()` returns `True`: the node reads the files;
* `video_batch_size()` asks for as many videos per batch as there are workers;
* reading `item_data` downloads the video, so the node first reads it for the whole batch in a thread pool: the downloads run at the same time;
* the worker processes use the `spawn` start method. The app runs threads, and `fork` in a process with threads can hang;
* each video's error is caught separately: a broken video goes to the **Failed** output, the others go on.

`src/custom_nodes/scan_frames/ui.py`:

```python
from os.path import dirname, realpath
from typing import Optional

from supervisely.app.widgets import Container, Field, InputNumber, NodesFlow

from src.ui.dtl.Action import VideoAction
from src.ui.dtl.Layer import Layer
from src.ui.dtl.utils import get_layer_docs


class ScanFramesAction(VideoAction):
    name = "scan_frames"
    title = "Scan Frames"
    docs_url = ""
    description = "Resize every frame and score it with OpenCV. Videos run in parallel."
    md_description = get_layer_docs(dirname(realpath(__file__)))

    @classmethod
    def create_new_layer(cls, layer_id: Optional[str] = None):
        width_input = InputNumber(value=320, min=16, step=16, size="small")
        workers_input = InputNumber(value=0, min=0, step=1, size="small")
        settings_container = Container(
            widgets=[
                Field(title="Resize to width", content=width_input),
                Field(
                    title="Parallel videos",
                    description="0: one per CPU core, at most 8.",
                    content=workers_input,
                ),
            ]
        )

        def get_settings(options_json: dict) -> dict:
            return {"width": int(width_input.get_value()), "workers": int(workers_input.get_value())}

        def create_options(src: list, dst: list, settings: dict) -> dict:
            width_input.value = settings.get("width", 320)
            workers_input.value = settings.get("workers", 0)
            return {
                "src": [],
                "dst": [],
                "settings": [
                    NodesFlow.Node.Option(
                        name="settings",
                        option_component=NodesFlow.WidgetOptionComponent(settings_container),
                    )
                ],
            }

        return Layer(
            action=cls,
            id=layer_id,
            create_options=create_options,
            get_settings=get_settings,
            need_preview=False,
        )

    @classmethod
    def create_outputs(cls):
        return [
            NodesFlow.Node.Output("destination_scanned", "Scanned"),
            NodesFlow.Node.Output("destination_failed", "Failed"),
        ]
```

## Step 5. Pick Frame Range

Uses the scores from **Scan Frames**: keeps the runs of frames with a score at or above the threshold that are at least the shortest length, widened by the padding. The ranges are inclusive `(start, end)` frame indexes, passed on as `vid_desc.frame_ranges`.

`src/custom_nodes/pick_frame_range/compute.py`:

```python
from src.compute.Layer import Layer
from src.exceptions import GraphError


def find_ranges(scores: list, threshold: float, min_length: int, padding: int) -> list:
    """Inclusive (start, end) ranges of frames whose score is at least `threshold`, at least
    `min_length` frames long, widened by `padding` frames on each side."""
    runs = []
    start = None
    for idx, score in enumerate(scores + [float("-inf")]):
        if score >= threshold and start is None:
            start = idx
        elif score < threshold and start is not None:
            if idx - start >= min_length:
                runs.append([max(0, start - padding), min(len(scores) - 1, idx - 1 + padding)])
            start = None
    merged = []
    for run in runs:
        if merged and run[0] <= merged[-1][1] + 1:
            merged[-1][1] = max(merged[-1][1], run[1])
        else:
            merged.append(run)
    return [tuple(run) for run in merged]


class PickFrameRangeLayer(Layer):
    action = "pick_frame_range"

    layer_settings = {
        "required": ["settings"],
        "properties": {
            "settings": {
                "type": "object",
                "required": ["threshold", "min_length", "padding"],
                "properties": {
                    "threshold": {"type": "number"},
                    "min_length": {"type": "integer", "minimum": 1},
                    "padding": {"type": "integer", "minimum": 0},
                },
            }
        },
    }

    def __init__(self, config, net):
        Layer.__init__(self, config, net=net)

    def process(self, data_el):
        vid_desc, ann = data_el
        scores = getattr(vid_desc, "frame_scores", None)
        if scores is None:
            raise GraphError("Pick Frame Range needs the scores of Scan Frames: connect it after that node.")
        # The scores come from decoding the file; never go past the frames the video has.
        scores = scores[: vid_desc.info.item_info.frames_count]
        s = self.settings
        vid_desc.frame_ranges = find_ranges(scores, s["threshold"], s["min_length"], s["padding"])
        if vid_desc.frame_ranges:
            yield vid_desc, ann, 0  # "Picked"
        else:
            yield vid_desc, ann, 1  # "Nothing picked"
```

`src/custom_nodes/pick_frame_range/ui.py`:

```python
from os.path import dirname, realpath
from typing import Optional

from supervisely.app.widgets import Container, Field, InputNumber, NodesFlow

from src.ui.dtl.Action import VideoAction
from src.ui.dtl.Layer import Layer
from src.ui.dtl.utils import get_layer_docs


class PickFrameRangeAction(VideoAction):
    name = "pick_frame_range"
    title = "Pick Frame Range"
    docs_url = ""
    description = "Pick the frame ranges whose score is above a threshold."
    md_description = get_layer_docs(dirname(realpath(__file__)))

    @classmethod
    def create_new_layer(cls, layer_id: Optional[str] = None):
        threshold_input = InputNumber(value=5, min=0, step=0.5, size="small")
        min_length_input = InputNumber(value=25, min=1, step=1, size="small")
        padding_input = InputNumber(value=0, min=0, step=1, size="small")
        settings_container = Container(
            widgets=[
                Field(title="Score threshold", content=threshold_input),
                Field(title="Shortest range, frames", content=min_length_input),
                Field(title="Padding, frames", content=padding_input),
            ]
        )

        def get_settings(options_json: dict) -> dict:
            return {
                "threshold": float(threshold_input.get_value()),
                "min_length": int(min_length_input.get_value()),
                "padding": int(padding_input.get_value()),
            }

        def create_options(src: list, dst: list, settings: dict) -> dict:
            threshold_input.value = settings.get("threshold", 5)
            min_length_input.value = settings.get("min_length", 25)
            padding_input.value = settings.get("padding", 0)
            return {
                "src": [],
                "dst": [],
                "settings": [
                    NodesFlow.Node.Option(
                        name="settings",
                        option_component=NodesFlow.WidgetOptionComponent(settings_container),
                    )
                ],
            }

        return Layer(
            action=cls,
            id=layer_id,
            create_options=create_options,
            get_settings=get_settings,
            need_preview=False,
        )

    @classmethod
    def create_outputs(cls):
        return [
            NodesFlow.Node.Output("destination_picked", "Picked"),
            NodesFlow.Node.Output("destination_nothing", "Nothing picked"),
        ]
```

## Step 6. Cut Clips

Cuts a clip for every frame range with the app's helper, `cut_clips` from `src/compute/utils/video_clips.py`:

```python
cut_clips(video_path, video_info, ann, ranges, result_dir=None, threads=0) -> List[Clip]
```

* `ranges` are inclusive `(start, end)` frame ranges. Each gives one clip, named `<video name>_<n>.mp4`;
* the clips are re-encoded (H.264), so the cut is frame exact, whatever the keyframes are;
* each `Clip` has `path`, `name`, `info` (its `VideoInfo`) and `ann`: the figures and the frame range tags inside the range, re-indexed so that the clip's first frame is frame 0. Video tags are kept.

`clip_to_item(vid_desc, clip)` turns a clip into an item for the next nodes. Connect **Clips** to **Create New Project** or **Add to Existing Project** to save them: the clips go to a dataset with the same name as the dataset of their video.

`src/custom_nodes/cut_clips/compute.py`:

```python
import os
from concurrent.futures import ThreadPoolExecutor

from src.compute.Layer import Layer
from src.compute.utils.video_clips import clip_to_item, cut_clips
from src.exceptions import GraphError


class CutClipsLayer(Layer):
    action = "cut_clips"

    layer_settings = {"required": ["settings"], "properties": {"settings": {}}}

    def __init__(self, config, net):
        Layer.__init__(self, config, net=net)

    def requires_item(self):
        return True

    def modifies_data(self):
        return True  # the node outputs new video files

    def has_batch_processing(self):
        return True

    def _cut(self, data_el, threads: int) -> list:
        vid_desc, ann = data_el
        ranges = getattr(vid_desc, "frame_ranges", None)
        if ranges is None:
            raise GraphError("Cut Clips needs frame ranges: connect it after Pick Frame Range.")
        if not ranges:
            return [(vid_desc, ann, 1)]  # "No clips"
        clips = cut_clips(vid_desc.item_data, vid_desc.info.item_info, ann, ranges, threads=threads)
        return [clip_to_item(vid_desc, clip) + (0,) for clip in clips]  # "Clips"

    def process_batch(self, data_els):
        # ffmpeg runs as a separate process, so threads are enough to cut several videos at once.
        workers = min(len(data_els), os.cpu_count() or 1)
        threads = max(1, (os.cpu_count() or 1) // workers)
        with ThreadPoolExecutor(workers) as pool:
            results = list(pool.map(lambda data_el: self._cut(data_el, threads), data_els))
        yield [output for outputs in results for output in outputs]
```

`ffmpeg` runs as its own process, so threads are enough to cut several videos at the same time. `threads` splits the CPU cores between the videos.

`src/custom_nodes/cut_clips/ui.py`:

```python
from os.path import dirname, realpath
from typing import Optional

from supervisely.app.widgets import NodesFlow

from src.ui.dtl.Action import VideoAction
from src.ui.dtl.Layer import Layer
from src.ui.dtl.utils import get_layer_docs


class CutClipsAction(VideoAction):
    name = "cut_clips"
    title = "Cut Clips"
    docs_url = ""
    description = "Cut one clip per picked frame range, with its annotation."
    md_description = get_layer_docs(dirname(realpath(__file__)))

    @classmethod
    def create_new_layer(cls, layer_id: Optional[str] = None):
        def get_settings(options_json: dict) -> dict:
            return {}

        def create_options(src: list, dst: list, settings: dict) -> dict:
            return {"src": [], "dst": [], "settings": []}

        return Layer(
            action=cls,
            id=layer_id,
            create_options=create_options,
            get_settings=get_settings,
            need_preview=False,
        )

    @classmethod
    def create_outputs(cls):
        return [
            NodesFlow.Node.Output("destination_clips", "Clips"),
            NodesFlow.Node.Output("destination_no_clips", "No clips"),
        ]
```

## Step 7. The full pipeline

Connect the nodes in the app: `Videos Project → Select Videos by Tag (Selected) → Take Subset (Subset) → Scan Frames (Scanned) → Pick Frame Range (Picked) → Cut Clips (Clips) → Create New Project`. Leave the other outputs unconnected.

You can also load it from a file. Save this as `/data-nodes/presets/videos/clips.json` in Team Files, replace `my-videos` with the name of your videos project and `tagged` with your tag, and open it with **LOAD** in the app:

```json
[
  {
    "action": "videos_project",
    "src": [
      "my-videos/*"
    ],
    "dst": "$vp_1",
    "settings": {
      "classes_mapping": "default",
      "tags_mapping": "default"
    }
  },
  {
    "action": "select_videos_by_tag",
    "src": [
      "$vp_1"
    ],
    "dst": [
      "$sel_2__selected",
      "$sel_2__other"
    ],
    "settings": {
      "tag_name": "tagged"
    }
  },
  {
    "action": "take_subset",
    "src": [
      "$sel_2__selected"
    ],
    "dst": [
      "$sub_3__subset",
      "$sub_3__rest"
    ],
    "settings": {
      "percent": 70,
      "seed": 0
    }
  },
  {
    "action": "scan_frames",
    "src": [
      "$sub_3__subset"
    ],
    "dst": [
      "$scan_4__scanned",
      "$scan_4__failed"
    ],
    "settings": {
      "width": 320,
      "workers": 0
    }
  },
  {
    "action": "pick_frame_range",
    "src": [
      "$scan_4__scanned"
    ],
    "dst": [
      "$pick_5__picked",
      "$pick_5__nothing"
    ],
    "settings": {
      "threshold": 5,
      "min_length": 25,
      "padding": 0
    }
  },
  {
    "action": "cut_clips",
    "src": [
      "$pick_5__picked"
    ],
    "dst": [
      "$cut_6__clips",
      "$cut_6__none"
    ],
    "settings": {}
  },
  {
    "action": "create_new_project",
    "src": [
      "$cut_6__clips"
    ],
    "dst": "clips",
    "settings": {
      "project_name": "clips"
    }
  }
]
```

The result is a new project with the clips. Every clip keeps its annotation, re-indexed to the clip.

## Run videos in parallel

* Videos go through the pipeline in batches. A batch has 1 video unless a node asks for more with `video_batch_size()`; the pipeline uses the largest value among its nodes. A pipeline that does not read or change any media file uses large batches anyway.
* Batches are made per dataset, before any node runs. If a filter before your node drops most of the videos, your node gets smaller batches.
* Use processes (`ProcessPoolExecutor`) for Python and OpenCV work, threads for downloads and for external programs like `ffmpeg`.
* Every video in a batch is on disk at the same time. Choose the batch size with the agent's disk and memory in mind.
* Nodes without `process_batch` get the items of a batch one by one.

## Step 8. Your own packages

Install your packages into the fork's Docker image. Start from the app's image, `supervisely/data-nodes:<version>`: take the version from `docker_image` in the app's `config.json`. Its Python is 3.12.

For example, if a private package `acme-video-signals` gives the frame score, `scoring.py` becomes:

```python
import cv2
from acme_video_signals import frame_score  # from your private package

...
            scores.append(0.0 if prev is None else frame_score(prev, gray))
```

Add `docker/custom.Dockerfile` to the fork:

```dockerfile
FROM supervisely/data-nodes:6.74.7

# Install your packages from your private index. The index URL, with its token, is passed as a
# build secret: it is used during the build only and is not stored in the image.
RUN --mount=type=secret,id=pip_index_url \
    /opt/venv/bin/python -m ensurepip && \
    /opt/venv/bin/python -m pip install --no-cache-dir \
        --index-url "$(cat /run/secrets/pip_index_url)" \
        "acme-video-signals==0.1.0" \
        "numpy<2"
```

* The app's packages are in `/opt/venv`. The image has no `pip` there, so `ensurepip` adds it first.
* Keep `numpy<2` in the same command: the app uses `imgaug`, which does not work with NumPy 2. Without the pin, one of your packages can upgrade NumPy and break the app.
* Your packages must support Python 3.12.

Build the image with your private index. For example, with AWS CodeArtifact (a repository with PyPI as its upstream also serves public packages):

```bash
export CODEARTIFACT_AUTH_TOKEN=$(aws codeartifact get-authorization-token \
    --domain my-domain --domain-owner 111122223333 --query authorizationToken --output text)
export PIP_INDEX_URL="https://aws:${CODEARTIFACT_AUTH_TOKEN}@my-domain-111122223333.d.codeartifact.eu-west-1.amazonaws.com/pypi/my-repo/simple/"

docker build --secret id=pip_index_url,env=PIP_INDEX_URL \
    -f docker/custom.Dockerfile -t registry.example.com/ml-pipelines-custom:1.0.0 .
docker push registry.example.com/ml-pipelines-custom:1.0.0
```

The token is used only by the build: it is not in the image, its layers or its history. The agents that run the app need access to your registry: run `docker login registry.example.com` on their machines.

Then set the image in the fork's `config.json`, and give the app its own name:

```json
{
  "name": "ML Pipelines (custom)",
  "docker_image": "registry.example.com/ml-pipelines-custom:1.0.0",
  ...
}
```

To try the image before you release, run the app in it locally against your instance, with the fork mounted into the container.

## Step 9. Release the fork as a private app

Release the fork with the Supervisely CLI from the root of the repository, as described in [Add private app](../basics/add-private-app.md):

```bash
supervisely release
```

The app appears in the **private** apps of the Ecosystem. Run it like ML Pipelines, on any agent: for example a separate machine or an on-demand worker for the heavy video work. More about `config.json` is in [config.json](../basics/app-json-config/config.json.md).

## Keep the fork up to date

Your nodes are new files in `src/custom_nodes/`, so taking our updates is a clean rebase:

```bash
git remote add upstream https://github.com/supervisely-ecosystem/data-nodes.git
git fetch upstream
git rebase upstream/master
```

The only file you share with us is `config.json`: when we change its `docker_image`, keep yours, and change the `FROM` line of your Dockerfile to our new version.
