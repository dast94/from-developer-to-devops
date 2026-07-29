# Changing Linux Permissions with `chmod`

> **Note:** This documentation uses `linuxUser` as a generic example username.  
> On the laboratory system, the real username may be different.

## Lesson Result

```text
Module: 02-users-groups-and-permissions
Lesson: 03-changing-permissions-with-chmod
Status: PASSED
```

This lesson explains how to modify Linux permissions safely using `chmod`.

The practical challenge required assigning specific permissions to configuration files, scripts, logs, private files and directories without using overly permissive or recursive commands.

---

## What You Will Learn

This lesson covers:

- the purpose of `chmod`;
- symbolic permission notation;
- numeric permission notation;
- permission values `4`, `2` and `1`;
- adding, removing and replacing permissions;
- common permission combinations;
- differences between permissions on files and directories;
- safe permission changes;
- verification with `ls -l` and `ls -ld`;
- script execution after adding `x`;
- avoiding `chmod 777`;
- avoiding unsafe recursive permission changes.

Main command:

```bash
chmod
```

Verification commands:

```bash
ls -l
ls -ld
```

---

## What `chmod` Means

`chmod` means:

```text
change mode
```

It changes the permission mode of a file or directory.

It does not normally change:

- the owner;
- the associated group;
- the filename;
- the file content.

Example:

```bash
chmod u+x deploy.sh
```

This adds execute permission to the file owner.

Example:

```bash
chmod 750 deploy.sh
```

This sets the permissions to:

```text
rwxr-x---
```

---

## Why Permission Changes Matter

Incorrect permissions can cause:

- scripts that cannot be executed;
- configuration files that cannot be read;
- logs that cannot be written;
- secrets visible to unauthorized users;
- shared directories that cannot be accessed;
- files that are modifiable by too many users;
- deployment failures;
- security vulnerabilities.

The objective is not to give the maximum possible permissions.

The correct objective is:

```text
Give each identity only the access required to perform its task.
```

This follows the:

```text
Principle of Least Privilege
```

---

## Symbolic Mode

The general symbolic syntax is:

```text
chmod [category][operation][permissions] target
```

Categories:

```text
u = user or owner
g = group
o = others
a = all categories
```

Operations:

```text
+ = add permissions
- = remove permissions
= = set the exact permissions
```

Permissions:

```text
r = read
w = write
x = execute
```

---

## Adding a Permission

Example:

```bash
chmod u+x deploy.sh
```

Meaning:

```text
Add execute permission to the owner.
```

Before:

```text
-rw-rw-r--
```

After:

```text
-rwxrw-r--
```

Only the owner block changes.

The group and others blocks remain unchanged.

---

## Removing a Permission

Example:

```bash
chmod g-w app.conf
```

Meaning:

```text
Remove write permission from the group.
```

Before:

```text
-rw-rw-r--
```

After:

```text
-rw-r--r--
```

---

## Setting Exact Permissions

Example:

```bash
chmod o= app.conf
```

Meaning:

```text
Set the permissions for others to no permissions.
```

Before:

```text
-rw-r--r--
```

After:

```text
-rw-r-----
```

The `=` operation does not add or subtract selectively.

It replaces the selected category with exactly the permissions provided.

Example:

```bash
chmod u=x file
```

The owner receives exactly:

```text
--x
```

Any previous `r` or `w` permission for the owner is removed.

---

## Multiple Symbolic Changes

Several symbolic changes can be combined:

```bash
chmod u+x,g-w,o-r deploy.sh
```

Meaning:

```text
u+x -> add execute permission to the owner
g-w -> remove write permission from the group
o-r -> remove read permission from others
```

The operations are separated by commas.

---

## Numeric Mode

Linux permissions can also be represented numerically.

Values:

```text
r = 4
w = 2
x = 1
```

The values are added for each permission category.

