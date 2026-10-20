---
title: Linux User and Group Management Guide
slug: managing-users-and-groups
author: Ajay Singh
excerpt: A practical Linux administration cheatsheet covering users, groups, passwords, sudo privileges, file ownership, permissions, ACLs, account security, and access troubleshooting.
tags: ["Linux", "Access Management", "SysAdmin"]
created_at: 2024-02-22T06:42:27Z
---

Linux is a multi-user operating system in which user accounts, groups, ownership, and permissions control access to files, directories, processes, and administrative operations.

This guide focuses primarily on Ubuntu and Debian-based systems while covering commands available on most Linux distributions. Use it as a practical reference for administering user accounts, assigning privileges, securing access, and troubleshooting permission problems.

## 1. Understanding Users, Groups, and IDs

Every Linux user account has a username and a numeric user ID (UID). Groups have names and numeric group IDs (GIDs).

Linux uses these IDs to determine ownership and access permissions.

| Concept | Description |
| :-- | :-- |
| UID | Numeric identifier for a user |
| GID | Numeric identifier for a group |
| Primary group | The user's default group, commonly used for newly created files |
| Supplementary groups | Additional groups that grant membership-based access |
| Home directory | Location for a user's personal files and configuration |
| Login shell | Program started for an interactive login |
| System account | Account intended for operating-system services or processes |
| Root | Superuser account with UID `0` |

### Important account files

| File | Purpose |
| :-- | :-- |
| `/etc/passwd` | User account records |
| `/etc/shadow` | Password hashes and password-aging information |
| `/etc/group` | Group records |
| `/etc/gshadow` | Protected group administration information |
| `/etc/sudoers` | Sudo policy configuration |
| `/etc/sudoers.d/` | Additional sudo policy files |
| `/etc/skel/` | Default files copied into newly created home directories |

Avoid editing account databases directly. Use the account-management utilities so the files remain consistent.

## 2. Listing and Inspecting Users

### List local user accounts

```bash
# List usernames
cut -d: -f1 /etc/passwd

# Display complete account records
getent passwd

# Find a specific user
getent passwd username

# Count local passwd-file entries
wc -l < /etc/passwd
```

`getent` can also retrieve accounts provided by configured directory services, such as LDAP. The contents of `/etc/passwd` alone may not represent every account known to the system.

### Understand a passwd entry

Example:

```text
ajay:x:1000:1000:Ajay Singh:/home/ajay:/bin/bash
```

The fields are:

1. Username: `ajay`
2. Password placeholder: `x`
3. UID: `1000`
4. Primary GID: `1000`
5. User description: `Ajay Singh`
6. Home directory: `/home/ajay`
7. Login shell: `/bin/bash`

The `x` indicates that password information is stored separately, typically in `/etc/shadow`.

### Inspect a user's account

```bash
id username
getent passwd username
groups username
```

For the currently logged-in user:

```bash
whoami
id
groups
```

### Find users by UID

```bash
# Find accounts with UID 0
awk -F: '$3 == 0 {print $1, $3, $6, $7}' /etc/passwd

# Display accounts with UID 1000 or higher
awk -F: '$3 >= 1000 {print $1, $3, $6, $7}' /etc/passwd
```

UID ranges vary by distribution and configuration. A UID of 1000 or higher is a common convention for human users, not a universal rule.

## 3. Listing and Inspecting Groups

### List groups

```bash
cut -d: -f1 /etc/group
getent group
getent group groupname
```

### Inspect group membership

```bash
# Show the groups of a user
groups username

# Display numeric UID, primary GID, and supplementary groups
id username

# Inspect a particular group
getent group developers
```

Example group entry:

```text
developers:x:1050:ajay,alex
```

This identifies the group name and GID, followed by supplementary members listed in that group record. A user's primary group may not appear in the member list.

## 4. Creating User Accounts

### Create a user interactively on Ubuntu

```bash
sudo adduser newusername
```

