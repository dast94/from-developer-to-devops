# Linux Fundamentals — Practical Exam

> **Note:** In this guide, `linuxUser` is used as an example username.  
> Replace it with your actual Linux username when running the commands.

## Exam Result

```text
Module: 01-linux-fundamentals
Status: PASSED
Score: 91/100
```

This practical exam verified the ability to combine the Linux fundamentals studied in the previous lessons.

The exam focused on:

- filesystem navigation;
- file and directory management;
- text-file inspection;
- output redirection;
- globbing;
- brace expansion;
- quoting and escaping;
- shell variables;
- operational safety;
- verification of results.

---

## Scenario

The task was to prepare a small deployment workspace for three environments:

```text
development
staging
production
```

The workspace had to contain:

- source configuration files;
- backup copies;
- deployment logs;
- environment documentation;
- a final deployment report.

The original configuration files had to remain unchanged and available after the backup process.

The final structure was:

```text
linux-fundamentals-exam/
├── source-configs/
│   ├── api-development.conf
│   ├── api-staging.conf
│   ├── api-production.conf
│   ├── database-development.conf
│   ├── database-staging.conf
│   └── database-production.conf
├── backups/
│   ├── api-development.conf.bak
│   ├── api-staging.conf.bak
│   ├── api-production.conf.bak
│   ├── database-development.conf.bak
│   ├── database-staging.conf.bak
│   └── database-production.conf.bak
├── logs/
│   ├── deployment-1.log
│   ├── deployment-2.log
│   ├── deployment-3.log
│   └── deployment-10.log
├── environments/
│   ├── development/
│   │   └── README.md
│   ├── staging/
│   │   └── README.md
│   └── production/
│       └── README.md
└── Deployment Report/
    └── summary.txt
```

---

## Exam Restrictions

The exam had to be completed without using:

```text
sudo
rm -r
rm -rf
grep
sed
awk
xargs
for
while
if
graphical text editors
```

The objective was to solve the task using only the commands studied in the Linux fundamentals module.

---

## Commands and Concepts Used

The exam required the practical use of:

```text
pwd
whoami
cd
ls
find
touch
mkdir
cp
mv
rm
cat
less
head
tail
wc
file
printf
echo
```

It also required:

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
> overwrite redirection
>> append redirection
```

---

## 1. Preparing the Structure

Before creating anything, the current user and directory were verified:

```bash
cd ~
pwd
whoami
```

Example output:

```text
/home/linuxUser
linuxUser
```

The main directory structure was created using `mkdir -p` and brace expansion:

```bash
mkdir -p linux-fundamentals-exam/{source-configs,backups,logs,environments,"Deployment Report"}
```

The three environment directories were created using an absolute path and brace expansion:

```bash
mkdir -p /home/linuxUser/linux-fundamentals-exam/environments/{development,staging,production}
```

The structure was verified with:

```bash
find linux-fundamentals-exam
```

---

## 2. Creating Configuration Files

The required configuration filenames were generated using brace expansion:

```bash
touch linux-fundamentals-exam/source-configs/{api,database}-{development,staging,production}.conf
```

This generated:

```text
api-development.conf
api-staging.conf
api-production.conf
database-development.conf
database-staging.conf
database-production.conf
```

Brace expansion generates strings before the command is executed.

The command above is conceptually equivalent to passing six different paths to `touch`.

---

## API Configurations

Each API configuration contained four lines.

Example:

```bash
printf "service=api\nenvironment=development\nport=8080\ndebug=true\n" \
  > linux-fundamentals-exam/source-configs/api-development.conf
```

Example content:

```text
service=api
environment=development
port=8080
debug=true
```

Equivalent files were created for:

```text
staging
production
```

Each filename and its internal `environment` value had to be consistent.

---

## Database Configurations

The database files used values appropriate for a database service.

Example:

```bash
printf "service=database\nenvironment=development\nport=5432\nlog_level=info\n" \
  > linux-fundamentals-exam/source-configs/database-development.conf
