---
description: Low-level labeling performance metrics used by developers via API
---

# Labeling Performance Metrics

This document describes low-level labeling performance metrics that are logged via the internal API and used by developers to analyze user activity and labeling behavior in annotation tools.

## Overview

These metrics reflect interactions at the system level (project, dataset, figure, etc.) and low-level annotation events. They're useful for building custom analytics, debugging activity flows, and tracking user engagement in data labeling tasks.

## Metrics Table

| Field Name | Description | Meta | Note |
|:-----------|:------------|:-----|:-----|
| `login` | Defines the moment when the user logs in | `{ ip: String }` | Meta may be empty |
| `logout` | Defines the moment when the user logs out | `{ ip: String }` | |
| `create_project` | Triggered when a new project is created | – | |
| `update_project` | Triggered when an existing project is updated | – | |
| `disable_project` | Marks a project as disabled | – | |
| `restore_project` | Restores a previously disabled project | – | |
| `create_dataset` | New dataset creation | – | |
| `update_dataset` | Dataset metadata or content updated | – | |
| `disable_dataset` | Dataset disabled | – | |
| `create_image` | Image is added to dataset | – | |
| `update_image` | Image properties updated | – | |
| `disable_image` | Image disabled | – | |
| `restore_image` | Previously disabled image restored | – | |
| `create_figure` | New annotation figure created | `{ source: 'clone' \| 'tracking', frame: Number, seekPosition: Number, sliceIndex: Number, normal: Object }` | `normal` example: `{"x": 0, "y": 0, "z": 1}` |
| `update_figure` | Annotation figure updated | `{ frame: Number, normal: Object, sliceIndex: Number }` | Meta may be empty. `normal` example: `{"x": 0, "y": 0, "z": 1}` |
| `disable_figure` | Annotation figure disabled or removed | `{ frame: Number, normal: Object, sliceIndex: Number }` | Meta may be empty. `normal` example: `{"x": 0, "y": 0, "z": 1}` |
| `restore_figure` | Annotation figure restored | `{ frame: Number, normal: Object, sliceIndex: Number }` | Meta may be empty. `normal` example: `{"x": 0, "y": 0, "z": 1}` |
| `create_class` | Annotation class created | – | |
| `update_class` | Annotation class updated | – | |
| `disable_class` | Annotation class disabled | – | |
| `restore_class` | Annotation class restored | – | |
| `create_backup` | Project or dataset backup created | – | |
| `export_project` | Export action triggered | – | |
| `model_train` | A training task started using a model | – | |
| `model_inference` | Inference operation performed | – | |
| `create_plugin` | Plugin created | – | |
| `disable_plugin` | Plugin disabled | – | |
| `restore_plugin` | Plugin restored | – | |
| `create_node` | Workflow or task node created | – | |
| `disable_node` | Node disabled | – | |
| `restore_node` | Node restored | – | |
| `create_workspace` | Workspace created | – | |
| `disable_workspace` | Workspace disabled | – | |
| `restore_workspace` | Workspace restored | – | |
| `create_model` | New machine learning model created | – | |
| `disable_model` | Model disabled | – | |
| `restore_model` | Model restored | – | |
| `add_member` | Team member added | – | |
| `remove_member` | Team member removed | – | |
| `login_to_team` | User logs into a specific team | `{ ip: String }` | |
| `attach_tag` | A tag is added to an image or figure | `{ id: Number, value: String, startFrame: Number, endFrame: Number }` | |
| `update_tag_value` | Tag value changed | `{ value: String, startFrame: Number, endFrame: Number }` | |
| `detach_tag` | Tag removed | `{ value: String, startFrame: Number, endFrame: Number }` | Meta may be empty |
| `annotation_duration` | Tracks time spent in the labeling tool per item, regardless of whether any annotation actions were taken. | `{ duration: Number }` | The counter starts when an item is opened in the labeling tool and stops when it's closed or the user moves to the next item. If a `figureId` value is present, it indicates that the user interacted with at least one annotation figure. If not, it means the item was only viewed or reviewed without changes to annotations. |
| `image_review_status_updated` | Status of image review changed | `{ reviewStatus: 'none' \| 'done' \| 'accepted' \| 'rejected' \| 'quality_check_accepted' }` | |
| `email_changed` | User email updated | `{ newEmail: String, oldEmail: String }` | |

## How to Access Activity Data

There are three ways to retrieve labeling activity data:

**REST API**

Use the [`GET /teams.activity`](https://api.docs.supervisely.com/#tag/Teams/paths/~1teams.activity/get) endpoint to query raw activity logs. Supports filtering by `type`, date range, user, and other parameters.

**Python SDK**

Use [`TeamApi.get_activity()`](https://supervisely.readthedocs.io/latest/sdk/supervisely.api.team_api.TeamApi.html#supervisely.api.team_api.TeamApi.get_activity) to fetch activity records programmatically. Returns a list of activity entries with `type`, `meta`, and contextual fields such as job ID and figure ID.

**Ecosystem App**

The [Export Activity as CSV](https://ecosystem.supervisely.com/apps/export-activity-as-csv) app provides a no-code way to export team activity logs directly from the Supervisely UI.