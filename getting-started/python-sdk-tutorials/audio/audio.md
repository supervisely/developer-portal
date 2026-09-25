---
description: >-
  Create Audio projects, upload recordings, label time segments, configure the
  project spectrogram and export spectrograms for training with the Python SDK.
---

# Audio

## Introduction

An **Audio project** holds recordings that are labeled with **time segments** and **recording tags**. A segment is a tag applied to a range of samples of one recording, optionally about one channel; a recording tag labels the whole recording. The labeling tool shows each recording as a waveform and a spectrogram, and the spectrogram is computed with settings that belong to the **project**: every recording in the project is analysed the same way, so every annotator is looking at the same picture and the picture can be reproduced later for training.

This tutorial covers:

* creating an Audio project and configuring its spectrogram
* uploading recordings
* adding, reading, editing and removing segments and recording tags
* rendering spectrograms of whole recordings and of labeled segments
* exporting training crops for a whole project
* downloading and uploading a project in Supervisely format

{% hint style="info" %}
Audio projects need a Supervisely instance with Audio support and a Supervisely Python SDK version that has `sly.ProjectType.AUDIO`.
{% endhint %}

## Prerequisites

**Step 1.** Prepare `~/supervisely.env` file with credentials. [Learn more here.](../../basics-of-authentication.md)

**Step 2.** Install the SDK.

```bash
pip install supervisely
```

Uncompressed WAV is decoded with the Python standard library. To decode FLAC, OGG, MP3 or M4A locally, install the optional decoders as well:

```bash
pip install "supervisely[audio]"
```

They are imported only when such a file is decoded. Uploading, importing and downloading never decode audio, so they work without them.

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

## Create a project and configure the spectrogram

```python
workspace_id = 123

project = api.project.create(workspace_id, "Engine noise", type=sly.ProjectType.AUDIO)
dataset = api.dataset.create(project.id, "bench-run-1")
```

The spectrogram is a transform of the audio, and the settings decide what is visible: two tones 40 Hz apart are one line at `fft_size=256` and two lines at `fft_size=1024`; an event 70 dB below full scale disappears at `min_db=-60`. Choose them once, before labeling starts.

```python
settings = sly.SpectrogramSettings(
    scale="mel",            # linear | log | mel
    fft_size=1024,          # 32, 64, ..., 32768
    hop_length=256,         # samples between frames, >= 1
    window="hann",          # hann | hamming | blackman
    mel_bands=64,           # 2..512, used by the mel scale
    min_db=-100.0,
    max_db=0.0,             # must be greater than min_db
    colormap="magma",       # viridis | magma | grayscale
    interpolation="sharp",  # sharp | smooth
)
api.audio.set_spectrogram_settings(project.id, settings)

api.audio.get_spectrogram_settings(project.id)
```

A project that was never configured returns the platform defaults, which are what the labeling tool shows in that case. Values outside the ranges above raise `ValueError` before any request is sent.

Changing the settings is a project-wide change that requires permission to edit the project, not only to label in it. Existing labels are not changed, but they were drawn on a picture that no longer matches — which is why the settings are meant to be set once.

`settings.fingerprint` is a short hash over the fields that change the numbers (`colormap` and `interpolation` only change the colours). Two projects with the same fingerprint analyse audio identically, so their labels can be merged or used for training together.

```python
settings.fingerprint   # 'sha256:...'
```

## Upload recordings

```python
recording = api.audio.upload_path(dataset.id, "bench-01.wav", "/data/bench-01.wav")
print(recording.id, recording.name, recording.file_meta)
# 7133022 bench-01.wav {'mime': 'audio/vnd.wave', 'size': 192044}
```

`api.audio.upload_paths(dataset_id, names, paths)` uploads several files at once. Files already on the server are recognised by hash and not sent again, so re-running an interrupted upload transfers only what is missing.

The platform does not store the sample rate, duration or number of channels of a recording. Read them from the file when you need them:

```python
info = sly.audio.get_audio_info("/data/bench-01.wav")
print(info)
# AudioFileInfo(sample_rate=16000, sample_count=48000, channels=2, duration=3.000s)
```

## Label segments

A segment is a tag, so create the tags first:

```python
meta = sly.ProjectMeta.from_json(api.project.get_meta(project.id))
meta = meta.add_tag_metas([
    sly.TagMeta("knock", sly.TagValueType.NONE),
    sly.TagMeta("source", sly.TagValueType.ONEOF_STRING, possible_values=["engine", "gearbox"]),
    sly.TagMeta("scene", sly.TagValueType.ANY_STRING),
])
api.project.update_meta(project.id, meta)

meta = sly.ProjectMeta.from_json(api.project.get_meta(project.id))
knock = meta.get_tag_meta("knock")
source = meta.get_tag_meta("source")
scene = meta.get_tag_meta("scene")
```

