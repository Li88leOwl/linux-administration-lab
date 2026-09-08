# 🐧 Lab 002 — Linux Filesystem & Structure

> Learning what actually lives inside `/` and, more importantly, where to start looking when something breaks.

---

## Why this lab exists

Lab 001 was mostly about getting comfortable inside the terminal.

I learned how to:

```text
move around
list files
create files
copy things
rename things
read files
find help
understand basic paths
```

That was useful, but there was still one problem.

I could move around Linux without necessarily understanding **where I was moving to**.

Commands like:

```bash
cd /etc
cd /var
cd /proc
cd /usr
```

aren't particularly useful if those directories are just random names in my head.

So Lab 002 was about building a filesystem mental map.

Not:

> "`/etc` means this because a table says so."

More like:

> "SSH configuration is broken. `/etc` is probably one of the first places I should investigate."

That's the level of understanding I'm aiming for.

---

# 🎯 Lab Objective

By the end of this lab I wanted to be able to look at the top-level Linux filesystem and understand the purpose of the major directories.

The main ones covered were:

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

The goal was to understand:

- what normally lives inside each directory;
- why a Linux administrator cares about it;
- what kind of problem would lead me there;
- which directories contain normal files;
- which directories expose virtual/kernel information;
- where users live;
- where configuration lives;
- where logs live;
- how devices appear in Linux;
- what mount points are;
- the difference between a block device and a filesystem;
- what should absolutely not be modified blindly.

---

# 🧪 Lab Environment

This lab was performed in an isolated Linux learning environment.

```text
Environment    : Linux under virtualization / subsystem
Shell          : Bash
User type      : Standard user with sudo access
Purpose        : Linux administration practice
```

Machine-specific information is intentionally left out of this public repository.

That includes:

```text
real usernames
hostnames
corporate device names
Windows profile paths
IP addresses
MAC addresses
internal DNS information
VPN information
device identifiers
credentials
tokens
SSH keys
```

Examples throughout this repository use generic values such as:

```text
user
linux-lab
/home/<user>
```

The useful part is the Linux knowledge.

The exact laptop I happened to learn it on isn't.

---

# 🌳 The Linux Filesystem Starts at `/`

The first thing I had to properly understand was that Linux does not organize storage in quite the same way I was used to seeing it on Windows.

Windows commonly exposes drives like:

```text
C:\
D:\
E:\
```

Linux presents one main filesystem tree beginning at:

```text
/
```

This is called the:

```text
filesystem root
```

Everything appears somewhere underneath it.

A simplified view:

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

That does **not** necessarily mean every directory physically lives on the same disk.

Linux can attach different:

```text
disks
partitions
network filesystems
virtual filesystems
external devices
```

into different points in this tree.

That idea becomes important when learning about mounting.

---

# `/` vs `/root`

This was one of the misconceptions I had to correct.

They are completely different things.

```text
/
```

means:

```text
the top of the Linux filesystem
```

Whereas:

```text
/root
```

normally means:

```text
the root user's home directory
```

Example:

```text
/
├── home
│   └── <user>
│
├── root
│
├── etc
├── var
└── usr
```

A normal user can run:

```bash
cd /
```

and still remain a normal user.

Changing directories does not change privileges.

For example:

```bash
cd /
whoami
```

might still return:

```text
user
```

because:

```text
filesystem location
≠
privilege level
```

Privileges depend on things such as:

```text
user identity
group membership
permissions
sudo/root access
```

not where I happen to be standing.

---

# `/etc` — System Configuration

If I had to associate `/etc` with one word, it would be:

```text
CONFIGURATION
```

A lot of system-wide Linux configuration lives here.

Explore it with:

```bash
cd /etc
ls
```

Common things found under `/etc` include configuration related to:

```text
users
groups
SSH
networking
package management
services
authentication
scheduled tasks
systemd
filesystems
hostname configuration
```

Some examples:

```text
/etc/hosts
/etc/hostname
/etc/passwd
/etc/group
/etc/shadow
/etc/fstab
/etc/ssh/
/etc/systemd/
/etc/apt/
```

The exact contents vary depending on the system.

---

## `/etc/os-release`

A useful file for identifying the operating system:

```bash
cat /etc/os-release
```

This may contain fields such as:

```text
NAME
VERSION
VERSION_ID
ID
PRETTY_NAME
```

---

## `/etc/passwd`

Can be inspected with:

```bash
head /etc/passwd
```

Despite the filename, modern Linux systems do not normally store password hashes directly inside `/etc/passwd`.

That topic belongs properly in the users and permissions labs.

For now, the important lesson is:

```text
/etc/passwd
→ user account information
```

