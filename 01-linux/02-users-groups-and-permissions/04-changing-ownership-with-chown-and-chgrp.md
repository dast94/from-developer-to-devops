# Changing Linux Ownership with `chown` and `chgrp`

> **Note:** This documentation uses `linuxUser` as a generic example username.  
> On the laboratory system, the real username may be different.

## Lesson Result

```text
Module: 02-users-groups-and-permissions
Lesson: 04-changing-ownership-with-chown-and-chgrp
Status: PASSED
```

This lesson explains how Linux file ownership and group ownership can be inspected and modified using `chown` and `chgrp`.

The practical challenge focused on:

- distinguishing ownership from permissions;
- changing a file group safely;
- understanding which group changes are permitted to a normal user;
- restoring the original group;
- attempting an unauthorized ownership change;
- verifying that a failed operation does not modify the file;
- recording UID, GID, owner, group and permissions in a report.

---

## What You Will Learn

This lesson covers:

- file ownership;
- group ownership;
- the difference between `chmod`, `chown` and `chgrp`;
- changing the owner of a file;
- changing owner and group together;
- changing only the group;
- restrictions applied to normal users;
- numerical ownership with UID and GID;
- inspection with `ls -l` and `ls -ln`;
- safe ownership troubleshooting;
- the risks of recursive ownership changes.

Main commands:

```bash
chown
chgrp
ls -l
ls -ln
id
getent group
```

---

## Ownership and Permissions Are Different

A Linux file normally has:

```text
Owner
Group
Permissions
```

Example:

```text
-rw-r----- 1 linuxUser developers 128 app.conf
```

Interpretation:

```text
Permissions: -rw-r-----
Owner:       linuxUser
Group:       developers
Filename:    app.conf
```

The permissions describe what the applicable categories may do.

The owner and group identify which users receive the owner and group permission blocks.

These are separate concepts.

---

## `chmod`, `chown` and `chgrp`

The three commands have different responsibilities:

```text
chmod -> changes permissions
chown -> changes owner and optionally group
chgrp -> changes group
```

Example:

```bash
chmod 640 app.conf
```

Changes:

```text
permissions
```

It does not normally change owner or group.

Example:

```bash
chown appuser app.conf
```

Changes:

```text
owner
```

Example:

```bash
chgrp developers app.conf
```

Changes:

```text
group
```

The symbolic permission string can remain exactly the same while ownership changes.

---

## Why Ownership Matters

Suppose a configuration file has:

```text
Owner: linuxUser
Group: linuxUser
Permissions: -rw-r-----
```

A service running as:

```text
www-data
```

may not be able to read it because:

- `www-data` is not the owner;
- `www-data` is not a member of the file group;
- others have no read permission.

The immediate temptation may be:

```bash
chmod 644 app.conf
```

This would grant read access to all users.

A safer solution may be to associate the file with the correct service group and preserve restricted permissions.

For example:

```text
Owner: linuxUser
Group: www-data
Permissions: -rw-r-----
```

This allows group members to read the file without exposing it to everyone.

---

## `chown`

`chown` means:

```text
change owner
```

General syntax:

```bash
chown OWNER FILE
```

Example:

```bash
chown appuser app.conf
```

This attempts to set:

```text
Owner: appuser
```

The group is not explicitly changed by this command form.

---

## Changing Owner and Group Together

Syntax:

```bash
chown OWNER:GROUP FILE
```

Example:

```bash
chown appuser:developers app.conf
```

Result:

```text
Owner: appuser
Group: developers
```

The permission string normally remains unchanged.

Example:

```text
Before:
-rw-r----- linuxUser linuxUser app.conf

After:
-rw-r----- appuser developers app.conf
```

---

## Changing Only the Group with `chown`

The owner may be omitted:

```bash
chown :developers app.conf
```

This changes only:

```text
Group: developers
```

The owner remains unchanged.

