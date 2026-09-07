# 🐧 Lab 001 — Linux Orientation

> Getting comfortable inside a Linux system before trying to administer one.

---

## 🎯 Objective

This lab is about building a basic mental model of working inside a Linux terminal.

The goal isn't to memorize a bunch of commands and hope they stick.

I want to understand:

- who I am on the system;
- which machine I'm working on;
- where I currently am;
- what Linux system I'm working with;
- how to move around the filesystem;
- how to create and manage files and directories;
- how to read files from the terminal;
- how Linux interprets paths and filenames;
- how to find help when I forget a command.

By the end of this lab, I should be comfortable enough inside a terminal that I’m not just firing commands blindly.

---

# 🧪 Lab Environment

```text
Platform      : Virtual Machine
OS Family     : Debian-based Linux
Shell         : Bash
Purpose       : Linux Administration Lab
```

> System-specific identifiers, network information, local usernames and other environment details are intentionally omitted from this public repository.

---

# 🧠 How Linux Commands Are Structured

Most Linux commands roughly follow:

```bash
command [options] [arguments]
```

Example:

```bash
ls -la /etc
```

Breakdown:

```text
ls       → command
-la      → options / flags
/etc     → argument / target
```

A simple mental model:

```text
WHAT DO I WANT TO DO?
        ↓
     COMMAND

HOW DO I WANT IT DONE?
        ↓
     OPTIONS

WHAT AM I DOING IT TO?
        ↓
     ARGUMENT
```

Another example:

```bash
cp -r source/ backup/
```

```text
cp        → command
-r        → option
source/   → source argument
backup/   → destination argument
```

---

# 👤 User Identity

## `whoami`

Shows the username of the current user.

```bash
whoami
```

Example:

```text
labuser
```

Mental model:

```text
whoami
   ↓
WHO AM I?
```

---

## `id`

Displays information about the current user, including:

- UID;
- primary GID;
- group memberships.

```bash
id
```

Example:

```text
uid=1000(labuser) gid=1000(labuser) groups=1000(labuser),27(sudo)
```

Meaning:

```text
uid      → User ID

gid      → Primary Group ID

groups   → Groups this user belongs to
```

The `sudo` group becomes important later when working with elevated privileges.

---

# 📍 Current Location

## `pwd`

`pwd` means:

```text
Print Working Directory
```

It shows exactly where I currently am inside the filesystem.

```bash
pwd
```

Example:

```text
/home/<user>
```

Mental model:

```text
pwd
 ↓
WHERE AM I?
```

---

# 🖥️ Machine Identity

## `hostname`

Displays the hostname assigned to the current machine.

```bash
hostname
```

Example:

```text
linux-lab
```

Important distinction:

```text
whoami
   ↓
CURRENT USER


hostname
   ↓
CURRENT MACHINE
```

A hostname is **not** the username of the person logged in.

---

# 🐧 Linux Distribution Information

Linux distributions may include systems such as:

```text
Debian
Ubuntu
Fedora
Rocky Linux
Arch Linux
Parrot
```

They are different Linux distributions, but they use the **Linux kernel** underneath.

---

## `/etc/os-release`

The `/etc/os-release` file contains information about the installed Linux distribution.

```bash
cat /etc/os-release
```

It may contain fields such as:

```text
NAME
VERSION
VERSION_ID
ID
PRETTY_NAME
```

Example:

```text
NAME="Example Linux"
VERSION="1.0"
```

Mental model:

```text
cat /etc/os-release
        ↓
WHICH LINUX DISTRIBUTION?
```

---

# ⚙️ Kernel Information

The **distribution** and the **kernel** are not the same thing.

For example:

```text
Distribution
     ↓
Debian / Ubuntu / Fedora / etc.

Kernel
     ↓
Linux
```

---

## `uname`

Displays basic system/kernel information.

```bash
uname
```

---

## `uname -r`

Displays the currently running kernel release.

```bash
uname -r
```

Example:

```text
6.x.x-amd64
```

---

## `uname -a`

Displays more detailed system information.

```bash
uname -a
```

The output may include:

```text
kernel name
hostname
kernel release
kernel version
architecture
operating-system family
```

Mental model:

```text
uname -r
    ↓
WHICH KERNEL RELEASE?


uname -a
    ↓
MORE SYSTEM INFORMATION
```

---

# 📂 Listing Files & Directories

## `ls`

Lists the contents of the current directory.