| Permissions | Calculation | Value |
|---|---:|---:|
| `---` | `0` | `0` |
| `--x` | `1` | `1` |
| `-w-` | `2` | `2` |
| `-wx` | `2 + 1` | `3` |
| `r--` | `4` | `4` |
| `r-x` | `4 + 1` | `5` |
| `rw-` | `4 + 2` | `6` |
| `rwx` | `4 + 2 + 1` | `7` |

A three-digit mode represents:

```text
owner | group | others
```

Example:

```text
750
```

Separation:

```text
7 | 5 | 0
```

Translation:

```text
7 = rwx
5 = r-x
0 = ---
```

Final permissions:

```text
rwxr-x---
```

---

## Common Numeric Modes

### `600`

```text
6 | 0 | 0
rw- --- ---
```

Result:

```text
-rw-------
```

Typical use:

- secrets;
- private keys;
- private credentials;
- personal configuration.

---

### `640`

```text
6 | 4 | 0
rw- r-- ---
```

Result:

```text
-rw-r-----
```

Typical use:

- configuration readable by the owner and group;
- no access for others.

---

### `644`

```text
6 | 4 | 4
rw- r-- r--
```

Result:

```text
-rw-r--r--
```

Typical use:

- ordinary readable files;
- public configuration;
- documentation;
- logs readable by all local users.

---

### `700`

```text
7 | 0 | 0
rwx --- ---
```

Result on a file:

```text
-rwx------
```

Result on a directory:

```text
drwx------
```

Typical use:

- private scripts;
- private directories;
- owner-only access.

---

### `750`

```text
7 | 5 | 0
rwx r-x ---
```

Result on a file:

```text
-rwxr-x---
```

Result on a directory:

```text
drwxr-x---
```

Typical use:

- scripts executable by owner and group;
- application directories accessible by owner and group;
- no access for others.

---

### `755`

```text
7 | 5 | 5
rwx r-x r-x
```

Result:

```text
-rwxr-xr-x
```

Typical use:

- scripts or directories accessible by everyone;
- commands installed for system-wide use;
- public application directories.

It should still be used intentionally.

---

## Symbolic and Numeric Comparison

The following commands can produce equivalent results:

```bash
chmod u=rwx,g=rx,o= deploy.sh
```

```bash
chmod 750 deploy.sh
```

Both result in:

```text
-rwxr-x---
```

Symbolic notation is useful when changing only one part.

Example:

```bash
chmod g-w app.conf
```

Numeric notation is useful when setting the complete final state.

Example:

```bash
chmod 640 app.conf
```

---

## Files and Directories

The same numeric mode can behave differently depending on the element type.

Example:

```bash
chmod 700 private-file
chmod 700 private-directory
```

The symbolic permissions are equivalent:

```text
rwx------
```

But their meaning differs.

For a file:

```text
r = read content
w = modify content
x = execute
```

For a directory:

```text
r = list entry names
w = create, remove or rename entries
x = traverse the directory
```

A directory usually needs `x` to be usable.

A configuration file usually does not need `x`.

---

## Safe Permission Changes

Before changing permissions:

```bash
ls -l file
```

or:

```bash
ls -ld directory
```

Determine:

```text
Who should access the element?
What operations are required?
Which category should receive the permission?
Should the change affect one block or the entire mode?
```

After changing permissions, always verify again:

```bash
ls -l file
ls -ld directory
```

Do not assume that the command produced the desired result.

---

## Practical Laboratory

The laboratory used:

```text
permission-lab/
├── configs/
│   └── app.conf
├── scripts/
│   └── deploy.sh
├── logs/
│   └── application.log
└── private/
    └── secret.conf
```

---

## Making a Script Executable

Initial permissions:

```text
-rw-rw-r--
```

The owner did not have execute permission.

Command:

```bash
chmod u+x permission-lab/scripts/deploy.sh
```

Expected change:

```text
Before: - | rw- | rw- | r--
After:  - | rwx | rw- | r--
```

Final result:

```text
-rwxrw-r--
```

The script could then be executed:

```bash
./permission-lab/scripts/deploy.sh
```