For readability, when changing only the group, `chgrp` is often clearer.

---

## `chgrp`

`chgrp` means:

```text
change group
```

Syntax:

```bash
chgrp GROUP FILE
```

Example:

```bash
chgrp developers app.conf
```

Possible result:

```text
Before:
-rw-r----- linuxUser linuxUser app.conf

After:
-rw-r----- linuxUser developers app.conf
```

The following remain unchanged:

- owner;
- permissions;
- content;
- filename.

Only the associated group changes.

---

## Restrictions for Normal Users

A normal user cannot usually assign a file freely to another owner.

Example:

```bash
chown root file.txt
```

A normal user will typically receive:

```text
Operation not permitted
```

Changing ownership generally requires administrative privileges or a process with the necessary capability.

---

## Why Ownership Changes Are Restricted

The owner of a file affects:

- access control;
- responsibility for the file;
- disk quotas;
- security auditing;
- which permission block applies;
- administration of shared systems.

If users could transfer ownership freely, they might:

- assign unwanted files to another user;
- bypass disk quota limits;
- hide responsibility for files;
- create misleading ownership states;
- abuse another user's permission context;
- interfere with system services.

For this reason, unrestricted ownership transfer is not allowed to ordinary users.

---

## Changing a File Group as a Normal User

A normal user may usually change the group of a file when both conditions are true:

1. the user owns the file;
2. the destination group is one of the user's groups.

Available groups can be listed with:

```bash
id -Gn
```

Example:

```text
linuxUser cdrom plugdev sambashare
```

If `linuxUser` owns a file, this may succeed:

```bash
chgrp cdrom file.txt
```

This will normally fail if the destination group is not one of the user's groups:

```bash
chgrp postgres file.txt
```

Ownership of the file alone is not sufficient.

The user must also belong to the destination group.

---

## Verifying Group Membership

Before changing a file group:

```bash
id -Gn
```

Then inspect the selected group:

```bash
getent group cdrom
```

Example:

```text
cdrom:x:24:linuxUser
```

Interpretation:

```text
Group name: cdrom
GID:        24
Members:    linuxUser
```

The real values must always come from the current system.

---

## Inspecting Ownership with `ls -l`

Example:

```bash
ls -l file.txt
```

Output:

```text
-rw-r--r-- 1 linuxUser developers 42 Jul 30 10:00 file.txt
```

Important fields:

```text
Permissions: -rw-r--r--
Owner:       linuxUser
Group:       developers
```

After `chgrp`, only the group field may change.

---

## Numerical Ownership with `ls -ln`

The command:

```bash
ls -ln file.txt
```

shows numerical UID and GID instead of resolving them to names.

Example:

```text
-rw-r--r-- 1 1000 1001 42 Jul 30 10:00 file.txt
```

Interpretation:

```text
Owner UID: 1000
Group GID: 1001
```

Comparison:

```bash
ls -l file.txt
```

may show:

```text
linuxUser developers
```

while:

```bash
ls -ln file.txt
```

may show:

```text
1000 1001
```

---

## Comparing File Ownership with Current Identity

Current identity:

```bash
id
```

Specific values:

```bash
id -u
id -g
id -un
id -gn
```

Meaning:

```text
id -u  -> current UID
id -g  -> primary GID
id -un -> current username
id -gn -> primary group name
```

The numerical owner and group shown by `ls -ln` can be compared with:

```bash
id -u
id -g
```

---

## `chown` Does Not Change Permissions Automatically

Suppose a file begins as:

```text
-rw------- linuxUser linuxUser secret.conf
```

A successful ownership change might produce:

```text
-rw------- appuser developers secret.conf
```

The permission string remains:

```text
-rw-------
```

The new owner receives the owner block:

```text
rw-
```

The group receives:

```text
---
```

Changing owner or group does not automatically grant additional permissions.

Ownership and permissions must be managed separately.

---

## Permission Evaluation After Ownership Changes

