# Linux Filesystem Navigation

> **Note:** In this guide, `linuxUser` is used as an example username.  
> Replace it with your actual Linux username when running the commands.

## What You Will Learn

Linux uses a hierarchical filesystem.

The highest level of the filesystem is the root directory:

```text
/
```

Every file and directory is located somewhere below the root directory.

In this guide, you will learn how to:

- identify the current user;
- identify the current working directory;
- list files and directories;
- display hidden files;
- navigate through the filesystem;
- distinguish between absolute and relative paths;
- return to the previously visited directory.

---

## Example Environment

This guide uses the following example environment:

```text
User: linuxUser
Home directory: /home/linuxUser
Shell: /bin/bash
```

You can retrieve this information with:

```bash
whoami
pwd
echo $SHELL
```

---

## Filesystem Hierarchy

The Linux filesystem starts from the root directory:

```text
/
```

A simplified Linux filesystem may look like this:

```text
/
├── home
├── etc
├── var
├── tmp
├── usr
└── opt
```

The `/home` directory normally contains the home directories of regular users.

For the example user `linuxUser`, the home directory is:

```text
/home/linuxUser
```

The symbol `~` represents the home directory of the current user.

For the example user:

```text
~ = /home/linuxUser
```

The root directory `/` and the home directory `~` are different:

- `/` is the highest point of the entire filesystem;
- `~` is the home directory of the current user.

---

## Absolute and Relative Paths

### Absolute Path

An absolute path describes the complete location of a file or directory.

It always starts with `/`.

Example:

```text
/home/linuxUser/projects
```

An absolute path has the same meaning regardless of the current working directory.

Example:

```bash
cd /home/linuxUser/projects
```

---

### Relative Path

A relative path is interpreted starting from the current working directory.

For example, when the current directory is:

```text
/home/linuxUser
```

the command:

```bash
cd projects
```

moves to:

```text
/home/linuxUser/projects
```

When the current directory is `/`, the example user can reach the home directory using this relative path:

```bash
cd home/linuxUser
```

A relative path may work from one directory and fail from another because its meaning depends on the current working directory.

---

## Commands

### `whoami`

Displays the name of the current user.

```bash
whoami
```

Example output:

```text
linuxUser
```

---

### `pwd`

`pwd` means:

```text
print working directory
```

It displays the absolute path of the current working directory.

```bash
pwd
```

Example output:

```text
/home/linuxUser
```

---

### `ls`

Displays the visible files and directories inside the current directory.

```bash
ls
```

---

### `ls -a`

Displays all entries, including hidden files and directories.

```bash
ls -a
```

Files and directories whose names start with `.` are considered hidden.

Examples:

```text
.bashrc
.config
.ssh
```

A hidden file is not necessarily protected or secure. It is simply not displayed by the standard `ls` command.

---

### `ls -l`

Displays detailed information about files and directories.

```bash
ls -l
```

The output normally includes:

- file type;
- permissions;
- owner;
- group;
- size;
- last modification time;
- file or directory name.

It does not display hidden entries unless it is combined with `-a`.

---

### `ls -la`

Combines the `-l` and `-a` options.

```bash
ls -la
```

The following command is equivalent:

```bash
ls -al
```

Both commands display detailed information about visible and hidden entries.

---

### `cd`

`cd` means:

```text
change directory
```

It is used to navigate through the filesystem.

Move to a directory using an absolute path:

```bash
cd /home/linuxUser
```

Move to a directory using a relative path:

```bash
cd projects
```

Move to the parent directory:

```bash
cd ..
```

Move to the current user's home directory:

```bash
cd ~
```

Move to the previous working directory:

```bash
cd -
```

Refer to the current directory:

```bash
cd .
```

---

## Special Path Symbols

### Current Directory: `.`

The dot represents the current working directory.

```bash
cd .
```

This command does not change the current location.

---

### Parent Directory: `..`

Two dots represent the parent directory.

For example, from:

```text
/home/linuxUser/projects
```