Ubuntu's `adduser` utility prompts for a password and optional user information. It normally creates a home directory and a matching primary group.

### Create a user with `useradd`

```bash
sudo useradd -m -s /bin/bash newusername
sudo passwd newusername
```

- `-m`: Create a home directory.
- `-s`: Set the login shell.

`useradd` is a lower-level utility; defaults and behavior can differ from Ubuntu's interactive `adduser`.

### Create a user with a custom home directory

```bash
sudo useradd -m -d /srv/home/newusername -s /bin/bash newusername
sudo passwd newusername
```

Check that the parent directory exists and that the chosen location is appropriate for home directories.

### Create a system account

For service accounts, use the distribution's recommended service-account procedure. A common pattern is:

```bash
sudo useradd --system --no-create-home --shell /usr/sbin/nologin appservice
```

A system account generally should not have an interactive login unless the service specifically requires it.

### Verify the new account

```bash
id newusername
getent passwd newusername
sudo passwd -S newusername
```

## 5. Modifying Existing Users

### Change a username

```bash
sudo usermod -l newusername oldusername
```

Changing the username does not automatically rename the home directory or update every application reference to the old name.

### Change the home directory

```bash
sudo usermod -d /home/newusername -m newusername
```

The `-m` option moves the contents of the old home directory to the new location when applicable. Review existing paths, mounts, permissions, and application dependencies first.

### Change the login shell

```bash
sudo usermod -s /bin/bash username
```

List permitted shells:

```bash
cat /etc/shells
```

### Set or change the user's descriptive name

```bash
sudo chfn username
```

The available fields depend on the system's account-management configuration.

### Change the user's primary group

```bash
sudo usermod -g groupname username
```

This changes the primary group recorded for the account. It does not automatically change the group ownership of all existing files.

### Lock or unlock an account password

```bash
sudo passwd -l username
sudo passwd -u username
```

Password locking is not a universal guarantee that all forms of access are disabled. SSH keys, existing sessions, and other authentication methods may still permit access. Use the appropriate account and service controls when disabling access completely.

## 6. Deleting User Accounts

### Delete a user but retain their files

```bash
sudo deluser username
```

This removes the account but normally leaves the user's home directory and other files in place.

### Delete a user and their home directory

```bash
sudo deluser --remove-home username
```

**Warning:** This is destructive. Verify the username and back up any required data before running it.

### Find files owned by a deleted user

If the UID is known:

```bash
sudo find / -xdev -uid 1005 -ls 2>/dev/null
```

Replace `1005` with the deleted user's UID. This searches one filesystem; additional mounted filesystems may need separate inspection.

### Before deleting an account

- Check whether the user owns application files, scheduled jobs, or service processes.
- Preserve required data and audit records.
- Review SSH access, sudo rules, and group membership.
- Consider disabling access before deleting the account.
- Check for files owned by the UID outside the home directory.

## 7. Creating and Managing Groups

### Create a group

```bash
sudo groupadd developers
```

### Rename a group

```bash
sudo groupmod -n engineers developers
```

### Change a group's GID

```bash
sudo groupmod -g 1050 developers
```

Changing a GID may leave existing files owned by the old numeric GID. Review ownership before making this change.

### Delete a group

```bash
sudo groupdel developers
```

A group cannot normally be deleted while it is the primary group of a user. Review group usage and ownership before removal.

### Add a user to a supplementary group

```bash
sudo usermod -aG developers username
```

The `-a` option is essential when using `-G` to append membership. Without `-a`, the supplementary group list can be replaced, potentially removing the user from important groups.

Ubuntu also supports:

```bash
sudo adduser username developers
```

### Remove a user from a group

```bash
sudo gpasswd -d username developers
```

Verify membership:

```bash
id username
getent group developers
```

A user may need to start a new login session for the updated group membership to take effect in their processes.

## 8. Password Management and Expiration

### Change your own password

```bash
passwd
```

### Set another user's password

```bash
sudo passwd username
```

### Inspect password status

```bash
sudo passwd -S username
sudo chage -l username
```