Linux evaluates the applicable permission block in simplified order:

1. owner block, if the process UID matches the file owner;
2. group block, if a process group matches the file group;
3. others block otherwise.

If ownership changes, a different user may now receive the owner block.

Example:

```text
-rw-r----- appuser developers app.conf
```

The user `appuser` receives:

```text
rw-
```

Members of `developers` receive:

```text
r--
```

Other users receive:

```text
---
```

---

## Recursive Ownership Changes

Recursive forms exist:

```bash
chown -R OWNER:GROUP directory
chgrp -R GROUP directory
```

These commands modify nested files and directories.

They are powerful and potentially dangerous.

Possible risks include:

- changing ownership of files unintentionally;
- breaking services;
- altering secret ownership;
- changing application-generated files;
- affecting mounted directories;
- modifying shared data;
- producing a difficult rollback.

Before using recursive ownership changes, verify:

```bash
pwd
find target-directory
ls -l target-directory
```

The recursive option was not used in this lesson.

---

## Practical Laboratory

The laboratory structure was:

```text
ownership-lab/
├── config/
│   └── app.conf
├── logs/
│   └── application.log
└── shared/
    └── notes.txt
```

Creation:

```bash
cd ~
mkdir -p ownership-lab/{config,logs,shared}
touch ownership-lab/config/app.conf
touch ownership-lab/logs/application.log
touch ownership-lab/shared/notes.txt
```

Content:

```bash
printf "environment=development\n" \
  > ownership-lab/config/app.conf

printf "INFO Application started\n" \
  > ownership-lab/logs/application.log

printf "Shared operational notes\n" \
  > ownership-lab/shared/notes.txt
```

---

## Initial State

Files were inspected using:

```bash
ls -l ownership-lab/config/app.conf
ls -l ownership-lab/logs/application.log
ls -l ownership-lab/shared/notes.txt
```

Typical result:

```text
Owner: linuxUser
Group: linuxUser
```

The files were created by processes running with the current user's identity.

Their group was initially the current primary group, assuming no special group inheritance rule changed it.

---

## Numerical Verification

The configuration file was inspected with:

```bash
ls -ln ownership-lab/config/app.conf
```

The numerical owner and group were compared with:

```bash
id -u
id -g
```

Example:

```text
File owner UID: 1000
Current UID:    1000

File group GID: 1000
Primary GID:    1000
```

The actual values must be verified on the current machine.

---

## Available Groups

The user's group names were listed with:

```bash
id -Gn
```

Example:

```text
linuxUser adm cdrom sudo plugdev sambashare
```

A non-administrative supplementary group was selected for the test.

The group was verified with:

```bash
getent group cdrom
```

Example:

```text
cdrom:x:24:linuxUser
```

---

## Temporary Group Change

The group of:

```text
ownership-lab/shared/notes.txt
```

was changed using:

```bash
chgrp cdrom ownership-lab/shared/notes.txt
```

Verification:

```bash
ls -l ownership-lab/shared/notes.txt
ls -ln ownership-lab/shared/notes.txt
```

Expected changes:

```text
Group name changes.
Group GID changes.
```

Expected unchanged values:

```text
Owner remains the same.
Permissions remain the same.
Content remains the same.
```

The operation was allowed because:

- the current user owned the file;
- the current user belonged to the `cdrom` group.

---

## Restoring the Primary Group

The original primary group was restored with:

```bash
chgrp "$(id -gn)" ownership-lab/shared/notes.txt
```

Command substitution executes:

```bash
id -gn
```

and inserts the resulting group name into `chgrp`.

If the primary group is:

```text
linuxUser
```

the effective command becomes:

```bash
chgrp linuxUser ownership-lab/shared/notes.txt
```

---

## Unauthorized Ownership Change

The following command was attempted without `sudo`:

```bash
chown root ownership-lab/config/app.conf
```

Result:

```text
Operation not permitted
```

