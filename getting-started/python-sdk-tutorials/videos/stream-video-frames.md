---
description: Stream and save video frames locally using real-time demuxing with PyAV — faster than downloading frames one-by-one via API.
---

# Stream video frames to directory

## Introduction

In this tutorial you will learn how to use `stream_video_frames_to_dir` to decode a video stored in Supervisely and save a range of frames to a local directory.

Unlike `api.video.frame.download_path` (which makes one HTTP request per frame), this function demuxes the video stream directly with **PyAV**, decodes only the requested frames, and writes them as image files — all in a single pass. This is significantly faster for large frame ranges.

You will learn how to:

1. [Save a range of frames to a directory (synchronous).](#save-a-range-of-frames-sync)
2. [Save all frames of a video.](#save-all-frames)
3. [Use the async version inside an async context.](#async-version)
4. [Track progress with a progress bar.](#progress-bar)

## Prerequisites

Install the `video-av` extra which pulls in PyAV:

```bash
pip install 'supervisely[video-av]'
```

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

## Save a range of frames (sync)

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

## Save all frames

Omit `start` and `end` to process the entire video:

```python
paths = stream_video_frames_to_dir(
    api=api,
    video_id=video_id,
    output_dir="/tmp/all_frames",
)
print(f"Total frames saved: {len(paths)}")
```

## Save as JPEG instead of PNG

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

For fine-grained control — e.g. processing each frame as it arrives — use the lower-level `async_stream_video_frames` generator:

```python
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

## Function reference

### `stream_video_frames_to_dir`

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

### `async_stream_video_frames_to_dir`

Same signature as above, but `async`. Returns `List[str]`.

### `async_stream_video_frames`

```python
async_stream_video_frames(
    api,
    video_id,
    start=None,
    end=None,
)
```

Async generator. Yields `(frame_index: int, image: np.ndarray)` tuples. The image is in **RGB** format.
