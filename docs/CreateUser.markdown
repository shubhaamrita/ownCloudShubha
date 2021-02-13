---
layout: page
title: "Create User"
permalink: /usercreation/
---

This guide provides instructions on creating a user in ownCloud
## User Authentication 
Configure additional user backends in ownCloud’s configuration file ```(config/config.php)``` using the following syntax:
```
<?php

"user_backends" => [
    0 => [
        "class"     => ...,
        "arguments" => [
            0 => ...
        ],
    ],
],
```
External user support app (user_external), which is not enabled by default, provides three backends:
- [IMAP](#imap)
- [SMB](#smb)
- [FTP](#ftp)

### IMAP

| Option | Value/Description |
| --------------| ---------|
| Class | OC_User_IMAP
| Arguments | A mailbox string as defined in the PHP documentation|
| Dependency | PHP’s IMAP extension.See Manual Installation on Linux|

### SMB

| Option | Value/Description |
| --------------| ---------|
| Class | OC_User_SMB
| Arguments | The samba server to authenticate against|
| Dependency | PECL’s smbclient extension or smbclient|

### FTP

| Option | Value/Description |
| --------------| ---------|
| Class | OC_User_FTP
| Arguments | The FTP server to authenticate against|
| Dependency | PHP’s FTP extension.See Source Installation|

## Create a user account
- Open Default View of Users screen
![N|Solid](https://doc.owncloud.com/server/10.6/admin_manual/_images/configuration/user/users-page.png)

-	Enter the new user’s Login Name and E-Mail in Username and Email textbox
-	Optionally, assign Groups memberships
![N|Solid](https://doc.owncloud.com/server/10.6/admin_manual/_images/configuration/user/users-page-new-user.png)
-	Click the Create button

#### User is now created. You will be able to view the created user in **Default View**