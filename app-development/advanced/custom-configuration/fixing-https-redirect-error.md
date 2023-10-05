---
description: In this tutorial you will learn how to fix 400 HTTP errors which usually occur when HTTPS is enabled and the client is using HTTP instead, which causes the client to skip the request body entirely for security reasons.
---

# Fixing 400 HTTP errors when using HTTP instead of HTTPS

## Introduction <a href="#user-content-fixing-https-redirect-error-in-supervisely" id="user-content-fixing-https-redirect-error-in-supervisely"></a>

When using HTTP -> HTTPS redirect on your Supervisely instance, you may face the problem, when all Supervisely applications would crash with same error, which in most cases, look like this in app's logs:

```
400 Client Error: Bad Request for url: https://your-sly-instance-domain/public/api/v3/ecosystem.file.download
```

Full log entry for example:

```json
{
  "level": "warn",
  "message": "TASK_END",
  "fields": {
    "exit_status": "400 Client Error: Bad Request for url: https://your-sly-instance-domain/api/v3/ecosystem.file.download ({\"error\":\"Validation error\",\"details\":[{\"message\":\"\\\"moduleId\\\" is required\",\"path\":[\"moduleId\"],\"type\":\"any.required\",\"context\":{\"key\":\"moduleId\",\"label\":\"moduleId\"}},{\"message\":\"\\\"value\\\" must contain at least one of [filePath, isArchive]\",\"path\":[],\"type\":\"object.missing\",\"context\":{\"peers\":[\"filePath\",\"isArchive\"],\"peersWithLabels\":[\"filePath\",\"isArchive\"],\"label\":\"value\"}}]})"
  },
  "timestamp": "2023-10-04T08:12:42.922Z",
  "task_id": 79
}
```

So, to fix this error, you just need to change http:// part of server adress to https:// in instance's settings.<br>
And there are two ways to do it:

1. [Using UI](#changing-server-address-using-ui)
2. [Using terminal](#changing-server-address-using-terminal)

## Changing Server Address Using UI <a href="#user-content-changing-server-address-using-ui" id="user-content-changing-server-address-using-ui"></a>

1. Log in into your Supervisely instance as admin user.
2. In top right corner, click on your user name and select `Instance settings` from dropdown menu.
3. Expand `Server configuration` section.
4. In `Server address` field, change `http://` part of server address to `https://` (e.g. `http://your-sly-instance-domain` -> `https://your-sly-instance-domain`).
5. Scroll the page down and click `Save` button.

ℹ️ After changing the server address you need to re-deploy all agents, which are connected to your instance.

<img src="https://github-production-user-asset-6210df.s3.amazonaws.com/118521851/272907722-fe034e21-63fb-4cc0-83b1-5bb04878b552.png"/>

## Changing Server Address Using Terminal <a href="#user-content-changing-server-address-using-terminal" id="user-content-changing-server-address-using-terminal"></a>

1. Open Supervisely directory in terminal (e.g. `cd /home/supervisely/supervisely`).
2. Open `.env` file in your favorite text editor (e.g. `nano .env`).
3. Find `DOMAIN` variable and change `http://` part of server address to `https://` (e.g. `http://your-sly-instance-domain` -> `https://your-sly-instance-domain`).
4. Repeat the same for `SERVER_ADDRESS` variable.
5. Save changes and close the file.
6. Restart Supervisely using `sudo supervisely up -d` command, otherwise changes won't take effect.

[![asciicast](https://asciinema.org/a/bJsPCmcOw4pkxw0FdQGhq1JsO.svg)](https://asciinema.org/a/bJsPCmcOw4pkxw0FdQGhq1JsO)

ℹ️ After changing the server address you need to re-deploy all agents, which are connected to your instance.
