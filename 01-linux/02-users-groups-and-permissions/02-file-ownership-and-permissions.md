# Linux File Ownership and Permissions

> **Note:** This documentation uses `linuxUser` as a generic example username.  
> On the laboratory system, the real username may be different.

## Lesson Result

```text
Module: 02-users-groups-and-permissions
Lesson: 02-file-ownership-and-permissions
Status: PASSED
```

This lesson explains how Linux associates files and directories with owners, groups and permission sets.

The practical challenge focused on investigating a `Permission denied` error without using:

```text
sudo
chmod
chown
chgrp
```

The objective was to understand the problem before attempting to fix it.

---

## What You Will Learn

This lesson covers:

- file ownership;
- group ownership;
- permission categories;
- file types;
- symbolic permissions;
- read, write and execute permissions;
- differences between file and directory permissions;
- directory traversal;
- evaluation order between owner, group and others;
- inspection with `ls -l`;
- inspection of directories with `ls -ld`;
- troubleshooting `Permission denied`;
- the relationship between readable and executable files.

Main commands:

```bash
ls -l
ls -ld
whoami
id
cat
printf
```

---

## Linux Ownership Model

Every ordinary Linux file or directory normally has:

- an owner;
- an associated group;
- permissions for the owner;
- permissions for the group;
- permissions for all other users.

Example:

```text
-rw-r----- 1 linuxUser developers 128 Jul 27 10:30 app.conf
```

The most important parts are:

```text
-rw-r-----
linuxUser
developers
app.conf
```

Meaning:

```text
Type and permissions: -rw-r-----
Owner:                linuxUser
Group:                developers
Filename:             app.conf
```

Ownership and permissions are separate concepts.

The owner is an identity.

The group is an identity collection.

The permission string describes what each category may do.

---

## Why Ownership and Permissions Matter

When an application cannot access a file, the existence of the file alone is not enough.

A complete investigation should ask:

```text
Which user runs the application?
Which groups does the user belong to?
Who owns the file?
Which group owns the file?
Which permission category applies?
Are all parent directories traversable?
Should the process have access?
```

A generic error such as:

```text
Permission denied
```

may be caused by:

- missing read permission;
- missing write permission;
- missing execute permission;
- incorrect ownership;
- incorrect group ownership;
- missing traversal permission on a parent directory;
- a process running under an unexpected user;
- a file owned by `root`;
- a service account that was not granted required access.

Using:

```bash
sudo
```

without understanding the problem may hide the real cause.

Using:

```bash
chmod 777
```

may remove the immediate restriction while creating an unnecessary security risk.

---

## Reading `ls -l`

The command:

```bash
ls -l
```

shows detailed information about files.

Example:

```text
-rw-r--r-- 1 linuxUser linuxUser 42 Jul 27 11:00 notes.txt
```

Fields:

| Field | Meaning |
|---|---|
| `-rw-r--r--` | Type and permissions |
| `1` | Hard-link count |
| `linuxUser` | Owner |
| `linuxUser` | Group |
| `42` | Size in bytes |
| `Jul 27 11:00` | Last modification |
| `notes.txt` | Filename |

The permission string must not be confused with the owner or group names.

Correct interpretation:

```text
-rw-rw-r-- 1 linuxUser linuxUser service.conf
│           │         │
│           │         └── group
│           └──────────── owner
└──────────────────────── type and permissions
```

---

## File Types

The first character of the permission string identifies the element type.

Common values:

```text
- = regular file
d = directory
l = symbolic link
```

Examples:

```text
-rw-r--r--  regular file
drwxr-xr-x  directory
lrwxrwxrwx  symbolic link
```

The first character is not part of the three permission groups.

For:

```text
drwxr-xr-x
```

the structure is:

```text
d | rwx | r-x | r-x
```

Meaning:

```text
d     directory
rwx   owner permissions
r-x   group permissions
r-x   others permissions
```

---

## Owner, Group and Others

The nine permission characters after the file type are divided into three categories:

```text
user | group | others
```

They can also be described as:

```text
owner | associated group | everyone else
```

