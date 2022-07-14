---
description: Learn about the basics of authentication in Supervisely
---

# Basics of authentication

The easiest and best way to authenticate with the Supervisely API is by using Basic Authentication via a personal access token.

You need only two environment variables:

1. ``[`SERVER_ADDRESS`](basics-of-authentication.md#server\_address-env) - address of your Supervisely instance
2. ``[`API_TOKEN`](basics-of-authentication.md#api\_token-env) - your personal access token

### `SERVER_ADDRESS` env

If you are using [🌎](basics-of-authentication.md#community) **Community Edition** [🌎](basics-of-authentication.md#community)your server address is `https://app.supervise.ly`

If you are using 🔐 <mark style="color:purple;">**Enterprise Edition**</mark>** ** 🔐 you have your own instance address. You can copy the URL address from the browser or contact instance administrator. For example on my private instance the address is the following:

![My private instance of Supervisely](https://user-images.githubusercontent.com/12828725/178995621-5d6b363b-e3c3-4653-8a58-95b9c8f62b34.png)

In the example above the server address is `https://dev.supervise.ly`

### `API_TOKEN` env

Every basic account has its own personal access token in account settings.&#x20;

![API token in account settings](https://user-images.githubusercontent.com/12828725/178999565-db05fdfb-2a72-49b2-8247-73873ee9f9ff.png)

You can revoke your current token and generate the new one at any time by clicking `re-generate api key` button.

