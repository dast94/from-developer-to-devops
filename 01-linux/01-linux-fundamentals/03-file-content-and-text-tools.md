# Linux File Content and Text Inspection

> **Note:** In this guide, `linuxUser` is used as an example username.  
> Replace it with your actual Linux username when running the commands.

## What You Will Learn

In this guide, you will learn how to read and inspect text files directly from the terminal without using a graphical editor.

The goal is not to memorise every command.

The goal is to understand which tool should be used depending on the situation.

The main commands are:

```text
cat
less
head
tail
wc
file
```

This guide also introduces:

```text
echo
printf
```

These commands can be used to quickly create text content from the terminal.

---

## Why This Is Important

A DevOps Engineer frequently works with:

- configuration files;
- application logs;
- system logs;
- environment files;
- scripts;
- deployment reports;
- generated pipeline output;
- service status files.

For example, imagine connecting to a server and finding this file:

```text
/var/log/my-application/application.log
```

The file may contain thousands or even millions of lines.

Using the wrong command can:

- fill the terminal with too much output;
- hide the important part of the file;
- slow down troubleshooting;
- make errors harder to find;
- waste time;
- accidentally modify a file that should only be inspected.

A DevOps Engineer should be able to answer questions such as:

- What type of file is this?
- How many lines does it contain?
- How does it begin?
- How does it end?
- How can I read it one screen at a time?
- Is the file empty?
- Did the problem appear only at the end of the log?
- Is the file currently being updated by another process?

---

## Reading Small Files with `cat`

### `cat`

`cat` means:

```text
concatenate
```

It displays the content of one or more files on the standard output, normally the terminal.

```bash
cat filename.txt
```

Example:

```bash
cat /etc/hostname
```

This displays the content of the system hostname configuration file.

---

### Reading Multiple Files

`cat` can display multiple files one after another:

```bash
cat file1.txt file2.txt
```

The content of `file1.txt` is displayed first, followed by the content of `file2.txt`.

---

### When to Use `cat`

`cat` is useful for:

- small text files;
- short configuration files;
- quick content inspection;
- displaying multiple files;
- checking whether a file contains the expected text.

Example:

```bash
cat config/application.conf
```

---

### When Not to Use `cat`

`cat` is not ideal for very large files.

Example:

```bash
cat huge-application.log
```

This may print thousands of lines to the terminal, making the output difficult to read.

For long files, use tools such as:

```bash
less
tail
head
```

---

## Reading Large Files with `less`

### `less`

`less` allows a file to be read one screen at a time.

```bash
less filename.txt
```

Example:

```bash
less application.log
```

This is useful for long files because the entire file is not printed at once in the terminal.

---

### Navigation Inside `less`

While `less` is open, the following keys can be used:

```text
Space        next page
b            previous page
Down Arrow   next line
Up Arrow     previous line
/word        search for a word
n            next search result
N            previous search result
g            beginning of the file
G            end of the file
q            quit
```

Example workflow:

```bash
less logs/application.log
```

Then, inside `less`:

```text
/ERROR
```

Press `Enter` to search.

Use:

```text
n
```

to move to the next occurrence.

Use:

```text
q
```

to exit.

`less` is normally used only to read a file and does not modify its content.

---

## Inspecting the Beginning with `head`

### `head`

`head` displays the beginning of a file.

```bash
head filename.txt
```

By default, it normally displays the first ten lines.

Example:

```bash
head application.log
```

---

### Displaying a Specific Number of Lines

```bash
head -n 5 filename.txt
```

This displays the first five lines.

The shorter form also works:

```bash
head -5 filename.txt
```

The `-n` form is more explicit and easier to read.

Example:

```bash
head -n 3 logs/application.log
```

---

## Inspecting the End with `tail`

### `tail`

`tail` displays the end of a file.

```bash
tail filename.txt
```

By default, it normally displays the last ten lines.

Example:

```bash
tail application.log
```

---

### Displaying a Specific Number of Lines

```bash
tail -n 5 filename.txt
```

This displays the last five lines.

The shorter form also works:

```bash
tail -5 filename.txt
```

Example:

```bash
tail -n 4 logs/application.log
```

---

## Following Logs in Real Time

### `tail -f`