---

## Why `/etc` matters to an administrator

Imagine:

> SSH worked yesterday. Someone changed the configuration. Nobody can connect anymore.

A reasonable thought process is:

```text
SSH problem
↓
possibly configuration
↓
system configuration
↓
/etc
↓
/etc/ssh
```

That's more useful than simply memorizing:

```text
/etc = configuration
```

---

## What can go wrong here?

Quite a lot.

Bad configuration under `/etc` can contribute to:

```text
SSH failures
authentication problems
services refusing to start
mount failures
networking issues
broken package configuration
incorrect permissions
boot problems
```

So one rule I want to keep:

```text
READ
↓
UNDERSTAND
↓
BACK UP
↓
EDIT
↓
VALIDATE
↓
RELOAD / RESTART
↓
VERIFY
```

Not:

```text
sudo nano something-important.conf

hope for the best
```

---

# `/home` — Normal Users' Home Directories

Think:

```text
/home
↓
normal users
```

Example:

```text
/home/
├── alice
├── bob
└── charlie
```

My own home directory can normally be reached with:

```bash
cd ~
```

and confirmed using:

```bash
pwd
```

or:

```bash
echo $HOME
```

Example:

```text
/home/<user>
```

---

## `/home` vs `~`

These are related but not identical.

```text
/home
```

is the directory that commonly contains multiple users' home directories.

```text
~
```

is shell shorthand for the **current user's home directory**.

So:

```bash
cd /home
```

might place me here:

```text
/home
```

while:

```bash
cd ~
```

might place me here:

```text
/home/<user>
```

---

## What normally lives in a user's home?

Things such as:

```text
documents
projects
downloads
scripts
personal configuration
SSH configuration
shell configuration
application settings
```

Including hidden files:

```text
.bashrc
.profile
.ssh
.config
```

To see hidden files:

```bash
ls -lah ~
```

---

## Administrator scenario

Suppose:

> SSH works for every user except one.

That changes the investigation.

Instead of immediately blaming the whole SSH service, I might investigate that specific user's configuration.

For example:

```text
/home/<user>/.ssh/
```

and eventually things such as:

```text
authorized_keys
permissions
ownership
```

This is why understanding `/home` matters.

---

# `/root` — Root User's Home

`/root` is normally the home directory belonging to the root user.

Compare:

```text
/home/<user>
→ normal user's home

/root
→ root user's home
```

Inspect the directory entry:

```bash
ls -ld /root
```

A normal user may not have permission to inspect its contents.

That's expected.

---

## A useful lesson from `Permission denied`

One thing I'm deliberately trying not to do is develop this habit:

```text
Permission denied
↓
sudo everything
```

Instead:

```text
Permission denied
↓
why am I being denied?
↓
should this user have access?
↓
do I actually need elevated privileges?
```

Sometimes Linux refusing access means Linux is working exactly as intended.

---

# `ls -ld`

This command came up during the lab:

```bash
ls -ld /root
```

The flags are:

```text
-l
→ long listing

-d
→ show the directory itself rather than listing its contents
```

So:

```bash
ls -ld /root
```

means:

> Show detailed information about the `/root` directory itself.

The short flags can also be written separately:

```bash
ls -l -d /root
```

or combined:

```bash
ls -ld /root
```

---

# `/var` — Variable Data

If `/etc` is mostly configuration, `/var` is where a lot of **changing system/application data** lives.

Mental model:

```text
/var
↓
variable data
```

Explore:

```bash
ls -lah /var
```

Common subdirectories include:

```text
/var/log
/var/lib
/var/cache
/var/spool
/var/tmp
```

---

# `/var/log` — Logs

This is one of the most important locations for troubleshooting.

Think:

```text
something broke
↓
what happened?
↓
logs
↓
/var/log
```

Explore:

```bash
ls -lah /var/log
```

Traditional log files may live here.

Modern Linux systems may also store logging information through the systemd journal, which will later introduce commands such as:

```bash
journalctl
```

But `/var/log` remains an important place to understand.

---

## Real troubleshooting example

Imagine:

> The application stopped working at 14:32.

A troubleshooting path might be:

```text
application failed
↓
find relevant service/application logs
↓
look around 14:32
↓
identify error
↓
work backwards from evidence
```

Later commands such as:

```bash
tail -f <logfile>
```

become useful here.

---

# `du` — Disk Usage

During this lab I used:

```bash
du -sh /var/log
```

`du` stands for:

```text
disk usage
```

Breakdown:

```text
du
→ disk usage

-s
→ summarize

-h
→ human-readable
```

So:

```bash
du -sh /var/log
```

roughly means:

> Tell me the total amount of disk space used by `/var/log` in a readable format.

Without `-s`:

```bash
du -h /var/log
```

I may see usage for many items underneath it.

With:

```bash
du -sh /var/log
```

I get the summary.

---

## `du` and permissions

Running:

```bash
du -sh /var/log
```

may produce something such as:

```text
du: cannot read directory '...': Permission denied
17M    /var/log
```

This means the current user could not inspect every part of the directory tree.

Running:

```bash
sudo du -sh /var/log
```

may allow the command to inspect areas that require elevated privileges.

The important lesson wasn't simply:

```text
sudo fixes it
```

It was:

```text
normal command
↓
observe permission boundary
↓
understand why
↓
elevate deliberately if required
```

---

# `du` vs `df`

I only introduced this distinction briefly here.

```text
du
→ how much space files/directories are using

df
→ how much space the filesystem has available/used
```

Example:

```bash
du -sh /var/log
```

asks about the contents of `/var/log`.

Whereas:

```bash
df -h
```

shows filesystem-level capacity information.

`df` will be covered properly in the storage module.

---

# `/tmp` — Temporary Files

Mental model:

```text
/tmp
↓
temporary working data
```

Applications and users may create short-lived files here.

Example:

```bash
echo "Lab 002 temporary test" > /tmp/lab002-test.txt
```

Read:

```bash
cat /tmp/lab002-test.txt
```

Remove:

```bash
rm /tmp/lab002-test.txt
```

Then:

```bash
ls /tmp/lab002-test.txt
```

should confirm that the file no longer exists.

---

## Important rule

Do not treat `/tmp` as permanent storage.

Depending on the system configuration, temporary files may be cleaned:

```text
periodically
during boot
by system services
after certain periods
```

Exact cleanup behaviour varies.

---

## Security relevance

Temporary directories are interesting from a security perspective because many processes and users may create content there.

That does **not** mean:

```text
file in /tmp
=
malware
```

It means `/tmp` can be worth investigating when behaviour is suspicious.

Context matters.

---

# `/usr` — User-Space Programs and Shared Resources

This directory caused a little confusion because the name looks like:

```text
user
```

But `/usr` is **not where normal user profiles live**.

That is primarily:

```text
/home
```

A useful mental model for `/usr` is:

```text
installed user-space programs
libraries
shared application resources
```

Common directories include:

```text
/usr/bin
/usr/sbin
/usr/lib
/usr/share
/usr/local
```

---

# `/usr/bin`

Contains many executable programs.

Example:

```bash
which python3
```

may return something under:

```text
/usr/bin/
```

Likewise:

```bash
which ls
which grep
```

may point to executables beneath `/usr/bin`.

This helped connect something from Lab 001:

```text
command
↓
shell searches PATH
↓
executable discovered
↓
filesystem location
```

---

# `/usr/sbin`

Traditionally contains many system-administration-oriented executables.

The exact distinction between `bin` and `sbin` is less strict on some modern systems than it historically was, but the concept is still useful.

---

# `/usr/lib`

Contains libraries and supporting application components.

Applications frequently depend on libraries rather than containing every function themselves.

This will matter more later when troubleshooting software dependencies.

---

# `/usr/share`

Commonly contains architecture-independent shared data.

Examples may include:

```text
documentation
manual data
icons
localization resources
application resources
```

---

# `/usr/local`

A useful place to understand:

```text
/usr/local
```

is commonly intended for software installed locally by the administrator rather than managed as part of the operating system's normal packaged files.

This can help separate:

```text
distribution-managed software
```

from:

```text
locally installed software
```

---

# `/home` vs `/usr`

The distinction I want to remember:

```text
/home
→ users' personal data and configuration


/usr
→ installed user-space software and shared resources
```

So:

```text
/home/alice
```

might contain Alice's personal files.

While:

```text
/usr/bin/python3
```

might contain an executable available to users of the system.

---

# `/opt` — Optional / Third-Party Software

Think:

```text
/opt
↓
optional software
```

It is commonly used for self-contained or third-party application installations.

Example structure:

```text
/opt/
└── vendor-application/
```

The directory may also be completely empty.

That is not a problem.

---

## Administrator scenario

Suppose someone says:

> An enterprise monitoring agent was installed manually and nobody remembers exactly where.

A sensible first place to investigate might be:

```text
/opt
```

The important word is:

```text
might
```

Linux conventions guide investigation.

They do not guarantee that every vendor follows them perfectly.

---

# `/boot` — Boot-Related Files

Think:

```text
/boot
↓
files involved in booting Linux
```

Depending on the system, this may contain things such as:

```text
kernel images
initial RAM filesystem images
bootloader files
```

