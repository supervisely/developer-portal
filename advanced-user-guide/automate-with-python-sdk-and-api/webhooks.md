---
description: Guide explains how to manage webhooks using Supervisely SDK and API
---

# Webhooks

## Introduction

In this tutorial you will learn how to manage `Webhooks` using Supervisely SDK and API. Webhooks allow you to receive real-time notifications when certain events occur in your team, such as labeling job completion or labeling queue updates.

## Webhooks automation

### Import libraries

```python
import os
from dotenv import load_dotenv
import supervisely as sly
```

### Init API client

Init API for communicating with Supervisely Instance. First, we load environment variables with credentials:

```python
if sly.is_development():
    load_dotenv(os.path.expanduser("~/supervisely.env"))
api = sly.Api.from_env()
```

### Get your Team ID from environment

```python
team_id = 123  # change to your team ID
```

## Available webhook actions

Supervisely supports the following webhook action types:

- `sly.LABELING_JOB_COMPLETED` - Triggered when a labeling job is completed
- `sly.LABELING_QUEUE_JOB_COMPLETED` - Triggered when a job in labeling queue is completed
- `sly.LABELING_QUEUE_COMPLETED` - Triggered when entire labeling queue is completed

## Create a webhook

Create a new webhook that will be triggered when a labeling job is completed:

```python
webhook = api.webhook.create(
    team_id=team_id,
    url="https://example.com/webhook-endpoint",
    action=sly.LABELING_JOB_COMPLETED,
    retries_count=3,
    retries_delay=10
)

print(f"Created webhook with ID: {webhook.id}")
print(f"Action: {webhook.action}")
print(f"URL: {webhook.url}")
```

### Create webhook with custom headers

You can add custom headers to your webhook requests, for example for authentication:

```python
webhook = api.webhook.create(
    team_id=team_id,
    url="https://example.com/webhook-endpoint",
    action=sly.LABELING_JOB_COMPLETED,
    headers={
        "Authorization": "Bearer your-secret-token",
        "X-Custom-Header": "custom-value"
    },
    retries_count=5,
    retries_delay=15,
    follow_redirect=True,
    reject_unauthorized=True
)
```

### Parameters explanation

- **team_id** - ID of your team where the webhook will be active
- **url** - Target URL that will receive webhook POST requests
- **action** - Type of event that triggers the webhook (use constants from `sly`)
- **retries_count** - Number of retry attempts if webhook delivery fails (default: 5)
- **retries_delay** - Delay in seconds between retry attempts (default: 10)
- **headers** - Dictionary of custom HTTP headers to include in webhook requests
- **follow_redirect** - Whether to follow HTTP redirects (default: True)
- **reject_unauthorized** - Whether to reject unauthorized SSL certificates (default: True)

## Get list of webhooks

Get all webhooks in your team:

```python
webhooks = api.webhook.get_list(team_id=team_id)

for webhook in webhooks:
    print(f"ID: {webhook.id}")
    print(f"Action: {webhook.action}")
    print(f"URL: {webhook.url}")
    print(f"Retries: {webhook.retries_count}")
    print(f"Created at: {webhook.created_at}")
```

### Filter webhooks

You can filter webhooks by specific criteria:

```python
# Get only webhooks for labeling job completion
filters = [{"field": "action", "operator": "=", "value": sly.LABELING_JOB_COMPLETED}]
webhooks = api.webhook.get_list(team_id=team_id, filters=filters)
```

## Get webhook information by ID

Retrieve detailed information about a specific webhook:

```python
webhook_id = 456  # your webhook ID
webhook_info = api.webhook.get_info_by_id(webhook_id=webhook_id)

print(f"Webhook ID: {webhook_info.id}")
print(f"Action: {webhook_info.action}")
print(f"URL: {webhook_info.url}")
print(f"Retries count: {webhook_info.retries_count}")
print(f"Retries delay: {webhook_info.retries_delay}")
print(f"Team ID: {webhook_info.team_id}")
print(f"Created at: {webhook_info.created_at}")
print(f"Updated at: {webhook_info.updated_at}")
```

## Update a webhook

Update an existing webhook's properties:

```python
webhook_id = 456  # your webhook ID

# Update webhook URL
updated_webhook = api.webhook.update(
    webhook_id=webhook_id,
    url="https://new-endpoint.com/webhook"
)

# Update multiple properties
updated_webhook = api.webhook.update(
    webhook_id=webhook_id,
    url="https://new-endpoint.com/webhook",
    retries_count=10,
    retries_delay=20,
    headers={
        "Authorization": "Bearer new-token"
    }
)

print(f"Updated webhook: {updated_webhook.url}")
```

**Note:** Only the parameters you specify will be updated. Other parameters will remain unchanged.