```bash
ls
```

Example:

```text
Documents
Downloads
Projects
notes.txt
```

Mental model:

```text
ls
 ↓
WHAT IS HERE?
```

---

# `ls -l`

The:

```text
-l
```

option means:

```text
long listing format
```

Run:

```bash
ls -l
```

Instead of only displaying filenames, Linux shows more information.

Example:

```text
-rw-r--r-- 1 labuser labuser 1240 Sep 7 20:14 notes.txt
```

This contains things such as:

```text
permissions
owner
group
size
timestamp
filename
```

We will cover the permission string:

```text
-rw-r--r--
```

properly in the permissions module.

---

# `ls -a`

The:

```text
-a
```

means:

```text
all
```

It includes hidden files.

```bash
ls -a
```

Linux hidden files normally begin with:

```text
.
```

Examples:

```text
.bashrc
.profile
.gitconfig
.ssh
```

A normal:

```bash
ls
```

may not show these.

But:

```bash
ls -a
```

will.

---

# `ls -la`

Combines:

```text
-l → long / detailed listing

-a → all files, including hidden files
```

```bash
ls -la
```

---

# `ls -h`

The:

```text
-h
```

means:

```text
human-readable
```

It makes file sizes easier to read.

Instead of seeing something like:

```text
104857600
```

you may see:

```text
100M
```

---

# `ls -lah`

One of the most useful forms of `ls`:

```bash
ls -lah
```

Breakdown:

```text
-l → long listing

-a → include hidden files

-h → human-readable sizes
```

So:

```bash
ls -lah
```

means roughly:

> Show me everything in this directory, including hidden files, with detailed information and readable file sizes.

---

## Short options can be combined

These commands are effectively equivalent here:

```bash
ls -l -a -h
```

```bash
ls -lah
```

```bash
ls -alh
```

The short options:

```text
-l
-a
-h
```

can be combined after one `-`.

So:

```text
-l -a -h
```

becomes:

```text
-lah
```

---

# `.` and `..` inside `ls -la`

When running:

```bash
ls -la
```

I may see:

```text
.
..
```

These are not random files.

```text
.     → current directory

..    → parent directory
```

This is also why:

```bash
cd ..
```

works.

---

# 🧭 Navigating the Filesystem

## `cd`

`cd` means:

```text
Change Directory
```

Example:

```bash
cd projects
```

This moves into the `projects` directory relative to the current location.

---

# Home Directory

The:

```text
~
```

symbol represents the current user's home directory.

```bash
cd ~
```

For a normal user, that may represent something like:

```text
/home/<user>
```

Running:

```bash
cd
```

with no argument normally takes me home as well.

---

# Filesystem Root

The:

```text
/
```

represents the top of the Linux filesystem.

```bash
cd /
```

Then:

```bash
pwd
```

returns:

```text
/
```

---

## `/` is NOT `/root`

This distinction is important.

```text
/
```

means:

```text
filesystem root
```

While:

```text
/root
```

normally means:

```text
root user's home directory
```

They are completely different things.

---

# Parent Directory

Two dots:

```text
..
```

represent the parent directory.

Example:

```bash
cd ..
```

Suppose I am here:

```text
/home/<user>/projects/linux
```

After:

```bash
cd ..
```

I would be here:

```text
/home/<user>/projects
```

---

# Moving Up Multiple Levels

```bash
cd ../..
```

means:

```text
go up two directory levels
```

---

# Current Directory

A single:

```text
.
```

means:

```text
current directory
```

Example:

```bash
ls .
```

means:

> List the contents of the directory I am currently inside.

Later, I may also see commands like:

```bash
./script.sh
```

which means:

> Execute `script.sh` from the current directory.

---

# Previous Directory

```bash
cd -
```

returns to the previous working directory.

Example:

```bash
cd /etc
cd /var/log
cd -
```

The final command returns to:

```text
/etc
```

Running:

```bash
cd -
```

again would return to:

```text
/var/log
```

---

# 🛣️ Absolute vs Relative Paths

This is one of the most important concepts in Linux navigation.

---

## Absolute Path

An absolute path starts from:

```text
/
```

Example:

```bash
cd /home/<user>/projects
```

This represents the complete route from the filesystem root.

It does not matter where I currently am.

---

## Relative Path

A relative path starts from the current working directory.

Suppose:

```bash
pwd
```

returns:

```text
/home/<user>
```

and I run:

```bash
cd projects
```

Linux interprets this as:

```text
/home/<user>/projects
```

because `projects` was relative to my current location.

---

# Special Path Symbols

```text
/      filesystem root

~      current user's home directory

.      current directory

..     parent directory

cd -   previous working directory
```

Examples:

```bash
cd /
cd ~
cd ..
cd ../..
cd -
ls .
```

---

# 🔠 Linux Is Case-Sensitive

Linux filenames and directory names are case-sensitive.

These can all exist as separate files:

```text
notes.txt
Notes.txt
NOTES.txt
```

Likewise:

```text
docs/
Docs/
DOCS/
```

can represent different directories.

So:

```bash
cd Docs
```

is not necessarily the same as:

```bash
cd docs
```

If Linux says:

```text
No such file or directory
```

one of the first things worth checking is capitalization.

---

# ␠ Filenames With Spaces

Suppose a file is named:

```text
linux notes.txt
```

Running:

```bash
cat linux notes.txt
```

does not necessarily work.

The shell sees:

```text
linux
```

and:

```text
notes.txt
```

as two separate arguments.

---

## Double Quotes

```bash
cat "linux notes.txt"
```

This tells the shell:

> Treat everything between these quotes as one argument.

---

## Single Quotes

This also works:

```bash
cat 'linux notes.txt'
```

---

## Escaping the Space

The space can also be escaped:

```bash
cat linux\ notes.txt
```

So these all reference the same filename:

```bash
cat "linux notes.txt"
```

```bash
cat 'linux notes.txt'
```

```bash
cat linux\ notes.txt
```

---

# Single vs Double Quotes

There is an important difference.

Double quotes allow variable expansion.

Example:

```bash
echo "$HOME"
```

Might output:

```text
/home/<user>
```

But:

```bash
echo '$HOME'
```

prints literally:

```text
$HOME
```

This becomes much more important later when working with Bash.

For now:

```text
" "     → protects spaces but still allows expansion

' '     → treats contents more literally
```

---

# 📁 Creating Directories

## `mkdir`

Creates a directory.

```bash
mkdir projects
```

---

## Creating Nested Directories

```bash
mkdir -p labs/linux/day1
```

The:

```text
-p
```

option allows Linux to create any missing parent directories along the way.

---

# 📄 Creating Files

## `touch`

Creates an empty file if it does not already exist.

```bash
touch notes.txt
```

If the file already exists, `touch` does **not** wipe the file.

Instead, it updates its timestamp.

---

# 📋 Copying Files

## `cp`

`cp` means:

```text
copy
```

Example:

```bash
cp notes.txt backup.txt
```

After this:

```text
notes.txt
backup.txt
```

both exist.

Mental model:

```text
cp
│
├── original stays
│
└── copy appears
```

---

# Copying Directories

Directories normally need recursive copying.

```bash
cp -r project/ project-backup/
```

The:

```text
-r
```

means:

```text
recursive
```

Linux copies the directory and everything inside it.

---

# 🚚 Moving & Renaming

## `mv`

`mv` can do two things:

```text
move
rename
```

---

## Moving

```bash
mv notes.txt documents/
```

Moves the file into:

```text
documents/
```

---

## Renaming

```bash
mv notes.txt linux-notes.txt
```

Changes the filename from:

```text
notes.txt
```

to:

```text
linux-notes.txt
```

---

## Renaming Directories

The same command works with directories.

```bash
mv docs documentation
```

This renames:

```text
docs/
```

to:

```text
documentation/
```

Mental model:

```text
mv = move OR rename
```

---

# `cp` vs `mv`

```text
cp
│
├── original stays
└── copy appears


mv
│
├── original location/name disappears
└── item exists at the new location/name
```

---

# 🗑️ Removing Files

## `rm`

Deletes files.

```bash
rm notes.txt
```

Terminal deletion should be treated carefully.

Unlike a graphical desktop environment, `rm` usually does not mean:

> Move this to the recycle bin.

---

# Removing Directories

```bash
rm -r old-directory/
```

The:

```text
-r
```

means recursively remove the directory and its contents.

---

# ⚠️ `rm -rf`

Commands such as:

```bash
sudo rm -rf ...
```

should never become something I type without thinking.

Possible flags:

```text
-r → recursive

-f → force
```

Adding:

```text
sudo
```

may also give the command elevated privileges.

A mistake involving the wrong path can become very destructive very quickly.

Rule:

> Always understand the exact path being targeted before running a destructive command.

---

# 🌳 Viewing Directory Structure

## `tree`

Displays files and directories as a hierarchy.

```bash
tree
```

Example:

```text
.
├── Backup
│   └── day1_backup.txt
├── Docs
│   └── commands.txt
└── Notes
    └── Day1_notes.txt
```

This is useful for seeing how directories relate to one another.

If `tree` is not installed, it can normally be installed using the distribution's package manager.

For Debian-based systems:

```bash
sudo apt install tree
```

---

# 📖 Reading Files

Linux provides several ways to inspect text files.

The ones covered in this lab are:

```text
cat
less
head
tail
```

---

# `cat`

Displays the contents of a file directly in the terminal.

```bash
cat notes.txt
```

This is best suited to relatively small files.

Example:

```bash
cat /etc/os-release
```

---

# `less`

Useful for interactively reading longer files.

```bash
less /etc/services
```

Useful controls:

```text
↑ / ↓       move through the file

Space       next page

b           previous page

g           beginning

G           end

/word       search for "word"

n           next matching result

q           quit
```

Important beginner reminder:

```text
q
```

exits `less`.

The terminal is not frozen 😂

---

# `head`

Displays the beginning of a file.

```bash
head notes.txt
```

By default, it normally displays the first few lines.

Specify an exact number:

```bash
head -n 5 notes.txt
```

Meaning:

> Show the first five lines.

---

# `tail`

Displays the end of a file.

```bash
tail notes.txt
```

Specific number:

```bash
tail -n 20 notes.txt
```

Meaning:

> Show the final 20 lines.

---

# Following a File

One of the most useful forms of `tail` for troubleshooting:

```bash
tail -f application.log
```

The:

```text
-f
```

means:

```text
follow
```

As new lines are written to the log file, they appear in the terminal.

This becomes useful when investigating live applications.

Stop it with:

```text
Ctrl + C
```

---

# ✍🏽 Writing Simple Text

## `echo`

Prints text to the terminal.

```bash
echo "hello"
```

Output:

```text
hello
```

---

# Writing to a File

```bash
echo "Linux Orientation 01" > notes.txt
```

This can create the file if it does not already exist.

---

# `>` — Overwrite

A single:

```text
>
```

redirects output into a file.

```bash
echo "hello" > notes.txt
```

Important:

> Existing content may be replaced.

Mental model:

```text
> = overwrite
```

---

# `>>` — Append

Double:

```text
>>
```

adds content to the end of a file.

```bash
echo "Learning Linux administration." >> notes.txt
```

Existing content remains.

Mental model:

```text
>     overwrite

>>    append
```

---

# 🆘 Getting Help

An important Linux skill is not memorizing every command.

It's knowing how to find the documentation.

---

# `--help`

Many commands provide basic help through:

```bash
command --help
```

Example:

```bash
ls --help
```

This may show:

```text
usage
options
flags
descriptions
```

---

# `man`

`man` means:

```text
manual
```

Example:

```bash
man ls
```

Other examples:

```bash
man cp
man mv
man rm
man uname
```

Manual pages commonly include sections such as:

```text
NAME
SYNOPSIS
DESCRIPTION
OPTIONS
EXAMPLES
```

Useful controls:

```text
Space       next page

b           previous page

/word       search

n           next result

q           quit
```

---

# Reading Command Syntax

A manual page may show something like:

```text
cp [OPTION]... SOURCE DEST
```

Meaning:

```text
cp          → command

[OPTION]    → optional command flags

SOURCE      → thing being copied

DEST        → where it should go
```

Example:

```bash
cp notes.txt backup.txt
```

---

# 🔎 Finding Commands

## `which`

Shows the executable found through the current shell's `PATH`.

```bash
which python3
```

Example:

```text
/usr/bin/python3
```

Later, `$PATH` will be covered in more detail.

---

# `type`

Shows how the shell interprets a command.

Example:

```bash
type cd
```

Possible result:

```text
cd is a shell builtin
```

This is useful because not everything typed into a shell is necessarily a standalone executable file.

Mental model:

```text
which
   ↓
WHERE IS THE EXECUTABLE?


type
   ↓
WHAT DOES THE SHELL THINK THIS COMMAND IS?
```

---

# 🕘 Command History

## `history`

Displays previously executed commands.

```bash
history
```

Example:

```text
1 pwd
2 ls
3 cd projects
4 mkdir lab
```