Example:

```text
-rwxr-x---
```

Separation:

```text
- | rwx | r-x | ---
```

Meaning:

```text
Owner:
- read
- write
- execute

Group:
- read
- execute

Others:
- no permissions
```

The three categories are not:

```text
r
w
x
```

Those are permission types, not identity categories.

---

## Read, Write and Execute

The symbolic permissions are:

```text
r = read
w = write
x = execute
- = permission absent
```

Their meaning depends on whether the target is a file or directory.

---

## Permissions on Files

For regular files:

```text
r = read the file content
w = modify the file content
x = execute the file as a program or script
```

Example:

```text
-rw-r-----
```

Separation:

```text
- | rw- | r-- | ---
```

Meaning:

```text
Owner:
- may read
- may modify
- may not execute

Group:
- may read
- may not modify
- may not execute

Others:
- no access
```

A readable file is not automatically executable.

A file can contain valid Bash code but still fail when launched directly if the applicable permission block does not contain `x`.

---

## Permissions on Directories

For directories:

```text
r = list directory names
w = create, remove or rename entries
x = traverse the directory and access entries by name
```

This is different from file permissions.

Example:

```text
drwxr-x---
```

Separation:

```text
d | rwx | r-x | ---
```

Meaning:

```text
Owner:
- may list entries
- may modify directory entries
- may traverse the directory

Group:
- may list entries
- may traverse the directory
- may not modify entries

Others:
- no access
```

---

## Directory Read Permission

The permission:

```text
r
```

on a directory allows reading the directory listing.

It makes it possible to see the names of entries, subject to the command and other permissions.

It does not automatically guarantee access to the files inside.

---

## Directory Write Permission

The permission:

```text
w
```

on a directory controls the ability to modify its entries.

This includes operations such as:

- creating a new file;
- deleting a file;
- renaming a file;
- creating a subdirectory;
- removing an entry.

For practical directory modification, `w` is normally used together with `x`.

---

## Directory Execute Permission

The permission:

```text
x
```

on a directory means traversal.

It allows a process to pass through the directory and access known entries by name.

For a path such as:

```text
/home/linuxUser/project/config/app.conf
```

the applicable user must be able to traverse:

```text
/home
/home/linuxUser
/home/linuxUser/project
/home/linuxUser/project/config
```

If one parent directory lacks the required `x` permission, access to the final file may fail even if the file itself is readable.

---

## File Deletion

The ability to delete a file depends mainly on the directory containing it.

Suppose the target is:

```text
project/logs/service.log
```

The important permissions for removing `service.log` are primarily those of:

```text
project/logs/
```

Usually, deletion requires:

```text
w + x on the parent directory
```

The file itself does not need the `x` permission to be deleted.

The file's own `w` permission is not the primary control for removing its directory entry.

This is one of the most important differences between file-content permissions and directory-entry permissions.

---

## Ownership Evaluation Order

When a process accesses a file, Linux determines which permission category applies.

In simplified form:

1. If the process UID matches the file owner, use the owner block.
2. Otherwise, if one of the process groups matches the file group, use the group block.
3. Otherwise, use the others block.

Linux does not combine the three blocks to find the most favorable result.

Example:

```text
----r----- 1 linuxUser developers app.conf
```

Separation:

```text
- | --- | r-- | ---
```

Suppose:

```text
Owner: linuxUser
Group: developers
```

and `linuxUser` is also a member of `developers`.

When `linuxUser` accesses the file, Linux applies the owner block:

```text
---
```

The user does not fall back to the group block:

```text
r--
```

Therefore the owner cannot read the file in this example.

---

## Inspecting Files with `ls -l`

To inspect an individual file:

```bash
ls -l app.conf
```

Example:

```text
-rw-rw-r-- 1 linuxUser linuxUser 34 Jul 27 21:14 app.conf
```

Interpretation:

```text
Type:        regular file
Owner:       linuxUser
Group:       linuxUser
Permissions: -rw-rw-r--
```

Permission separation:

```text
- | rw- | rw- | r--
```