Names may resemble:

```text
vmlinuz-...
initrd.img-...
grub/
```

The exact contents depend heavily on the environment.

---

## Virtualization and subsystem note

A Linux environment running under a subsystem or special virtualization layer may not have the same `/boot` contents as a traditional physical Linux installation or normal VM.

That doesn't change what `/boot` means conceptually.

It just means:

> Different Linux environments do not always boot in exactly the same way.

---

## Safety rule

For this lab:

```text
inspect /boot
do not experiment inside /boot
```

Deleting or changing boot-critical files is not something to learn through random trial and error.

There will be safer ways to learn the Linux boot process later.

---

# `/dev` — Device Interfaces

This was probably one of the coolest concepts in the lab.

`/dev` contains **special filesystem entries that provide interfaces to devices and kernel-managed resources**.

They're not ordinary documents like:

```text
notes.txt
```

They are file-like interfaces that programs can interact with through operations such as:

```text
open
read
write
close
```

Mental model:

```text
program
↓
/dev/<device>
↓
kernel
↓
driver / kernel subsystem
↓
device
```

---

# The "Everything Is a File" Idea

Unix/Linux is famous for the idea:

> Everything is a file.

That isn't literally true in every technical sense, but it's a useful mental model.

Linux exposes many resources through file-like interfaces.

Examples:

```text
/dev/null
/dev/tty
/dev/sda
/dev/nvme0n1
```

Exact devices depend on the environment.

---

# `/dev/null`

Probably the easiest device to understand.

Think:

```text
/dev/null
↓
black hole
```

Example:

```bash
echo "hello" > /dev/null
```

The output from `echo` is written to `/dev/null`.

The kernel accepts it and discards it.

Nothing useful is stored.

Reading from `/dev/null`:

```bash
cat /dev/null
```

returns no content.

Mental model:

```text
write to /dev/null
→ disappears


read from /dev/null
→ empty result
```

---

# Device Types

Inspecting a device:

```bash
ls -l /dev/null
```

may show something beginning with:

```text
c
```

The first character indicates the filesystem entry type.

Some useful ones:

```text
- → regular file

d → directory

c → character device

b → block device
```

---

# Character Devices

Character devices generally provide stream-like access.

Examples can include things such as:

```text
terminals
serial interfaces
pseudo-devices
```

Data is handled more like a stream of bytes/characters.

---

# Block Devices

Block devices are commonly associated with storage.

Examples may look like:

```text
/dev/sda
/dev/sda1
/dev/nvme0n1
/dev/nvme0n1p1
```

A block device is accessed in chunks called:

```text
blocks
```

Common examples include:

```text
hard drives
SSDs
partitions
virtual disks
```

---

# Whole Disk vs Partition

Important distinction:

```text
/dev/sda
→ whole disk
```

while:

```text
/dev/sda1
→ first partition on that disk
```

Conceptually:

```text
/dev/sda
│
├── /dev/sda1
├── /dev/sda2
└── /dev/sda3
```

So `/dev/sda` and `/dev/sda1` are not normally two separate physical disks.

One represents the whole device.

The other represents a partition on it.

---

# Block Device vs Filesystem

This was another important correction during the lab.

A **block device** is not the same thing as a **filesystem**.

Think:

```text
block device
→ storage
```

Examples:

```text
/dev/sda
/dev/sda1
```

A filesystem is the structure used to organize files and directories on that storage.

Examples include:

```text
ext4
xfs
btrfs
FAT32
NTFS
```

So:

```text
block device
↓
raw storage

filesystem
↓
rules/structure used to organize data on that storage
```

---

# Linux Filesystem Hierarchy vs Filesystem Type

The word "filesystem" can mean different things depending on context.

These:

```text
/etc
/var
/home
/usr
```

are part of the:

```text
Linux filesystem hierarchy
```

Whereas:

```text
ext4
xfs
btrfs
NTFS
```

are:

```text
filesystem types
```

Those concepts should not be mixed together.

---

# Storage Mental Model

A useful simplified chain:

```text
physical / virtual disk
        ↓
block device
        ↓
partition
        ↓
filesystem
        ↓
mount point
        ↓
files and directories
```

Example:

```text
disk
↓
/dev/sdb
↓
/dev/sdb1
↓
ext4
↓
/mnt/data
↓
files
```

This is simplified, but it gives me the right starting mental model.

The storage lab will go much deeper.

---

# Mount Points

A mount point is:

> the directory where a filesystem is attached to the Linux filesystem tree and made accessible.

Example:

```text
/dev/sdb1
→ block device / partition

ext4
→ filesystem

/mnt/data
→ mount point
```

