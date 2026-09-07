# 🐧 Lab 001 — Linux Orientation

> Getting comfortable inside a Linux system before trying to administer one.

---

## 🎯 Objective

The purpose of this lab is to build a basic mental model of working inside a Linux terminal.

By the end of this lab I should be able to:

* identify the current user;
* identify the machine I'm connected to;
* identify the Linux distribution and kernel;
* understand where I am in the filesystem;
* navigate using absolute and relative paths;
* create, copy, move and inspect files;
* read text files;
* understand basic shell behaviour;
* find documentation without relying entirely on Google.

---

# 🧠 Command Structure

Most Linux commands follow:

```bash
command [options] [arguments]
```

Example:

```bash
ls -la /etc
```

```text
ls       → command
-la      → options
/etc     → target / argument
```

---

# 👤 Identity

## `whoami`

Displays the username of the current user.

```bash
whoami
```

Example:

```text
li88leowl
```

---

## `id`

Displays the user's UID, primary GID and group memberships.

```bash
id
```

Example:

```text
uid=1000(li88leowl) gid=1000(li88leowl) groups=1000(li88leowl),27(sudo)
```

```text
uid     → user identifier
gid     → primary group identifier
groups  → groups the user belongs to
```

---

# 📍 Current Location

## `pwd`

**Print Working Directory**

Shows the directory I am currently inside.

```bash
pwd
```

Example:

```text
/home/li88leowl
```

---

# 🖥️ Machine Identity

## `hostname`

Shows the hostname assigned to the machine.

```bash
hostname
```

Example:

```text
linux-lab-01
```

Important:

```text
whoami   → user

hostname → machine
```

---

# 🐧 Linux Distribution

## `/etc/os-release`

Contains information about the installed Linux distribution.

```bash
cat /etc/os-release
```

Possible information includes:

```text
NAME
VERSION
VERSION_ID
ID
```

Example:

```text
NAME="Ubuntu"
VERSION="24.04 LTS"
```

---

# ⚙️ Kernel Information

## `uname`

Displays system/kernel information.

```bash
uname
```

---

## Kernel release

```bash
uname -r
```

---

## Detailed system information

```bash
uname -a
```

The output can include:

```text
kernel
hostname
kernel release
architecture
operating-system family
```

---

# 📂 Listing Files

## `ls`

Lists directory contents.

```bash
ls
```

---

## `ls -l`

Long listing format.

```bash
ls -l
```

Shows information such as:

```text
permissions
owner
group
size
timestamp
filename
```

---

## `ls -a`

Shows all files, including hidden files.

```bash
ls -a
```

Linux hidden files generally begin with:

```text
.
```

Example:

```text
.bashrc
.profile
.gitconfig
```

---

## `ls -la`

Combines:

```text
-l → detailed listing

-a → include hidden files
```

```bash
ls -la
```

---

# 🧭 Navigation

## `cd`

Changes directory.

```bash
cd projects
```

---

## Home directory

```bash
cd ~
```

or simply:

```bash
cd
```

---

## Filesystem root

```bash
cd /
```

---

## Parent directory

```bash
cd ..
```

---

## Two levels upward

```bash
cd ../..
```

---

## Previous directory

```bash
cd -
```

---

# 🛣️ Absolute vs Relative Paths

## Absolute Path

Starts from:

```text
/
```

Example:

```bash
cd /home/li88leowl/projects
```

It represents the complete path from the filesystem root.

---

## Relative Path

Starts from the current working directory.

Example:

```bash
cd projects
```

If:

```bash
pwd
```

returns:

```text
/home/li88leowl
```

then:

```bash
cd projects
```

means:

```text
/home/li88leowl/projects
```

---

# 🧭 Special Path Symbols

```text
/     filesystem root

~     current user's home directory

.     current directory

..    parent directory

-     previous directory when used with cd
```

Examples:

```bash
cd ~
cd ..
cd /
cd -
ls .
```

---

# 📁 Creating Directories

## `mkdir`

Creates a directory.

```bash
mkdir projects
```

---

## Nested directories

```bash
mkdir -p labs/linux/day1
```

The:

```text
-p
```

option allows required parent directories to be created as well.

---

# 📄 Creating Files

## `touch`

Creates an empty file when it does not already exist.

```bash
touch notes.txt
```

If the file already exists, `touch` updates its timestamps rather than deleting its contents.

---

# 📋 Copying

## `cp`

Copies files.

```bash
cp notes.txt backup.txt
```

---

## Copying directories

```bash
cp -r project/ project-backup/
```

The:

```text
-r
```

means recursively copy the directory and its contents.

---

# 🚚 Moving & Renaming

## `mv`

Can move files:

```bash
mv notes.txt documents/
```

It can also rename files:

```bash
mv notes.txt linux-notes.txt
```

Mental model:

```text
mv = move OR rename
```

---

# 🗑️ Removing Files

## `rm`

Removes files.

```bash
rm notes.txt
```

---

## Removing directories recursively

```bash
rm -r old-directory/
```

`rm` should be treated carefully.

Terminal deletion may not behave like a normal desktop recycle bin.

Commands involving:

```bash
sudo rm -rf
```

should never be run casually without understanding exactly what path they target.

---

# 🌳 Viewing Directory Structure

## `tree`

Shows files and directories as a hierarchy.

```bash
tree
```

Example:

```text
.
├── docs
│   └── notes.txt
├── scripts
│   └── backup.sh
└── README.md
```

`tree` may need to be installed depending on the Linux distribution.

Ubuntu:

```bash
sudo apt install tree
```

---

# 📖 Reading Files

## `cat`

Displays file contents.

```bash
cat notes.txt
```

Best suited to relatively small text files.

---

## `less`