### Configure password aging

```bash
# Set a maximum password age of 90 days
sudo chage -M 90 username

# Set a minimum password age of 1 day
sudo chage -m 1 username

# Warn 7 days before expiration
sudo chage -W 7 username

# Set an account expiration date
sudo chage -E 2027-12-31 username
```

Check the resulting policy:

```bash
sudo chage -l username
```

Account expiration and password expiration are different controls. Password-aging policies should reflect your organization's security requirements rather than being applied blindly.

### Require a password change at next login

```bash
sudo chage -d 0 username
```

## 9. Sudo and Administrative Privileges

Sudo allows authorized users to execute commands with elevated privileges.

### Add a user to the Ubuntu sudo group

```bash
sudo usermod -aG sudo username
```

The user should start a new session before relying on the new membership.

### Verify sudo access

```bash
id username
sudo -l -U username
```

The second command displays the sudo privileges configured for that user when run by an authorized administrator.

### Run a command as another user

```bash
sudo -u username whoami
sudo -i
```

- `sudo -u username`: Run a command as the specified user.
- `sudo -i`: Start a root login shell.

Exit the elevated shell when finished.

### Configure a dedicated sudo rule

Use `visudo` to validate sudo policy syntax:

```bash
sudo visudo -f /etc/sudoers.d/developers
```

Example rule:

```text
%developers ALL=(ALL:ALL) /usr/bin/systemctl status nginx
```

This grants members of the `developers` group permission to run the specified command through sudo, subject to the exact sudoers matching rules. Verify the command path and policy carefully.

Avoid granting unrestricted sudo access unless it is genuinely required. Never edit `/etc/sudoers` or a sudoers drop-in with an ordinary editor without syntax validation.

## 10. File Ownership and Permissions

Linux permissions determine which users can read, write, or execute files and directories.

### Inspect ownership and permissions

```bash
ls -l filename
ls -ld /home/username
stat filename
```

Example:

```text
-rw-r----- 1 ajay developers 1024 Oct 20 10:00 report.txt
```

The permission string contains:

- File type: `-` for a regular file, `d` for a directory.
- Owner permissions: `rw-`.
- Group permissions: `r--`.
- Other-user permissions: `---`.

### Change the owner

```bash
sudo chown username filename
```

### Change owner and group

```bash
sudo chown username:developers filename
```

### Change group ownership

```bash
sudo chgrp developers filename
```

### Change ownership recursively

```bash
sudo chown -R username:developers /srv/project
```

Use recursive ownership changes carefully. A mistaken path can modify large parts of the system. Verify the target directory before executing the command.

## 11. Understanding and Changing Permissions

The basic permission values are:

| Permission | Numeric value | Regular file | Directory |
| :-- | --: | :-- | :-- |
| Read (`r`) | 4 | Read file contents | List directory entries |
| Write (`w`) | 2 | Modify file contents | Create or remove entries, subject to other permissions |
| Execute (`x`) | 1 | Execute a program or script | Traverse or enter the directory |

### Common numeric modes

| Mode | Meaning |
| :-- | :-- |
| `600` | Owner read/write only |
| `640` | Owner read/write; group read |
| `644` | Owner read/write; others read |
| `700` | Owner has full access |
| `750` | Owner full; group read/execute |
| `755` | Owner full; others read/execute |
| `770` | Owner and group full access |
| `775` | Owner and group full; others read/execute |
| `777` | Everyone has full permissions; usually unsafe |

### Change permissions using numeric mode

```bash
chmod 640 filename
chmod 750 script.sh
```

### Change permissions symbolically

```bash
# Add execute permission for the owner
chmod u+x script.sh

# Remove write permission for others
chmod o-w filename

# Give the group read and execute permission
chmod g+rx directory

# Remove all permissions for others
chmod o-rwx filename
```

### Apply permissions recursively

```bash
# Directories
find /srv/project -type d -exec chmod 750 {} +

# Regular files
find /srv/project -type f -exec chmod 640 {} +
```

