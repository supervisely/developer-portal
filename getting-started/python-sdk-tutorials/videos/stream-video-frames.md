---
description: Stream and save video frames locally using real-time demuxing with PyAV — faster than downloading frames one-by-one via API.
---

# Stream video frames to directory

## Introduction

In this tutorial you will learn how to use `stream_video_frames_to_dir` to decode a video stored in Supervisely and save a range of frames to a local directory.

Unlike `api.video.frame.download_path` (which makes one HTTP request per frame), this function demuxes the video stream directly with **PyAV**, decodes only the requested frames, and writes them as image files — all in a single pass.

{% hint style="success" %}
Significantly faster than downloading frames one-by-one via API, especially for large frame ranges.
{% endhint %}

You will learn how to:

1. [Save a range of frames to a directory (synchronous).](#save-a-range-of-frames-sync)
2. [Save all frames of a video.](#save-all-frames)
3. [Use the async version inside an async context.](#async-version)
4. [Track progress with a progress bar.](#progress-bar)
5. [Compare performance against the standard API.](#performance-comparison)

## Prerequisites

{% hint style="warning" %}
This function requires the `video-av` extra. Install it before use:

```bash
pip install 'supervisely[video-av]'
```

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
video_id = 12345  # ⬅️ change value
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
For fine-grained control — e.g. processing each frame as it arrives — use the lower-level `async_stream_video_frames` generator:

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

## Performance comparison

Benchmark over 50 frames, averaged across 3 runs. Measured end-to-end including network I/O and decoding.

| Method                                  | Time    |
| --------------------------------------- | ------- |
| `api.video.frame.download_nps(...)`     | 20.72 s |
| `async_stream_video_frames(...)`        | 2.31 s  |
| `download_paths` in asyncio thread pool | 21.13 s |
| `async_stream_video_frames_to_dir(...)` | 3.6 s   |

As you can see, the streaming functions demonstrate a significant speedup compared to the standard API methods. This is because `download_nps` / `download_paths` make one HTTP request per frame to the Supervisely render endpoint. The streaming functions open the raw video stream once with PyAV and decode locally — no per-frame server round-trips.

## Function reference

<details>

<summary>stream_video_frames_to_dir</summary>

```python
stream_video_frames_to_dir(
    api,
    video_id,
    output_dir,
    start=None,   # first frame index (inclusive, 0-based); default: 0
    end=None,     # last frame index (inclusive, 0-based); default: last frame
    ext="png",    # image extension: "png", "jpg", etc.
    progress_cb=None,    # callable(1) called after each saved frame
    image_writer=None,   # custom writer fn(path, np_array); default: sly.image.write
)
```

Returns a `List[str]` of absolute paths to saved frame files.

</details>

<details>

<summary>async_stream_video_frames_to_dir</summary>

Same signature as `stream_video_frames_to_dir`, but `async`. Returns `List[str]`.

</details>

<details>

<summary>async_stream_video_frames</summary>

```python
async_stream_video_frames(
    api,
    video_id,
    start=None,
    end=None,
)
```

Async generator. Yields `(frame_index: int, image: np.ndarray)` tuples. Image is in **RGB** format.

</details>