Meaning:

```text
Owner: read and write
Group: read and write
Others: read only
```

---

## Inspecting Directories with `ls -ld`

The command:

```bash
ls -l directory
```

normally lists the directory contents.

The command:

```bash
ls -ld directory
```

shows the directory itself.

Example:

```bash
ls -ld permission-lab
```

Output:

```text
drwxrwxr-x 5 linuxUser linuxUser 4096 Jul 27 20:00 permission-lab
```

Interpretation:

```text
Type:        directory
Owner:       linuxUser
Group:       linuxUser
Permissions: drwxrwxr-x
```

Separation:

```text
d | rwx | rwx | r-x
```

Meaning:

```text
Owner:
- read
- write
- traverse

Group:
- read
- write
- traverse

Others:
- read
- traverse
- no write
```

---

## Practical Laboratory

A laboratory workspace was created:

```text
permission-lab/
├── configs/
│   └── app.conf
├── scripts/
│   └── deploy.sh
└── logs/
    └── application.log
```

Creation commands:

```bash
cd ~
mkdir -p permission-lab/{configs,scripts,logs}
```

Files:

```bash
touch permission-lab/configs/app.conf
touch permission-lab/scripts/deploy.sh
touch permission-lab/logs/application.log
```

---

## Configuration Content

```bash
printf "environment=development\nport=8080\n" \
  > permission-lab/configs/app.conf
```

Result:

```text
environment=development
port=8080
```

---

## Script Content

A safe quoting form was used to avoid history expansion problems with `!`:

```bash
printf '%s\n' \
  '#!/bin/bash' \
  'printf "Deployment started\n"' \
  > permission-lab/scripts/deploy.sh
```

Result:

```bash
#!/bin/bash
printf "Deployment started\n"
```

Using:

```bash
printf "#!/bin/bash\n..."
```

inside an interactive Bash session may trigger history expansion because of:

```text
!
```

and produce:

```text
event not found
```

Single quotes prevent Bash from interpreting the exclamation mark.

---

## Log Content

```bash
printf "INFO Application started\n" \
  > permission-lab/logs/application.log
```

---

## Identity Verification

Commands:

```bash
whoami
id
```

Observed information:

```text
UID: 1000
Username: linuxUser
Primary group: linuxUser
Supplementary groups: system-dependent
```

The exact values must be taken from the real system.

---

## Initial Ownership

Files were inspected with:

```bash
ls -l permission-lab/configs/app.conf
ls -l permission-lab/scripts/deploy.sh
ls -l permission-lab/logs/application.log
```

Example result:

```text
-rw-rw-r-- 1 linuxUser linuxUser app.conf
-rw-rw-r-- 1 linuxUser linuxUser deploy.sh
-rw-rw-r-- 1 linuxUser linuxUser application.log
```

The files had the same owner because they were created by processes running with the same user UID.

They had the same group because the creating process used the same primary group and no other group-inheritance rule changed the result.

---

## Initial Directory Permissions

Directories were inspected with:

```bash
ls -ld permission-lab
ls -ld permission-lab/configs
ls -ld permission-lab/scripts
ls -ld permission-lab/logs
```

Observed permissions:

```text
drwxrwxr-x
```

Separation:

```text
d | rwx | rwx | r-x
```

All categories had `x`, so all categories could traverse the directories.

Only owner and group had `w`, so others could not modify the directory entries.

---

## Symbolic Analysis

### `app.conf`

```text
- | rw- | rw- | r--
```

Meaning:

```text
Regular file
Owner: read and write
Group: read and write
Others: read
```

### `deploy.sh`

```text
- | rw- | rw- | r--
```

Meaning:

```text
Regular file
Owner: read and write
Group: read and write
Others: read
No execute permission
```

### `application.log`

```text
- | rw- | rw- | r--
```

Meaning:

```text
Regular file
Owner: read and write
Group: read and write
Others: read
```

### `permission-lab/`

```text
d | rwx | rwx | r-x
```

Meaning:

```text
Directory
Owner: full directory access
Group: full directory access
Others: read and traverse
```