The `-f` option means:

```text
follow
```

It keeps the command active and displays new lines as they are appended to the file.

```bash
tail -f application.log
```

This is especially useful when:

- an application is currently running;
- a service is writing logs;
- a deployment is in progress;
- an error must be observed in real time;
- another process is updating the file.

Example:

```bash
tail -f ~/linux-text-lab/logs/application.log
```

In another terminal, a new line can be appended:

```bash
echo "2026-07-13 10:10:00 INFO Health check completed" \
  >> ~/linux-text-lab/logs/application.log
```

The new line immediately appears in the terminal running `tail -f`.

To stop the command:

```text
Ctrl + C
```

This interrupts the foreground process.

---

## Counting Lines and Words with `wc`

### `wc`

`wc` means:

```text
word count
```

It can count:

- lines;
- words;
- bytes;
- characters.

Example:

```bash
wc filename.txt
```

Possible output:

```text
12 48 301 filename.txt
```

The values normally represent:

```text
lines words bytes filename
```

---

### Counting Lines

```bash
wc -l filename.txt
```

Example:

```bash
wc -l logs/application.log
```

---

### Counting Words

```bash
wc -w filename.txt
```

Example:

```bash
wc -w config/application.conf
```

---

### Counting Bytes

```bash
wc -c filename.txt
```

This counts bytes.

Bytes and characters are not always the same, especially when Unicode characters are used.

---

### Counting Characters

```bash
wc -m filename.txt
```

This counts characters.

---

## Detecting File Types with `file`

### `file`

The `file` command attempts to identify the actual type of a file.

```bash
file filename
```

Example:

```bash
file deploy.sh
```

Possible output:

```text
deploy.sh: ASCII text
```

or:

```text
deploy.sh: Bourne-Again shell script, ASCII text executable
```

Linux does not depend only on the file extension.

A file named:

```text
configuration.jpg
```

may contain plain text if it has only been renamed.

The `file` command analyses the content and other file characteristics.

Possible outputs include:

```text
ASCII text
Unicode text, UTF-8 text
empty
Bourne-Again shell script
PNG image data
ELF executable
```

The result is not necessarily the word `TXT`.

---

## Writing Text with `echo`

### `echo`

`echo` prints text to the terminal.

```bash
echo "Hello Linux"
```

Output:

```text
Hello Linux
```

---

### Writing Text into a File

```bash
echo "application=demo" > app.conf
```

This writes the text into `app.conf`.

If the file does not exist, it is created.

If the file already exists, its previous content is replaced.

---

### Appending Text

```bash
echo "environment=development" >> app.conf
```

This adds the new line at the end of the existing file.

If the file does not exist, it is created.

---

## Writing Formatted Text with `printf`

### `printf`

`printf` provides more precise control over text formatting than `echo`.

```bash
printf "first line\nsecond line\n"
```

The sequence:

```text
\n
```

represents a newline.

Output:

```text
first line
second line
```

---

### Writing Multiple Lines into a File

```bash
printf "first line\nsecond line\n" > notes.txt
```

Example configuration:

```bash
printf "application=devops-demo\n\
environment=development\n\
port=8080\n\
debug=true\n" > config/application.conf
```

The resulting file contains:

```text
application=devops-demo
environment=development
port=8080
debug=true
```

`printf` is often preferred in scripts because its behaviour is more predictable across different environments.

---

## Output Redirection

### `>`

The `>` operator redirects output into a file.

```bash
echo "new content" > filename.txt
```

Behaviour:

- creates the file if it does not exist;
- replaces the content if the file already exists.

Example:

```bash
echo "application=demo" > app.conf
```

This can cause data loss if `app.conf` already contains important configuration.

---

### `>>`

The `>>` operator appends output to the end of a file.

```bash
echo "new content" >> filename.txt
```

Behaviour:

- creates the file if it does not exist;
- adds content at the end if the file already exists.

Example:

```bash
echo "log_level=info" >> app.conf
```

---

### Important Difference

```text
>   creates or replaces
>>  creates or appends
```

This single-character difference is extremely important.

A wrong redirection operator can overwrite a configuration file or destroy important data.

---

## Practical Example

Create a laboratory structure:

```bash
cd ~

mkdir linux-text-lab
cd linux-text-lab

mkdir config logs documentation
```

Create a configuration file:

```bash
printf "application=devops-demo\n\
environment=development\n\
port=8080\n\
debug=true\n" > config/application.conf
```

Read the file:

```bash
cat config/application.conf
```

Append a new configuration value:

```bash
printf "log_level=info\n" >> config/application.conf
```

Verify that the previous content still exists:

```bash
cat config/application.conf
```

Display the first two lines:

```bash
head -n 2 config/application.conf
```

Display the last two lines:

```bash
tail -n 2 config/application.conf
```

Count the lines:

```bash
wc -l config/application.conf
```

Count the words:

```bash
wc -w config/application.conf
```

Determine the file type:

```bash
file config/application.conf
```

---

## Creating and Inspecting a Log File

Create a log file with multiple events:

```bash
printf "2026-07-13 10:00:00 INFO Application started\n\
2026-07-13 10:00:01 INFO Database connection established\n\
2026-07-13 10:01:12 WARN Response time exceeded threshold\n\
2026-07-13 10:02:33 ERROR Database connection lost\n\
2026-07-13 10:02:40 WARN Reconnection attempt started\n\
2026-07-13 10:02:45 INFO Reconnection attempt 1\n\
2026-07-13 10:02:50 INFO Reconnection attempt 2\n\
2026-07-13 10:02:55 INFO Reconnection attempt 3\n\
2026-07-13 10:03:00 ERROR Connection failed\n\
2026-07-13 10:03:10 WARN Waiting before retry\n\
2026-07-13 10:03:20 INFO New connection attempt\n\
2026-07-13 10:03:30 INFO Database connection restored\n" > logs/application.log
```

Display the first three lines:

```bash
head -n 3 logs/application.log
```

Display the last four lines:

```bash
tail -n 4 logs/application.log
```

Open the file with `less`:

```bash
less logs/application.log
```

Search for errors:

```text
/ERROR
```

Move to the next result:

```text
n
```

Exit:

```text
q
```

---

## Safe Workflow

Before inspecting or modifying a file, verify the context.

```bash
pwd
ls
```

Before using `>` on an important file, inspect the current content:

```bash
cat important.conf
```

or:

```bash
less important.conf
```

Before writing, ask:

- Does the file already exist?
- Do I want to replace the content?
- Do I want to append a new line?
- Am I using the correct path?
- Is the file currently used by a service?
- Could an incorrect change break the application?

Use `>` only when replacement is intentional.

Use `>>` when existing content must be preserved.

---

## Common Mistakes

### Using `cat` on Very Large Files

```bash
cat huge-file.log
```

This does not normally damage the file, but it can produce an unmanageable amount of terminal output.

Better options:

```bash
less huge-file.log
```

```bash
tail huge-file.log
```

---

### Thinking `head` Shows Only One Line

By default:

```bash
head filename.txt
```

normally shows the first ten lines.

To show only one line:

```bash
head -n 1 filename.txt
```

---

### Thinking `tail` Shows Only One Line

By default:

```bash
tail filename.txt
```

normally shows the last ten lines.

To show only one line:

```bash
tail -n 1 filename.txt
```

---

### Forgetting the Space Before an Option

Incorrect:

```bash
tail-n 4 application.log
```

Correct:

```bash
tail -n 4 application.log
```

The shell interprets `tail-n` as a different command name.

---

### Confusing `>` and `>>`

This command replaces the content:

```bash
echo "test" > important.conf
```

This command appends the content:

```bash
echo "test" >> important.conf
```

Using `>` accidentally can destroy the previous file content.

---

### Forgetting the Correct Path

If the file is located at:

```text
config/application.conf
```

this command may fail:

```bash
wc -l application.conf
```

The correct command is:

```bash
wc -l config/application.conf
```

unless the current directory is already `config/`.

---

### Missing `\n` in `printf`

This:

```bash
printf "first linesecond line"
```

creates one continuous line.

This:

```bash
printf "first line\nsecond line\n"
```

creates two separate lines.

---

### Using a String Without `echo` or `printf`

Incorrect:

```bash
"new log line" >> application.log
```

The shell tries to execute the string as a command.

