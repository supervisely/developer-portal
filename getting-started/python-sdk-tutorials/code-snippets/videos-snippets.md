# Python code snippets for working with videos

In this section, you'll find Python code snippets for working with videos for different use cases.

## Copy or move videos without annotations to another project or dataset

**Use case:** you need to copy or move videos without annotations to another project or dataset.

```python
src_dataset_id = 77612

# The destination dataset may be in the same project or in another project.
# Remember that destination dataset must be created before copying videos.
dst_dataset_id = 77613

# Retrieve sly.VideoInfo from the source dataset.
src_videos = api.video.get_list(src_dataset_id)

# For example: copy first 10 videos.
src_videos = src_videos[:10]

# Prepare lists of video names and video IDs.
src_video_ids = [video.id for video in src_videos]
src_video_names = [video.name for video in src_videos]

dst_videos = api.video.upload_ids(dst_dataset_id, src_video_names, src_video_ids)

# If you need to move videos (not copy), you can delete them from the source dataset.
api.video.remove_batch(src_video_ids)
```

## Copy or move videos with annotations to another project

**Use case:** you need to copy or move videos with annotations to another project.<br>
**Context:** when working with annotated videos, you also need to ensure that destination project meta contains all the necessary classes and tags.

```python
src_project_id = 29276
src_dataset_id = 77612

dst_project_id = 29277
dst_dataset_id = 77613

# Retrieve sly.VideoInfo from the source dataset.
src_videos = api.video.get_list(src_dataset_id)

# For example: copy first 10 videos.
src_videos = src_videos[:10]

# Prepare lists of video names and video IDs.
src_video_ids = [video.id for video in src_videos]
src_video_names = [video.name for video in src_videos]

# Retrieve source project meta.
src_project_meta = sly.ProjectMeta.from_json(api.project.get_meta(src_project_id))

# Download corresponding annotations for the videos in JSON format.
anns_json = api.video.annotation.download_bulk(src_dataset_id, src_video_ids)

# Create sly.VideoAnnotation objects from the JSON annotations.
anns = [
    sly.VideoAnnotation.from_json(ann_json, src_project_meta, sly.KeyIdMap())
    for ann_json in anns_json
]

# Retrieve destination project meta.
dst_project_meta = sly.ProjectMeta.from_json(api.project.get_meta(dst_project_id))

# Iterate over annotattions to update the destination project meta.
for ann in anns:
    for video_obj in ann.objects:
        if video_obj.obj_class not in dst_project_meta.obj_classes:
            dst_project_meta = dst_project_meta.add_obj_class(video_obj.obj_class)
        if video_obj.tags is not None:
            for tag in video_obj.tags:
                if tag.meta not in dst_project_meta.tag_metas:
                    dst_project_meta = dst_project_meta.add_tag_meta(tag.meta)

# Update destination project meta on the server.
api.project.update_meta(dst_project_id, dst_project_meta)

# Upload videos to the destination dataset.
dst_videos = api.video.upload_ids(dst_dataset_id, src_video_names, src_video_ids)
dst_video_ids = [video.id for video in dst_videos]

# Upload annotations to the destination dataset.
for ann, video_id in zip(anns, dst_video_ids):
    api.video.annotation.append(video_id, ann)

# If you need to move videos (not copy), you can delete them from the source dataset.
api.video.remove_batch(src_video_ids)
```
