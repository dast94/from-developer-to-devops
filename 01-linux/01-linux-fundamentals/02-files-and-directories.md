# Linux Files and Directories

> **Note:** In this guide, `linuxUser` is used as an example username.  
> Replace it with your actual Linux username when running the commands.

## What You Will Learn

In this guide, you will learn how to:

- create files and directories;
- copy files and directories;
- move and rename files and directories;
- remove files and directories;
- verify the current working directory before changing the filesystem;
- avoid common mistakes that can cause data loss.

The main commands are:

```text
touch
mkdir
cp
mv
rm
rmdir
```

When working with files and directories, it is important to verify the execution context using:

```bash
pwd
ls
```

A wrong command can:

- overwrite a configuration file;
- delete important data;
- remove the wrong directory;
- break an application or service.

---

## Creating Files

### `touch`

The `touch` command can create an empty file.

```bash
touch notes.txt
```

If the file does not exist, it is created.

If the file already exists, `touch` normally updates its timestamps without changing its content.

Example:

```bash
touch app.conf
```

Create multiple files with one command:

```bash
touch app.conf database.conf notes.txt
```

Create files using relative paths:

```bash
touch configs/app.conf configs/database.conf
```

`touch` does not write text inside a file. It only creates an empty file or updates timestamps.

---

## Creating Directories

### `mkdir`

`mkdir` means:

```text
make directory
```

It creates a new directory.

```bash
mkdir projects
```

Create multiple directories:

```bash
mkdir configs backups scripts
```

This creates three different directories:

```text
configs/
backups/
scripts/
```

---

### Creating Nested Directories

To create nested directories, use the `-p` option:

```bash
mkdir -p application/config/nginx
```

This creates the entire directory structure:

```text
application/
└── config/
    └── nginx/
```

The `-p` option is useful when one or more parent directories do not exist.

Without `-p`, the command fails if the parent directory is missing.

The `-p` option also avoids an error when the requested directory already exists.

---

## Copying Files

### `cp`

`cp` means:

```text
copy
```

The general syntax is:

```bash
cp source destination
```

Example:

```bash
cp app.conf app.conf.backup
```

This creates a copy named:

```text
app.conf.backup
```

The original file remains unchanged.

---

### Copying a File into a Directory

```bash
cp app.conf backups/
```

The copied file keeps the original name:

```text
backups/app.conf
```

To copy the file and assign a new name:

```bash
cp app.conf backups/app.conf.bak
```

The result is:

```text
backups/app.conf.bak
```

---

### Copying Directories

To copy a directory and its contents, use the recursive option:

```bash
cp -r source-directory destination-directory
```

Example:

```bash
cp -r configs configs-backup
```

The `-r` option means:

```text
recursive
```

It tells `cp` to copy the directory and everything inside it.

---

## Moving and Renaming

### `mv`

`mv` means:

```text
move
```

It can be used to:

- move a file;
- move a directory;
- rename a file;
- rename a directory;
- move and rename in the same command.

---

### Moving a File

```bash
mv notes.txt documentation/
```

After the command, the file no longer exists in the original location.

It is now located at:

```text
documentation/notes.txt
```

Unlike `cp`, `mv` does not leave a copy at the original location.

---

### Renaming a File

```bash
mv old-name.txt new-name.txt
```

The file remains in the same directory, but its name changes.

---

### Moving and Renaming at the Same Time

```bash
mv notes.txt backups/linux-notes.txt
```

This command:

1. moves `notes.txt` into `backups/`;
2. renames it to `linux-notes.txt`.

The final path is:

```text
backups/linux-notes.txt
```

---

## Removing Files

### `rm`

`rm` means:

```text
remove
```

It removes files.

```bash
rm temporary.txt
```

On a typical Linux terminal, the file is removed directly and is not moved to a graphical trash folder.

For this reason, `rm` must be used carefully.

Before removing an important file, verify:

```bash
pwd
ls -l temporary.txt
```

---

## Removing Directories

### `rmdir`

`rmdir` removes empty directories.

```bash
rmdir empty-directory
```

It fails if the directory contains files or subdirectories.

This makes `rmdir` safer for simple empty-directory cleanup.

---

### `rm -r`

To remove a directory and its contents, use:

```bash
rm -r directory
```

The `-r` option performs the operation recursively.

It can remove:

- the directory;
- files inside it;
- nested directories;
- all nested contents.

Example:

```bash
rm -r temporary-directory
```

This command is more dangerous than `rmdir` because it can remove an entire directory tree.

---

## Safe Workflow

Before performing a destructive operation, verify the current context.

### Check the Current User

```bash
whoami
```

### Check the Current Directory

```bash
pwd
```

### Inspect the Target

For a file:

```bash
ls -l target-file
```

For a directory:

```bash
ls -ld target-directory
```

You should understand:

- where you are;
- what you are modifying;
- whether the target is a file or directory;
- whether the path is correct;
- whether the operation is really necessary.

Do not use `sudo` simply because a command fails.

First identify the actual cause of the problem.

---

## Practical Example

Create a laboratory directory:

```bash
mkdir -p linux-file-lab/configs
```

Enter the directory:

```bash
cd linux-file-lab
```

Create additional directories:

```bash
mkdir backups scripts
```

Create a notes file:

```bash
touch notes.txt
```

Create configuration and script files:

```bash
touch configs/app.conf \
      configs/database.conf \
      scripts/deploy.sh \
      scripts/backup.sh
```

Copy a configuration backup:

```bash
cp configs/app.conf backups/app.conf.backup
```

Move and rename the notes file:

```bash
mv notes.txt backups/linux-notes.txt
```

Create a temporary file:

```bash
touch temporary.txt
```

Verify that it exists:

```bash
ls -l temporary.txt
```

Remove it:

```bash
rm temporary.txt
```

Verify that it no longer exists:

```bash
ls
```

Create and remove an empty directory:

```bash
mkdir empty-directory
rmdir empty-directory
```

---

## Challenge

The challenge was to create this structure:

```text
devops-simulation/
├── config/
│   ├── app.conf
│   └── database.conf
├── backup/
│   ├── app.conf.bak
│   └── database.conf.bak
├── scripts/
│   ├── deploy.sh
│   └── rollback.sh
└── documentation/
    └── README.md
```

The requirements included:

- starting from the home directory;
- using at least one absolute path;
- using relative paths;
- using `mkdir -p`;
- using `cp` to create backups;
- using `mv`;
- creating and removing a temporary file;
- avoiding `sudo`;
- avoiding `rm -rf`.

A possible command sequence is:

```bash
cd ~

mkdir -p devops-simulation/config
mkdir /home/linuxUser/devops-simulation/backup

cd devops-simulation

mkdir scripts documentation

cd config

touch app.conf database.conf

cp app.conf ../backup/app.conf.bak
cp database.conf ../backup/database.conf

cd ../backup

mv database.conf database.conf.bak

cd ..

touch scripts/deploy.sh scripts/rollback.sh
touch documentation/README.md

touch documentation/temporary.md
rm documentation/temporary.md
```

The final structure can be verified with:

```bash
cd ~
find devops-simulation
```

Expected structure:

```text
devops-simulation
devops-simulation/config
devops-simulation/config/app.conf
devops-simulation/config/database.conf
devops-simulation/backup
devops-simulation/backup/app.conf.bak
devops-simulation/backup/database.conf.bak
devops-simulation/scripts
devops-simulation/scripts/deploy.sh
devops-simulation/scripts/rollback.sh
devops-simulation/documentation
devops-simulation/documentation/README.md
```

---

## Common Mistakes

### Reversing Source and Destination

The syntax is:

```bash
cp source destination
```

The first path is the source.

The second path is the destination.

The same principle applies to `mv`.

---

### Confusing `cp` and `mv`

`cp` creates another copy.

```bash
cp file.txt backup/
```

The original file remains.

`mv` changes the file's location or name.

```bash
mv file.txt backup/
```

The original path no longer contains the file.

---

### Thinking `touch` Writes Content

```bash
touch notes.txt
```

creates an empty file or updates its timestamps.

It does not write text into the file.

---

### Using `rmdir` on a Non-Empty Directory

```bash
rmdir directory
```

works only when the directory is empty.

For a directory containing files, a recursive operation is required, but it must be used carefully.

---

### Using `rm -r` Without Checking the Path

Before running:

```bash
rm -r directory
```

verify:

```bash
pwd
ls -ld directory
```

A small path mistake can remove the wrong directory tree.

---

### Assuming `rm` Always Requests Confirmation

`rm` does not always request confirmation.

Its behaviour may depend on:

- the command options;
- shell aliases;
- system configuration.

Never depend only on a confirmation prompt for safety.

---

### Forgetting That Existing Files Can Be Overwritten

Commands such as:

```bash
cp source.txt destination.txt
mv source.txt destination.txt
```

may overwrite the destination if it already exists.

Always inspect the destination when the data is important.

---

## Lessons Learned

The most important lesson is not memorising commands.

It is understanding the effect of each command before running it.

The main differences are:

```text
touch  -> create an empty file or update timestamps
mkdir  -> create directories
cp     -> copy files or directories
mv     -> move or rename files and directories
rm     -> remove files
rmdir  -> remove empty directories
rm -r  -> remove directories recursively
```

Before modifying or removing data, verify:

```bash
whoami
pwd
ls
```

A safe DevOps workflow requires both technical knowledge and operational discipline.
