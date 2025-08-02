# AI Search: How to Use Embeddings and Semantic Search

## Overview

This tutorial demonstrates how to use the Supervisely Python SDK to work with AI Search functionality. AI Search allows you to intelligently search for images within a project using semantic similarity, leveraging CLIP embeddings stored in a dedicated vector database (Qdrant).

## Prerequisites

### Instance Requirements

**Minimum Instance Version:** `6.14.4`

For AI Search functionality to work properly, your Supervisely instance must have the following services running:

-   `Embeddings Generator` - Handles the calculation of CLIP embeddings for images
-   `Embeddings Auto-Updater` - Automatically updates embeddings when new images are added
-   `Qdrant Vector Database` - Configured to store and retrieve the calculated embeddings for projects
-   `CLIP Service Application` - Provides the neural network model for generating image embeddings

{% hint style="info" %}

These services are typically configured by your instance administrator. If AI Search is not working, contact your administrator to ensure all required services are properly deployed and running.

{% endhint %}

### SDK Setup

Before starting, ensure you have set up your development environment and installed Supervisely SDK version `6.73.410` or higher

To install the required SDK version:

```bash
pip install supervisely>=6.73.410
```

### Initialize API Client

Once you have your credentials configured, initialize the API client:

```python
import os
from dotenv import load_dotenv
import supervisely as sly

# Load secrets and create API object from .env file (recommended)
# Learn more here: https://developer.supervisely.com/getting-started/basics-of-authentication
if sly.is_development():
    load_dotenv(os.path.expanduser("~/supervisely.env"))

api = sly.Api.from_env()
```

## Core Methods

### Calculate Embeddings

The `calculate_embeddings` method initiates an asynchronous calculation of CLIP embeddings for all images in the specified project. The embeddings are generated and stored in the Qdrant vector database for AI Search operations.

```python
project_id = 123456
api.project.calculate_embeddings(id: project_id)
```

Parameters:

| Argument | Type  | Description                                                                       |
| -------- | ----- | --------------------------------------------------------------------------------- |
| `id`     | `int` | Required. The unique identifier of the project for which to calculate embeddings. |

Returns:

-   `None`

Notes:

-   This method sends a request to the embeddings generator service and returns immediately. The actual calculation happens asynchronously in the background.
-   Embeddings must be calculated before AI Search can be performed on the project.
-   The calculation time depends on the number of images in the project and the performance of your instance.
-   Progress can be tracked through the `ProjectInfo` where `embeddings_in_progress` should be `False` and `embeddings_updated_at` timestamp should be present to indicate completion.
-   If the embeddings auto-updater service is running, new images added to the project will have embeddings calculated automatically.

### Perform AI Search

The `perform_ai_search` method executes an AI-powered search within a project using one of three mutually exclusive search modes: semantic text search, image similarity search, or diverse sampling.

```python
project_id = 123456
perform_ai_search(project_id: project_id)
```

Parameters:
| Argument | Type | Description |
|--------------------|-------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------|
| `project_id` | `int` | Required. Unique identifier of the project to search within. |
| `dataset_id` | `Optional[int]` | Optional. Restricts the search to images within this dataset. Default is `None` (searches entire project). |
| `image_id` | `Optional[List[int]]` | Optional. IDs of a reference image for similarity search. Finds images visually similar to this images. |
| `prompt` | `Optional[str]` | Optional. Natural language text for semantic search. Finds images matching this description. |
| `method` | `Optional[str]` | Optional. Sampling method for diverse search: `"centroids"` (representative samples from clusters), `"random"` (evenly across clusters). |
| `limit` | `Optional[int]` | Optional. Maximum number of images to return. Default is `100`. |
| `clustering_method` | `Optional[Literal["kmeans", "dbscan"]]` | Optional. Clustering method for results: `"kmeans"` or `"dbscan"`. If `None`, no clustering is applied. |
| `num_clusters` | `Optional[int]` | Optional. Number of clusters to create if `clustering_method` is specified. Required for `"kmeans"` method. |
| `image_id_scope` | `Optional[List[int]]` | Optional. List of image IDs to limit the search scope. If `None`, search is performed across all images unless other filters are set. |
| `threshold` | `Optional[float]` | Optional. Similarity threshold. Only images with similarity above this value are returned. In search results, this parameter is also referred to as `score`. |

