---
description: Low-level labeling performance metrics used by developers via API
---

# Labeling Performance Metrics

This document describes low-level labeling performance metrics that are logged via the internal API and used by developers to analyze user activity and labeling behavior in annotation tools.

## Overview

These metrics reflect interactions at the system level (project, dataset, figure, etc.) and low-level annotation events. They're useful for building custom analytics, debugging activity flows, and tracking user engagement in data labeling tasks.

## Metrics Table

| Field Name | Description | Value | Note |
|:-----------|:------------|:------|:-----|
| `login` | Defines the moment when the user logs in | – |
| `logout` | Defines the moment when the user logs out | – |
| `create_project` | Triggered when a new project is created | – |
| `update_project`| Triggered when an existing project is updated | – |
| `disable_project` | Marks a project as disabled | – |
| `restore_project` | Restores a previously disabled project | – |
| `create_dataset` | New dataset creation | – |
| `update_dataset` | Dataset metadata or content updated | – |
| `disable_dataset` | Dataset disabled | – |
| `create_image` | Image is added to dataset | – |
| `update_image` | Image properties updated | – |
| `disable_image` | Image disabled | – |
| `restore_image` | Previously disabled image restored | – |
| `create_figure` | New annotation figure created | – |
| `update_figure` | Annotation figure updated | – |
| `disable_figure` | Annotation figure disabled or removed | – |
| `restore_figure` | Annotation figure restored | – |
| `create_class` | Annotation class created | – |
| `update_class` | Annotation class updated | – |
| `disable_class` | Annotation class disabled | – |
| `restore_class` | Annotation class restored | – |
| `create_backup` | Project or dataset backup created | – |
| `export_project` | Export action triggered | – |
| `model_train` | A training task started using a model | – |
| `model_inference` | Inference operation performed | – |
| `create_plugin` | Plugin created | – |
| `disable_plugin` | Plugin disabled | – |
| `restore_plugin` | Plugin restored | – |
| `create_node` | Workflow or task node created | – |
| `disable_node` | Node disabled | – |
| `restore_node` | Node restored | – |
| `create_workspace` | Workspace created | – |
| `disable_workspace` | Workspace disabled | – |
| `restore_workspace` | Workspace restored | – |
| `create_model` | New machine learning model created | – |
| `disable_model` | Model disabled | – |
| `restore_model` | Model restored | – |
| `add_member` | Team member added | – |
| `remove_member` | Team member removed | – |
| `login_to_team` | User logs into a specific team | – |
| `attach_tag` | A tag is added to an image or figure | – |
| `update_tag_value` | Tag value changed | – |
| `detach_tag` | Tag removed | – |
| `annotation_duration` |Tracks time spent in the labeling tool per item, regardless of whether any annotation actions were taken. | {'duration': {number} sec} | The counter starts when an item is opened in the labeling tool and stops when it’s closed or the user moves to the next item. If a `figureId` value is present, it indicates that the user interacted with at least one annotation figure. If not, it means the item was only viewed or reviewed without changes to annotations.|
| `image_review_status_updated` | Status of image review changed | – |
| `email_changed` | User email updated | – |

## Notes

- These events are tracked internally and can be exported via APIs like `Teams.activity` for audit or analytics purposes.
- Many of these metrics log to the `activity_log` table, which can include contextual metadata (e.g., job ID, figure ID).
- Filtering by `type` and `meta` fields allows developers to group or aggregate behaviors.