These are example policies, not universal defaults. Executable files, public assets, private keys, and application files may require different permissions.

## 12. Special Permissions: SUID, SGID, and Sticky Bit

### Set-user-ID (SUID)

For executable files, SUID can cause the program to run with the file owner's effective UID.

```bash
chmod u+s executable
```

SUID has security implications. Do not set it on arbitrary executables.

### Set-group-ID (SGID)

On a directory, SGID causes newly created files and subdirectories to inherit the directory's group on typical Linux filesystems.

```bash
sudo chgrp developers /srv/shared
sudo chmod 2770 /srv/shared
```

This is useful for shared team directories.

### Sticky bit

On a shared directory, the sticky bit restricts deletion and renaming of entries to their owners, the directory owner, or a privileged user.

```bash
sudo chmod 1777 /srv/shared-temp
```

The `/tmp` directory commonly uses sticky-bit permissions. Do not use world-writable directories without understanding the security consequences.

## 13. Default Permissions: umask

The `umask` determines which permission bits are removed from newly created files and directories.

### View the current umask

```bash
umask
umask -S
```

### Set a temporary umask

```bash
umask 027
```

This affects newly created files and directories in the current shell and its descendants.

With a typical creation mode, `umask 027` commonly results in files with mode `640` and directories with mode `750`. Programs can choose more restrictive creation modes, so the final permissions may differ.

Configure a persistent umask through the appropriate shell or system policy for your environment.

## 14. Access Control Lists (ACLs)

ACLs provide more granular permissions than the traditional owner/group/other model.

### Inspect ACLs

```bash
getfacl filename
getfacl /srv/shared
```

### Grant a user access to a file

```bash
sudo setfacl -m u:username:r-- filename
```

### Grant a group access to a directory

```bash
sudo setfacl -m g:developers:rwx /srv/shared
```

### Set default ACLs for new directory entries

```bash
sudo setfacl -d -m g:developers:rwx /srv/shared
```

Default ACLs are inherited by newly created entries under the directory, subject to the creating program's requested permissions and filesystem behavior.

### Remove ACL entries

```bash
sudo setfacl -x u:username filename
sudo setfacl -b filename
```

`setfacl -b` removes extended ACL entries. Review the result before using it on directories with deliberate access-control policies.

## 15. Shared Directory for a Team

A shared directory can use a dedicated group and SGID to make collaboration easier.

### Create the group and directory

```bash
sudo groupadd developers
sudo mkdir -p /srv/shared
sudo chown root:developers /srv/shared
sudo chmod 2770 /srv/shared
```

### Add team members

```bash
sudo usermod -aG developers username
```

Each member should start a new login session to receive the updated group membership.

### Verify the configuration

```bash
ls -ld /srv/shared
getent group developers
id username
```

Expected directory permissions typically begin with `drwxrws---`. The `s` indicates SGID, which helps new entries inherit the shared group.

Note that SGID does not automatically guarantee group write permission on every new file. If consistent group write access is required, consider a suitable `umask` and default ACL.

## 16. SSH Access and User Authentication

### Inspect SSH authorized keys

For a user:

```bash
sudo ls -la /home/username/.ssh
sudo cat /home/username/.ssh/authorized_keys
```

Review the keys carefully; avoid publishing their contents.

### Set secure ownership and permissions

```bash
sudo chown -R username:username /home/username/.ssh
sudo chmod 700 /home/username/.ssh
sudo chmod 600 /home/username/.ssh/authorized_keys
```

Adjust the group ownership if the account's primary group is not named after the user.

### Test SSH configuration

```bash
sudo sshd -t
sudo systemctl status ssh
```

On Ubuntu, the service is commonly named `ssh`; other distributions may use `sshd`.

### Disable an account's access

Account removal or password locking alone may not disable every authentication method. For SSH access, review:

- Authorized public keys.
- `AllowUsers` and `AllowGroups` directives.
- Sudo privileges.
- Existing login sessions.
- Service accounts and other authentication providers.

