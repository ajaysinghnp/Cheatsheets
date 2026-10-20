---
title: Linux User and Group Management Guide
slug: managing-users-and-groups
author: ajay-singh
excerpt: Linux commands to manage users and groups
tags: ["Linux", "Access Management", "SysAdmin"]
created_at: 2024-02-22T06:42:27Z
---

# Note for managing Users and Groups in the Ubuntu OS

Ubuntu, like any other multi-user operating system, implements a system of file permissions and ownership as part of its security protocol. Managing users and groups in Ubuntu is crucial for setting up a system’s access rights and ensuring the appropriate segregation of rights. Whether you are an Ubuntu novice or an experienced administrator, knowing how to effectively manage users and groups is key to maintaining a secure system.

This note will give you an in-depth understanding of user and group management in Ubuntu, taking you through various commands and tools that could be used, illustrated by multiple examples.

## Understanding Users and Groups

In Ubuntu, each user has a username and a numeric user ID (UID). Similarly, groups contain zero or more users and are identified by both group names and group IDs (GIDs). When a user creates a file, it is owned by that user and their primary group.

## Listing User Groups

To list all the users on your system, you can use the following command:

```bash
cut -d: -f1 /etc/passwd
```

To list all the groups, you can use:

```bash
cut -d: -f1 /etc/group
```

## Adding Users

To add a new user to your system, use the adduser command:

```bash
sudo adduser newusername
```

After running this command, you will be prompted to set a password and optional user information. An associated group will automatically be created for the new user.

## Modifying Users

To change the details of an existing user, such as their username, you can use the usermod command:

```bash
sudo usermod -l newusername oldusername
```

To add a user to a supplementary group, you can use:

```bash
sudo usermod -aG groupname username
```

## Deleting Users

To remove a user from your system, the deluser command is used:

```bash
sudo deluser username
```

If you also wish to remove their home directory and mail spool, use the `--remove-home` option:

```bash
sudo deluser --remove-home username
```

## Working with Groups

Similar to users, you can `add`, `modify` and `delete` groups with `groupadd`, `groupmod`, and `groupdel` commands.

### Creating Groups

```bash
sudo groupadd groupname
```

### Modifying Groups

```bash
sudo groupmod -n newgroupname oldgroupname
```

### Deleting Groups

```bash
sudo groupdel groupname
```

### Managing Group Membership

To add a user to a group, you can use:

```bash
sudo adduser username groupname
```

To view the groups a user is a part of, type:

```bash
groups username
```

Or, to see the numeric user and group IDs, use the id command:

```bash
id username
```

## Advanced Management

### Managing Passwords

Passwords can be changed using passwd. To change another user’s password, you need superuser privileges:

```bash
sudo passwd username
```

### User Login Information

To display the last login information for all users, you can use:

```bash
lastlog
```

### Changing File Ownership and Permissions

To change the owner of a file, use:

```bash
sudo chown username filename
```

To change the group of a file, use:

```bash
sudo chgrp groupname filename
```

Modifying permissions can be done with the chmod command. For instance, to give the owner of a file execute permission:

```bash
chmod u+x filename
```