Output:

```text
Deployment started
```

The script worked because the applicable owner permission block now contained:

```text
x
```

---

## Setting Configuration Permissions

Required state:

```text
Owner: read and write
Group: read
Others: no access
```

Calculation:

```text
Owner: rw- = 6
Group: r-- = 4
Others: --- = 0
```

Mode:

```text
640
```

Command:

```bash
chmod 640 permission-lab/configs/app.conf
```

Result:

```text
-rw-r-----
```

---

## Setting Log Permissions

Required state:

```text
Owner: read and write
Group: read
Others: read
```

Calculation:

```text
Owner: rw- = 6
Group: r-- = 4
Others: r-- = 4
```

Mode:

```text
644
```

Command:

```bash
chmod 644 permission-lab/logs/application.log
```

Result:

```text
-rw-r--r--
```

---

## Setting Script Permissions Numerically

Required state:

```text
Owner: read, write and execute
Group: read and execute
Others: no access
```

Calculation:

```text
Owner: rwx = 7
Group: r-x = 5
Others: --- = 0
```

Mode:

```text
750
```

Command:

```bash
chmod 750 permission-lab/scripts/deploy.sh
```

Result:

```text
-rwxr-x---
```

---

## Temporary Symbolic Changes

### Removing Read Permission from Others

Initial state:

```text
-rw-r--r--
```

Command:

```bash
chmod o-r permission-lab/logs/application.log
```

Temporary result:

```text
-rw-r-----
```

Restoration:

```bash
chmod o+r permission-lab/logs/application.log
```

Final result:

```text
-rw-r--r--
```

---

### Adding Write Permission to the Group

Initial state:

```text
-rw-r-----
```

Command:

```bash
chmod g+w permission-lab/configs/app.conf
```

Temporary result:

```text
-rw-rw----
```

Restoration:

```bash
chmod g-w permission-lab/configs/app.conf
```

Final result:

```text
-rw-r-----
```

---

## Private Directory

A private directory was created:

```bash
mkdir permission-lab/private
```

A private file was added:

```bash
printf "token=example-not-a-real-secret\n" \
  > permission-lab/private/secret.conf
```

Directory requirements:

```text
Owner: read, write and execute
Group: no access
Others: no access
```

Mode:

```text
700
```

Command:

```bash
chmod 700 permission-lab/private
```

Result:

```text
drwx------
```

File requirements:

```text
Owner: read and write
Group: no access
Others: no access
```

Mode:

```text
600
```

Command:

```bash
chmod 600 permission-lab/private/secret.conf
```

Result:

```text
-rw-------
```

---

## Permission Repair Challenge

### Scenario

The challenge required creating:

```text
permission-repair/
├── config/
│   ├── app.conf
│   └── secret.conf
├── scripts/
│   ├── deploy.sh
│   └── rollback.sh
├── logs/
│   └── application.log
├── private/
│   └── credentials.txt
└── report/
    └── permissions.txt
```

Required final permissions:

| Element | Mode | Symbolic permissions |
|---|---:|---|
| `config/app.conf` | `640` | `-rw-r-----` |
| `config/secret.conf` | `600` | `-rw-------` |
| `scripts/deploy.sh` | `750` | `-rwxr-x---` |
| `scripts/rollback.sh` | `700` | `-rwx------` |
| `logs/application.log` | `644` | `-rw-r--r--` |
| `private/` | `700` | `drwx------` |
| `private/credentials.txt` | `600` | `-rw-------` |
| main directories | `750` | `drwxr-x---` |

Restrictions:

```text
No sudo
No chmod 777
No chmod -R
No chown
No chgrp
```

---

## Creating the Structure

```bash
mkdir -p permission-repair/{config,scripts,logs,private,report}
cd permission-repair
```

Files:

```bash
touch config/{app,secret}.conf
touch scripts/{deploy,rollback}.sh
touch logs/application.log
touch private/credentials.txt
touch report/permissions.txt
```