The:

```text
↑
↓
```

arrow keys can also move through previously used commands.

---

# 🧹 Clearing the Terminal

```bash
clear
```

clears the visible terminal.

Keyboard shortcut:

```text
Ctrl + L
```

This does **not** remove shell history.

It only clears the visible screen.

---

# ⌨️ Useful Keyboard Controls

```text
TAB
    autocomplete paths / commands


↑ / ↓
    navigate command history


Ctrl + C
    interrupt the current foreground command


Ctrl + L
    clear the terminal display


Ctrl + D
    send EOF / may close the shell
```

---

# TAB Completion

TAB completion should become a habit.

Instead of typing:

```bash
cd linux-administration-lab
```

I may be able to type:

```text
cd lin
```

then press:

```text
TAB
```

The shell may complete the directory name automatically.

This reduces typing and helps avoid mistakes.

---

# 🧪 Identifying File Types

## `file`

Linux does not depend completely on filename extensions.

A file may simply be called:

```text
backup
```

without:

```text
.txt
.exe
.zip
```

To inspect the actual file type:

```bash
file backup
```

Possible results include:

```text
ASCII text
gzip compressed data
ELF 64-bit executable
```

Example:

```bash
file /bin/ls
```

---

# 🌍 Environment Variables — First Look

Environment variables will be covered properly later.

For now, a few useful examples are:

```bash
echo $HOME
```

Displays the current user's home directory.

```bash
echo $USER
```

Displays the current username.

```bash
echo $SHELL
```

Displays the current configured shell.

Some common environment variables:

```text
$HOME
$USER
$SHELL
$PATH
```

---

# 🧠 Orientation Mental Model

```text
whoami
   ↓
WHO AM I?


id
   ↓
WHAT USER / GROUPS?


pwd
   ↓
WHERE AM I?


hostname
   ↓
WHICH MACHINE?


cat /etc/os-release
   ↓
WHICH DISTRIBUTION?


uname -r
   ↓
WHICH KERNEL?


ls / ls -lah
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
HOW DO I COPY, MOVE OR RENAME?


rm
   ↓
HOW DO I REMOVE THINGS?


cat / less / head / tail
   ↓
HOW DO I READ THINGS?


echo / > / >>
   ↓
HOW DO I WRITE SIMPLE CONTENT?


man / --help
   ↓
HOW DO I LEARN A COMMAND?


which / type
   ↓
HOW DOES THE SHELL FIND / INTERPRET COMMANDS?


history
   ↓
WHAT HAVE I ALREADY RUN?
```

---

# 🧪 Lab 001 Practical

The practical goal was to create and manipulate a small directory structure using the commands covered during the orientation.

Target structure:

```text
linux-orientation-lab/
├── Backup/
│   └── day1_backup.txt
├── Docs/
│   └── commands.txt
└── Notes/
    └── Day1_notes.txt
```

---

## Practical Tasks

### 01 — Identify the Environment

```text
Identify:

- current user
- current directory
- hostname
- Linux distribution
- kernel release
```

Commands involved:

```bash
whoami
pwd
hostname
cat /etc/os-release
uname -r
```

---

### 02 — Return Home

Navigate to the current user's home directory.

```bash
cd ~
```

---

### 03 — Create the Lab

Create:

```text
linux-orientation-lab
```

---

### 04 — Create Directories

Inside the lab create:

```text
Backup
Docs
Notes
```

---

### 05 — Create a Notes File

Create:

```text
Notes/Day1_notes.txt
```

---

### 06 — Write Initial Content

Add:

```text
Linux Orientation 01
```

---

### 07 — Append Another Line

Add:

```text
Learning Linux administration.
```

without deleting the previous content.

---

### 08 — Read the File

Display its contents from the terminal.

---

### 09 — Create a Backup

Copy the notes file into:

```text
Backup/
```

as:

```text
day1_backup.txt
```

---

### 10 — Practice Renaming

Create a file in:

```text
Docs/
```

and rename it to:

```text
commands.txt
```

using:

```bash
mv
```

---

### 11 — Inspect the Directory

Use:

```bash
ls -lah
```

and understand:

```text
-l
-a
-h
```

---

### 12 — Practice Relative Navigation

Enter:

```text
Notes/
```

using a relative path.

---

### 13 — Navigate Up

Use:

```bash
cd ..
```

---

### 14 — Practice an Absolute Path