the command:

```bash
cd ..
```

moves to:

```text
/home/linuxUser
```

---

### Home Directory: `~`

The tilde represents the home directory of the current user.

```bash
cd ~
```

For the example user, this moves to:

```text
/home/linuxUser
```

---

### Previous Directory: `-`

The dash represents the previous working directory.

```bash
cd -
```

This does not necessarily move to the parent directory.

It switches between the current directory and the previously visited directory.

Example:

```bash
cd projects
cd ..
cd -
```

After the final command, the current directory is:

```text
/home/linuxUser/projects
```

Running `cd -` again returns to:

```text
/home/linuxUser
```

---

## Practical Examples

### Navigating from the Root Directory

Move to the root directory:

```bash
cd /
```

Move to the example user's home using an absolute path:

```bash
cd /home/linuxUser
```

The equivalent relative path, when starting from `/`, is:

```bash
cd home/linuxUser
```

---

### Entering a Directory with a Relative Path

Starting from:

```text
/home/linuxUser
```

Enter the `projects` directory using:

```bash
cd projects
```

The resulting path is:

```text
/home/linuxUser/projects
```

---

### Returning to the Parent Directory

From:

```text
/home/linuxUser/projects
```

use:

```bash
cd ..
```

The resulting directory is:

```text
/home/linuxUser
```

---

### Returning to the Previous Directory

Use:

```bash
cd -
```

This returns to:

```text
/home/linuxUser/projects
```

Running the same command again returns to:

```text
/home/linuxUser
```

---

### Displaying Hidden Files

Use:

```bash
ls -a
```

This displays both visible and hidden files, including entries such as:

```text
.bashrc
.config
.profile
.ssh
```

---

## Common Mistakes

### Forgetting the Space Between a Command and an Option

Incorrect:

```bash
ls-a
```

The shell interprets `ls-a` as the name of a command.

Correct:

```bash
ls -a
```

The space separates the command `ls` from the option `-a`.

---

### Confusing `/` and `~`

The following symbols are not equivalent:

```text
/
~
```

- `/` represents the root of the entire filesystem;
- `~` represents the current user's home directory.

---

### Confusing `cd ..` and `cd -`

These commands perform different operations.

```bash
cd ..
```

Moves to the parent directory.

```bash
cd -
```

Moves to the previously visited directory.

They may sometimes produce the same result, but only because of the previous navigation sequence.

---

### Assuming Hidden Files Are Secure

A filename beginning with `.` is hidden from the default `ls` output.

This does not mean that the file is encrypted, protected or inaccessible.

---

### Forgetting That Linux Is Case-Sensitive

Linux distinguishes between uppercase and lowercase letters.

The following names represent different files or directories:

```text
projects
Projects
PROJECTS
```

---

### Using Spaces Without Quotes

Spaces are allowed in filenames, but the shell interprets them as separators.

For a directory named:

```text
Linux Course
```

use quotes:

```bash
cd "Linux Course"
```

Alternatively, escape the space:

```bash
cd Linux\ Course
```

For technical projects, names such as these are generally easier to manage:

```text
linux-course
linux_course
```

---

### Using `sudo` Without Understanding the Problem

Using `sudo` for every failed command is dangerous.

Before executing a command with elevated privileges, verify:

```bash
whoami
pwd
```

You should also understand:

- which permission is missing;
- who owns the target file or directory;
- whether you are operating in the correct location;
- whether administrative privileges are actually required.

---

## Lessons Learned

Before running commands that modify the filesystem, always verify the execution context.

The most useful commands for doing this are:

```bash
whoami
pwd
```

Absolute paths identify an exact location and do not depend on the current directory.

Relative paths are shorter, but their meaning depends on the current working directory.

Small syntax details are important in the shell. For example:

```bash
ls -a
```

and:

```bash
ls-a
```

are interpreted as completely different commands.

Also remember:

- `cd ..` moves to the parent directory;
- `cd -` returns to the previous working directory.
