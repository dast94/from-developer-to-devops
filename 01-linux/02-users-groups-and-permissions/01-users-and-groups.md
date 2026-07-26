# Linux Users and Groups

> **Note:** This documentation uses `linuxUser` as a generic example username.  
> On the laboratory system, the real username may be different.

## Lesson Result

```text
Module: 02-users-groups-and-permissions
Lesson: 01-users-and-groups
Status: PASSED
```

This lesson introduces Linux identities, users, groups, UID, GID, system accounts and centralized identity resolution.

The practical challenge consisted of performing a read-only identity audit and creating a report containing real information from the system.

---

## What You Will Learn

This lesson covers:

- Linux as a multi-user operating system;
- users as system identities;
- human users and service accounts;
- User Identifiers — UID;
- Group Identifiers — GID;
- primary and supplementary groups;
- the `root` account;
- system accounts;
- login shells;
- `/etc/passwd`;
- `/etc/group`;
- identity resolution with `getent`;
- basic concepts of LDAP and Active Directory;
- the principle of least privilege.

Main commands:

```bash
whoami
id
groups
getent passwd
getent group
```

---

## Linux Is a Multi-User System

Linux is a multi-user operating system.

This does not only mean that several people can use the same computer. Linux associates files, directories, processes and services with precise identities.

When a command accesses a resource, Linux evaluates information such as:

- the identity executing the command;
- the groups associated with that identity;
- the owner of the target file;
- the group associated with the file;
- the permissions assigned to the owner;
- the permissions assigned to the group;
- the permissions assigned to other users;
- the permissions of parent directories;
- whether the process has additional privileges.

A message such as:

```text
Permission denied
```

does not explain the complete cause of the problem.

Possible causes include:

- the user does not own the file;
- the user does not belong to the required group;
- the file is associated with a different group;
- a parent directory cannot be traversed;
- the process is running with an unexpected identity;
- the resource belongs to `root`;
- the application should not have access to that resource.

For this reason, immediately adding `sudo` is not a proper troubleshooting strategy.

A better analysis begins with questions such as:

```text
Which user is running the service?
Which groups does that user belong to?
Who owns the configuration files?
Which group owns the configuration files?
Should the process actually have access?
```

---

## Linux Identities

A Linux user is an identity recognized by the operating system.

An identity can represent:

- a human user;
- an administrator;
- a service;
- an application;
- an automated process.

A process is not itself a user.

A process runs under a user identity and one or more group identities.

For example:

```text
User identity: postgres
Process: PostgreSQL database server
```

The process uses the permissions available to the `postgres` identity.

---

## Users and UIDs

Each Linux user has a numerical identifier called:

```text
UID
```

UID means:

```text
User Identifier
```

Example:

```text
Username: linuxUser
UID: 1000
```

The username is a human-readable label.

The UID is the numerical identifier used internally by Linux for many ownership and permission checks.

This distinction is important:

```text
linuxUser -> readable username
1000      -> numerical system identity
```

The username associated with an UID may theoretically change, while the numerical ownership stored on files remains based on the UID.

---

## Groups and GIDs

A group is a collection of one or more user identities.

Groups allow administrators to assign shared access without configuring every user individually.

Each group has a numerical identifier called:

```text
GID
```

GID means:

```text
Group Identifier
```

Example:

```text
Group name: developers
GID: 1001
```

A group may represent:

- a development team;
- system administrators;
- Docker users;
- web server operators;
- database operators;
- users allowed to access a shared resource.

---

## Primary and Supplementary Groups

Every user normally has one primary group.

A user may also belong to several supplementary groups.

Example:

```text
User: linuxUser
Primary group: linuxUser

Supplementary groups:
- sudo
- docker
- developers
```

The primary group is normally used as the group ownership for newly created files, unless another rule changes that behavior.

Supplementary groups provide additional access to resources associated with those groups.

When an ordinary command is executed, Linux considers:

- the user UID;
- the primary GID;
- all supplementary group IDs.

The user does not normally choose one supplementary group manually for each command.

All group memberships of the current process may participate in permission checks.

---

## Inspecting the Current Identity

### `whoami`

The command:

```bash
whoami
```

prints the effective username of the current process.

Example:

```text
linuxUser
```

It does not show:

- the UID;
- the primary GID;
- supplementary groups;
- numerical group identifiers.

---

### `id`

The `id` command provides more complete identity information:

```bash
id
```

Example:

```text
uid=1000(linuxUser) gid=1000(linuxUser) groups=1000(linuxUser),27(sudo),999(docker)
```

The fields mean:

```text
uid     User Identifier
gid     primary Group Identifier
groups  all groups associated with the identity
```

---

## Useful `id` Options

### Current UID

```bash
id -u
```

Example:

```text
1000
```

### Current username

```bash
id -un
```

Example:

```text
linuxUser
```

### Primary GID

```bash
id -g
```

Example:

```text
1000
```

### Primary group name

```bash
id -gn
```

Example:

```text
linuxUser
```

### All group IDs

```bash
id -G
```

Example:

```text
1000 27 999
```

### All group names

```bash
id -Gn
```

Example:

```text
linuxUser sudo docker
```

The capital letter changes the meaning:

```text
-g -> primary GID
-G -> all group IDs
```

Similarly:

```text
-gn -> primary group name
-Gn -> all group names
```

---

## The `groups` Command

The command:

```bash
groups
```

shows the groups associated with the current identity.

Example:

```text
linuxUser sudo docker developers
```

A specific user can also be queried:

```bash
groups linuxUser
```

Compared with `id`, the output is simpler but contains fewer details.

To identify only the primary group, use:

```bash
id -gn
```

---

## The Root Account

`root` is the traditional Linux administrative account.

Its most important characteristic is:

```text
UID: 0
```

A typical record is:

```text
root:x:0:0:root:/root:/bin/bash
```

The account normally has:

```text
Username: root
UID: 0
Primary GID: 0
Home directory: /root
Login shell: /bin/bash
```

The numerical UID `0` is the crucial detail.

An identity with UID `0` has special administrative privileges and can bypass many ordinary access restrictions.

`root` should not be considered merely a normal user with a few additional permissions.

Using administrative privileges unnecessarily can cause problems such as:

- modifying system files accidentally;
- deleting important directories;
- creating files owned by `root` inside a normal user's workspace;
- changing configurations without understanding the consequences;
- bypassing security controls instead of correcting permissions.

Before using `sudo`, verify the current context:

```bash
whoami
id
pwd
```

Then investigate the actual ownership and permission problem.

---

## System Accounts

Not every Linux user represents a human being.

Examples of system accounts include:

```text
root
daemon
www-data
systemd-network
systemd-timesync
messagebus
syslog
postgres
```

System accounts are commonly used to run services with dedicated identities.

Examples:

```text
Nginx or Apache -> www-data
PostgreSQL      -> postgres
System logging  -> syslog
```

A service account helps:

- isolate the service from human users;
- limit access to unrelated resources;
- make file ownership clearer;
- assign only required permissions;
- reduce the potential impact of a compromised service.

For example, the `postgres` account should normally be able to access PostgreSQL data, but it should not freely modify a web server configuration or users' personal files.

---

## Principle of Least Privilege

The security concept behind service isolation is called:

```text
Principle of Least Privilege
```

It means:

> A user, process or service should receive only the permissions required to perform its task.

Examples:

- a web server should read website files but not users' private documents;
- a database should access its database files but not unrelated application secrets;
- a deployment process should modify only the deployment directories it manages;
- a monitoring agent should read metrics without having complete administrative control.

More privileges increase the possible damage caused by:

- human errors;
- application bugs;
- misconfiguration;
- security vulnerabilities;
- compromised services.

---

## Understanding `/etc/passwd`

Basic local account information is traditionally stored in:

```text
/etc/passwd
```

A typical entry is:

```text
linuxUser:x:1000:1000:Linux User:/home/linuxUser:/bin/bash
```

Fields are separated by `:`.

The seven main fields are:

```text
1. username
2. password placeholder
3. UID
4. primary GID
5. description or GECOS field
6. home directory
7. login shell
```

For the example:

```text
Username:             linuxUser
Password placeholder: x
UID:                  1000
Primary GID:          1000
Description:          Linux User
Home directory:       /home/linuxUser
Login shell:          /bin/bash
```

Commas inside the description field do not create additional `/etc/passwd` fields.

The separator between the seven main fields is the colon:

```text
:
```

---

## Does `/etc/passwd` Contain Passwords?

Despite its name, `/etc/passwd` does not normally contain readable user passwords on modern Linux systems.

The value:

```text
x
```

is usually a placeholder.

Password hashes and password-aging information are normally stored in:

```text
/etc/shadow
```

`/etc/shadow` does not normally contain the original passwords in plain text.

It contains cryptographic password hashes and account-aging information.

A normal user should not be able to read it.

Do not use administrative privileges merely to inspect it out of curiosity.

---

## Understanding `/etc/group`

Local group information is traditionally stored in:

```text
/etc/group
```

Example:

```text
developers:x:1001:linuxUser,alice
```

The fields are:

```text
1. group name
2. password placeholder
3. GID
4. supplementary members
```

For the example:

```text
Group name:            developers
Password placeholder: x
GID:                   1001
Members:               linuxUser, alice
```

The final members field may be empty:

```text
linuxUser:x:1000:
```

This does not necessarily mean that nobody belongs to the group.

A user's primary group is determined by the primary GID stored in the user's account record.

The final field of `/etc/group` mainly lists explicit supplementary memberships.

---

## Querying Identities with `getent`

Account files can be read directly:

```bash
cat /etc/passwd
cat /etc/group
```

However, `getent` is often more appropriate:

```bash
getent passwd
getent group
```

To query a specific user:

```bash
getent passwd linuxUser
```

To query a specific group:

```bash
getent group developers
```

`getent` means conceptually:

> Get entries from a system database configured for name resolution.

It uses the identity sources configured through the system's Name Service Switch, commonly called NSS.

This matters because an account may not be stored only in local files.

Identity information may also come from:

- LDAP;
- Active Directory;
- centralized directory services;
- other NSS-compatible sources.

Therefore:

```bash
cat /etc/passwd
```

mainly shows local account records.

Meanwhile:

```bash
getent passwd username
```

can search all configured account sources.

---

## LDAP

LDAP means:

```text
Lightweight Directory Access Protocol
```

LDAP is a protocol used to access and manage structured directory information.

An organization may centrally store information such as:

```text
Username
UID
Group memberships
Email address
Department
Organization
Home directory
Access attributes
```

Instead of creating the same user independently on many servers, the servers can query a central directory.

Simplified example:

```text
Central LDAP directory
        |
        +-- Linux server 1
        +-- Linux server 2
        +-- VPN service
        +-- Git platform
        +-- Internal applications
```

LDAP is not simply a password database.

It is a protocol for reading and managing hierarchical directory data.

---

## Active Directory

Active Directory is Microsoft's centralized identity and domain-management platform.

It can manage:

- users;
- groups;
- computers;
- authentication;
- authorization;
- organizational units;
- security policies;
- domain resources.

Active Directory commonly works with technologies such as:

```text
LDAP     -> querying directory information
Kerberos -> authentication
DNS      -> locating domain services
Group Policy -> centralized Windows configuration
```

Linux systems can be integrated with an Active Directory domain.

In that situation, an employee may log in with a centralized company account even when the account does not appear as a local row in `/etc/passwd`.

A command such as:

```bash
getent passwd company-user
```

may still return the identity if the Linux system is configured to resolve users through Active Directory.

---

## Login Shells

The last field of a user record normally indicates the configured login shell.

Possible values include:

```text
/bin/bash
/bin/sh
/usr/sbin/nologin
/bin/false
```

### Interactive shells

Examples:

```text
/bin/bash
/bin/sh
```

These may allow interactive command sessions, depending on the rest of the system configuration.

### Non-interactive account shells

Examples:

```text
/usr/sbin/nologin
/bin/false
```

These values are commonly used for service accounts that should not start ordinary interactive login sessions.

An account with `/usr/sbin/nologin` may still:

- own files;
- run a service;
- execute processes started by systemd;
- access application resources;
- be used internally by software.

`nologin` does not mean that the identity is unused.

---

## Practical Laboratory

### Inspecting the Current Identity

Commands:

```bash
whoami
id
groups
```

Example observations:

```text
Current username: linuxUser
Current UID: 1000
Primary group: linuxUser
Primary GID: 1000
Supplementary groups: sudo, docker, developers
```

---

### Inspecting Individual Identity Values

Commands:

```bash
id -u
id -un
id -g
id -gn
id -G
id -Gn
```

Example table:

| Command | Meaning |
|---|---|
| `id -u` | Current UID |
| `id -un` | Current username |
| `id -g` | Primary GID |
| `id -gn` | Primary group name |
| `id -G` | All group IDs |
| `id -Gn` | All group names |

---

### Inspecting the Current User Record

```bash
getent passwd linuxUser
```

Example:

```text
linuxUser:x:1000:1000:Linux User:/home/linuxUser:/bin/bash
```

Values obtained:

```text
UID: 1000
Primary GID: 1000
Home directory: /home/linuxUser
Login shell: /bin/bash
```

---

### Inspecting Root

```bash
getent passwd root
id root
```

Example:

```text
root:x:0:0:root:/root:/bin/bash
```

```text
uid=0(root) gid=0(root) groups=0(root)
```

The UID confirms the privileged identity:

```text
UID 0
```