Interactive file viewer.

```bash
less /etc/services
```

Useful controls:

```text
↑ / ↓      move
Space      next page
b          previous page
g          beginning
G          end
/word      search
n          next match
q          quit
```

---

## `head`

Displays the beginning of a file.

```bash
head notes.txt
```

Specific number of lines:

```bash
head -n 5 notes.txt
```

---

## `tail`

Displays the end of a file.

```bash
tail notes.txt
```

Specific number:

```bash
tail -n 20 notes.txt
```

---

## Follow a changing file

```bash
tail -f application.log
```

Useful for monitoring logs as new entries are written.

Stop using:

```text
Ctrl + C
```

---

# ✍🏽 Writing Simple Content

## `echo`

Print text:

```bash
echo "hello"
```

Write to a file:

```bash
echo "Linux Orientation" > notes.txt
```

---

## `>`

Redirects output and **replaces existing file contents**.

```bash
echo "hello" > notes.txt
```

Use carefully.

---

## `>>`

Appends output to the end of a file.

```bash
echo "Learning Linux administration" >> notes.txt
```

Mental model:

```text
>     overwrite

>>    append
```

---

# 🆘 Getting Help

## `--help`

Many commands expose basic documentation through:

```bash
ls --help
```

---

## `man`

Displays a command's manual page.

```bash
man ls
```

Examples:

```bash
man cp
man rm
man uname
```

Useful controls:

```text
Space     next page
b         previous page
/word     search
n         next search result
q         quit
```

---

# 🔎 Finding Commands

## `which`

Shows the executable resolved through the current shell PATH.

```bash
which python3
```

Example:

```text
/usr/bin/python3
```

---

## `type`

Shows how the shell interprets a command.

```bash
type cd
```

Possible result:

```text
cd is a shell builtin
```

---

# 🕘 Command History

## `history`

Displays previously executed shell commands.

```bash
history
```

Previous commands can also be accessed using:

```text
↑
↓
```

---

# 🧹 Clearing the Terminal

```bash
clear
```

Keyboard shortcut:

```text
Ctrl + L
```

This clears the visible terminal but does not erase command history.

---

# ⌨️ Useful Keyboard Controls

```text
TAB        autocomplete paths and commands

↑ / ↓      previous/next history entries

Ctrl + C   interrupt a running foreground command

Ctrl + L   clear terminal display

Ctrl + D   EOF / may close the current shell
```

---

# 🧪 Identifying File Types

## `file`

Inspects the actual type of a file.

```bash
file notes.txt
```

Possible output:

```text
ASCII text
```

Example:

```bash
file /bin/ls
```

Linux does not depend entirely on filename extensions to determine what a file actually is.

---

# 🔠 Case Sensitivity

Linux filenames are case-sensitive.

These can be three separate files:

```text
file.txt
File.txt
FILE.txt
```

---

# ␠ Filenames With Spaces

Use quotes:

```bash
cat "linux notes.txt"
```

or escape the space:

```bash
cat linux\ notes.txt
```

For server environments and scripts, names without spaces are often easier to work with.

---

# 🧠 Orientation Mental Model

```text
whoami
    ↓
WHO AM I?

id
    ↓
WHAT USER/GROUPS?

pwd
    ↓
WHERE AM I?

hostname
    ↓
WHICH MACHINE?

cat /etc/os-release
    ↓
WHICH DISTRIBUTION?

uname
    ↓
WHICH KERNEL/SYSTEM?

ls
    ↓
WHAT IS HERE?

cd
    ↓
HOW DO I MOVE?

mkdir / touch
    ↓
HOW DO I CREATE THINGS?

cp / mv
    ↓
HOW DO I COPY OR MOVE THINGS?

rm
    ↓
HOW DO I REMOVE THINGS?

cat / less / head / tail
    ↓
HOW DO I READ THINGS?

man / --help
    ↓
HOW DO I LEARN A COMMAND?
```

---

# 🧪 Lab 001 Practical

Build:

```text
linux-orientation-lab/
├── docs/
│   └── commands.txt
├── notes/
│   └── day1.txt
└── backup/
    └── day1-backup.txt
```

Tasks:

```text
01. Identify the current user.
02. Identify the current working directory.
03. Identify the hostname.
04. Identify the Linux distribution.
05. Identify the kernel release.
06. Navigate to the home directory.
07. Create linux-orientation-lab.
08. Enter the lab.
09. Create docs, notes and backup directories.
10. Create notes/day1.txt.
11. Add "Linux Orientation 01".
12. Append "Learning Linux administration."
13. Display the file.
14. Copy it into backup/day1-backup.txt.
15. Create docs/notes.txt.
16. Rename it to docs/commands.txt.
17. Inspect the lab directory.
18. Navigate using a relative path.
19. Navigate upward using ..
20. Navigate to /etc using an absolute path.
21. Return using cd -.
22. Inspect the first lines of /etc/passwd.
23. Inspect the final lines of /etc/passwd.
24. Open /etc/services using less.
25. Read the ls manual.
26. Locate python3.
27. Determine what type of command cd is.
28. Review command history.
```

---

# ✅ Completion Criteria

Lab 001 is complete when I can explain, without simply memorizing:

```text
user vs machine

distribution vs kernel

absolute vs relative path

/ vs /root

~ vs /

. vs ..

copy vs move

cat vs less

head vs tail

> vs >>

which vs type
```

---

## 📡 Status

```bash
li88leowl@linux:~$ ./status

module      : Linux Orientation
lab         : 001
status      : learning
next        : Linux Filesystem & Structure
destination : Cloud Infrastructure + Security Engineering
```

---

> The goal isn't to memorize the terminal.
> The goal is to stop being lost inside it.
