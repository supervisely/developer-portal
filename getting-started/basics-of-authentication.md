---
description: Learn about the basics of authentication in Supervisely
---

# Basics of authentication

## Basics of Authentication

The easiest and best way to authenticate with the Supervisely API is by using Basic Authentication via a personal access token.

You need only two environment variables:

1. [`SERVER_ADDRESS`](basics-of-authentication.md#server\_address-env) - address of your Supervisely instance
2. [`API_TOKEN`](basics-of-authentication.md#api\_token-env) - your personal access token

{% embed url="https://youtu.be/fObjPz5AnpE" %}
Video tutorial - basics of authentication for python developers
{% endembed %}

{% hint style="info" %}
You can try examples shown in the video for yourself: find the repository with the scripts on [GitHub](https://github.com/supervisely-ecosystem/example-creds-storage).
{% endhint %}

#### `SERVER_ADDRESS` env

If you are using [🌎](basics-of-authentication.md#community) **Community Edition** [🌎](basics-of-authentication.md#community) your server address is `https://app.supervise.ly`

If you are using 🔐 <mark style="color:purple;">**Enterprise Edition**</mark> 🔐 you have your own instance address. You can copy the URL address from the browser or contact instance admin. For example on my private instance the address is the following:

![My private instance of Supervisely](https://user-images.githubusercontent.com/12828725/178995621-5d6b363b-e3c3-4653-8a58-95b9c8f62b34.png)

In the example above the server address is `https://dev.supervise.ly`

#### `API_TOKEN` env

Every basic account has its own personal access token in account settings:

1. Find `Account Settings` under your name in the right top corner.
2. Go to `API Token` tab.
3. Press copy button.

![API token in account settings](https://user-images.githubusercontent.com/12828725/178999565-db05fdfb-2a72-49b2-8247-73873ee9f9ff.png)

You can revoke your current token and generate the new one at any time by clicking `re-generate api key` button.

#### How to use in Python

To communicate with the Supervisely platform, you first need to instantiate a client. The easiest way to do that is by calling the function [`from_env()`](https://supervisely.readthedocs.io/en/latest/sdk/supervisely.api.api.Api.html#supervisely.api.api.Api.from\_env) or pass values of environment variables in the [constructor](https://supervisely.readthedocs.io/en/latest/sdk/supervisely.api.api.Api.html#supervisely.api.api.Api).

**Use `.env` file - recommended 👍**

It is the default practice to store your secrets as environment variables and keep them safe in `.env` files for local development.

1. Create .env file (recommended: `~/supervisely.env`) with the following content:

```python
SERVER_ADDRESS="https://app.supervise.ly"
API_TOKEN="4r47N...xaTatb"
```

2\. Use it the following way

```python
import os
from dotenv import load_dotenv
import supervisely as sly

if sly.is_development():
    load_dotenv(os.path.expanduser("~/supervisely.env"))
api = sly.Api.from_env()
```

**Pass values into the API constructor - optional, not recommended**

```python
import supervisely as sly

api = sly.Api(server_address="https://app.supervise.ly", token="4r47N...xaTatb")
```

{% hint style="warning" %}
We do not recommend using this way in production.\
\
It is the fastest way, but remember, it is not safe to store the secrets right in your sources.\
\
Avoid (_accidentally_) committing (_exposing_) your _private keys_, _passwords_ or other _sensitive details_ (_by hard-coding in them in your script_) to Git by storing them as environment variables.
{% endhint %}

## Fast Authentication with CLI Tool

{% hint style="warning" %}
#### Beta. Release coming soon.
{% endhint %}

If you already use or are about to install our [Enterprise CLI Tool](command-line-interface/cli-tool/), you will be able to do all the `.env` file preparation using just one command.

```bash
# bash
supervisely instance login [OPTIONS]
```

**Command options:**

* `-s` / `--server-address` - Server address.
* `-l` / `--login` - User login.
* `-p` / `--password` - User password.

This will help you automatically create the `.env` file. If you have this file and try to log in as a different user, the existing `.env` file will be overwritten. A backup file will be created for the previous `.env` file. Backup files will be saved for the last 5 authorizations.

More information you can find on [this page](command-line-interface/cli-tool/instance.md#login)