---

## Execution Test

The script was launched with:

```bash
./permission-lab/scripts/deploy.sh
```

Result:

```text
Permission denied
```

The file permissions were:

```text
-rw-rw-r--
```

The owner block was:

```text
rw-
```

The script was:

- readable;
- writable;
- not executable.

The missing permission was:

```text
x
```

Specifically:

```text
owner execute permission
```

The script content could still be viewed using:

```bash
cat permission-lab/scripts/deploy.sh
```

This demonstrates:

```text
Readable does not mean executable.
```

---

## Path Traversal Analysis

For the file:

```text
/home/linuxUser/permission-lab/configs/app.conf
```

the following directories must be traversable:

```text
/home
/home/linuxUser
/home/linuxUser/permission-lab
/home/linuxUser/permission-lab/configs
```

The target file requires read permission for the applicable category.

Summary:

```text
Directories in path -> x
Target file to read -> r
```

---

## Permission Investigation Challenge

### Scenario

The following project structure was created:

```text
permission-investigation/
├── config/
│   └── service.conf
├── scripts/
│   └── start-service.sh
├── logs/
│   └── service.log
└── report/
    └── analysis.txt
```

The goal was to investigate a failed script execution without correcting ownership or permissions.

Restrictions:

```text
No sudo
No chmod
No chown
No chgrp
No recursive deletion
```

The challenge was an investigation, not a repair.

---

## Creating the Challenge Structure

```bash
mkdir -p permission-investigation/{config,scripts,logs,report}
```

Configuration file:

```bash
printf "environment=development\nport=8080\n" \
  > permission-investigation/config/service.conf
```

Script:

```bash
printf '%s\n' \
  '#!/bin/bash' \
  'printf "Deployment started\n"' \
  > permission-investigation/scripts/start-service.sh
```

Log file:

```bash
printf "INFO Application started\n" \
  > permission-investigation/logs/service.log
```

---

## Inspecting Challenge Directories

Command:

```bash
ls -ld permission-investigation
ls -ld permission-investigation/*
```

Observed output:

```text
drwxrwxr-x permission-investigation
drwxrwxr-x permission-investigation/config
drwxrwxr-x permission-investigation/logs
drwxrwxr-x permission-investigation/report
drwxrwxr-x permission-investigation/scripts
```

All directories were:

```text
Owner: linuxUser
Group: linuxUser
Permissions: drwxrwxr-x
```

---

## Inspecting Challenge Files

Command:

```bash
ls -l permission-investigation/*
```

Observed file permissions:

```text
-rw-rw-r-- service.conf
-rw-rw-r-- service.log
-rw-rw-r-- start-service.sh
```

All files were owned by:

```text
Owner: linuxUser
Group: linuxUser
```

---

## Failed Script Execution

Command:

```bash
./permission-investigation/scripts/start-service.sh
```

Result:

```text
Permission denied
```

Analysis:

```text
Script permissions: -rw-rw-r--
Owner permissions: rw-
Owner execute permission: absent
```

Cause:

```text
The owner does not have execute permission on start-service.sh.
```

Required permission:

```text
x for the owner
```

No repair was performed because the challenge required diagnosis only.

---

## Creating the Investigation Report

The report was written to:

```text
permission-investigation/report/analysis.txt
```

Final content:

```text
Linux Permission Investigation

Current user: linuxUser
Current UID: 1000
Primary group: linuxUser

Configuration owner: linuxUser
Configuration group: linuxUser
Configuration permissions: -rw-rw-r--

Script owner: linuxUser
Script group: linuxUser
Script permissions: -rw-rw-r--
Script executable by owner: no

Log owner: linuxUser
Log group: linuxUser
Log permissions: -rw-rw-r--

Project directory permissions: drwxrwxr-x
Config directory permissions: drwxrwxr-x
Scripts directory permissions: drwxrwxr-x
Logs directory permissions: drwxrwxr-x

Execution result: Permission denied
Cause: The owner does not have execute permission on start-service.sh
Required permission: x for the owner
```

The report was verified with:

```bash
cat permission-investigation/report/analysis.txt
```

---

## Important Correction: Owner Is Not `rw-`

An early mistake was to interpret:

```text
-rw-rw-r-- 1 linuxUser linuxUser service.conf
```

as:

```text
Owner: rw-
Group: rw-
Permissions: r--
```

This was incorrect.

The correct interpretation is:

```text
Owner: linuxUser
Group: linuxUser
Permissions: -rw-rw-r--
```

The blocks:

```text
rw-
rw-
r--
```

describe permissions for owner, group and others.

They are not owner or group names.

---

## Important Correction: Group Name and GID

The command:

```bash
id -g
```

returns the primary GID numerically.

Example:

```text
1000
```

The command:

```bash
id -gn
```

returns the primary group name.

Example:

```text
linuxUser
```

For a report field called:

```text
Primary group
```

the appropriate command is normally:

```bash
id -gn
```

For a field called:

```text
Primary GID
```

the appropriate command is:

```bash
id -g
```

---

## Common Mistakes

### Confusing Permission Categories with Permission Types

Categories:

```text
owner
group
others
```

Permission types:

```text
read
write
execute
```

These are different concepts.

---

### Confusing Owner and Permission Block

Incorrect:

```text
Owner: rw-
```

Correct:

```text
Owner: linuxUser
Owner permissions: rw-
```

---

### Thinking `x` Means Delete

On a file:

```text
x = execute
```

It does not mean delete.

File deletion depends mainly on the permissions of the parent directory.

---

### Thinking `x` on a Directory Means Modify

On a directory:

```text
x = traverse
```

Modification of directory entries depends mainly on:

```text
w + x
```

---

### Thinking a Readable Script Is Executable

A script can contain valid code and be readable while direct execution still fails.

Direct execution requires the applicable `x` permission.

---

### Ignoring Parent Directories

A readable file may still be inaccessible if a parent directory cannot be traversed.

Every directory in the path must allow traversal for the applicable identity category.

---

### Using `ls -l` Instead of `ls -ld`

To inspect a directory itself:

```bash
ls -ld directory
```

To inspect its contents:

```bash
ls -l directory
```

---

### Using `chmod 777`

`777` grants:

```text
read
write
execute
```

to:

```text
owner
group
others
```

It is usually an excessive and insecure response to a permission problem.

The proper approach is:

```text
Determine who needs access.
Determine what access is required.
Grant only those permissions.
```

---

### Using `sudo` Before Understanding the Cause

Before elevating privileges, inspect:

```bash
whoami
id
ls -l file
ls -ld directory
```

The objective is to solve the ownership or permission problem, not bypass it.

---

## Lesson Evaluation

The laboratory and challenge demonstrated:

- correct reading of `ls -l`;
- correct reading of `ls -ld`;
- distinction between file type and permissions;
- distinction between owner, group and others;
- understanding of `r`, `w` and `x`;
- understanding of directory traversal;
- understanding of parent-directory requirements;
- recognition that readable does not mean executable;
- identification of a missing execute permission;
- distinction between owner name and permission block;
- distinction between group name and GID;
- investigation without `sudo`;
- documentation using real values.

Final result:

```text
02-file-ownership-and-permissions: PASSED
```

---

## Lessons Learned

Linux permission troubleshooting should follow a structured process.

First identify the active identity:

```bash
whoami
id
```

Then inspect the target:

```bash
ls -l file
```

Then inspect every relevant directory:

```bash
ls -ld directory
```

Separate the permission string:

```text
type | owner | group | others
```

Example:

```text
-rw-rw-r--
- | rw- | rw- | r--
```

Ask:

```text
Which category applies?
Which permission is required?
Does the file have it?
Can every directory in the path be traversed?
Does the process actually need this access?
```

The central principles are:

```text
Readable does not mean executable.
File permissions and directory permissions have different meanings.
Deletion depends mainly on the parent directory.
Ownership and permission blocks must not be confused.
Do not use sudo or chmod 777 as a substitute for analysis.
Apply the Principle of Least Privilege.
```
