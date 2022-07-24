---
description: >-
  Supervisely sets default environment variables for each Supervisely
  Application run. You can also use these environment variables for your
  development and debugging.
---

# Environment variables

Supervisely has the set default environment variables, Supervisely Application has access to them and we recommend using the same variables during development and debugging.&#x20;

{% hint style="info" %}
Environment variables are case-sensitive.
{% endhint %}

For development and debugging purposes you can manually **copy the ID to a clipboard** for every item you are working on: team, workspace, project, dataset, image, labeling job, team member, etc... Just open the context menu of the item and press **`Copy ID`** button.

![](https://user-images.githubusercontent.com/12828725/180638373-7f0d81c1-9e53-454e-82c6-50abd184bd00.png)

### **`SERVER_ADDRESS`**

address of your Supervisely instance, for Community Edition the value should be `https://app.supervise.ly`. For the Enterprise Edition the value is custom and depends on your configuration. Learn more [here](../basics-of-authentication.md#server\_address-env). This variable is always passed to an App.&#x20;

### **`API_TOKEN`**

Your personal access token for authentication. Learn more [here](../basics-of-authentication.md#api\_token-env). This variable is always passed to an App.&#x20;

### **`TASK_ID`**

When you run an app on Supervisely, the platform creates a task for this app to store all relevant information for this task (logs, persistent data, cache, temporary files, ...).  Task ID is needed to access this data (read or write). This variable is always passed to an App. &#x20;

![Aplication task on page "Workspace tasks"](https://user-images.githubusercontent.com/12828725/180637942-73b9b411-8251-48f6-a0bf-3b341346d55e.png)

### **`context.teamId`**

The ID of the currently opened team. This variable is always passed to an App.&#x20;

![Current team](https://user-images.githubusercontent.com/12828725/180637662-83b572ee-c49f-41df-9114-241b92207e82.png)

### **`context.workspaceId`**

The ID of the currently opened workspace. This variable is always passed to an App.&#x20;

![Current workspace](https://user-images.githubusercontent.com/12828725/180637666-c3778b97-f616-4f93-9c8c-e66b82da0257.png)