```

Example content:

```text
service=database
environment=development
port=5432
log_level=info
```

Equivalent configurations were created for staging and production.

---

## Verifying Configuration Contents

The configuration structure was checked using:

```bash
find linux-fundamentals-exam/source-configs
```

The number of lines in every configuration was checked using:

```bash
wc -l linux-fundamentals-exam/source-configs/*.conf
```

Each file contained four lines.

Expected total:

```text
24 total
```

The first two lines of one configuration were verified using:

```bash
head -n 2 linux-fundamentals-exam/source-configs/api-development.conf
```

Expected output:

```text
service=api
environment=development
```

---

## 3. Verifying the Configuration Glob

Before using a glob for a file operation, the matching paths were inspected:

```bash
printf '%s\n' linux-fundamentals-exam/source-configs/*.conf
```

This displayed the six configuration files without modifying them.

This verification is important before operations such as:

```bash
cp
mv
rm
```

A pattern should not be trusted without first checking what it matches.

---

## 4. Creating Backups

Each source configuration was copied to the `backups` directory while adding the `.bak` suffix.

Example:

```bash
cp source-configs/api-development.conf \
   backups/api-development.conf.bak
```

The same operation was performed for all six configurations.

The original files remained in:

```text
source-configs/
```

The backup copies were created in:

```text
backups/
```

The final backup structure was verified with:

```bash
find backups
```

Expected files:

```text
backups/api-development.conf.bak
backups/api-staging.conf.bak
backups/api-production.conf.bak
backups/database-development.conf.bak
backups/database-staging.conf.bak
backups/database-production.conf.bak
```

At this stage, six explicit `cp` commands were clearer and safer than attempting to create source-destination pairs using one complex command.

Brace expansion can generate strings, but it does not automatically pair each source path with its corresponding destination path.

---

## 5. Creating Deployment Logs

The deployment log filenames were generated with nested brace expansion:

```bash
touch logs/deployment-{{1..3},10}.log
```

This generated:

```text
deployment-1.log
deployment-2.log
deployment-3.log
deployment-10.log
```

The sequence:

```text
{1..3}
```

generated the numbers from 1 to 3.

The surrounding brace expression added `10` to the generated list.

---

## Main Deployment Log

The file:

```text
logs/deployment-3.log
```

contained multiple deployment events.

The log included:

- `INFO` events;
- `WARN` events;
- `ERROR` events;
- service recovery;
- a final post-deployment health check.

Example content:

```text
2026-07-13 10:00:00 INFO Application started
2026-07-13 10:00:01 INFO Database connection established
2026-07-13 10:01:12 WARN Response time exceeded threshold
2026-07-13 10:02:33 ERROR Database connection lost
2026-07-13 10:02:33 WARN Try to reconnect to server
2026-07-13 10:03:33 ERROR Impossible to connect to server
2026-07-13 10:03:01 INFO Database connection established
2026-07-13 10:04:12 WARN Response time exceeded threshold
2026-07-13 10:05:33 ERROR Database connection lost
2026-07-13 10:06:33 WARN Reconnection attempt started
2026-07-13 10:07:33 ERROR Impossible to connect to server
2026-07-20 10:09:30 INFO Service restored successfully
2026-07-20 10:30:00 INFO Post-deployment health check successful
```

The final file contained:

```text
13 lines
```

---

## 6. Inspecting the Deployment Log

### Determining the File Type

```bash
file logs/deployment-3.log
```

Output:

```text
logs/deployment-3.log: ASCII text
```

The log level `INFO` is not the file type.

The file type is the result returned by the `file` command.

---

### Counting the Lines

```bash
wc -l logs/deployment-3.log
```

Output:

```text
13 logs/deployment-3.log
```

---

### Inspecting the Beginning

```bash
head -n 2 logs/deployment-3.log
```

Output:

```text
2026-07-13 10:00:00 INFO Application started
2026-07-13 10:00:01 INFO Database connection established
```

---

### Inspecting the End

```bash
tail -n 3 logs/deployment-3.log
```

This displayed the three most recent log events.

---

### Searching with `less`

The log was opened with:

```bash
less logs/deployment-3.log
```

Inside `less`, the following commands were used:

```text
/ERROR
Enter
n
q
```

Meaning:

```text
/ERROR -> search for ERROR
Enter  -> execute the search
n      -> move to the next result
q      -> exit less
```

---

## 7. Following the Log in Real Time

In the first terminal:

```bash
tail -f logs/deployment-3.log
```

In a second terminal, a new event was appended:

```bash
printf "2026-07-20 10:30:00 INFO Post-deployment health check successful\n" \
  >> logs/deployment-3.log
```

The new event appeared immediately in the terminal running `tail -f`.

The process was interrupted using:

```text
Ctrl + C
```

After the new event was added, the line count and final event were recalculated:

```bash
wc -l logs/deployment-3.log
tail -n 1 logs/deployment-3.log
```

The final report used the values obtained after this test.

---

## 8. Creating Environment Documentation

The three README files were generated using brace expansion:

```bash
touch environments/{development,staging,production}/README.md
```

Each file contained environment-specific information.

Example for development:

```text
Environment: development
Purpose: deployment validation
```

Example for staging:

```text
Environment: staging
Purpose: deployment validation
```

Example for production:

```text
Environment: production
Purpose: deployment validation
```

The contents were verified with:

```bash
cat environments/development/README.md
cat environments/staging/README.md
cat environments/production/README.md
```

---

## 9. Working with a Directory Containing Spaces

The report directory was named:

```text
Deployment Report
```

It could be referenced using double quotes:

```bash
cd "Deployment Report"
```

or by escaping the space:

```bash
cd Deployment\ Report
```

A variable was also used:

```bash
REPORT_DIR="$HOME/linux-fundamentals-exam/Deployment Report"
```

The variable was safely used with double quotes:

```bash
cd "$REPORT_DIR"
```

Quoting was necessary because the variable value contained a space.

Without quotes:

```bash
cd $REPORT_DIR
```

the shell could split the value into multiple arguments.

---

## 10. Creating the Final Report

Before writing the report, real values were collected using:

```bash
wc -l logs/deployment-3.log
head -n 1 logs/deployment-3.log
tail -n 1 logs/deployment-3.log
file source-configs/api-development.conf
file logs/deployment-3.log
```

The report contained:

```text
Linux Fundamentals Deployment Report
Configuration files: 6
Backup files: 6
Deployment log lines: 13
First deployment event: 2026-07-13 10:00:00 INFO Application started
Last deployment event: 2026-07-20 10:30:00 INFO Post-deployment health check successful
Configuration type: ASCII text
Log type: ASCII text
```

The report was verified using:

```bash
cat "Deployment Report/summary.txt"
```

The number of lines recorded in the report matched the real output of:

```bash
wc -l logs/deployment-3.log
```

This consistency check was an important part of the exam.

---

## 11. Pattern Verification

### API Configurations

Expected files:

```text
api-development.conf
api-staging.conf
api-production.conf
```

Pattern:

```bash
ls source-configs/api-*.conf
```

---

### Database Configurations

Expected files:

```text
database-development.conf
database-staging.conf
database-production.conf
```

Pattern:

```bash
ls source-configs/database-*.conf
```

---

### Deployment Logs 1 to 3

Expected files:

```text
deployment-1.log
deployment-2.log
deployment-3.log
```

Pattern:

```bash
ls logs/deployment-[1-3].log
```

The character class excludes:

```text
deployment-10.log
```

Using:

```text
?
```

would be less precise because it could also match a file such as:

```text
deployment-a.log
```

---

### All Backup Files

```bash
ls backups/*.conf.bak
```

This matched all six backup files.

---

### Printing a Pattern Literally

```bash
echo 'source-configs/*.conf'
```

Output:

```text
source-configs/*.conf
```

Single quotes prevented pathname expansion.

Compare this with:

```bash
ls source-configs/*.conf
```

where the shell expands the pattern into matching filenames.

---

## 12. Handling a Filename Beginning with `-`

A temporary file was deliberately created:

```bash
touch ./-temporary.conf
```

The explicit `./` made it clear that the value was a path.

The file was displayed safely using:

```bash
ls -l -- -temporary.conf
```

A filename beginning with `-` is problematic because programs may interpret it as an option.

The file was safely removed using:

```bash
rm -- -temporary.conf
```

The `--` separator means:

> Stop interpreting the following arguments as command options.

The file was then verified as removed.

---

## 13. Final Verification

The complete structure was verified with:

```bash
find linux-fundamentals-exam
```

The final report was displayed with:

```bash
cat "linux-fundamentals-exam/Deployment Report/summary.txt"
```

The final log count was verified with:

```bash
wc -l linux-fundamentals-exam/logs/deployment-3.log
```

Output:

```text
13 linux-fundamentals-exam/logs/deployment-3.log
```

This matched the value recorded in the report:

```text
Deployment log lines: 13
```

---

## Errors Encountered

### Incorrect Environment Directory Name

The directory was initially created as:

```text
developments
```

instead of:

```text
development
```

It was corrected using:

```bash
mv environments/developments environments/development
```

This demonstrated the importance of verifying generated directory names immediately.

---

### Reusing API Values in Database Configurations

The first database configurations used values such as:

```text
port=8080
debug=true
```

These values were technically valid text, but they looked like an accidental copy of the API configuration.

They were replaced with more coherent values:

```text
port=5432
log_level=info
```

This demonstrated that syntactically correct configuration is not necessarily operationally meaningful.

---

### Outdated Backups After Configuration Changes

After correcting the database configuration files, the previous backup copies no longer represented the updated originals.

The backup files had to be recreated.

This demonstrated an important rule:

> A backup created before a configuration change does not automatically represent the current configuration.

---

### Confusing File Type with Configuration Data

The following values were initially confused:

```text
development
INFO
```

with file types.

However:

```text
development
```

is an environment name.

```text
INFO
```

is a log level.

The real file type was obtained using:

```bash
file filename
```

and was:

```text
ASCII text
```

---

### Inconsistent Log Timeline

Some log events used different dates.

The commands and structure were valid, but a real incident log should maintain a coherent chronological sequence unless multiple incidents are intentionally combined.

Operational data must be technically valid and logically coherent.

---

## What Went Well

The exam successfully demonstrated:

- correct filesystem navigation;
- use of both absolute and relative paths;
- compact directory creation using brace expansion;
- preservation of original configuration files;
- correct creation of six backup copies;
- safe verification of globs;
- selective glob patterns;
- handling of paths containing spaces;
- safe use of quoted variables;
- inspection of log files;
- searching inside `less`;
- real-time log monitoring with `tail -f`;
- safe handling of filenames beginning with `-`;
- creation of a final report using real data;
- verification that report values matched actual command output.

---

## Areas to Improve

The main improvement areas are:

- verify names immediately after creation;
- avoid careless copy-and-paste between different service configurations;
- maintain coherent timestamps in logs;
- distinguish file metadata from file content;
- verify backups after changing source files;
- treat operational constraints as mandatory;
- show evidence instead of relying on assumptions.

In DevOps work, small inconsistencies can become production incidents.

Correct commands are not enough. Precision and verification are also required.

---

## Final Evaluation

| Area | Result |
|---|---:|
| Structure | Passed |
| Configuration files | Passed |
| Backup files | Passed |
| Log creation and inspection | Passed |
| Globbing and brace expansion | Passed |
| Quoting and variables | Passed |
| Operational safety | Passed |
| Final report | Passed |

Final score:

```text
91/100
```

Final status:

```text
01-linux-fundamentals: PASSED
```

---

## Lessons Learned

The main lesson from this exam is that Linux competence is not only the ability to remember commands.

A reliable workflow consists of:

1. understanding the requested final state;
2. planning the directory and file structure;
3. using expansions only when their result is predictable;
4. checking paths before modifying data;
5. verifying patterns before applying operations;
6. preserving original files;
7. validating generated content;
8. comparing reports with real command output;
9. correcting mistakes without creating additional damage;
10. performing a final structural and content verification.

Useful verification commands include:

```bash
whoami
pwd
find
ls
cat
file
wc
head
tail
printf '%s\n' pattern
```

The module demonstrated that the following tools can be combined in a realistic workflow:

```text
Filesystem navigation
File management
Text inspection
Output redirection
Globbing
Brace expansion
Variables
Quoting
Escaping
Operational verification
```

The Linux fundamentals module is complete.