---

## Creating the Scripts

Deployment script:

```bash
printf '%s\n' \
  '#!/bin/bash' \
  'printf "Deployment started\n"' \
  > scripts/deploy.sh
```

Rollback script:

```bash
printf '%s\n' \
  '#!/bin/bash' \
  'printf "Rollback started\n"' \
  > scripts/rollback.sh
```

---

## Applying File Permissions

Application configuration:

```bash
chmod g-w,o-r config/app.conf
```

Final result:

```text
-rw-r-----
```

Equivalent numeric mode:

```text
640
```

Secret configuration:

```bash
chmod 600 config/secret.conf
```

Deployment script:

```bash
chmod 750 scripts/deploy.sh
```

Rollback script:

```bash
chmod 700 scripts/rollback.sh
```

Application log:

```bash
chmod g-w logs/application.log
```

Final result:

```text
-rw-r--r--
```

Equivalent numeric mode:

```text
644
```

Credentials file:

```bash
chmod 600 private/credentials.txt
```

---

## Applying Directory Permissions

Private directory:

```bash
chmod 700 private
```

Main directories:

```bash
chmod 750 config scripts logs report
```

Project root:

```bash
cd ..
chmod 750 permission-repair
cd permission-repair
```

Final directory states:

```text
permission-repair/  drwxr-x---  750
config/             drwxr-x---  750
logs/               drwxr-x---  750
report/             drwxr-x---  750
scripts/            drwxr-x---  750
private/            drwx------  700
```

---

## Final File Verification

Command:

```bash
ls -l config scripts logs private report
```

Final results:

```text
config/app.conf           -rw-r-----  640
config/secret.conf        -rw-------  600
scripts/deploy.sh         -rwxr-x---  750
scripts/rollback.sh       -rwx------  700
logs/application.log      -rw-r--r--  644
private/credentials.txt   -rw-------  600
```

---

## Script Execution

Deployment script:

```bash
./scripts/deploy.sh
```

Output:

```text
Deployment started
```

Rollback script:

```bash
./scripts/rollback.sh
```

Output:

```text
Rollback started
```

Both scripts executed successfully because their owner permission blocks contained `x`.

---

## Final Permission Report

The report was stored in:

```text
report/permissions.txt
```

Content:

```text
Linux Permission Repair Report

config/app.conf: 640 (-rw-r-----)
config/secret.conf: 600 (-rw-------)
scripts/deploy.sh: 750 (-rwxr-x---)
scripts/rollback.sh: 700 (-rwx------)
logs/application.log: 644 (-rw-r--r--)
private/: 700 (drwx------)
private/credentials.txt: 600 (-rw-------)
Main directories: 750 (drwxr-x---)

deploy.sh execution: successful
rollback.sh execution: successful
Recursive chmod used: no
chmod 777 used: no
```

---

## Verification Commands

Directory verification:

```bash
ls -ld . config scripts logs private report
```

File verification:

```bash
ls -l config scripts logs private report
```

Execution verification:

```bash
./scripts/deploy.sh
./scripts/rollback.sh
```

Report verification:

```bash
cat report/permissions.txt
```

---

## Important Correction: Script Extensions

The scripts were initially created with:

```text
deploy.conf
rollback.conf
```

instead of:

```text
deploy.sh
rollback.sh
```

Because the incorrect names appeared in the real `ls` output, this was not only a transcription error.

They were corrected using:

```bash
mv scripts/deploy.conf scripts/deploy.sh
mv scripts/rollback.conf scripts/rollback.sh
```

The rename did not change the existing permissions.

---

## Important Correction: Script Content

An empty executable file may technically be launched, but it does not perform a useful task.

The rollback script initially had a size of:

```text
0 bytes
```

It was replaced with a real Bash script containing:

```bash
#!/bin/bash
printf "Rollback started\n"
```

A realistic operational script should contain:

- a valid interpreter line;
- a meaningful command;
- clear output;
- correct permissions.