Keep an administrative recovery path when modifying SSH access on a remote system.

## 17. Login History and Active Sessions

### View logged-in users

```bash
who
w
users
```

### View recent login history

```bash
last
last username
```

### View failed login attempts

```bash
sudo lastb
```

Availability and the amount of history depend on the distribution, audit configuration, and log retention.

### Inspect a user's processes

```bash
ps -u username -f
pgrep -a -u username
```

Before terminating processes during account removal, determine whether they belong to active workloads or services.

## 18. Account Locking and Access Revocation

### Lock a password

```bash
sudo passwd -l username
```

### Expire an account

```bash
sudo chage -E 2026-10-21 username
```

The example date is illustrative; choose the intended expiration date.

### Unlock a password

```bash
sudo passwd -u username
```

Unlocking a password does not necessarily restore an expired account or reverse other restrictions.

For a full offboarding process, review authentication, SSH keys, group membership, sudo rules, running processes, scheduled jobs, and owned files. Choose controls appropriate to the access method and service involved.

## 19. Troubleshooting Permission Problems

### Identify the current user and groups

```bash
whoami
id
groups
```

### Inspect the target and its parent directories

```bash
ls -ld /srv /srv/project /srv/project/file.txt
namei -l /srv/project/file.txt
```

A user needs directory traversal permission on each parent directory to access a path.

### Inspect ACLs

```bash
getfacl /srv/project
getfacl /srv/project/file.txt
```

### Check file ownership and mode

```bash
stat /srv/project/file.txt
```

### Check filesystem mount options

```bash
findmnt /srv/project
```

A read-only mount or filesystem-level restriction can prevent writes even when Unix permissions appear correct.

### Common causes of access errors

- The user is not a member of the required group.
- The user has not started a new session after a group change.
- A parent directory lacks execute permission.
- An ACL grants or restricts access.
- The filesystem is mounted read-only.
- The file is on a network share with its own permission mapping.
- SELinux or AppArmor policy blocks the operation.
- The application is running under a different UID or GID than expected.

Use the relevant security and filesystem diagnostics rather than applying broad permissions such as `chmod -R 777`.

## 20. Quick Reference

| Task | Command |
| :-- | :-- |
| Identify current user | `whoami` |
| Show UID and groups | `id username` |
| List users | `getent passwd` |
| List groups | `getent group` |
| Create Ubuntu user | `sudo adduser username` |
| Delete user | `sudo deluser username` |
| Create group | `sudo groupadd groupname` |
| Add supplementary group | `sudo usermod -aG groupname username` |
| Remove group membership | `sudo gpasswd -d username groupname` |
| Change password | `sudo passwd username` |
| Inspect password aging | `sudo chage -l username` |
| Grant Ubuntu sudo group access | `sudo usermod -aG sudo username` |
| Change owner | `sudo chown username file` |
| Change group | `sudo chgrp groupname file` |
| Change permissions | `chmod 640 file` |
| Inspect ACLs | `getfacl file` |
| Inspect login history | `last` |
| List active sessions | `who` |
| Inspect path permissions | `namei -l /path/to/file` |

## 21. Security Best Practices

- Apply the principle of least privilege: grant only the permissions needed for each task.
- Use individual accounts rather than sharing administrative credentials.
- Grant sudo access selectively and validate policy changes with `visudo`.
- Use strong authentication and SSH keys where appropriate.
- Protect `/etc/shadow`, private keys, and sensitive application files.
- Review supplementary groups and sudo rules regularly.
- Avoid unnecessary SUID executables and world-writable directories.
- Back up important data before deleting users or changing ownership recursively.
- Verify account and permission changes using `id`, `stat`, `getfacl`, and `namei`.
- Establish an account-offboarding process that includes keys, active sessions, service access, and owned files.

User and group management is not just about creating accounts; it is about ensuring that every account has the appropriate access and that privileges can be audited, maintained, and revoked safely.
