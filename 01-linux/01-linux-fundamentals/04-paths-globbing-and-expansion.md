# Linux Paths, Globbing and Shell Expansion

> **Note:** In this guide, `linuxUser` is used as an example username.  
> Replace it with your actual Linux username when running the commands.

## What You Will Learn

In this guide, you will learn how Bash processes paths and arguments before executing a command.

The main topics are:

```text
*
?
[...]
{...}
~
$VARIABLE
"double quotes"
'single quotes'
\ escape
-- end of options
```

The objective is not only to recognise this syntax.

The objective is to understand:

- what the shell expands;
- when the expansion happens;
- which arguments are passed to a command;
- how quoting changes shell behaviour;
- how to safely operate on multiple files;
- how to handle paths containing spaces.

---

## How the Shell Processes Arguments

Consider this command:

```bash
ls *.log
```

The `ls` program does not necessarily receive the literal string:

```text
*.log
```

Before executing `ls`, Bash searches the current directory for filenames matching the pattern.

For example, Bash may transform:

```bash
ls *.log
```

into:

```bash
ls application.log error.log service.log
```

This process is called:

```text
pathname expansion
```

It is commonly known as:

```text
globbing
```

This distinction is important because the shell performs the expansion before the command is executed.

A command such as:

```bash
rm *.log
```

does not simply mean “remove logs”.

It may become:

```bash
rm application.log error.log service.log
```

The exact files depend on:

- the current working directory;
- the files that currently exist;
- the pattern used;
- shell configuration.

---

## Why This Is Important

Using shell expansions incorrectly can cause serious mistakes.

A wrong pattern may:

- copy the wrong files;
- move more files than expected;
- overwrite existing data;
- remove too many files;
- break a script;
- pass the wrong arguments to a command;
- split one path into multiple arguments;
- operate in the wrong directory.

Before executing:

```bash
rm *.log
```

a DevOps Engineer should ask:

- Which directory am I currently in?
- Which files match this pattern?
- Are any of these files important?
- Does the pattern include more files than expected?
- What happens if there are no matches?

Before a destructive operation, inspect the expansion using a non-destructive command:

```bash
printf '%s\n' *.log
```

Another possible verification is:

```bash
ls -l -- *.log
```

The first command is often clearer when the purpose is only to inspect the expanded arguments.

---

## How Expansions Are Used in DevOps

Copy all YAML files:

```bash
cp *.yaml backup/
```

Count the lines in multiple log files:

```bash
wc -l logs/*.log
```

Select configuration files with one variable character:

```bash
ls configs/app-?.conf
```

Create multiple environment directories:

```bash
mkdir -p environments/{development,staging,production}
```

Use the current user's home directory:

```bash
cd ~/projects
```

Use a variable inside a path:

```bash
cd "$HOME/projects"
```

Access a directory whose name contains spaces:

```bash
cd "Project Files"
```

These concepts are used in:

- Bash scripts;
- deployment procedures;
- CI/CD pipelines;
- backup operations;
- Docker build instructions;
- configuration management;
- log management;
- cloud automation.

---

## How Spaces Separate Arguments

The shell normally uses spaces to separate command arguments.

This command:

```bash
touch application config.txt
```

creates two files:

```text
application
config.txt
```

To create a single file named:

```text
application config.txt
```

use double quotes:

```bash
touch "application config.txt"
```

Alternatively, escape the space:

```bash
touch application\ config.txt
```

In both cases, the filename is passed to `touch` as one argument.

---

## Wildcard `*`

The asterisk matches zero or more characters in a filename.

Suppose the current directory contains:

```text
app
app.log
application.log
database.log
README.md
```

The pattern:

```bash
ls *.log
```

may match:

```text
app.log
application.log
database.log
```

The pattern:

```bash
ls app*
```

may match:

```text
app
app.log
application.log
```

The asterisk can match zero characters. This is why `app*` can also match a file named exactly `app`.

---

### Hidden Files

The pattern:

```bash
ls *
```

does not normally include filenames beginning with a dot.

For example:

```text
.hidden.log
.config
.env
```

are normally excluded.

Therefore:

```text
*
```

does not literally mean every file in the directory.

---

## Single-Character Pattern `?`

The question mark matches exactly one character.

Suppose the directory contains:

```text
app-1.log
app-2.log
app-a.log
app-10.log
```

The pattern:

```bash
ls app-?.log
```

may match:

```text
app-1.log
app-2.log
app-a.log
```

It does not match:

```text
app-10.log
```

because there are two characters between `app-` and `.log`.