Returns:

-   The ID `int` of the created entities collection containing search results, or `None` if no collection was created (e.g., no results found).

Raises:

-   `ValueError`: If more than one of prompt, image_id, or method is provided (they are mutually exclusive).
-   `ValueError`: If method is provided but is not one of the allowed values ("centroids" or "random").

Notes:

-   Only one search mode parameter (`prompt`, `image_id`, or `method`) can be used per search.
-   The returned collection ID can be used to retrieve the actual image results using the image API methods.
-   Collections created by AI Search are temporary and will be overwritten by subsequent searches unless explicitly saved.
-   Embeddings must be calculated for the project before performing any search operations.

## Complete Examples

### Enable AI Search for a Project

```python
import supervisely as sly
import time

api = sly.Api.from_env()
project_id = 123456

# Enable embeddings for the project
api.project.enable_embeddings(project_id)
print(f"Embeddings enabled for project {project_id}")

# Start calculating embeddings
api.project.calculate_embeddings(project_id)
print("Embeddings calculation started...")

# Check and wait until process is finished
while api.project.get_embeddings_in_progress(project_id):
    time.sleep(60)

print("AI Search is now enabled and ready!")

```

### Text Prompt Search

```python
from supervisely.api.entities_collection_api import CollectionTypeFilter

project_id = 123456
prompt = "person riding a bicycle in the park"
limit = 20

# Perform text-based search
collection_id = api.project.perform_ai_search(project_id=project_id, prompt=prompt, limit=limit)

if collection_id:
    print(f"Search completed! Collection ID: {collection_id}")

    # Get collection info
    images = api.entities_collection.get_items(
        collection_id, collection_type=CollectionTypeFilter.AI_SEARCH
    )
    print(f"Found {len(images)} similar images")

    # Display results
    for idx, img in enumerate(images[:5]):  # Show first 5
        print(f"{idx+1}. {img.name} (ID: {img.id})")

else:
    print("No results found")
```

### Image Similarity Search

```python
from supervisely.api.entities_collection_api import CollectionTypeFilter

project_id = 123456
reference_image_id = 789012
limit = 20

# Get the reference image info
ref_image = api.image.get_info_by_id(reference_image_id)
print(f"Reference image: {ref_image.name}")

# Search for similar images
collection_id = api.project.perform_ai_search(
    project_id=project_id, image_id=reference_image_id, limit=limit
)

if collection_id:
    # Get similar images
    similar_images = api.entities_collection.get_items(
    collection_id, collection_type=CollectionTypeFilter.AI_SEARCH
)

    print(f"Found {len(similar_images)} similar images:")
    for img in similar_images[:10]:  # Show top 10
        if img.id != reference_image_id:  # Skip the reference image itself
            print(f"- {img.name} (ID: {img.id})")

else:
    print("No results found")
```

### Diverse Search

```python
from supervisely.api.entities_collection_api import CollectionTypeFilter

project_id = 123456
limit = 20
method = "centroids"

# Perform diverse search
collection_id = api.project.perform_ai_search(project_id=project_id, method=method, limit=limit)

if collection_id:
    print(f"Diverse search completed! Collection ID: {collection_id}")
    print(f"Method used: {method}")

    # Get diverse samples
    diverse_images = api.entities_collection.get_items(
        collection_id, collection_type=CollectionTypeFilter.AI_SEARCH
    )

    print(f"\nRetrieved {len(diverse_images)} diverse samples")

    # Group by dataset for analysis
    datasets = {}
    for img in diverse_images:
        ds_id = img.dataset_id
        if ds_id not in datasets:
            datasets[ds_id] = []
        datasets[ds_id].append(img)

    print("\nDistribution across datasets:")
    for ds_id, imgs in datasets.items():
        ds_info = api.dataset.get_info_by_id(ds_id)
        print(f"- {ds_info.name}: {len(imgs)} images")

else:
    print("No results found")
```

## Other Embeddings Methods

Here's a comprehensive table of all embeddings-related methods in the Supervisely SDK:

| Method                         | Module        | Description                                         | Parameters                                                                           | Returns                          |
| ------------------------------ | ------------- | --------------------------------------------------- | ------------------------------------------------------------------------------------ | -------------------------------- |
| `enable_embeddings()`          | `api.project` | Enable embeddings for the project                   | `id: int` - Project ID<br>`silent: bool = True`                                      | `None`                           |
| `disable_embeddings()`         | `api.project` | Disable embeddings for the project                  | `id: int` - Project ID<br>`silent: bool = True`                                      | `None`                           |
| `is_embeddings_enabled()`      | `api.project` | Check if embeddings are enabled for the project.    | `id: int` - Project ID<br>                                                           | `bool`                           |
| `set_embeddings_in_progress()` | `api.project` | Set embeddings calculation status                   | `id: int` - Project ID<br>`in_progress: bool`                                        | `None`                           |
| `get_embeddings_in_progress()` | `api.project` | Get embeddings calculation status                   | `id: int` - Project ID<br>                                                           | `bool`                           |
| `set_embeddings_updated_at()`  | `api.project` | Set the timestamp when embeddings were last updated | `id: int` - Project ID<br>`timestamp: Optional[str] = None`<br>`silent: bool = True` | `None`                           |
| `get_embeddings_updated_at()`  | `api.project` | Get the timestamp when embeddings were last updated | `id: int` - Project ID                                                               | `str` - YYYY-MM-DDTHH:MM:SS.fffZ |

## Possible Use Cases

### Integration with Annotation Workflow

```python
import time

from supervisely.api.entities_collection_api import CollectionTypeFilter


def create_annotation_job_from_search(project_id: int, prompt: str, annotator_id: int):
    """Create annotation job from AI search results."""

    # Ensure embeddings are calculated before performing AI search
    api.project.calculate_embeddings(project_id)

    while api.project.get_embeddings_in_progress(project_id) is True:
        print("Waiting for embeddings to be calculated...")
        time.sleep(10)

    # Search for images
    collection_id = api.project.perform_ai_search(project_id=project_id, prompt=prompt, limit=100)

    if not collection_id:
        print("No images found")
        return None

    # Get images from collection
    images = api.entities_collection.get_items(
        collection_id=collection_id, collection_type=CollectionTypeFilter.AI_SEARCH
    )

    # Map defective images by their dataset
    dataset_to_images = {}
    for img in images:
        ds_id = img.dataset_id
        if ds_id not in dataset_to_images:
            dataset_to_images[ds_id] = []
        dataset_to_images[ds_id].append(img)

    # Create a new dataset for annotation
    dataset_name = f"annotation_job_{prompt.replace(' ', '_')}"
    dataset = api.dataset.create(
        project_id,
        dataset_name,
        change_name_if_conflict=True,
    )

    # Copy images to annotation dataset
    for src_ds_id, image_infos in dataset_to_images.items():
        if not image_infos:
            continue
        # Copy images with annotations to the new dataset
        api.image.copy_batch_optimized(
            src_dataset_id=src_ds_id,
            src_image_infos=image_infos,
            dst_dataset_id=dataset.id,
            with_annotations=True,
        )

    # Create labeling job
    jobs = api.labeling_job.create(
        name=f"Annotate {prompt}",
        dataset_id=dataset.id,
        user_ids=[annotator_id],
        readme=f"Annotate all objects matching: {prompt}",
    )

    print(f"Created annotation jobs: {len(jobs)} for dataset {dataset_name}")
    for job in jobs:
        print(f"Job ID: {job.id}, Name: {job.name}")

    return [job.id for job in jobs]
```

### Active Learning Sample Selection