The start and end of a segment are **zero-based sample indices, both inclusive**, into the original recording — not milliseconds and not frames. At 16 kHz, `start=16000, end=23999` is the half-second from 1.0 s to 1.5 s. `channel` is the zero-based channel the label is about, or `None` for all channels.

```python
segments = [
    sly.AudioSegment(tag_id=knock.sly_id, start=16000, end=23999, channel=1),
    # seconds are converted to samples; end_sec is exclusive
    sly.AudioSegment.from_seconds(
        source.sly_id, start_sec=0.0, end_sec=3.0, sample_rate=info.sample_rate, value="engine"
    ),
]
api.audio.add_segments(project.id, recording.id, segments)
```

A recording tag — "Entire recording" in the labeling tool — has no range and no channel. A tag can be on a recording only once:

```python
api.audio.add_recording_tag(
    project.id, recording.id, sly.AudioRecordingTag(tag_id=scene.sly_id, value="test bench")
)

# or both kinds in one request
api.audio.add_tags(project.id, recording.id, segments=segments, recording_tags=[...])
```

## Read, edit and remove labels

```python
for s in api.audio.get_segments(recording.id):
    print(s.id, s.tag_id, s.start, s.end, s.channel, s.value, s.duration_seconds(info.sample_rate))
# 2122881 53651 16000 23999 1 None 0.5
# 2122882 53652 0 47999 None engine 3.0
```

```python
for t in api.audio.get_recording_tags(recording.id):
    print(t.id, t.tag_id, t.value)
# 2122883 53653 test bench

segments, recording_tags = api.audio.get_tags(recording.id)  # both in one request
```

To edit or remove a label, pass it as it was read back — it carries the ids the platform needs. An edit replaces the label's `meta` whole, so change the object you read rather than building a new one:

```python
segment = api.audio.get_segments(recording.id)[0]
segment.start, segment.end = 17000, 23999
api.audio.update_segment(segment)

tag = api.audio.get_recording_tags(recording.id)[0]
tag.value = "road"
api.audio.update_recording_tag(tag)

api.audio.remove_segment(segment)
api.audio.remove_recording_tag(tag)
```

`api.audio.get_list(dataset_id)` lists the recordings of a dataset, each with its tags.

## Render a spectrogram

Download the recording, decode it, and render it under the project's settings:

```python
api.audio.download_path(recording.id, "/tmp/bench-01.wav")
samples, rate = sly.audio.read_audio("/tmp/bench-01.wav")   # float32, shape (samples, channels)

settings = api.audio.get_spectrogram_settings(project.id)
spec = sly.audio.render_spectrogram(samples, rate, settings)  # dB, shape (rows, frames)

image = sly.audio.to_image(spec, settings)                    # RGB, as the tool paints it
sly.image.write("/tmp/bench-01.png", image)
```

Rows are mel bands for the mel scale and FFT bins otherwise, lowest frequency first; `to_image` puts the lowest frequency at the bottom, as in the labeling tool. Decibels are absolute (`10*log10(power)`, a full-scale sine reads 0 dB) and clipped to `[min_db, max_db]`; pass `as_db=False` for raw power. Multichannel audio is averaged into a mixdown unless you pass `channel=`.

To reproduce the tool's on-screen grid rather than the natural resolution, pass the number of display rows:

```python
spec = sly.audio.render_spectrogram(samples, rate, settings, rows=512)
```

## Export training crops

`render_segment` renders just the labeled range, on the segment's channel:

```python
segment = api.audio.get_segments(recording.id)[0]
crop = sly.audio.render_segment(
    samples, rate, segment.start, segment.end, settings, channel=segment.channel
)
```

The same for every segment in a project, saved as NumPy arrays named after the recording, the label and its tag:

```python
import numpy as np

meta = sly.ProjectMeta.from_json(api.project.get_meta(project.id))
tag_names = {tag.sly_id: tag.name for tag in meta.tag_metas}
settings = api.audio.get_spectrogram_settings(project.id)
os.makedirs("/tmp/crops", exist_ok=True)

for dataset in api.dataset.get_list(project.id, recursive=True):
    for recording in api.audio.get_list(dataset.id, recursive=False):
        local_path = os.path.join("/tmp/audio", recording.name)
        api.audio.download_path(recording.id, local_path)
        samples, rate = sly.audio.read_audio(local_path)
        for segment in api.audio.get_segments(recording.id):
            crop = sly.audio.render_segment(
                samples, rate, segment.start, segment.end, settings, channel=segment.channel
            )
            name = f"{recording.id}_{segment.id}_{tag_names[segment.tag_id]}.npy"
            np.save(os.path.join("/tmp/crops", name), crop)
```

{% hint style="info" %}
Store the settings next to the crops (`settings.to_json()`): they are what makes a crop comparable with one rendered later.
{% endhint %}

### Rendering in PyTorch or TensorFlow

The stored settings are enough to rebuild the same analysis in a training framework instead of the SDK. Six conventions are not implied by the field names, and getting any of them wrong is a visible difference rather than a rounding one:

| | What the settings mean |
|---|---|
| Window | **Periodic** (`2*pi*i/N`): `torch.hann_window(n, periodic=True)` and its Hamming and Blackman siblings |
| Padding | Frames **centred** on `k*hop_length`, padded with **zeros** (`pad_mode="constant"`), not reflected |
| Frames | `(n - 1) // hop_length + 1`. `torch.stft(center=True)` returns one more when `n` is a multiple of the hop: drop it |
| Scaling | Power divided by `sum(window)^2`, then multiplied by 4 on every bin except DC and Nyquist |
| Mel | **HTK** mel scale from 0 to Nyquist, and a weighted **average**: divide by the sum of each band's weights |
| dB | `10*log10(power)`, floor `1e-20`, clipped to `[min_db, max_db]`; no `top_db`, no normalisation to the loudest value |

In PyTorch, from the decoded `samples` and the project's `settings`:

```python
import numpy as np
import torch

WINDOWS = {"hann": torch.hann_window, "hamming": torch.hamming_window, "blackman": torch.blackman_window}


def htk_mel_filters(fft_size, sample_rate, mel_bands):
    """Triangles on HTK mel edges from 0 to Nyquist, shape (mel_bands, fft_size // 2 + 1)."""
    edges = 700 * np.expm1(np.arange(mel_bands + 2) / (mel_bands + 1) * np.log1p(sample_rate / 1400))
    freqs = np.arange(fft_size // 2 + 1) * sample_rate / fft_size
    lo, mid, hi = edges[:-2, None], edges[1:-1, None], edges[2:, None]
    return torch.from_numpy(np.clip(np.minimum((freqs - lo) / (mid - lo), (hi - freqs) / (hi - mid)), 0, None))


def spectrogram(samples, sample_rate, settings, channel=None):
    x = samples.mean(axis=1) if channel is None else samples[:, channel]
    x = torch.from_numpy(np.ascontiguousarray(x)).double()
    win = WINDOWS[settings.window](settings.fft_size, periodic=True, dtype=torch.float64)
    spec = torch.stft(x, settings.fft_size, settings.hop_length, window=win,
                      center=True, pad_mode="constant", return_complex=True)
    spec = spec[:, : (len(x) - 1) // settings.hop_length + 1]
    power = spec.abs() ** 2 / win.sum() ** 2
    power[1:-1] *= 4.0
    if settings.scale == "mel":
        fb = htk_mel_filters(settings.fft_size, sample_rate, settings.mel_bands)
        power = (fb @ power) / fb.sum(1, keepdim=True)
    db = 10 * torch.log10(power.clamp_min(1e-20))
    return db.clamp(settings.min_db, settings.max_db)
```

This matches `sly.audio.render_spectrogram` to within 0.002 dB for every scale and window. TensorFlow's `tf.signal.stft` has no centring, so pad `fft_size // 2` zeros in front yourself; `tf.signal.linear_to_mel_weight_matrix(lower_edge_hertz=0, upper_edge_hertz=rate / 2)` is the same HTK filter bank.

## Download and upload a project

`sly.download` works for Audio projects as for any other type:

```python
sly.download(api, project.id, "/tmp/engine-noise", save_audio_info=True)
```

```text
📦 engine-noise
├── 📄 meta.json
└── 📂 bench-run-1
    ├── 📂 audio
    │   └── 🎵 bench-01.wav
    ├── 📂 ann
    │   └── 📄 bench-01.wav.json
    └── 📂 audio_info
        └── 📄 bench-01.wav.json
```

`meta.json` carries the tags and the project's spectrogram settings; each annotation carries the recording's segments and recording tags, identified by tag **name**, plus its sample rate, sample count and channel count. See [Audio Annotation](../../supervisely-annotation-format/audio.md) for the format.

```python
project_fs = sly.AudioProject("/tmp/engine-noise", sly.OpenMode.READ)
dataset_fs = project_fs.datasets.get("bench-run-1")

ann = dataset_fs.get_ann("bench-01.wav", project_fs.meta)
print(ann.sample_rate, ann.sample_count, ann.channels)
for segment in ann.tags:
    print(segment.name, segment.start, segment.end, segment.channel, segment.value)
for tag in ann.recording_tags:
    print(tag.name, tag.value)

settings = sly.SpectrogramSettings.from_json(project_fs.meta.project_settings.spectrogram)
```

Upload it back as a new project, with its nested datasets, labels and spectrogram settings:

```python
project_id, project_name = sly.upload_audio_project(
    "/tmp/engine-noise", api, workspace_id, "Engine noise copy"
)
```

To add a project in this format, or a folder of plain recordings, to an existing project, use the [Import](https://docs.supervisely.com/import-and-export/import/supported-annotation-formats/audio) app or `sly.ImportManager` inside an app. The imported spectrogram settings are applied only when the destination project has none yet and holds no recordings; otherwise it keeps its own, because changing them would change what its existing labels mean.
