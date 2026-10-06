---
description: Run your own Python code on videos inside ML Pipelines and keep the frame ranges it selects
---

# ML Pipelines: Custom Code node

## Introduction

The **Custom Code** node of the [ML Pipelines](https://ecosystem.supervisely.com/apps/data-nodes) app runs a Python script you write on every video that reaches it. The script looks at the video (its frames, its annotation, its metadata) and returns the frame ranges to keep. Each range becomes a clip with its annotations, so one node can select, trim and split videos by any logic: OpenCV, a model, or code from your own packages.

A typical pipeline:

`Videos Project` → `Filter Videos by Tags` → `Custom Code` → `Create New Project`

This section has three pages:

- this page: how the node works, the script contract and its settings;
- [Examples](examples.md): short scripts for every step of a video clip pipeline;
- [Your own packages](own-packages.md): run the node with your own Python packages, including a private package index.

{% hint style="warning" %}
The node runs arbitrary code inside the app, so it is off unless the app's image enables it with the environment variable `ML_PIPELINES_CUSTOM_CODE=1`. The image of the ML Pipelines app in the Ecosystem does not set it. Build your own image with it, as [Your own packages](own-packages.md) shows. Without it the node is not listed, and a preset that contains it stops with an error.
{% endhint %}

## The script

The script is a `.py` file in Team Files. It defines one function:

```python
def process(video_path: str, video_info, ann, params: dict) -> list[tuple[int, int]]:
    ...
```

| Argument | What it is |
| --- | --- |
| `video_path` | Local path of the video file, downloaded for this node. |
| `video_info` | `VideoInfo` of the video: `name`, `id`, `frames_count`, `frames_to_timecodes`, `frame_width`, `frame_height`, ... |
| `ann` | The video's `sly.VideoAnnotation`: tags, objects, figures per frame. |
| `params` | The node's **Parameters** JSON as a `dict`. |

It returns inclusive `(start, end)` frame indexes, counted from `0`:

| Return | Result |
| --- | --- |
| `[(250, 624), (1000, 1249)]` | Two clips: frames 250 to 624 and frames 1000 to 1249. |
| `[]` | The video is dropped. |
| `[(0, video_info.frames_count - 1)]` | The video passes through unchanged. |

A minimal script:

```python
def process(video_path, video_info, ann, params):
    # keep the first 10 seconds of a 25 fps video
    return [(0, min(249, video_info.frames_count - 1))]
```

Per-frame logic simply lives inside `process`: open `video_path` with OpenCV, go over the frames and decide. See [Examples](examples.md).

### Clips

- Every clip is cut at exact frames and re-encoded to H.264 MP4, named `<video name>_frames_<start>-<end>.mp4`.
- Figures and frame-range tags are re-indexed to the clip: frame `start` becomes frame `0`. Video tags are kept.
- Connect an output node (`Create New Project`, `Add to Existing Project`) to save the clips. Datasets keep the names of the source datasets.

### Errors

If `process` raises an exception, or returns a range outside the video, that video is skipped. The error and the traceback are written to the app log, and the run continues with the other videos. A script that crashes its process (for example, in native code or out of memory) is isolated the same way. At the end of the run, the log has a summary: videos processed, clips, videos passed through, dropped and failed.

## Scripts in Team Files

The node keeps the **path** of the script in Team Files, never its text. A script is written once and then reused across sessions, presets and the whole team. A script uploaded or edited elsewhere (Team Files, API) is used as it is on the next run.

In the node's settings (**EDIT**):

- **Script**: pick a script of the team from `/ml-pipelines/custom-code/`, or **New from template** to start from an example.
- **Or any .py file in Team Files**: browse Team Files and **Open selected file**.
- **Code**: edit the script right in the node. **Save** writes it back to the same file. **Save as** creates a new file: a bare name goes to `/ml-pipelines/custom-code/`, a path starting with `/` is used as it is.
- Unsaved changes are saved when the pipeline starts, so every run uses a file in Team Files. A new script that was never saved gets a free name such as `/ml-pipelines/custom-code/script.py`.

Each run writes the script's path and the SHA-256 of the code that ran to the log:

```
Custom Code: running /ml-pipelines/custom-code/select_motion.py (sha256 012c061d...) with 8 workers
```

To manage scripts from Python, use the Team Files API:

```python
import supervisely as sly

api = sly.Api.from_env()
team_id = sly.env.team_id()
api.file.upload(team_id, "select_motion.py", "/ml-pipelines/custom-code/select_motion.py")
```

## Settings

| Setting | Description |
| --- | --- |
| **Script** | Path of the `.py` file in Team Files. |
| **Parameters** | JSON object passed to `process` as `params`. Use it for thresholds, sizes and seeds, so one script serves many pipelines. |
| **Workers** | How many videos run at the same time. `0`: one per CPU core of the agent, up to 8. |

The node's JSON in a preset:

```json
{
    "action": "custom_code",
    "src": ["$filter_videos_by_tag_2__true"],
    "dst": "$custom_code_3",
    "settings": {
        "script_path": "/ml-pipelines/custom-code/select_motion.py",
        "params": {"width": 320, "threshold": 5.0, "min_length": 25},
        "workers": 0
    }
}
```

## Parallel processing and where it runs

Each video runs in its own worker process, so several videos are processed at the same time on all cores of the agent. Each worker loads the SDK and your script once and is reused for the following videos. Plan about 350 MB of memory per worker plus what your script loads. Video decoding in OpenCV already uses several cores, so more workers do not always mean faster: try a few values on your own data.

Videos are downloaded only when a node reads the file. Videos removed earlier in the pipeline, for example by `Filter Videos by Tags`, are never downloaded.

The whole pipeline runs in one app session, on the agent you choose when you launch the app. To process videos on a dedicated machine, connect it as an agent ([Connect your computer](../../getting-started/connect-your-computer/README.md)) and launch ML Pipelines on it, for example on an on-demand worker in your cloud. A single run does not spread across several agents.
