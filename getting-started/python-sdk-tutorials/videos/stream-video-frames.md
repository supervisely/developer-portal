---
description: Decode video frames directly from a raw stream using PyAV — bypasses server-side metadata requirements.
---

# Stream video frames to directory

{% hint style="info" %}
Supervisely SDK version ≥ **v6.73.578**
{% endhint %}

## Introduction

Use `stream_video_frames_to_dir` to decode frames directly from the raw video stream using **PyAV** without requiring server-side metadata.

The standard `api.video.frame.download_path` requires video metadata (frame count, timestamps) to be pre-calculated on the server. If that metadata has not been computed yet, the call will fail. `stream_video_frames_to_dir` bypasses this limitation by opening the raw video stream directly with **PyAV** and demuxing it locally.

{% hint style="warning" %}
Requires the `video-av` extra:

```bash
pip install 'supervisely[video-av]'
```

{% endhint %}

{% hint style="info" %}
Performance depends on the video and the requested frame range. Use this function when server-side metadata is unavailable, not as a general-purpose speed optimization.
{% endhint %}

## How to debug this tutorial

**Step 1.** Prepare `~/supervisely.env` file with credentials. [Learn more here.](../../basics-of-authentication.md)

**Step 2.** Set the required environment variables in `local.env`:

```
SERVER_ADDRESS=https://app.supervisely.com  # ⬅️ change value
API_TOKEN=your-token-here                   # ⬅️ change value
VIDEO_ID=12345                              # ⬅️ change value
```

**Step 3.** Run the script below.

### Import libraries

```python
import os
from dotenv import load_dotenv
import supervisely as sly
from supervisely.video.sampling import stream_video_frames_to_dir
```

### Init API client

```python
if sly.is_development():
    load_dotenv("local.env")
    load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api.from_env()
```

## Save frames to directory

{% tabs %}
{% tab title="Frame range (PNG)" %}
Download frames 0–9 (first 10 frames) and save as PNG files:

```python
video_id = int(os.environ["VIDEO_ID"])
output_dir = "/tmp/frames"

paths = stream_video_frames_to_dir(
    api=api,
    video_id=video_id,
    output_dir=output_dir,
    start=0,
    end=9,
)

print(f"Saved {len(paths)} frames:")
for p in paths:
    print(p)
```

**Output:**

```
Saved 10 frames:
/tmp/frames/frame_000000.png
/tmp/frames/frame_000001.png
...
/tmp/frames/frame_000009.png
```

Files are named `frame_<index:06d>.<ext>`. The output directory is created automatically if it does not exist.
{% endtab %}

{% tab title="All frames" %}
Omit `start` and `end` to process the entire video:

```python
paths = stream_video_frames_to_dir(
    api=api,
    video_id=video_id,
    output_dir="/tmp/all_frames",
)
print(f"Total frames saved: {len(paths)}")
```

{% endtab %}

{% tab title="JPEG output" %}
Pass `ext="jpg"` to change the output format:

```python
paths = stream_video_frames_to_dir(
    api=api,
    video_id=video_id,
    output_dir="/tmp/frames_jpg",
    start=0,
    end=49,
    ext="jpg",
)
```

{% endtab %}
{% endtabs %}

## Progress bar

Pass a `progress_cb` callable — it is called with `1` after each frame is saved:

```python
with sly.tqdm_sly(total=50, message="Streaming frames") as pbar:
    paths = stream_video_frames_to_dir(
        api=api,
        video_id=video_id,
        output_dir="/tmp/frames",
        start=0,
        end=49,
        progress_cb=pbar.update,
    )
```

## Decoding progress

Pass `show_decoding_progress=True` to display low-level tqdm progress bars for the demuxing phase (building the PTS map) and, for B-frame videos, the full-decode phase. Both bars count packets and are independent of `progress_cb`:

```python
paths = stream_video_frames_to_dir(
    api=api,
    video_id=video_id,
    output_dir="/tmp/frames",
    start=0,
    end=49,
    show_decoding_progress=True,
)
```

## Tuning parallelism and memory

Two parameters control the internal pipeline:

- `max_write_workers` (default `4`) — number of threads writing frames to disk in parallel. Increase on fast NVMe storage.
- `queue_maxsize` (default `32`) — size of the decode prefetch buffer in frames. Peak RAM ≈ `(queue_maxsize + max_write_workers + 1) × frame_bytes`. Reduce if you hit OOM errors.

```python
paths = stream_video_frames_to_dir(
    api=api,
    video_id=video_id,
    output_dir="/tmp/frames",
    max_write_workers=8,  # more disk parallelism
    queue_maxsize=16,     # smaller decode buffer
)
```

## Async version

{% tabs %}
{% tab title="Save to directory" %}
Use `async_stream_video_frames_to_dir` inside an `async` function:

```python
import asyncio
from supervisely.video.sampling import async_stream_video_frames_to_dir

async def main():
    paths = await async_stream_video_frames_to_dir(
        api=api,
        video_id=video_id,
        output_dir="/tmp/frames_async",
        start=0,
        end=9,
    )
    print(f"Saved {len(paths)} frames")

asyncio.run(main())
```

{% endtab %}

{% tab title="Frame-by-frame generator" %}
For fine-grained control — process each frame as it arrives — use `async_stream_video_frames`:

```python
import asyncio
from supervisely.video.sampling import async_stream_video_frames

async def process_frames():
    async for frame_idx, img in async_stream_video_frames(
        api=api,
        video_id=video_id,
        start=0,
        end=9,
    ):
        print(f"Got frame {frame_idx}, shape: {img.shape}")
        # img is an RGB NumPy array (H, W, 3)

asyncio.run(process_frames())
```

{% endtab %}
{% endtabs %}

## Function reference

<details>

<summary>stream_video_frames_to_dir</summary>

```python
stream_video_frames_to_dir(
    api,
    video_id,
    output_dir,
    start=None,                # first frame index (inclusive, 0-based); default: 0
    end=None,                  # last frame index (inclusive, 0-based); default: last frame
    ext="png",                 # image extension: "png", "jpg", etc.
    progress_cb=None,          # callable(1) called after each saved frame
    image_writer=None,         # custom writer fn(path, np_array); default: sly.image.write
    max_write_workers=4,       # parallel write threads; increase on fast storage
    queue_maxsize=32,          # decode prefetch buffer in frames; peak RAM ≈ (queue_maxsize + max_write_workers + 1) × frame_bytes
    show_decoding_progress=False,  # show tqdm bars for demux / B-frame decode phases
)
```

Returns a `List[str]` of absolute paths to saved frame files.

</details>

<details>

<summary>async_stream_video_frames_to_dir</summary>

Same signature as `stream_video_frames_to_dir` (including `max_write_workers`, `queue_maxsize`, and `show_decoding_progress`), but `async`. Returns `List[str]`.

</details>

<details>

<summary>async_stream_video_frames</summary>

```python
async_stream_video_frames(
    api,
    video_id,
    start=None,                    # first frame index (inclusive, 0-based); default: 0
    end=None,                      # last frame index (inclusive, 0-based); default: last frame
    queue_maxsize=32,              # decode prefetch buffer in frames; peak RAM ≈ (queue_maxsize + 1) × frame_bytes
    show_decoding_progress=False,  # show tqdm bars for demux / B-frame decode phases
)
```

Async generator. Yields `(frame_index: int, image: np.ndarray)` tuples. Image is in **RGB** format.

</details>