A command might eventually look like:

```bash
mount /dev/sdb1 /mnt/data
```

Conceptually:

```text
take the filesystem on /dev/sdb1
↓
attach it to the Linux directory tree
↓
make it accessible through /mnt/data
```

The mount point is therefore not the storage itself.

It is the **directory through which that filesystem becomes accessible**.

A mental shortcut I like:

```text
mount point
=
doorway into a filesystem
```

---

# `/mnt`

Although `/mnt` wasn't one of the main directories originally targeted in this lab, it became relevant while learning mount points.

`/mnt` is commonly used as a location for mounting filesystems.

In subsystem-based Linux environments, host operating-system drives may also appear underneath `/mnt`.

For example:

```text
host storage
↓
mounted into Linux
↓
/mnt/<something>
```

This made mount points much less abstract.

---

# `findmnt`

A useful command for inspecting mounted filesystems:

```bash
findmnt
```

To inspect the filesystem mounted at `/`:

```bash
findmnt /
```

To inspect a known mount point:

```bash
findmnt /mnt/<mount>
```

This helps answer questions such as:

```text
what filesystem is mounted here?
where did it come from?
what type is it?
```

We'll use this much more during storage.

---

# `/proc` — Live Process and Kernel Information

This was another major mental-model correction.

`/proc` is **not just another ordinary directory full of files stored on disk**.

It is a virtual filesystem.

Think:

```text
/proc
↓
live process + kernel information
```

---

# Process IDs Under `/proc`

Inside `/proc` there are numbered directories such as:

```text
/proc/1
/proc/500
/proc/4242
```

Those numbers correspond to:

```text
PID
=
Process ID
```

If PID `4242` is running:

```text
/proc/4242
```

may exist.

If that process exits:

```text
/proc/4242
```

disappears.

That's because `/proc` reflects live system state.

---

# `$$` — Current Shell PID

In Bash:

```bash
echo $$
```

returns the PID of the current shell.

That means I can inspect my own shell through:

```bash
head /proc/$$/status
```

Conceptually:

```text
current Bash shell
↓
has PID
↓
/proc/<PID>
↓
kernel exposes information about it
```

---

# `/proc/meminfo`

Displays memory-related information:

```bash
head /proc/meminfo
```

May include values such as:

```text
MemTotal
MemFree
MemAvailable
Buffers
Cached
```

---

# `/proc/cpuinfo`

Provides CPU information:

```bash
head /proc/cpuinfo
```

---

# `ls` vs `cat` vs `head`

I made a useful mistake while exploring `/proc`.

Running:

```bash
ls /proc/meminfo
```

does not display the contents of `meminfo`.

It simply lists the filesystem entry.

To read it:

```bash
cat /proc/meminfo
```

Or to read only the beginning:

```bash
head /proc/meminfo
```

Mental model:

```text
ls
→ show/list filesystem entries


cat
→ display file contents


head
→ display beginning of file/content
```

That mistake made the distinction much easier to remember.

---

# `/proc` Is Not "Processes Started at Boot"

Another misconception I corrected:

`/proc` does not only contain processes that started during boot.

It reflects **currently running processes**.

A process can start:

```text
during boot
after login
five minutes ago
because I launched a program
because a service restarted
```

and still receive a PID represented under `/proc`.

Likewise, when the process ends, its PID directory disappears.

---

# `/run` — Current Runtime State

Think:

```text
/run
↓
short-lived runtime state
```

It commonly contains information created while the current system instance is running.

Examples can include:

```text
PID files
sockets
runtime metadata
service state
locks
```

The exact contents depend on the machine and services running.

---

# `/etc` vs `/run`

This distinction matters.

```text
/etc
→ persistent configuration
```

Example:

```text
how a service SHOULD be configured
```

Whereas:

```text
/run
→ runtime state
```

Example:

```text
information relating to what is happening during the current boot
```

Shortcut:

```text
/etc
=
how the system should be configured


/run
=
what the running system currently needs/has at runtime
```

---

# `/proc` vs `/run`

These are also not the same thing.

```text
/proc
→ virtual view of live processes and kernel information
```

```text
/run
→ runtime state files/data created for the current system instance
```

---

# `/proc`, `/dev` and `/run`

A useful comparison:

```text
/proc
→ process + kernel information


/dev
→ device interfaces


/run
→ current runtime state
```

They're all important, but for completely different reasons.

---

# Filesystem Directories Are NOT Kernel Layers

This was probably the biggest conceptual correction in Lab 002.

I originally thought directories such as:

```text
/etc
/var
/proc
/dev
/run
```

were different "levels" of Linux somehow leading closer toward the kernel.