The error was not bypassed.

The file was verified with:

```bash
ls -l ownership-lab/config/app.conf
```

The owner remained unchanged.

The failed `chown` did not modify:

- owner;
- group;
- permissions;
- content.

---

## Ownership Investigation Challenge

### Required Structure

```text
ownership-investigation/
├── config/
│   └── service.conf
├── shared/
│   └── runbook.txt
└── report/
    └── ownership.txt
```

The challenge required:

- recording owner and group names;
- recording UID and GID values;
- recording permissions;
- changing the group of `runbook.txt`;
- verifying that permissions did not change;
- restoring the primary group;
- attempting `chown root`;
- documenting the failed operation;
- producing a final report.

Restrictions:

```text
No sudo
No chown -R
No chgrp -R
No chmod 777
```

---

## Creating the Challenge Structure

```bash
cd ~
mkdir -p ownership-investigation/{config,shared,report}
```

Configuration content:

```bash
printf "service=api\nenvironment=development\n" \
  > ownership-investigation/config/service.conf
```

Runbook content:

```bash
printf "Deployment runbook\nRestart the service after configuration changes\n" \
  > ownership-investigation/shared/runbook.txt
```

Report file:

```bash
touch ownership-investigation/report/ownership.txt
```

---

## Challenge Permissions

Directories:

```bash
chmod 750 ownership-investigation/{config,shared,report}
```

Configuration:

```bash
chmod 640 ownership-investigation/config/service.conf
```

Runbook:

```bash
chmod 644 ownership-investigation/shared/runbook.txt
```

Report:

```bash
chmod 640 ownership-investigation/report/ownership.txt
```

None of the text files required execute permission.

The directories required `x` for traversal.

---

## Initial Ownership State

The current identity was:

```text
Current user: linuxUser
Current UID: 1000
Primary group: linuxUser
Primary GID: 1000
```

The actual laboratory values were taken from the real system.

`service.conf` initially had:

```text
Owner:       linuxUser
Group:       linuxUser
UID:         1000
GID:         1000
Permissions: -rw-r-----
```

`runbook.txt` initially had:

```text
Owner:       linuxUser
Group:       linuxUser
UID:         1000
GID:         1000
Permissions: -rw-r--r--
```

---

## Challenge Group Change

The temporary group selected was:

```text
cdrom
```

Group lookup:

```bash
getent group cdrom
```

Result used in the report:

```text
Temporary group: cdrom
Temporary GID: 24
```

The group was changed using:

```bash
chgrp cdrom ownership-investigation/shared/runbook.txt
```

Verification showed:

```text
Owner changed:       no
Group changed:       yes
GID changed:         yes
Permissions changed: no
```

The symbolic permissions remained:

```text
-rw-r--r--
```

---

## Restoring the Group

The original group was restored using:

```bash
chgrp "$(id -gn)" ownership-investigation/shared/runbook.txt
```

Final state:

```text
Restored group: linuxUser
Restored GID: 1000
```

---

## Failed `chown root`

Command:

```bash
chown root ownership-investigation/config/service.conf
```

Result:

```text
Operation not permitted
```

The final verification showed:

```text
Owner changed after failed chown: no
```

The configuration remained owned by the current user.

---

## Final Challenge Report

The report was stored in:

```text
ownership-investigation/report/ownership.txt
```

Final content:

```text
Linux Ownership Investigation

Current user: linuxUser
Current UID: 1000
Primary group: linuxUser
Primary GID: 1000

service.conf owner: linuxUser
service.conf group: linuxUser
service.conf UID: 1000
service.conf GID: 1000
service.conf permissions: -rw-r-----

runbook.txt initial owner: linuxUser
runbook.txt initial group: linuxUser
runbook.txt initial UID: 1000
runbook.txt initial GID: 1000
runbook.txt permissions: -rw-r--r--

Temporary group: cdrom
Temporary GID: 24
Permissions changed after chgrp: no

Restored group: linuxUser
Restored GID: 1000

chown root result: Operation not permitted
Owner changed after failed chown: no

Conclusion:
chmod changes permissions.
chown changes owner and optionally group.
chgrp changes only the group.
```

