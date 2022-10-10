---
description: >-
  Guide explains how to manipulate user and team members using Supervisely SDK
  and API
---

# User automation

## Introduction

In this tutorial you will learn how to work with `User` and manage team members using Supervisely SDK and API.

📗 Everything you need to reproduce [this tutorial is on GitHub](https://github.com/supervisely-ecosystem/automation-with-python-sdk-and-api/tree/master/examples/user-automation): source code and demo data.

## How to debug this tutorial

**Step 1.** Prepare `~/supervisely.env` file with credentials. [Learn more here.](https://developer.supervise.ly/getting-started/basics-of-authentication#how-to-use-in-python)

**Step 2.** Clone [repository](https://github.com/supervisely-ecosystem/automation-with-python-sdk-and-api) with source code and demo data and create [Virtual Environment](https://docs.python.org/3/library/venv.html).

```bash
git clone https://github.com/supervisely-ecosystem/automation-with-python-sdk-and-api
cd automation-with-python-sdk-and-api
./create_venv.sh
```

**Step 3.** Open repository directory in Visual Studio Code.

```bash
code -r .
```

**Step 4.** change ✅ IDs ✅ in `local.env` file by copying the IDs from Supervisely instance.

*   Copy your TEAM ID from context menu of the team.



    <pre class="language-python"><code class="lang-python"><strong>CONTEXT_TEAMID=8                 # ⬅️ change it</strong></code></pre>

    <figure><img src="https://user-images.githubusercontent.com/48913536/194893071-69757371-d15c-439b-89c0-9ad7d3c51118.png" alt=""><figcaption></figcaption></figure>
*   You can find your user ID and login at `Start` -> `Team members` page



    ```python
    CONTEXT_USERID=7                 # ⬅️ change it
    CONTEXT_USERLOGIN="my_username"  # ⬅️ change it
    ```

    <figure><img src="https://user-images.githubusercontent.com/48913536/194893088-59f87b0d-cba9-4bd2-9a1b-2b2227fd0efb.png" alt=""><figcaption></figcaption></figure>

**Step 5.** Start debugging `examples/user-automation/main.py`

## User automation

### Import libraries

```python
import os
from dotenv import load_dotenv
import supervisely as sly
```

### Init API client

​Init API for communicating with Supervisely Instance. First, we load environment variables with credentials:

```python
load_dotenv("local.env")
load_dotenv(os.path.expanduser("~/supervisely.env"))
api = sly.Api()
```

### Get your IDs and username from environment

```python
TEAM_ID = int(os.environ["CONTEXT_TEAMID"])
USER_ID = int(os.environ["CONTEXT_USERID"])
USER_LOGIN = os.environ["CONTEXT_USERLOGIN"]
```

### List available roles on Supervisely instance

Print all roles that are available on Supervisely instance.

```python
roles = api.role.get_list()
print(roles)
```

Output:

```
[
  RoleInfo(id=1, role='admin', created_at='2019-04-11T10:52:06.517Z', updated_at='2019-04-11T10:52:06.517Z'),
  RoleInfo(id=2, role='developer', created_at='2019-04-11T10:52:06.517Z', updated_at='2019-04-11T10:52:06.517Z'),
  RoleInfo(id=3, role='annotator', created_at='2019-04-11T10:52:06.517Z', updated_at='2019-04-11T10:52:06.517Z'),
  RoleInfo(id=4, role='viewer', created_at='2019-04-11T10:52:06.517Z', updated_at='2019-04-11T10:52:06.517Z')
]
```

#### Get UserInfo about yourself

```python
my_info = api.user.get_my_info()
print("my login:", my_info.login)
print("my ID:", my_info.id)
```

Output:

```
my login: my_username
my ID: 100
```

### Get UserInfo by ID

Get general information about user by ID.

**Note:** requires admin permission if requested ID is not equal to user id

```python
user_info = api.user.get_info_by_id(USER_ID)
print(user_info)
```

Output:

```
UserInfo(
  id=100, 
  login='my_username', 
  name='user', 
  email=None, 
  logins=7, 
  disabled=False, 
  last_login='2019-08-02T09:18:09.155Z', 
  created_at='2019-04-11T10:59:50.472Z', 
  updated_at='2019-08-04T17:06:39.414Z'
)
```

### List all team users with corresponding roles

```python
team = api.team.get_info_id(TEAM_ID)
members = api.user.get_team_members(team.id)
print(f"All members in team: '{team.name}'")
print(members)
```

Output:

```
All members in team 'my_team'
[
  UserInfo(
    id=4, 
    login='max', 
    name='max_k', 
    email=None, 
    logins=7, 
    disabled=False,
    last_login='2019-08-02T09:18:09.155Z',
    created_at='2019-04-11T10:59:50.472Z',
    updated_at='2019-08-05T08:42:20.463Z'
  ),

  ...

  UserInfo(
    id=31, 
    login='labeler_03',
    name='',
    email=None, 
    logins=0, 
    disabled=False,
    last_login=None,
    created_at='2019-07-20T15:12:52.779Z',
    updated_at='2019-07-20T15:12:52.779Z'
  )
]
```

## Methods that require admin permisssion

### List all registered users

Print all registered users on Supervisely instance.

```python
users = api.user.get_list()
print('Total number of users: ', len(users))
for user in users:
    print("Id: {:<5} Login: {:<25s} logins_count: {:<5}".format(user.id, user.login, user.logins))
```

Output:

```
Total number of users:  100
Id: 1     Login: admin                     logins_count: 88   
Id: 2     Login: supervisely               logins_count: 55    
...  
Id: 99    Login: alex                      logins_count: 0    
Id: 100   Login: my_username               logins_count: 3
```

### Get UserInfo by login

Get general information about user by name.

```python
user_info = api.user.get_info_by_login(USER_LOGIN)
print(user_info)
```

Output:

```
UserInfo(
  id=100, 
  login='my_username', 
  name='user', 
  email=None, 
  logins=7, 
  disabled=False, 
  last_login='2019-08-02T09:18:09.155Z', 
  created_at='2019-04-11T10:59:50.472Z', 
  updated_at='2019-08-04T17:06:39.414Z'
)
```

### List all user teams with corresponding roles

Print user teams, with user's role in them.

```python
def print_user_teams(login):
    user = api.user.get_info_by_login(login)
    user_teams = api.user.get_teams(user.id)
    print("\nTeams of user {!r}:".format(login))
    for team in user_teams:
        print("[team_id={}] {:<25s} [role_id={}] {}".format(team.id, team.name, team.role_id, team.role))

print_user_teams(USER_LOGIN)
```

Output:

```
Teams of user 'my_username':
[team_id=7] team_x                    [role_id=1] admin
[team_id=5] my_team                   [role_id=1] admin
[team_id=3] jupyter_tutorials         [role_id=1] admin
```

### Create new user

```python
new_user = api.user.get_info_by_login('demo_user_451')
if new_user is None:
    new_user = api.user.create(login='demo_user_451', password='123abc', is_restricted=False)
print(new_user)
```

Output:

```
UserInfo(
  id=101, 
  login='demo_user_451',
  name='',
  email=None,
  logins=0,
  disabled=False,
  last_login=None,
  created_at='2019-07-19T09:44:45.750Z',
  updated_at='2019-08-03T16:17:15.228Z'
)
```

### Update user info

Change user's name and password.

```python
new_password = 'abc123'
new_name = 'Bob'
user_info = api.user.update(user_info.id, password=new_password, name=new_name)
print(user_info)
```

Output:

```
UserInfo(
  id=101, 
  login='demo_user_451',
  name='Bob',
  email=None,
  logins=0,
  disabled=False,
  last_login=None,
  created_at='2019-07-19T09:44:45.750Z',
  updated_at='2019-08-03T16:17:15.228Z'
)
```

### Disable/Enable user

```python
api.user.disable(new_user.id)
api.user.enable(new_user.id)
```

### Invite user to team

```python
user = api.user.get_info_by_login('demo_user_451')
team = api.team.get_info_by_id(TEAM_ID)
if api.user.get_team_role(user.id, team.id) is None:
    api.user.add_to_team(user.id, team.id, api.role.DefaultRole.ANNOTATOR)
```

Output:

```
Teams of user 'demo_user451':
[team_id=22] demo_user451              [role_id=1] admin
[team_id=4] my_team                    [role_id=3] annotator
```

### Change user role in team

```python
user = api.user.get_info_by_login('demo_user_451')
team = api.team.get_info_by_id(TEAM_ID)
api.user.change_team_role(user.id, team.id, api.role.DefaultRole.VIEWER)
print_user_teams('demo_user_451')
```

Output:

```
Teams of user 'demo_user451':
[team_id=22] demo_user451              [role_id=4] viewer
[team_id=4] max                        [role_id=1] admin
```

### Remove user from team

```python
team = api.team.get_info_by_id(TEAM_ID)
user = api.user.get_info_by_login('demo_user_451')
api.user.remove_from_team(user.id, team.id)
print_user_teams('demo_user_451')
```

Output:

```
Teams of user 'demo_user451':
[team_id=22] demo_user451              [role_id=4] viewer
```
