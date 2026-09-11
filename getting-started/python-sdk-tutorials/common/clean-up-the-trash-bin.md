---
description: >-
  How removal actually works in Supervisely — the archive step, the permanent step, what the
  Trash Bin holds, and how to empty it from Python without surprises.
---

# Clean up the Trash Bin

## Introduction

Deleting a Project in Supervisely does not free any storage. It archives it: the Project
disappears from the workspace and moves to the **Trash Bin**, where it can be restored. Storage
is only released by a second, irreversible step.

That two-step design is deliberate, but it means a long-running instance quietly accumulates
archived Teams, Workspaces, Projects and Datasets that still occupy disk. Clearing them through
the UI is fine for a handful of entities and painful for hundreds.

This tutorial covers:

1. [The two-step removal model](#the-two-step-removal-model) — what archive and permanent removal each do.
2. [Looking inside the Trash Bin](#looking-inside-the-trash-bin) — listing archived entities.
3. [Emptying it](#emptying-the-trash-bin) — the one-call version and the explicit version.
4. [How storage is actually reclaimed](#how-storage-is-actually-reclaimed) — why the disk does not shrink immediately.
5. [Things that will surprise you](#things-that-will-surprise-you) — the traps worth knowing before you automate this.

{% hint style="danger" %}
Permanent removal cannot be undone. There is no trash bin behind the Trash Bin, no restore, and
no backup taken on your behalf. Export anything you might still need before you start.
{% endhint %}

## The two-step removal model

Removal is two steps at every level of the hierarchy:

| Level | Step 1 — archive (soft, reversible) | Step 2 — remove permanently (irreversible, root only) |
| --- | --- | --- |
| Team | `api.team.archive(id)` | `api.team.remove_permanently(ids)` |
| Workspace | `api.workspace.archive(id)` | `api.workspace.remove_permanently(ids)` |
| Project | `api.project.remove(id)` | `api.project.remove_permanently(ids)` |
| Dataset | `api.dataset.remove(id)` | `api.dataset.remove_permanently(ids)` |

Two things about that table are worth pausing on.

**`api.project.remove()` does not remove the project.** It archives it. The name predates the
two-step model and is kept for backwards compatibility; it maps to the `projects.archive` API
method. The same is true of `api.dataset.remove()`.

**Step 2 only accepts entities that are already on step 1.** Calling `remove_permanently` on a
live Project is rejected — archive it first. Removal is also hierarchical: permanently removing
a Team takes its Workspaces, Projects and Datasets with it, so you never need to remove those
separately.

{% hint style="warning" %}
Do not confuse `api.project.remove_permanently()` with `api.project.archive(id, archive_url)`.
Despite the name, the latter is an unrelated legacy method that offloads a project to an
external backup archive. It is not the archive step described here.
{% endhint %}

## Looking inside the Trash Bin

The list methods return live entities by default — that is why an archived Project vanishes from
`api.project.get_list()`. Pass `archived=True` to see the other side:

```python
import os
from dotenv import load_dotenv

import supervisely as sly

if sly.is_development():
    load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api.from_env()

workspace_id = 7

live = api.project.get_list(workspace_id)
archived = api.project.get_list(workspace_id, archived=True)

print(f"{len(live)} live, {len(archived)} in the Trash Bin")
for project in archived:
    print(project.id, project.name)
```

The same parameter exists on `api.team.get_list()`, `api.workspace.get_list()` and
`api.dataset.get_list()`. It takes three values:

| Value | Returns |
| --- | --- |
| `None` (default) | live entities only — the request is sent exactly as before |
| `True` | archived entities instead |
| `"forever"` | archived entities including those already queued for permanent removal, where the entity has that state. Root only |

{% hint style="info" %}
`archived` requires Supervisely instance **6.17.22** or newer. On older instances the list
methods are hard-scoped to live entities and the Trash Bin cannot be enumerated over the API.
{% endhint %}

Each level needs its parent's id — `workspaces.list` needs a Team, `projects.list` needs a
Workspace, `datasets.list` needs a Project — so enumerating a whole instance means walking all
four levels. `api.trash.get_list()` does that walk for you:

```python
for item in api.trash.get_list():
    print(item.type, item.id, item.name)

# team 42 Old experiments
# workspace 7 Q1 imports
# project 111 lemons_annotated
# dataset 222 ds0
```

Each entry is a `TrashItem` with `type`, `id`, `name`, `team_id`, `workspace_id` and
`project_id`. Entities already covered by an archived parent are **not** listed separately: if a
Team is archived, its Projects do not appear, because removing the Team removes them too. What
you see is exactly what `clear()` will act on.

## Emptying the Trash Bin

### The short version

```python
import supervisely as sly

api = sly.Api.from_env()

print(api.trash.get_list())   # read this first
print(api.trash.clear())
# Output: {'team': 1, 'workspace': 0, 'project': 4, 'dataset': 2}
```

`clear()` removes everything in the order that respects the hierarchy — Teams, then Workspaces,
then Projects, then Datasets — waits for the background Team and Workspace removal tasks to
reach a terminal status before descending, and finally triggers the storage garbage collector.

Useful parameters:

```python
# Restrict the sweep to one Team
api.trash.clear(team_id=42)

# Skip the hunt for archived Datasets inside live Projects.
# That scan costs one API call per live Project, which adds up on a large instance.
api.trash.clear(include_datasets=False)

# Do not trigger the garbage collector; it also runs daily on its own
api.trash.clear(cleanup_unused=False)

# Track progress. `sly.tqdm_sly` is callable, which is what `progress_cb` expects
api.trash.clear(progress_cb=sly.tqdm_sly(desc="Removing", unit="entity"))
```

{% hint style="danger" %}
`clear()` is root only and irreversible. A regular user token is not enough, and there is no
confirmation prompt.
{% endhint %}

### The explicit version

If you want to select what goes rather than take everything, drive the levels yourself. The
pattern below archives a Project and then removes it permanently:

```python
import supervisely as sly

api = sly.Api.from_env()

project_id = 111

api.project.remove(project_id)                 # step 1: archive
api.project.remove_permanently(project_id)     # step 2: irreversible
```

Batching is built in — `remove_permanently` accepts a list and splits it into API calls of at
most 50 ids:

```python
workspace_id = 7

archived = api.project.get_list(workspace_id, archived=True)
old = [p.id for p in archived if p.updated_at < "2026-01-01"]

api.project.remove_permanently(old)
```

When passing a list to `api.project.remove_permanently()`, all ids must belong to the same Team.
Group them before calling.

Team and Workspace removal is asynchronous and returns a task id. Poll it before assuming the
work is done:

```python
import time

responses = api.team.remove_permanently([42])
task_id = responses[0]["taskId"]

while True:
    status = api.task.get_status(task_id)
    if status in (api.task.Status.FINISHED, api.task.Status.ERROR):
        break
    time.sleep(5)

api.task.raise_for_status(status)
```

A status of `error` is terminal — the removal stopped and will not resume on its own. Permanent
removal is idempotent, so retrying the same batch after a failure is safe.

## How storage is actually reclaimed

This is the part that generates support tickets, so it is worth understanding before you measure
your bucket.

Image and video data is stored **instance-globally and reference-counted by content hash**. One
stored object can be referenced from any number of Projects, Workspaces and Teams — that is what
makes cloning a project cheap and stops duplicate uploads from consuming space twice.

So permanently removing an entity drops its **references** to the data, not the data itself. The
underlying objects are reclaimed once nothing references them any more *and* a grace window has
passed, in two waves:

* **Wave 1 — inline, during the removal.** Data whose last request is older than 12 hours is
  reclaimed as part of the removal itself. Recently-requested data is deliberately left alone, so
  an in-flight download or an open labeling session is not pulled out from under it.
* **Wave 2 — the garbage collector.** Everything left over is swept by the instance-wide GC,
  which runs daily and can also be triggered on demand. It applies a **3-day grace window**
  before reclaiming an unreferenced object.

```python
api.trash.cleanup_unused()   # returns the background task id
```

`api.trash.clear()` calls this for you unless you pass `cleanup_unused=False`.

{% hint style="warning" %}
Bucket usage does not drop to its final value the instant a removal finishes. Expect the
remainder to be released over the following days as the two waves complete. Running the cleanup
twice in a row does not shorten the grace windows.
{% endhint %}

## Things that will surprise you

**`api.project.remove()` archives, it does not delete.** Covered above, and it is the single most
common misreading of this API.

**Not everything in the Trash Bin is reachable from Python.** The Trash Bin page also lists
models, checkpoints, python notebooks and DTL archives. Those have no public API, so
`api.trash.clear()` leaves them alone — remove them from the Trash Bin page in the UI. If your
goal is "the instance reports zero trash", the API gets you most of the way and the UI finishes
the job.

**If you call the REST method directly, send `preserveProjectCard` explicitly.** The
`projects.remove.permanently` endpoint defaults it to `true`, which is a *different operation*:
it keeps the project card and drops only its data, so the project stays in the Trash Bin while
the call still answers `{"success": true}`. The SDK always sends `false`, so
`api.project.remove_permanently()` and `api.trash.clear()` are not affected — this only bites
hand-rolled `api.post("projects.remove.permanently", ...)` calls.

```python
# Deletes the project — what you almost always want
api.post("projects.remove.permanently",
         {"projects": [{"id": 111}], "preserveProjectCard": False})

# Keeps the project card, drops only its data, leaves the row in the Trash Bin
api.post("projects.remove.permanently", {"projects": [{"id": 111}]})
```

**An entity can get stuck mid-restore.** Restoring an item from the Trash Bin puts it into a
`sync` status while its data is restored. If that job cannot finish, the UI shows
`This asset is syncing. You can delete or restore it only after synchronization is complete` and
refuses to select the row. The permanent-removal API methods have no such check, so
`api.trash.clear()` clears those entities anyway.

**The admin Team (id `1`) cannot be removed** at all, and is skipped by `api.trash.get_list()`.

## Related

* [Permanent removal](https://docs.supervisely.com/data-organization/storage/permanent-removal) — the same model from the platform documentation, with the raw API reference
* [Server trash bin](https://docs.supervisely.com/collaboration/admin-panel/server-trash-bin) — the UI view
* [Cloning projects for development](cloning-projects.md) — how to keep a copy before you prune