## Test a webhook

Send a test event to verify your webhook is working correctly:

```python
webhook_id = 456  # your webhook ID

# Test existing webhook
response = api.webhook.test(
    team_id=team_id,
    webhook_id=webhook_id
)

print(f"Test response: {response}")
```

### Test webhook with custom payload

You can also test a webhook with a custom payload:

```python
# Test with custom payload
response = api.webhook.test(
    team_id=team_id,
    webhook_id=webhook_id,
    payload={
        "custom_field": "test_value",
        "test_mode": True
    }
)
```

## Remove a webhook

Delete a webhook when it's no longer needed:

```python
webhook_id = 456  # your webhook ID

api.webhook.remove(webhook_id=webhook_id)
print(f"Webhook {webhook_id} has been removed")
```

## Webhook payload structure

When a webhook is triggered, Supervisely sends a POST request to your URL with the following structure:

### Labeling Job Completed Event

```json
{
  "event": "labeling_job.completed",
  "timestamp": "2026-02-06T12:34:56.789Z",
  "team_id": 123,
  "data": {
    "job_id": 789,
    "job_name": "Labeling Job #1",
    "project_id": 456,
    "dataset_id": 234,
    "status": "completed",
    "completed_by": {
      "user_id": 7,
      "user_login": "annotator"
    }
  }
}
```

### Labeling Queue Job Completed Event

```json
{
  "event": "labeling_queue.job.completed",
  "timestamp": "2026-02-06T12:34:56.789Z",
  "team_id": 123,
  "data": {
    "queue_id": 111,
    "job_id": 789,
    "project_id": 456,
    "dataset_id": 234,
    "status": "completed"
  }
}
```

### Labeling Queue Completed Event

```json
{
  "event": "labeling_queue.completed",
  "timestamp": "2026-02-06T12:34:56.789Z",
  "team_id": 123,
  "data": {
    "queue_id": 111,
    "queue_name": "My Labeling Queue",
    "total_jobs": 50,
    "completed_jobs": 50,
    "status": "completed"
  }
}
```

## Complete example

Here's a complete example that demonstrates webhook management:

```python
import os
from dotenv import load_dotenv
import supervisely as sly

# Initialize API
if sly.is_development():
    load_dotenv(os.path.expanduser("~/supervisely.env"))
api = sly.Api.from_env()

team_id = 123  # change to your team ID

# Create a webhook
webhook = api.webhook.create(
    team_id=team_id,
    url="https://example.com/webhook",
    action=sly.LABELING_JOB_COMPLETED,
    headers={"Authorization": "Bearer secret-token"},
    retries_count=3,
    retries_delay=10
)
print(f"Created webhook ID: {webhook.id}")

# List all webhooks
webhooks = api.webhook.get_list(team_id=team_id)
print(f"Total webhooks: {len(webhooks)}")
# Get webhook info
webhook_info = api.webhook.get_info_by_id(webhook_id=webhook.id)
print(f"Webhook URL: {webhook_info.url}")

# Test webhook
test_response = api.webhook.test(team_id=team_id, webhook_id=webhook.id)
print(f"Test response: {test_response}")

# Update webhook
updated = api.webhook.update(
    webhook_id=webhook.id,
    retries_count=5
)
print(f"Updated retries to: {updated.retries_count}")

# Remove webhook
api.webhook.remove(webhook_id=webhook.id)
print(f"Removed webhook ID: {webhook.id}")
```

## Best practices

1. **Use HTTPS endpoints** - Always use secure HTTPS URLs for your webhook endpoints
2. **Validate webhook signatures** - Implement signature validation on your server to ensure requests are from Supervisely
3. **Handle retries gracefully** - Make your endpoint idempotent to handle potential duplicate deliveries
4. **Set appropriate retry settings** - Configure `retries_count` and `retries_delay` based on your endpoint's reliability
5. **Monitor webhook health** - Regularly check webhook delivery status and update if needed
6. **Use custom headers for authentication** - Add authentication tokens in headers rather than in URL parameters
7. **Test before production** - Use the `test()` method to verify your webhook endpoint works correctly

## Troubleshooting

### Webhook not receiving events

1. Verify the webhook URL is publicly accessible
2. Check that the action type matches the events you expect
3. Ensure your server accepts POST requests
4. Check server logs for incoming requests

### SSL/TLS errors

If you're getting SSL errors, you can temporarily disable certificate validation:

```python
webhook = api.webhook.create(
    team_id=team_id,
    url="https://self-signed-cert.example.com/webhook",
    action=sly.LABELING_JOB_COMPLETED,
    reject_unauthorized=False  # Only for development/testing!
)
```

**Warning:** Never use `reject_unauthorized=False` in production environments.