Correct:

```bash
echo "new log line" >> application.log
```

or:

```bash
printf "new log line\n" >> application.log
```

---

### Forgetting to Exit `less`

Use:

```text
q
```

to exit normally.

---

### Leaving `tail -f` Running

`tail -f` remains active until interrupted.

Use:

```text
Ctrl + C
```

---

### Trusting the File Extension

A file ending in `.txt` is not guaranteed to contain text.

A file without an extension may still be a text file.

Use:

```bash
file filename
```

---

## Challenge

The challenge was to create this structure:

```text
incident-analysis/
├── config/
│   └── service.conf
├── logs/
│   └── service.log
└── report/
    └── summary.txt
```

---

### Configuration File

The file:

```text
config/service.conf
```

contains:

```text
service=payment-api
environment=production
port=9000
log_level=warning
```

It can be created with:

```bash
printf "service=payment-api\n\
environment=production\n\
port=9000\n\
log_level=warning\n" > config/service.conf
```

---

### Log File

The file:

```text
logs/service.log
```

contains at least 15 lines, including:

- at least five `INFO` events;
- at least three `WARN` events;
- at least two `ERROR` events;
- a final line indicating service recovery.

The initial content can be written using:

```bash
printf "first event\nsecond event\n" > logs/service.log
```

Additional events can be appended using:

```bash
printf "another event\n" >> logs/service.log
```

---

### Incident Information

Count the total log lines:

```bash
wc -l logs/service.log
```

Display the first event:

```bash
head -n 1 logs/service.log
```

Display the last event:

```bash
tail -n 1 logs/service.log
```

---

### Incident Report

The report file:

```text
report/summary.txt
```

contains real values obtained from the log.

Example:

```text
Incident report
Total log lines: 15
First event: 2026-07-13 10:00:00 INFO Application started
Last event: 2026-07-13 10:10:00 INFO Service restored
```

The report can be written using `printf`:

```bash
printf "Incident report\n\
Total log lines: 15\n\
First event: 2026-07-13 10:00:00 INFO Application started\n\
Last event: 2026-07-13 10:10:00 INFO Service restored\n" \
> report/summary.txt
```

At this stage, the values are manually inserted after being retrieved with `wc`, `head` and `tail`.

Later, shell scripting will allow these values to be captured automatically.

---

### Inspecting the Challenge Files

Display the report:

```bash
cat report/summary.txt
```

Inspect each file type:

```bash
file config/service.conf
file logs/service.log
file report/summary.txt
```

Possible output may include:

```text
ASCII text
Unicode text, UTF-8 text
```

The exact output depends on the file content and encoding.

---

### Following the Incident Log

In the first terminal:

```bash
tail -f logs/service.log
```

In a second terminal:

```bash
echo "2026-07-13 10:11:00 INFO Service health check successful" \
  >> logs/service.log
```

The new line appears immediately in the first terminal.

Stop `tail -f` with:

```text
Ctrl + C
```

---

### Verifying the Structure

From the parent directory:

```bash
find incident-analysis
```

Expected structure:

```text
incident-analysis
incident-analysis/config
incident-analysis/config/service.conf
incident-analysis/logs
incident-analysis/logs/service.log
incident-analysis/report
incident-analysis/report/summary.txt
```

---

## Lessons Learned

The most important lesson is choosing the correct tool for the situation.

```text
cat     -> display the complete content of small files
less    -> read long files one screen at a time
head    -> inspect the beginning of a file
tail    -> inspect the end of a file
tail -f -> follow new lines in real time
wc      -> count lines, words, bytes or characters
file    -> identify the actual file type
echo    -> print or write simple text
printf  -> write formatted and predictable text
```

The most important redirection rule is:

```text
>   create or replace
>>  create or append
```

Before using `>` on an existing file, verify that overwriting is intentional.

For long log files, use:

```bash
less application.log
```

For recent events, use:

```bash
tail application.log
```

For real-time monitoring, use:

```bash
tail -f application.log
```

A DevOps Engineer should not only know commands.

A DevOps Engineer must understand:

- what the command reads;
- what the command writes;
- whether data can be overwritten;
- which part of the file is relevant;
- whether the file is changing in real time.