---

## Final Verification

Report:

```bash
cat ownership-investigation/report/ownership.txt
```

Configuration:

```bash
ls -l ownership-investigation/config/service.conf
```

Final result:

```text
-rw-r----- linuxUser linuxUser service.conf
```

Runbook:

```bash
ls -l ownership-investigation/shared/runbook.txt
```

Final result:

```text
-rw-r--r-- linuxUser linuxUser runbook.txt
```

Both files were restored to the expected owner and group.

---

## Common Mistakes

### Confusing Ownership with Permissions

Incorrect assumption:

```text
chown changes rwx permissions.
```

Correct:

```text
chown changes owner and optionally group.
chmod changes rwx permissions.
```

---

### Thinking `chgrp` Changes the Owner

`chgrp` changes only the associated group.

Example:

```text
Before:
linuxUser linuxUser

After:
linuxUser cdrom
```

The owner remains `linuxUser`.

---

### Thinking File Ownership Alone Allows Any Group Change

A user normally needs both:

- ownership of the file;
- membership in the destination group.

Being the owner alone does not allow arbitrary group assignment.

---

### Confusing `root` and `sudo`

`root` is an identity with UID `0`.

`sudo` is a tool that may permit authorized users to execute commands with elevated privileges.

They are related but not the same thing.

---

### Expecting `chown` to Grant Group Access

Changing:

```text
Group: developers
```

does not automatically add group read or write permission.

The group permission block must already contain the needed permission or be adjusted separately using `chmod`.

---

### Ignoring Numerical IDs

Names are human-readable representations.

Linux ownership is based on:

```text
UID
GID
```

Use:

```bash
ls -ln
```

when numerical identity information is important.

---

### Using Recursive Ownership Changes Carelessly

Commands such as:

```bash
chown -R
chgrp -R
```

can affect an entire application tree.

Always inspect the target structure first.

---

### Using `sudo` to Bypass the Exercise

The failed `chown root` was intentional.

The objective was to understand the security restriction, not bypass it.

---

## Lesson Evaluation

The laboratory and challenge demonstrated:

- correct distinction between permissions and ownership;
- use of `chgrp`;
- understanding of `chown`;
- understanding of `chown OWNER:GROUP`;
- understanding of `chown :GROUP`;
- inspection with `ls -l`;
- numerical inspection with `ls -ln`;
- comparison with `id`;
- validation with `getent group`;
- safe temporary group changes;
- restoration of the original group;
- understanding of normal-user restrictions;
- analysis of `Operation not permitted`;
- verification that failed operations do not change ownership;
- avoidance of recursive ownership changes;
- creation of a complete ownership report.

Final result:

```text
04-changing-ownership-with-chown-and-chgrp: PASSED
```

---

## Lessons Learned

Ownership troubleshooting should follow a structured process.

Inspect the current identity:

```bash
id
```

Inspect owner, group and permissions:

```bash
ls -l file
```

Inspect numerical UID and GID:

```bash
ls -ln file
```

Verify group membership:

```bash
id -Gn
getent group GROUP
```

Then determine whether the problem involves:

```text
permissions
owner
group
group membership
directory traversal
```

Important rules:

```text
chmod changes permissions.
chown changes owner and optionally group.
chgrp changes only group.
```

```text
A normal user cannot normally transfer ownership freely.
```

```text
A normal user can usually assign a file only to a group they belong to.
```

```text
Changing owner or group does not automatically change permissions.
```

```text
The new owner uses the owner permission block.
```

```text
Recursive ownership changes must be used with extreme care.
```

The central security principle remains:

```text
Grant only the ownership and access required for the task.
```