```python
import time
from supervisely.api.entities_collection_api import CollectionTypeFilter


def select_diverse_training_samples(project_id: int, sample_size: int = 100):
    """Select diverse samples for active learning."""

    # Ensure embeddings are calculated before performing AI search
    api.project.calculate_embeddings(project_id)

    while api.project.get_embeddings_in_progress(project_id) is True:
        print("Waiting for embeddings to be calculated...")
        time.sleep(10)

    # Get diverse samples using centroids method
    collection_id = api.project.perform_ai_search(
        project_id=project_id,
        method="centroids",
        clustering_method="kmeans",
        limit=sample_size,
        num_clusters=10,
    )

    if collection_id:
        # Get selected images
        diverse_samples = api.entities_collection.get_items(
            collection_id=collection_id, collection_type=CollectionTypeFilter.AI_SEARCH
        )
        # Map defective images by their dataset
        dataset_to_images = {}
        for img in diverse_samples:
            ds_id = img.dataset_id
            if ds_id not in dataset_to_images:
                dataset_to_images[ds_id] = []
            dataset_to_images[ds_id].append(img)
        # Create training dataset
        training_dataset = api.dataset.create(
            project_id,
            f"active_learning_batch_{len(diverse_samples)}",
            change_name_if_conflict=True,
        )

        # Move samples to training dataset.
        for src_ds_id, image_infos in dataset_to_images.items():
            if not image_infos:
                continue
            api.image.move_batch_optimized(
                src_dataset_id=src_ds_id,
                src_image_infos=image_infos,
                dst_dataset_id=training_dataset.id,
                with_annotations=True,
                progress_cb=tqdm(desc=f"Moving images to training dataset", total=len(image_infos)),
            )

        print(f"Selected {len(diverse_samples)} diverse samples for training")
        print(f"Training dataset ID: {training_dataset.id}")

        # Recalculate embeddings after moving images
        api.project.calculate_embeddings(project_id)

        return training_dataset.id

    return None

# Example usage
project_id = 123456
training_dataset_id = select_diverse_training_samples(project_id, sample_size=200)
```

### Quality Control in Manufacturing

```python
import time
from supervisely.api.entities_collection_api import CollectionTypeFilter

def find_defective_products(project_id: int, defect_description: str):
    """Find potentially defective products using AI search."""

    # Ensure embeddings are calculated before performing AI search
    api.project.calculate_embeddings(project_id)

    while api.project.get_embeddings_in_progress(project_id) is True:
        print("Waiting for embeddings to be calculated...")
        time.sleep(10)

    # Search for defects using text description
    collection_id = api.project.perform_ai_search(
        project_id=project_id, prompt=defect_description, limit=20
    )

    if collection_id:
        # Get all potentially defective items
        defective_images = api.entities_collection.get_items(
            collection_id=collection_id, collection_type=CollectionTypeFilter.AI_SEARCH
        )
        # Map defective images by their dataset
        dataset_to_images = {}
        for img in defective_images:
            ds_id = img.dataset_id
            if ds_id not in dataset_to_images:
                dataset_to_images[ds_id] = []
            dataset_to_images[ds_id].append(img)
        # Create a dataset for review
        dataset_name = f"potential_defects_{defect_description.replace(' ', '_')}"
        dataset = api.dataset.create(project_id, dataset_name, change_name_if_conflict=True)

        # Copy images to review dataset
        for src_ds_id, image_infos in dataset_to_images.items():
            if not image_infos:
                continue
            # Copy images with annotations to the new dataset
            api.image.copy_batch_optimized(
                src_dataset_id=src_ds_id,
                src_image_infos=image_infos,
                dst_dataset_id=dataset.id,
                with_annotations=True,
            )

        print(f"Created review dataset '{dataset_name}' with {len(defective_images)} images")

        return dataset.id

    return None

# Example usage
project_id = 123456
defect_type = "scratched surface metal parts"
review_dataset_id = find_defective_products(project_id, defect_type)
```

## Summary

The AI Search functionality in Supervisely provides powerful capabilities for:

1. **Semantic Search**: Find images based on natural language descriptions
2. **Similarity Search**: Locate visually similar images
3. **Diverse Sampling**: Get representative samples from your dataset
4. **Dataset Exploration**: Understand the diversity and structure of your data

Key points to remember:

-   Always enable and calculate embeddings before using AI Search
-   Use appropriate search methods based on your use case
-   Manage collections efficiently to avoid clutter
-   Leverage batch operations for large-scale tasks
-   Monitor embeddings status and update as needed

For more information, refer to the [AI Search](https://docs.supervisely.com/data-organization/project-dataset/ai-search) documentation, which provides a visual overview of how AI Search works.