---

## Newline in Script Output

The deployment script initially printed:

```text
Deployment starteddas94@ubuntu...
```

This indicated that the output did not end with a newline.

Incorrect:

```bash
printf "Deployment started"
```

Correct:

```bash
printf "Deployment started\n"
```

The missing newline did not affect permission handling, but it reduced terminal output readability.

---

## Why `chmod -R` Was Avoided

The recursive option:

```bash
chmod -R MODE directory
```

applies permission changes to all nested elements.

This can be dangerous because files and directories often require different modes.

For example:

```text
Directory: 750
Configuration: 640
Secret: 600
Script: 750
Log: 644
```

Applying the same mode to everything could:

- make configuration files executable;
- remove traversal from directories;
- expose secrets;
- make logs writable by unauthorized users;
- break application access.

The challenge applied permissions individually and intentionally.

---

## Why `chmod 777` Was Avoided

Mode:

```text
777
```

means:

```text
owner: rwx
group: rwx
others: rwx
```

It grants complete access to every category.

This may allow unauthorized users to:

- read sensitive data;
- modify files;
- replace scripts;
- execute files;
- delete or create directory entries.

`777` is not a professional default solution for permission errors.

The correct process is:

```text
Identify the required access.
Identify the correct category.
Grant only the necessary permission.
Verify the final state.
```

---

## Common Mistakes

### Confusing `+` and `=`

```bash
chmod u+x file
```

adds `x`.

```bash
chmod u=x file
```

sets the owner permissions exactly to `--x`.

---

### Using an Invalid Permission Letter

Valid symbolic permission letters are:

```text
r
w
x
```

A command such as:

```bash
chmod o+e file
```

is invalid because `e` is not a permission symbol.

The intended command for read permission is:

```bash
chmod o+r file
```

---

### Confusing Numeric Modes with Quantities

```text
640
```

does not mean six hundred and forty.

It represents:

```text
6 | 4 | 0
rw- r-- ---
```

---

### Giving Execute Permission to Every File

Files such as:

```text
app.conf
application.log
credentials.txt
```

normally do not need execute permission.

Only programs and scripts that must be executed need `x`.

---

### Removing Execute Permission from Directories

A directory without appropriate `x` cannot be traversed.

Even when files inside are readable, they may become inaccessible through the path.

---

### Forgetting Verification

After every permission change:

```bash
ls -l file
```

or:

```bash
ls -ld directory
```

A successful `chmod` command does not prove that the selected mode was correct.

---

## Lesson Evaluation

The lesson demonstrated:

- correct symbolic `chmod` usage;
- correct numeric `chmod` usage;
- understanding of `r = 4`, `w = 2`, `x = 1`;
- conversion between symbolic and numeric permissions;
- intentional file permission design;
- intentional directory permission design;
- successful script execution;
- private file and directory protection;
- use of both symbolic and numeric changes;
- avoidance of recursive permission changes;
- avoidance of `777`;
- final verification with real commands;
- creation of a permission audit report.

Final result:

```text
03-changing-permissions-with-chmod: PASSED
```

---

## Lessons Learned

Permission modification should begin with a required final state.

Ask:

```text
Who is the owner?
Which group should have access?
Should others have access?
Is the target a file or directory?
Does the file need execution?
Does the directory need traversal?
```

Symbolic mode is useful for targeted changes:

```bash
chmod u+x script.sh
chmod g-w config.conf
chmod o-r file
```

Numeric mode is useful for complete states:

```bash
chmod 600 secret.conf
chmod 640 app.conf
chmod 644 application.log
chmod 700 private
chmod 750 deploy.sh
```

Always verify:

```bash
ls -l file
ls -ld directory
```

The central rules are:

```text
Use the minimum required permissions.
Do not use chmod 777 as a shortcut.
Do not use recursive chmod without analyzing the structure.
Files and directories often require different modes.
Readable does not mean executable.
Directory execute permission means traversal.
Permission changes must be verified.
```