They are not.

They are different locations in the Linux filesystem hierarchy.

Some contain ordinary persistent data.

Some expose virtual information.

Some expose devices.

Their position in the directory tree does not mean they are:

```text
closer to the kernel
```

or:

```text
further away from the kernel
```

A better model:

```text
                         Linux System
                              │
             ┌────────────────┼────────────────┐
             │                │                │
          Kernel          Processes       Filesystems
             │
             ├── exposes process/kernel info through /proc
             │
             ├── exposes device interfaces through /dev
             │
             └── interacts with the rest of userspace
```

Meanwhile directories such as:

```text
/etc
/var
/home
/usr
```

organize files and data used by the system and its applications.

---

# `mkdir -p`

Another flag introduced during the practical:

```bash
mkdir -p labs/linux/day1
```

`mkdir` means:

```text
make directory
```

The:

```text
-p
```

flag stands for:

```text
parents
```

It tells `mkdir` to create missing parent directories along the path.

Without `-p`, this may fail if the parent directories do not exist:

```bash
mkdir labs/linux/day1
```

With:

```bash
mkdir -p labs/linux/day1
```

Linux can create:

```text
labs/
└── linux/
    └── day1/
```

in one command.

---

# Brace Expansion

This command was used during the practical:

```bash
mkdir -p linux-filesystem-lab/{Notes,Checks}
```

Two separate concepts are happening here.

First:

```text
-p
→ mkdir option
```

Second:

```text
{Notes,Checks}
→ Bash brace expansion
```

Bash expands:

```bash
linux-filesystem-lab/{Notes,Checks}
```

into approximately:

```bash
linux-filesystem-lab/Notes linux-filesystem-lab/Checks
```

So the command effectively becomes:

```bash
mkdir -p linux-filesystem-lab/Notes linux-filesystem-lab/Checks
```

The braces are a shell feature.

They are not part of `mkdir` itself.

---

# 🧪 Lab Practical

The practical workspace was:

```text
linux-filesystem-lab/
├── Checks/
└── Notes/
```

Created with:

```bash
mkdir -p linux-filesystem-lab/{Notes,Checks}
```

---

## 1. Map the filesystem root

```bash
cd /
pwd
ls -lah
```

Then inspect the major directory entries:

```bash
ls -ld / /etc /home /root /var /tmp /usr /opt /boot /dev /proc /run
```

Goal:

Recognize the names and associate them with their purpose.

---

## 2. Inspect `/etc`

```bash
cd /etc
pwd
ls | head
cat os-release
head -n 5 passwd
```

Goal:

Understand `/etc` as the main area for system-wide configuration.

---

## 3. Inspect `/home`

```bash
cd /home
pwd
ls -lah
```

Then:

```bash
cd ~
pwd
echo $HOME
ls -lah
```

Goal:

Understand:

```text
/home
→ collection of normal users' home directories


~
→ current user's home
```

---

## 4. Compare `/` and `/root`

```bash
ls -ld /
ls -ld /root
```

Goal:

Lock in:

```text
/
→ filesystem root


/root
→ root user's home
```

---

## 5. Inspect `/var/log`

```bash
ls -lah /var/log
```

Then:

```bash
du -sh /var/log
```

If part of the tree is inaccessible:

```bash
sudo du -sh /var/log
```

Goal:

Understand both:

```text
/var/log
→ logs
```

and:

```text
permissions can limit what a normal user can inspect
```

---

## 6. Use `/tmp`

Create:

```bash
echo "Lab 002 temporary test" > /tmp/lab002-test.txt
```

Read:

```bash
cat /tmp/lab002-test.txt
```

Inspect:

```bash
ls -l /tmp/lab002-test.txt
```

Remove:

```bash
rm /tmp/lab002-test.txt
```

Verify:

```bash
ls /tmp/lab002-test.txt
```

Goal:

Understand temporary working data and cleanup.

---

## 7. Connect commands to `/usr`

```bash
which ls
which python3
which grep
```

Then:

```bash
file "$(which ls)"
file "$(which python3)"
```

Goal:

Connect executable commands to actual filesystem locations.

---

## 8. Inspect `/opt`

```bash
ls -lah /opt
```

Goal:

Understand where optional or third-party application installations may live.

An empty `/opt` is completely valid.

---

## 9. Inspect `/boot`

```bash
ls -lah /boot
```

Goal:

Recognize the purpose of boot-related storage.

No modification required.

---

## 10. Inspect `/dev`

```bash
ls -l /dev/null
```

Then:

```bash
echo "this disappears" > /dev/null
```

And:

```bash
cat /dev/null
```

Goal:

Understand that `/dev` contains special device interfaces rather than ordinary stored documents.

---

## 11. Inspect `/proc`

Current shell PID:

```bash
echo $$
```

Process information:

```bash
head /proc/$$/status
```

Memory:

```bash
head /proc/meminfo
```

CPU:

```bash
head /proc/cpuinfo
```

Goal:

Understand `/proc` as a live virtual interface into process/kernel information.

---

## 12. Inspect `/run`

```bash
ls -lah /run | head -n 20
```

Goal:

Understand `/run` as short-lived runtime state rather than permanent configuration.

---

## 13. Inspect mount points

```bash
findmnt /
```

If another known filesystem is mounted somewhere:

```bash
findmnt /mnt/<mount>
```

Goal:

Understand that a filesystem can be attached into the Linux directory tree through a mount point.

---

# 🧠 Troubleshooting Mental Map

This is probably the most useful part for future me.

Instead of remembering definitions, start with the incident.

```text
"SSH configuration is broken."
        ↓
      /etc
        ↓
   /etc/ssh


"Logs are filling the disk."
        ↓
      /var
        ↓
    /var/log


"Only one user's SSH keys are failing."
        ↓
     /home
        ↓
 /home/<user>/.ssh


"I need information about PID 4242."
        ↓
      /proc
        ↓
  /proc/4242


"What storage devices does Linux expose?"
        ↓
      /dev


"Where was that optional enterprise application installed?"
        ↓
      /opt
     maybe


"I need boot-related files."
        ↓
      /boot


"I need somewhere temporary."
        ↓
      /tmp


"I need runtime state from this boot."
        ↓
      /run
```

---

# 🧭 Directory Quick Reference

```text
/
    top of the Linux filesystem tree


/etc
    system-wide configuration


/home
    normal users' home directories


/root
    root user's home directory


/var
    variable/changing application and system data


/var/log
    traditional log storage


/tmp
    temporary working data


/usr
    installed user-space programs, libraries and shared resources


/usr/bin
    many common executables


/usr/sbin
    many administration-oriented executables


/usr/lib
    libraries/supporting components


/usr/share
    shared architecture-independent resources


/usr/local
    locally administered software/files


/opt
    optional or third-party software


/boot
    boot-related files


/dev
    device interfaces


/proc
    virtual live process and kernel information


/run
    short-lived runtime state for the current boot


/mnt
    common location used for mounted filesystems
```

---

# 🔧 Command Quick Reference

```bash
# Show current location
pwd


# Move to filesystem root
cd /


# Move home
cd ~


# List detailed directory information
ls -ld /etc


# Human-readable listing
ls -lah


# Inspect directory usage
du -sh /var/log


# Show filesystem/mount information
findmnt /


# Identify current shell PID
echo $$


# Inspect current shell process
head /proc/$$/status


# Inspect memory information
head /proc/meminfo


# Inspect CPU information
head /proc/cpuinfo


# Find executable
which python3


# Identify file/device type
file <path>


# Create nested directories
mkdir -p path/to/directory


# Send output into the Linux black hole
echo "ignore this" > /dev/null
```

---

# 🧩 Things I Got Wrong During This Lab

This section is staying because it's probably more useful than pretending I understood everything immediately.

---

## 1. I thought `/` represented a normal-user environment

Wrong.

```text
/
→ filesystem root
```

Privilege is unrelated.

I can stand in `/` while still being a normal user.

---

## 2. I confused `/root` with filesystem root

Correct distinction:

```text
/
→ filesystem root


/root
→ root user's home
```

---

## 3. I thought `/proc` contained processes started during boot

Wrong.

`/proc` reflects live process and kernel information.

Processes may appear and disappear throughout the entire lifetime of the system.

---

## 4. I treated filesystem directories like "levels toward the kernel"

Wrong.

```text
/etc
/var
/home
/usr
/proc
/dev
/run
```

are not layers of Linux stacked toward the kernel.

They're different parts of the filesystem hierarchy with different jobs.

Some, such as `/proc` and `/dev`, expose kernel-managed information/interfaces.

That doesn't make them "deeper levels."

---

## 5. I mixed up block devices and filesystems

Correct version:

```text
block device
→ storage interface


filesystem
→ structure used to organize data on storage
```

Example:

```text
/dev/sda1
→ partition/block device


ext4
→ possible filesystem on it
```

---

## 6. I mixed up whole disks and partitions

Correct version:

```text
/dev/sda
→ whole disk


/dev/sda1
→ first partition
```

---

## 7. I initially described mount points awkwardly

The cleaner definition:

> A mount point is the directory where a filesystem is attached to the Linux filesystem tree and made accessible.