The `?` pattern does not specifically mean a digit. It means any single character.

To require one digit, use a character class:

```bash
ls app-[0-9].log
```

---

## Character Classes

Square brackets match one character from a defined set or range.

### Specific Characters

```bash
ls app-[123].log
```

may match:

```text
app-1.log
app-2.log
app-3.log
```

---

### Numeric Range

```bash
ls app-[1-5].log
```

may match:

```text
app-1.log
app-2.log
app-3.log
app-4.log
app-5.log
```

The range applies to exactly one character.

It does not match:

```text
app-10.log
```

---

### Alphabetic Range

```bash
ls server-[a-c].conf
```

may match:

```text
server-a.conf
server-b.conf
server-c.conf
```

---

### Negation

```bash
ls app-[!1].log
```

matches filenames containing exactly one character in that position, provided that the character is not `1`.

Possible matches include:

```text
app-2.log
app-a.log
```

The exact behaviour of ranges can be affected by locale settings, so complex patterns should always be verified.

---

## Brace Expansion

Brace expansion generates strings.

It does not search for existing files.

Example:

```bash
echo environment-{dev,test,prod}
```

The shell generates:

```text
environment-dev
environment-test
environment-prod
```

These strings are generated even if no files with those names exist.

---

### Creating Multiple Directories

```bash
mkdir -p environments/{development,staging,production}
```

This is conceptually equivalent to:

```bash
mkdir -p environments/development \
         environments/staging \
         environments/production
```

---

### Numeric Sequences

```bash
echo file-{1..5}.txt
```

produces:

```text
file-1.txt
file-2.txt
file-3.txt
file-4.txt
file-5.txt
```

The sequence uses two dots:

```text
{1..5}
```

This is not equivalent to:

```text
{1-5}
```

---

### Combining Brace Expansions

Brace expansions can generate combinations.

```bash
touch configs/{app,database}-{development,staging,production}.conf
```

This generates:

```text
configs/app-development.conf
configs/app-staging.conf
configs/app-production.conf
configs/database-development.conf
configs/database-staging.conf
configs/database-production.conf
```

Nested brace expansion is also possible:

```bash
touch logs/deploy-{{1..3},10}.log
```

This generates:

```text
logs/deploy-1.log
logs/deploy-2.log
logs/deploy-3.log
logs/deploy-10.log
```

Although nested brace expansion can be compact, readability should still be considered in production scripts.

---

## Globbing vs Brace Expansion

Globbing searches for existing filenames.

Example:

```bash
ls *.log
```

The shell searches for existing files ending in `.log`.

Brace expansion generates strings.

Example:

```bash
echo {app,database}.log
```

The shell produces:

```text
app.log
database.log
```

whether the files exist or not.

The two mechanisms serve different purposes:

```text
Globbing        -> matches existing filenames
Brace expansion -> generates strings
```

---

## Expansion Order and Variables

Brace expansion happens before variable expansion.

This attempt does not create a reusable list in the expected way:

```bash
ENVIRONMENT={development,staging,production}
touch configs/{app-$ENVIRONMENT,database-$ENVIRONMENT}.conf
```

A variable is not automatically interpreted later as new brace-expansion syntax.

The direct and correct solution is:

```bash
touch configs/{app,database}-{development,staging,production}.conf
```

A variable normally contains data, not delayed Bash code.

Do not store shell syntax inside a variable and expect Bash to evaluate it again automatically.

---

## Tilde Expansion

The tilde normally represents the current user's home directory.

```bash
cd ~
```

For the example user, it expands to:

```text
/home/linuxUser
```

It can also be used inside a path:

```bash
cd ~/projects
```

which becomes:

```text
/home/linuxUser/projects
```

---

### Quoted Tilde

This command:

```bash
echo "~"
```

normally prints:

```text
~
```

The tilde is not expanded because it is inside double quotes.

---

## Variables in Paths

A shell variable associates a name with a value.

```bash
PROJECT_DIR="/home/linuxUser/projects"
```

Read the variable:

```bash
echo "$PROJECT_DIR"
```

Output:

```text
/home/linuxUser/projects
```

Use it in a command:

```bash
cd "$PROJECT_DIR"
```

The dollar sign requests variable expansion:

```text
$PROJECT_DIR
```

The shell replaces the reference with the variable's value.

---

## Why Variables Should Usually Be Quoted

Suppose:

```bash
PROJECT_DIR="/home/linuxUser/Project Files"
```

This is fragile:

```bash
cd $PROJECT_DIR
```

After expansion, the shell may split the value into two arguments:

```text
/home/linuxUser/Project
Files
```

This is safer:

```bash
cd "$PROJECT_DIR"
```

The double quotes preserve the complete value as a single argument.

A useful default habit is:

```bash
"$VARIABLE"
```

instead of:

```bash
$VARIABLE
```

There are cases where unquoted expansion is intentional, but those should be deliberate exceptions.

---

## Double Quotes

Double quotes:

- preserve spaces;
- prevent pathname expansion;
- allow variable expansion;
- allow some other shell expansions.

Example:

```bash
NAME="linuxUser"
echo "Hello $NAME"
```

Output:

```text
Hello linuxUser
```

This command:

```bash
echo "*.log"
```

prints:

```text
*.log
```

The asterisk is not expanded because it is quoted.

---

## Single Quotes

Single quotes treat almost all characters as literal text.

```bash
NAME="linuxUser"
echo 'Hello $NAME'
```

Output:

```text
Hello $NAME
```

The variable is not expanded.

Compare:

```bash
echo "$HOME"
```

This prints the value of the variable.

```bash
echo '$HOME'
```

This prints:

```text
$HOME
```

---

## Escaping Characters

The backslash protects the next character from its normal shell interpretation.

### Escaping a Space

```bash
cd Project\ Files
```

The space becomes part of the directory name.

---

### Escaping a Dollar Sign

```bash
echo \$HOME
```

Output:

```text
$HOME
```

The dollar sign is treated literally, so variable expansion does not happen.

---

### Escaping an Asterisk

```bash
echo \*.log
```

Output:

```text
*.log
```

The asterisk is treated literally, so globbing does not happen.

---

## Comparing Expansion Behaviour

Suppose the `logs/` directory contains `.log` files.

### Unquoted Pattern

```bash
echo logs/*.log
```

Bash expands the pattern and passes the matching filenames to `echo`.

Possible output:

```text
logs/app-1.log logs/app-2.log logs/database.log
```

---

### Double-Quoted Pattern

```bash
echo "logs/*.log"
```

Output:

```text
logs/*.log
```

The double quotes prevent globbing.

---

### Escaped Asterisk

```bash
echo logs/\*.log
```

Output:

```text
logs/*.log
```

Only the asterisk is escaped, so it loses its special meaning.

---

## End of Options with `--`

Many Linux commands use:

```text
--
```

to mark the end of command options.

Suppose a file is named:

```text
-important.conf
```

This command is ambiguous:

```bash
rm -important.conf
```

The filename may be interpreted as one or more command options.

Safer forms are:

```bash
rm -- -important.conf
```

or:

```bash
rm ./-important.conf
```

In the first example, `--` tells the command:

> Stop interpreting the following arguments as options.

In the second example, `./` makes the argument an explicit path.

`--` does not distinguish commands from variables. It normally separates command options from positional arguments.

---

## Safe Pattern Workflow

Before using a pattern in a destructive command, inspect what it matches.

Dangerous without verification:

```bash
rm logs/*.log
```

Safer workflow:

```bash
pwd
printf '%s\n' logs/*.log
```

After confirming the exact files, the operation may be executed if it is really required.

For an important operation, also inspect details:

```bash
ls -l -- logs/*.log
```

A safe workflow is:

1. verify the current directory;
2. inspect the pattern expansion;
3. confirm the target files;
4. execute the operation;
5. verify the result.

Do not rely only on memory or assumptions about which files are present.

---

## Practical Laboratory

Create the complete laboratory structure:

```bash
cd ~

mkdir -p linux-expansion-lab/{logs,configs,environments}
```

The resulting structure is:

```text
linux-expansion-lab/
├── logs/
├── configs/
└── environments/
```

---

### Creating Numbered Log Files

```bash
touch linux-expansion-lab/logs/app-{1..5}.log
```

Create additional files:

```bash
touch linux-expansion-lab/logs/{{app-10,database}.log,README.txt}
```

The final files are:

```text
app-1.log
app-2.log
app-3.log
app-4.log
app-5.log
app-10.log
database.log
README.txt
```

A more explicit alternative is:

```bash
touch linux-expansion-lab/logs/{app-10,database}.log \
      linux-expansion-lab/logs/README.txt
```

---

### Testing Patterns

Enter the log directory:

```bash
cd ~/linux-expansion-lab/logs
```

Show all `.log` files:

```bash
ls *.log
```

Expected matches:

```text
app-1.log
app-2.log
app-3.log
app-4.log
app-5.log
app-10.log
database.log
```

Show all names beginning with `app-`:

```bash
ls app-*
```

