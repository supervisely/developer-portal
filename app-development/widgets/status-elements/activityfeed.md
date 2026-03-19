# Activity Feed

## Introduction

**Activity Feed** widget displays a vertical timeline of activity items with status indicators. Each item can contain custom widget content and shows a visual status (completed, in progress, pending, or error). This widget is useful for showing task progress, workflow steps, or sequential operations in applications.

[Read this tutorial in developer portal.](https://developer.supervisely.com/app-development/widgets/status-elements/activity-feed)

[![activity-feed](https://github.com/supervisely-ecosystem/ui-widgets-demos/releases/download/v0.0.8/screenshot-localhost-8000-1765982017937.png)

## Function signature

```python
ActivityFeed(
    items: Optional[List[ActivityFeed.Item]] = None,
    widget_id: Optional[str] = None,
)
```

## Parameters

| Parameters  |                Type                 |            Description            |
| :---------: | :---------------------------------: | :-------------------------------: |
|   `items`   | `Optional[List[ActivityFeed.Item]]` | List of activity items to display |
| `widget_id` |           `Optional[str]`           |         ID of the widget          |

### items

List of `ActivityFeed.Item` objects representing individual activities.

**type:** `Optional[List[ActivityFeed.Item]]`

**default value:** `None`

```python
item1 = ActivityFeed.Item(content=Text("Processing dataset"), status="completed")
item2 = ActivityFeed.Item(content=Text("Training model"), status="in_progress", number=2)
feed = ActivityFeed(items=[item1, item2])
```

### widget_id

ID of the widget.

**type:** `Optional[str]`

**default value:** `None`

## ActivityFeed.Item

Inner class for creating activity feed items.

### Function signature

```python
ActivityFeed.Item(
    content: Widget,
    status: Literal["pending", "in_progress", "completed", "failed"] = "pending",
    number: Optional[int] = None,
)
```

### Parameters

| Parameters |                            Type                            |                     Description                     |
| :--------: | :--------------------------------------------------------: | :-------------------------------------------------: |
| `content`  |                          `Widget`                          |          Widget to display as item content          |
|  `status`  | `Literal["pending", "in_progress", "completed", "failed"]` |         Visual status of the activity item          |
|  `number`  |                      `Optional[int]`                       | Position number in the feed (auto-assigned if None) |

## Methods and attributes

| Attributes and Methods | Description                                 |
| :--------------------: | ------------------------------------------- |
|      `add_item()`      | Add a new item to the activity feed.        |
|    `remove_item()`     | Remove an item from the feed by its number. |
|     `set_status()`     | Update the status of an item by its number. |
|     `get_status()`     | Get the status of an item by its number.    |
|     `get_items()`      | Get all items in the activity feed.         |
|       `clear()`        | Remove all items from the feed.             |
|     `set_items()`      | Replace all items in the activity feed.     |

## Mini App Example

You can find this example in our GitHub repository:

[supervisely-ecosystem/ui-widgets-demos/status elements/010_activity_feed/src/main.py](https://github.com/supervisely-ecosystem/ui-widgets-demos/blob/master/status%20elements/010_activity_feed/src/main.py)

### Import libraries

```python
import os
import supervisely as sly
from supervisely.app.widgets import Card, Container, ActivityFeed, Text
from dotenv import load_dotenv
```

### Init API client

First, we load environment variables with credentials and init API for communicating with Supervisely Instance:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api()
```

### Create activity items with custom content

Create individual activity items with different statuses and custom widget content:

```python
# Create items with custom content
item1 = ActivityFeed.Item(content=Text("Processing dataset"), status="completed")
item2 = ActivityFeed.Item(content=Text("Training model"), status="in_progress", number=2)
item3 = ActivityFeed.Item(content=Text("Generating report"), status="pending")
```

### Initialize `ActivityFeed` widget

```python
# Create activity feed
feed = ActivityFeed(items=[item1, item2, item3])

# Add item during runtime
new_item = ActivityFeed.Item(content=Text("Deploy model"), status="pending")
feed.add_item(new_item)
```

### Create app layout

Prepare a layout for app using `Card` widget with the `content` parameter and place widget that we've just created in the `Container` widget.

```python
container = Container([feed])

card = Card(
    title="Activity Feed with Progress",
    content=container,
)
layout = card
```

### Create app using layout

Create an app object with layout parameter.

```python
app = sly.Application(layout=layout)
```

The activity feed will display a vertical timeline showing all activities with their current status indicators. Items with "completed" status show a checkmark, "in_progress" shows a spinner, "pending" shows an empty circle, and "failed" shows an error indicator.

### Using methods to control the feed

```python
# Update status by item number
feed.set_status(2, "completed")

# Get item status
status = feed.get_status(2)
print(f"Item 2 status: {status}")

# Remove an item
feed.remove_item(1)

# Clear all items
feed.clear()

# Set new items
new_items = [
    ActivityFeed.Item(content=Text("Step 1"), status="completed"),
    ActivityFeed.Item(content=Text("Step 2"), status="in_progress"),
]
feed.set_items(new_items)
```