That one is worth remembering.

---

# 🔐 Safety Notes

Some directories are fine to explore.

Others deserve a lot more respect.

For beginner labs:

```text
/etc
→ inspect first, don't randomly edit


/boot
→ inspect only


/dev
→ inspect device entries, do not write randomly to block devices


/proc
→ mostly inspect


/run
→ inspect, don't randomly modify service runtime state


/root
→ don't force access just because sudo exists
```

Especially avoid experimenting with commands that write directly to storage devices such as:

```text
/dev/sda
/dev/nvme0n1
```

unless the environment was specifically built to be destroyed.

---

# 🛡️ Public Repository Hygiene

Terminal screenshots can accidentally reveal more than expected.

Before publishing anything, check for:

```text
real username
hostname
workstation name
company naming convention
Windows profile name
IP address
MAC address
DNS names
VPN interfaces
SSH keys
tokens
API credentials
internal paths
```

Public examples in this repository use:

```text
user@linux-lab
/home/<user>
```

rather than the actual environment identifiers.

---

# ✅ Knowledge Check

By the end of the lab I should be able to answer these without looking them up.

### What is `/`?

```text
The top of the Linux filesystem hierarchy.
```

### What is `/root`?

```text
The root user's home directory.
```

### Where would I usually look for system configuration?

```text
/etc
```

### Where would I normally investigate traditional logs?

```text
/var/log
```

### Where do normal users commonly have their home directories?

```text
/home
```

### What is `/tmp` for?

```text
Short-lived temporary working data.
```

### Is `/usr` where user profiles live?

```text
No.

/usr contains user-space software and shared resources.

/home contains normal users' homes.
```

### What might `/opt` contain?

```text
Optional or third-party application installations.
```

### What does `/dev` expose?

```text
Device interfaces.
```

### What is `/proc`?

```text
A virtual filesystem exposing live process and kernel information.
```

### What happens to `/proc/5000` when PID 5000 exits?

```text
The entry disappears.
```

### What does `/run` contain?

```text
Short-lived runtime state for the current system instance.
```

### What is a block device?

```text
A storage device/interface Linux accesses in blocks.
```

### What is a filesystem?

```text
The structure used to organize files and directories on storage.
```

### What is a mount point?

```text
A directory where a filesystem is attached and made accessible.
```

### Are `/etc`, `/var`, `/proc`, `/dev` and `/run` different levels leading toward the kernel?

```text
No.

They are different locations/interfaces within the Linux filesystem hierarchy.
```

---

# 📌 What Changed After This Lab

Before this lab, the top of a Linux filesystem looked mostly like this:

```text
etc
var
usr
proc
dev
opt

random Linux stuff
```

Now it looks more like:

```text
/etc
→ where configuration probably lives


/var/log
→ where troubleshooting evidence may live


/home/<user>
→ where user-specific configuration may live


/usr
→ where a lot of installed software lives


/opt
→ where third-party software might live


/dev
→ where Linux exposes device interfaces


/proc
→ where I can inspect live process/kernel information


/run
→ where current runtime state may live
```

That's a much more useful way of seeing Linux.

I don't need to memorize every directory Linux has.

I need enough of a map that when something goes wrong, I can ask:

> **Where would Linux normally keep the thing I'm looking for?**

---

# 🧪 Lab Result

```text
LAB 002 — Linux Filesystem & Structure

Theory              : COMPLETE
Filesystem tour     : COMPLETE
Hands-on practical  : COMPLETE
Knowledge test      : PASSED WITH CORRECTIONS
Corrections reviewed: COMPLETE

STATUS               : COMPLETE ✅
```

---

# 🛣️ Progress

```text
Linux Administration
│
├── Lab 001 — Linux Orientation
│   └── COMPLETE ✅
│
├── Lab 002 — Linux Filesystem & Structure
│   └── COMPLETE ✅
│
└── Lab 003 — Users, Groups & Permissions
    └── NEXT
```

---

# Next Up — Lab 003

## Users, Groups & Permissions

The next lab moves from:

```text
where things live
```

into:

```text
who is allowed to do what
```

Topics will include:

```text
users
UIDs
groups
GIDs
/etc/passwd
/etc/group
/etc/shadow
useradd
adduser
usermod
groups
id
su
sudo
file ownership
chown
chmod
rwx
directory permissions
numeric permissions
symbolic permissions
permission troubleshooting
```

This should also finally make lines such as:

```text
-rw-r--r--
drwx------
```

stop looking like encrypted messages.

---

> Lab 001 taught me how to move around Linux.
>
> Lab 002 taught me where I was actually going.

**Lab 002 complete. 🐧**