Show only files numbered from 1 to 5:

```bash
ls app-[1-5].log
```

Show files containing exactly one character after `app-`:

```bash
ls app-?.log
```

This excludes:

```text
app-10.log
```

Show all non-hidden entries:

```bash
ls
```

---

### Creating Environment Directories

Return to the laboratory root:

```bash
cd ..
```

Create the environment directories:

```bash
mkdir -p environments/{development,staging,production}
```

Create one README file inside each directory:

```bash
touch environments/{development,staging,production}/README.md
```

Verify recursively:

```bash
find environments
```

Expected structure:

```text
environments
environments/development
environments/development/README.md
environments/staging
environments/staging/README.md
environments/production
environments/production/README.md
```

A quick, non-recursive alternative for this specific structure is:

```bash
ls environments/*/
```

However, this is not generally equivalent to `find` because:

- it follows only the selected pattern;
- it does not recursively traverse arbitrary depth;
- globbing normally excludes hidden entries;
- it depends on the exact directory structure.

---

### Working with Spaces

Create a directory containing a space:

```bash
mkdir "Project Files"
```

Enter using double quotes:

```bash
cd "Project Files"
```

Verify:

```bash
pwd
```

Return and enter using an escaped space:

```bash
cd ..
cd Project\ Files
```

Both forms treat the path as one argument.

---

### Using a Variable with a Path

From the laboratory directory:

```bash
PROJECT_DIR="/home/linuxUser/linux-expansion-lab/Project Files"
```

Enter the directory:

```bash
cd "$PROJECT_DIR"
```

Verify:

```bash
pwd
```

Quoting the variable is necessary because the value contains a space.

---

### Comparing Quotes and Escaping

```bash
echo "$HOME"
```

This expands the variable and prints its value.

```bash
echo '$HOME'
```

This prints the literal text:

```text
$HOME
```

```bash
echo \$HOME
```

This also prints:

```text
$HOME
```

In the third case, only the dollar sign is escaped.

---

## Common Mistakes

### Using `*` Without Verification

Before:

```bash
rm logs/*.log
```

inspect the matches:

```bash
printf '%s\n' logs/*.log
```

A broad pattern can affect more files than expected.

---

### Assuming `*` Includes Hidden Files

Normally:

```bash
*
```

does not match filenames beginning with `.`.

---

### Confusing `?` with a Digit

This:

```bash
app-?.log
```

matches exactly one character, not necessarily one number.

To require a digit:

```bash
app-[0-9].log
```

---

### Confusing Globbing and Brace Expansion

```bash
*.log
```

matches existing filenames.

```bash
{app,database}.log
```

generates two strings.

---

### Using `{1-5}` Instead of `{1..5}`

Incorrect sequence syntax:

```bash
file-{1-5}.txt
```

Correct:

```bash
file-{1..5}.txt
```

Character ranges such as `[1-5]` are used in glob patterns, not for generating a numeric sequence.

---

### Expecting a Variable to Trigger Brace Expansion

This approach does not work as delayed brace expansion:

```bash
ENVIRONMENT="{development,staging,production}"
touch configs/app-$ENVIRONMENT.conf
```

The variable contains a string. Bash does not normally reinterpret its expanded value as new brace syntax.

---

### Not Quoting Variables

Fragile:

```bash
cd $PROJECT_DIR
```

Safer:

```bash
cd "$PROJECT_DIR"
```

Without quotes, whitespace in the value may split it into several arguments.

---

### Quoting a Pattern That Should Expand

This expands:

```bash
ls *.log
```

This does not:

```bash
ls "*.log"
```

The quoted command searches for a literal filename containing an asterisk, which normally does not exist.

---

### Confusing Single and Double Quotes

Double quotes allow variable expansion:

```bash
echo "$HOME"
```

Single quotes do not:

```bash
echo '$HOME'
```

---

### Handling Filenames Beginning with `-`

Unsafe or ambiguous:

```bash
rm -important.conf
```

Safe:

```bash
rm -- -important.conf
```

or:

```bash
rm ./-important.conf
```

---

### Breaking a Path with Spaces

Incorrect:

```bash
cd Project Files
```

The shell sees two arguments.

Correct:

```bash
cd "Project Files"
```

or:

```bash
cd Project\ Files
```

---

## Challenge

The challenge was to build:

```text
deployment-preparation/
├── configs/
│   ├── app-development.conf
│   ├── app-staging.conf
│   ├── app-production.conf
│   ├── database-development.conf
│   ├── database-staging.conf
│   └── database-production.conf
├── logs/
│   ├── deploy-1.log
│   ├── deploy-2.log
│   ├── deploy-3.log
│   └── deploy-10.log
├── environments/
│   ├── development/
│   │   └── README.md
│   ├── staging/
│   │   └── README.md
│   └── production/
│       └── README.md
└── Release Notes/
    └── summary.txt
```

---

### Creating the Directory Structure

```bash
cd ~

mkdir -p deployment-preparation/{configs,logs,environments/{development,staging,production},Release\ Notes}
```

This command uses:

- brace expansion;
- nested brace expansion;
- an escaped space.

---

### Creating Configuration Files

```bash
touch deployment-preparation/configs/{app,database}-{development,staging,production}.conf
```

This generates six files:

```text
app-development.conf
app-staging.conf
app-production.conf
database-development.conf
database-staging.conf
database-production.conf
```

---

### Creating Deployment Logs

```bash
touch deployment-preparation/logs/deploy-{{1..3},10}.log
```

This creates:

```text
deploy-1.log
deploy-2.log
deploy-3.log
deploy-10.log
```

---

### Creating Environment Documentation

```bash
touch deployment-preparation/environments/{development,staging,production}/README.md
```

---

### Creating the Release Notes

```bash
touch "deployment-preparation/Release Notes/summary.txt"
```

This command uses double quotes to preserve the space in the directory name.

---

### Using a Variable to Reach `Release Notes`

```bash
RELEASE_DIR="$HOME/deployment-preparation/Release Notes"
cd "$RELEASE_DIR"
pwd
```

The variable must be quoted because its value contains a space.

Return home:

```bash
cd ~
```

---

### Selecting Only Logs 1 to 3

```bash
ls deployment-preparation/logs/deploy-[1-3].log
```

Expected matches:

```text
deployment-preparation/logs/deploy-1.log
deployment-preparation/logs/deploy-2.log
deployment-preparation/logs/deploy-3.log
```

It does not include:

```text
deployment-preparation/logs/deploy-10.log
```

---

### Selecting All Configuration Files

```bash
ls deployment-preparation/configs/*.conf
```

The glob is expanded to all six `.conf` files.

---

### Printing a Pattern Literally

Using double quotes:

```bash
echo "deployment-preparation/configs/*.conf"
```

Output:

```text
deployment-preparation/configs/*.conf
```

The asterisk is not expanded.

An escaped asterisk would also work:

```bash
echo deployment-preparation/configs/\*.conf
```

---

### Final Verification

```bash
find deployment-preparation
```

Expected structure:

```text
deployment-preparation
deployment-preparation/configs
deployment-preparation/configs/app-development.conf
deployment-preparation/configs/app-staging.conf
deployment-preparation/configs/app-production.conf
deployment-preparation/configs/database-development.conf
deployment-preparation/configs/database-staging.conf
deployment-preparation/configs/database-production.conf
deployment-preparation/logs
deployment-preparation/logs/deploy-1.log
deployment-preparation/logs/deploy-2.log
deployment-preparation/logs/deploy-3.log
deployment-preparation/logs/deploy-10.log
deployment-preparation/environments
deployment-preparation/environments/development
deployment-preparation/environments/development/README.md
deployment-preparation/environments/staging
deployment-preparation/environments/staging/README.md
deployment-preparation/environments/production
deployment-preparation/environments/production/README.md
deployment-preparation/Release Notes
deployment-preparation/Release Notes/summary.txt
```

During the exercise, incorrectly created log files were removed and recreated with the required `.log` extension.

This correction demonstrated an important operational lesson: verify generated names before continuing with later steps.

In production work, restrictions such as “do not use `rm`” must be treated as real operational constraints rather than optional suggestions.

---

## Lessons Learned

The shell can transform command arguments before the program is executed.

The main mechanisms are:

```text
*              -> zero or more characters
?              -> exactly one character
[...]          -> one character from a set or range
{...}          -> generate strings
~              -> current user's home directory
$VARIABLE      -> replace a variable reference with its value
"..."          -> preserve spaces and allow variable expansion
'...'          -> preserve literal text
\              -> escape the following character
--             -> normally marks the end of command options
```

The most important distinctions are:

```text
Globbing        -> matches existing filenames
Brace expansion -> generates strings
```

and:

```text
Double quotes -> allow variable expansion
Single quotes -> prevent variable expansion
```

Before executing a broad or destructive command:

```bash
pwd
printf '%s\n' pattern
```

should be part of the verification process.

A DevOps Engineer should understand not only what the written command appears to mean, but also which exact arguments the shell will pass to the executed program.