---

### Inspecting System Accounts

The account database was opened using:

```bash
getent passwd | less
```

Inside `less`:

```text
/nologin
n
q
```

Meaning:

```text
/nologin -> search for nologin
n        -> move to the next match
q        -> exit less
```

Examples of identified system accounts:

```text
systemd-network
systemd-timesync
messagebus
syslog
```

Possible deductions based on their names:

```text
systemd-network  -> likely related to network management
systemd-timesync -> likely related to time synchronization
messagebus       -> likely related to the system message bus
syslog           -> likely related to system logging
```

These are reasonable deductions, but they should not be presented as verified facts based only on the account name.

---

### Inspecting Groups

The group database was opened with:

```bash
getent group | less
```

Individual groups were queried with:

```bash
getent group linuxUser
getent group sudo
getent group cdrom
```

Example:

```text
linuxUser:x:1000:
```

The empty final field does not invalidate the primary group membership.

Example supplementary membership:

```text
sudo:x:27:linuxUser
```

---

## Command Substitution

The syntax:

```bash
$(command)
```

executes a command and replaces the expression with its output.

Example:

```bash
echo "$(whoami)"
```

If `whoami` returns:

```text
linuxUser
```

the shell effectively passes that value to `echo`.

Another example:

```bash
getent passwd "$(whoami)"
```

Execution occurs conceptually in two steps:

```text
1. Execute whoami
2. Use its output as the username passed to getent
```

This produces an effect similar to:

```bash
getent passwd linuxUser
```

Command substitution is useful when a value must be derived dynamically instead of being written manually.

---

## Formatted Output with `printf`

The `printf` command prints formatted text.

Basic example:

```bash
printf "Current user: %s\n" "$(id -un)"
```

Output:

```text
Current user: linuxUser
```

Important elements:

```text
%s -> string placeholder
\n -> newline
```

Multiple values can be supplied:

```bash
printf "User: %s\nUID: %s\n" \
  "$(id -un)" \
  "$(id -u)"
```

Output:

```text
User: linuxUser
UID: 1000
```

Using placeholders is clearer and safer than repeatedly closing and reopening quoted strings.

---

## `print` and `printf`

The command required for this laboratory was:

```bash
printf
```

`print` is not the standard Bash equivalent used for this formatted-output task.

Depending on the installed software and shell environment, `print` may refer to a different program and interpret its arguments in an unexpected way.

This can cause words from an output such as:

```text
linuxUser sudo docker developers
```

to be treated as separate arguments or filenames.

Use:

```bash
printf
```

for predictable formatted output in Bash.

---

## Output Redirection

To overwrite a file:

```bash
printf "New content\n" > report.txt
```

To append content:

```bash
printf "Additional content\n" >> report.txt
```

Difference:

```text
>  -> overwrite or create
>> -> append or create
```

When a previous attempt may have produced invalid content, use `>` to regenerate the file cleanly.

---

## Common Mistakes

### Treating a Process as a User

Incorrect:

```text
A process is a user.
```

Correct:

```text
A process runs under a user and group identity.
```

---

### Confusing Username and UID

Incorrect:

```text
linuxUser and 1000 are the same kind of value.
```

Correct:

```text
linuxUser -> readable account name
1000      -> numerical UID
```

---

### Confusing Primary and Supplementary Groups

The primary group is not the only group considered during ordinary permission checks.

Linux considers the process's primary and supplementary group memberships.

---

### Confusing `id -g` and `id -G`

```bash
id -g
```

returns the primary GID.

```bash
id -G
```

returns all group IDs.

The uppercase letter changes the result.

---

### Assuming Every Account Is a Person

Many accounts exist only to run applications or system services.

Do not delete or modify an unfamiliar account without understanding its role.

---

### Assuming `/etc/passwd` Contains Plain-Text Passwords

Modern Linux systems normally use a placeholder such as:

```text
x
```

The original password is not available there in plain text.

---

### Assuming `/etc/shadow` Contains Readable Passwords

`/etc/shadow` normally stores password hashes and password-aging information, not the original password.

---

### Assuming `nologin` Means the Account Does Nothing

A service account with:

```text
/usr/sbin/nologin
```

can still own files and execute a service.

It is mainly prevented from opening a normal interactive login shell.

---

### Assuming Empty Group Members Means No Membership

A record such as:

```text
linuxUser:x:1000:
```

may still be the primary group of `linuxUser`.

Primary group association comes from the GID stored in the user's account record.

---

### Reading Only `/etc/passwd` in an Enterprise Environment

Local files may not include identities stored in:

- LDAP;
- Active Directory;
- other centralized identity systems.

Use `getent` to query the configured system identity sources.

---

### Using `sudo` Instead of Troubleshooting

`sudo` can hide the real cause of a permission problem.

Before elevating privileges, verify:

```bash
whoami
id
groups
pwd
```

Then inspect ownership and permissions.

---

## Identity Audit Challenge

### Objective

Create a read-only identity report without:

- creating users;
- creating groups;
- changing permissions;
- modifying system files;
- using `sudo`.

Required structure:

```text
identity-audit/
└── report.txt
```

---

### Creating the Workspace

```bash
mkdir identity-audit
touch identity-audit/report.txt
```

---

### Collecting User Information

Current identity:

```bash
whoami
id
groups
```

Current account record:

```bash
getent passwd linuxUser
```

Root account record:

```bash
getent passwd root
```

Root identity:

```bash
id root
```

System accounts:

```bash
getent passwd systemd-network
getent passwd systemd-timesync
getent passwd messagebus
```

---

### Creating the Report

Example:

```bash
printf "Linux Identity Audit\nCurrent user: %s\nCurrent UID: %s\nPrimary group: %s\nPrimary GID: %s\nAll groups: %s\nHome directory: %s\nLogin shell: %s\nRoot UID: %s\nRoot home: %s\nRoot shell: %s\n" \
  "$(id -un)" \
  "$(id -u)" \
  "$(id -gn)" \
  "$(id -g)" \
  "$(id -Gn)" \
  "/home/linuxUser" \
  "/bin/bash" \
  "0" \
  "/root" \
  "/bin/bash" \
  > identity-audit/report.txt
```

The home directory and login shell values must be obtained from:

```bash
getent passwd linuxUser
```

They should not be invented.

---

### Adding System Accounts

```bash
printf "System account 1: %s\nSystem account 2: %s\nSystem account 3: %s\n" \
  "systemd-network - shell /usr/sbin/nologin - likely related to network management" \
  "systemd-timesync - shell /usr/sbin/nologin - likely related to time synchronization" \
  "messagebus - shell /usr/sbin/nologin - likely related to the system message bus" \
  >> identity-audit/report.txt
```

The word:

```text
likely
```

distinguishes a deduction from a verified fact.

---

### Final Report Structure

```text
Linux Identity Audit
Current user: linuxUser
Current UID: 1000
Primary group: linuxUser
Primary GID: 1000
All groups: linuxUser sudo docker developers
Home directory: /home/linuxUser
Login shell: /bin/bash
Root UID: 0
Root home: /root
Root shell: /bin/bash
System account 1: systemd-network - shell /usr/sbin/nologin - likely related to network management
System account 2: systemd-timesync - shell /usr/sbin/nologin - likely related to time synchronization
System account 3: messagebus - shell /usr/sbin/nologin - likely related to the system message bus
```

---

### Verifying the Report

```bash
cat identity-audit/report.txt
```

The report must use real system values.

The following should be checked:

- current username;
- current UID;
- primary group;
- primary GID;
- all groups;
- home directory;
- login shell;
- root UID;
- root home directory;
- root login shell;
- three valid system accounts;
- `nologin` shells;
- deductions clearly marked as deductions.

---

## Laboratory Result

The practical laboratory successfully demonstrated:

- identity inspection with `whoami`;
- detailed identity inspection with `id`;
- group inspection with `groups`;
- local and centralized-style account resolution with `getent`;
- interpretation of `/etc/passwd` records;
- interpretation of `/etc/group` records;
- distinction between UID and username;
- distinction between GID and group name;
- distinction between primary and supplementary groups;
- identification of system accounts;
- interpretation of login shells;
- safe generation of an audit report with `printf`;
- command substitution;
- output redirection;
- read-only system investigation.

Final result:

```text
01-users-and-groups: PASSED
```

---

## Lessons Learned

Linux security and access control begin with identity.

Before investigating permissions, it is necessary to understand:

```text
Who is executing the operation?
Which UID is being used?
Which primary group is associated?
Which supplementary groups are active?
Which account owns the resource?
Is the account human or technical?
Should the process have access?
```

Important distinctions:

```text
Username != UID
Group name != GID
Primary group != only active group
Service account != human account
nologin != inactive identity
/etc/passwd != plain-text password database
```

A reliable troubleshooting process begins with:

```bash
whoami
id
groups
getent passwd USERNAME
getent group GROUPNAME
```

The central DevOps principle introduced in this lesson is:

```text
Principle of Least Privilege
```

Every account and service should receive only the access required to perform its task.
