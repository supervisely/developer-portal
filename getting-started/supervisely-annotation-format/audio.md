# Audio Annotation

An Audio project is stored like the other project types: a `meta.json` with the tags and project settings, and one directory per dataset.

```text
📦 project_name
├── 📄 meta.json
└── 📂 dataset_name
    ├── 📂 audio
    │   ├── 🎵 recording_01.wav
    │   └── 🎵 recording_02.flac
    ├── 📂 ann
    │   ├── 📄 recording_01.wav.json
    │   └── 📄 recording_02.flac.json
    ├── 📂 audio_info             (optional)
    └── 📂 datasets               (nested datasets, same layout)
```

Supported recording formats: `.wav`, `.flac`, `.mp3`, `.ogg`, `.m4a`. Sample indices refer to the decoded recording as the labeling tool plays it; for `.m4a` that is the container's timeline, without the AAC encoder delay or the padding of the last frame.

## Annotation file

A label on a recording is a tag, applied either to a range of samples (a segment) or to the whole recording (a recording tag, "Entire recording" in the labeling tool). Each recording has an annotation file named after it:

```json
{
  "description": "",
  "sampleCount": 48000,
  "sampleRate": 16000,
  "channels": 2,
  "tags": [
    {
      "name": "knock",
      "frameRange": [16000, 23999],
      "channel": 1
    },
    {
      "name": "source",
      "frameRange": [0, 47999],
      "channel": null,
      "value": "engine"
    },
    {
      "name": "scene",
      "frameRange": null,
      "value": "test bench"
    }
  ]
}
```

* `sampleCount`, `sampleRate`, `channels` — the shape of the recording. The platform does not store them, so they are written on export to make sample ranges convertible to seconds. Optional on import.
* `tags` — the labels of the recording: segments and recording tags in one list.
* `name` — the tag's name in `meta.json`.
* `frameRange` — first and last sample of the segment, **both inclusive**, as zero-based indices into the original recording. The name is shared with videos; for audio the numbers are samples, not frames and not milliseconds. At 16 kHz, `[16000, 23999]` is 1.0 s to 1.5 s. `null` marks a recording tag, which labels the whole file; a tag can be on a recording only once.
* `channel` — zero-based channel a segment is about, or `null` for all channels. A recording tag has no channel.
* `value` — the tag value, for tags that have one.
* `meta` — optional free-form object, kept as is.
* `customData` — optional object any client may attach to a label, kept as is.
* `tagId`, `id`, `labelerLogin` — written on export, ignored on import: ids are replaced with the destination project's.

## Spectrogram settings

The spectrogram every recording in the project is analysed with is a project setting, stored in `meta.json` under `projectSettings.spectrogram`:

```json
{
  "classes": [],
  "tags": [
    { "name": "knock", "value_type": "none", "color": "#148A0F" }
  ],
  "projectType": "audio",
  "projectSettings": {
    "multiView": { "enabled": false, "tagName": null, "tagId": null, "isSynced": false },
    "spectrogram": {
      "scale": "mel",
      "fftSize": 1024,
      "hopLength": 256,
      "window": "hann",
      "melBands": 64,
      "minDb": -100.0,
      "maxDb": 0.0,
      "colormap": "magma",
      "interpolation": "sharp"
    }
  }
}
```

| Field | Values |
|---|---|
| `scale` | `linear`, `log`, `mel` |
| `fftSize` | 32, 64, 128, ..., 32768 |
| `hopLength` | integer, at least 1 |
| `window` | `hann`, `hamming`, `blackman` |
| `melBands` | 2 to 512 |
| `minDb`, `maxDb` | numbers, `maxDb` greater than `minDb` |
| `colormap` | `viridis`, `magma`, `grayscale` |
| `interpolation` | `sharp`, `smooth` |

All nine fields are required when `spectrogram` is present. Without it the platform defaults apply. The channel on screen is not a setting: it is chosen while viewing.

See [Audio](../python-sdk-tutorials/audio/audio.md) for reading and writing this format with the Python SDK.