Navigate to:

```text
/etc
```

using:

```bash
cd /etc
```

---

### 15 — Return to Previous Directory

Use:

```bash
cd -
```

---

### 16 — Inspect `/etc/passwd`

View the beginning:

```bash
head -n 5 /etc/passwd
```

And the end:

```bash
tail -n 5 /etc/passwd
```

---

### 17 — Use `less`

Open:

```bash
less /etc/services
```

Exit with:

```text
q
```

---

### 18 — Read a Manual

```bash
man ls
```

Exit with:

```text
q
```

---

### 19 — Locate Python

```bash
which python3
```

---

### 20 — Inspect a Shell Command

```bash
type cd
```

---

### 21 — Review History

```bash
history
```

---

### 22 — Verify the Final Structure

```bash
tree ~/linux-orientation-lab
```

Expected structure:

```text
linux-orientation-lab
├── Backup
│   └── day1_backup.txt
├── Docs
│   └── commands.txt
└── Notes
    └── Day1_notes.txt
```

---

# 🧩 Things I Initially Got Wrong

Part of this lab was figuring out where my mental model of Linux was still shaky.

A few things I corrected:

```text
hostname
    ≠ current username

hostname
    = machine name
```

```text
ps
    ≠ current directory

ps
    = process information
```

```text
/etc /var /home /tmp
    ≠ levels toward the kernel

they are directories inside the Linux filesystem hierarchy
```

```text
chkdsk
    = Windows tooling

Linux uses different tools for disk and filesystem inspection
```

```text
cp
    = copy
```

And:

```text
r → read

w → write

x → execute / traverse
```

Permissions will be covered properly in a later module rather than trying to memorize them here.

---

# 💡 What Changed After This Lab

Before this lab, I knew a few Linux commands but didn't always have a strong mental model of what the shell was actually doing.

After working through the orientation, I can now distinguish:

```text
user
vs
machine
```

```text
distribution
vs
kernel
```

```text
absolute path
vs
relative path
```

```text
/
vs
/root
```

```text
~
vs
/
```

```text
.
vs
..
```

```text
copy
vs
move
```

```text
cat
vs
less
```

```text
head
vs
tail
```

```text
>
vs
>>
```

```text
which
vs
type
```

More importantly, the terminal feels less like a collection of commands and more like an environment I can navigate and inspect.

---

# ✅ Completion Criteria

Lab 001 is considered complete when I can explain, without relying entirely on notes:

- what user I am;
- which machine I am working on;
- which distribution is installed;
- which kernel is running;
- where I am in the filesystem;
- how absolute and relative paths differ;
- what `/`, `~`, `.`, and `..` mean;
- how `ls`, `ls -l`, `ls -a`, and `ls -lah` differ;
- how to create files and directories;
- how to copy, move and rename them;
- how Linux handles filenames with spaces;
- how case sensitivity affects paths;
- how to inspect files using `cat`, `less`, `head`, and `tail`;
- how `>` and `>>` differ;
- how to use `man` and `--help`;
- how `which` and `type` differ.

---

# 🔐 Public Repository Note

This repository documents the concepts and commands used during the lab without publishing unnecessary details about the machine used to perform the exercises.

Information intentionally excluded includes:

```text
real hostname
local username
IP addresses
MAC addresses
VPN configuration
internal DNS details
device identifiers
SSH keys
API keys
tokens
credentials
```

Examples therefore use generic values such as:

```text
<user>
labuser
linux-lab
10.0.0.x
```

The point of the repository is to document the engineering knowledge, not fingerprint the lab environment.

---

# 📡 Current Status

```bash
user@linux-lab:~$ ./status

module      : Linux Orientation
lab         : 001
theory      : complete
practical   : complete
status      : COMPLETE ✅
next        : Linux Filesystem & Structure
destination : Cloud Infrastructure + Security Engineering
```

---

## 🛣️ Next

### Lab 002 — Linux Filesystem & Structure

Next up:

```text
/
├── boot
├── dev
├── etc
├── home
├── opt
├── proc
├── root
├── run
├── tmp
├── usr
└── var
```

The next goal is to stop seeing these as random Linux directories and understand:

```text
what lives there
why it lives there
which processes use it
what an administrator normally changes
what should usually be left alone
what can break if it is misconfigured
```

---

> The goal isn't to memorize the terminal.
>
> The goal is to stop being lost inside it.

**Lab 001 complete. 🐧**
