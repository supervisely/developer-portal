# Modal Window config

### Example 2. Modal Window

Modal Window is designed to have all app pre-launch configuration options in a centralized dialog within one tab. We'll use [`Import Images`](https://ecosystem.supervise.ly/apps/import-images) app as an example in this section.

You can initialize default values for variables in modal window in the config.json file.

[supervisely-ecosystem/import-images/config.json](https://github.com/supervisely-ecosystem/import-images/blob/master/config.json)

```json
{
  "name": "Import Images",
  "type": "app",
  "categories": ["import", "images", "essentials"],
  "description": "Drag and drop images to Supervisely, supported formats: .jpg, .jpeg, jpe, .mpo, .bmp, .png, .tiff, .tif, .webp, .nrrd",
  "docker_image": "supervisely/base-py-sdk:6.68.1",
  "main_script": "src/main.py",
  "modal_template": "src/modal.html",
  "modal_template_state": {
    "normalize_exif": false,
    "remove_alpha_channel": false,
    "remove_source": true,
    "project_name": ""
  },
  "task_location": "workspace_tasks",
  "icon": "https://github.com/supervisely-ecosystem/import-images/releases/download/v1.0.0/icon.png",
  "icon_cover": true,
  "icon_background": "#FFFFFF",
  "min_agent_version": "6.7.4",
  "min_instance_version": "6.5.46",
  "headless": true,
  "context_menu": {
    "context_category": "Import",
    "target": ["files_folder", "images_project", "images_dataset", "agent_folder"]
  },
  "poster": "https://github.com/supervisely-ecosystem/import-images/releases/download/v1.0.0/poster.png"
}
```

<figure><img src="../../.gitbook/assets/modal-props.png" alt=""><figcaption><p>Modal window properties</p></figcaption></figure>
