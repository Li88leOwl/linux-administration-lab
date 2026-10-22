# 🐧 Lab 003 — Users, Groups & Permissions

> Linux doesn't just care about what a file is.  
> It cares about **who is asking** and **what they're allowed to do**.

---

## Why this lab exists

The first two labs gave me a map.

Lab 001 taught me how to move around Linux.

Lab 002 taught me what places such as:

```text
/etc
/var
/home
/proc
/dev
/run
```

actually mean.

Lab 003 adds another layer:

> **Who is allowed to access all of this?**

This one took a little longer to settle in because permissions don't really click from reading `rwx` definitions.

They make more sense after:

```text
creating users
creating groups
breaking access
switching identities
getting Permission denied
checking ownership
fixing only what was wrong
```

By the end of this lab, permission strings such as:

```text
-rwxr-x---
```

should no longer look like encrypted Linux nonsense.

---

# 🎯 Objective

The goal of Lab 003 was to understand:

```text
users
UIDs

groups
GIDs

primary groups
supplementary groups

/etc/passwd
/etc/group
/etc/shadow

sudo
su

file ownership
group ownership

chmod
chown

symbolic permissions
numeric permissions

file permissions
directory permissions

permission troubleshooting
least privilege
```

More importantly, I wanted to stop responding to:

```text
Permission denied
```

with:

```text
sudo everything
```

or worse:

```bash
chmod 777 everything
```

The goal is to understand **why access failed first**.

---

# 🧪 Lab Environment

This lab was performed in an isolated Linux learning environment.

```text
Environment     : Linux lab
Shell           : Bash
Account         : Standard user with sudo authorization
Purpose         : Linux administration practice
```

Machine-specific information is intentionally omitted from this public repository.

Examples use generic identities such as:

```text
labuser
labguest
labteam
linux-lab
/home/<user>
```

No real workstation names, company identifiers, usernames, network information or credentials are included.

---

# 🧠 The Basic Permission Model

Linux is designed as a multi-user operating system.

That means Linux constantly has to answer questions such as:

```text
Who is asking?

Which user are they?

Which groups do they belong to?

Who owns this file?

Which group owns it?

What permissions were granted?

What exactly are they trying to do?
```

A simplified access decision looks like:

```text
USER REQUESTS ACCESS
        │
        ▼
Who is the user?
        │
        ▼
Are they the owner?
   │           │
  YES          NO
   │           │
   ▼           ▼
OWNER       Are they in
BITS        file's group?
               │
           ┌───┴───┐
          YES      NO
           │        │
           ▼        ▼
        GROUP     OTHER
         BITS      BITS
```

This becomes the foundation for almost everything else in this lab.

---

# 👤 Users

A Linux user is an identity that can own files, run processes and receive permissions.

Check the current user:

```bash
whoami
```

Mental model:

```text
whoami
↓
WHO AM I OPERATING AS RIGHT NOW?
```

---

# `id`

`id` gives much more information.

```bash
id
```

A result may look structurally like:

```text
uid=1000(labuser) gid=1000(labuser) groups=1000(labuser),27(sudo),1002(labteam)
```

Breakdown:

```text
uid
→ User Identifier


gid
→ primary Group Identifier


groups
→ primary + supplementary group memberships
```

---

# UID — User Identifier

Linux users have numeric identifiers called:

```text
UID
```

Meaning:

```text
User Identifier
```

The username is the human-friendly name.

Underneath that, Linux works heavily with numeric identities.

Conceptually:

```text
labuser
   ↓
UID 1000
```

Exact UID values vary between systems.

---

# UID 0

One UID is especially important:

```text
UID 0
```

Traditionally:

```text
UID 0
=
root / superuser
```

Check:

```bash
id root
```

You will normally see:

```text
uid=0(root)
```

So:

```text
root
→ username

0
→ UID
```

The powerful identity is fundamentally associated with UID `0`.

Modern Linux also has more granular mechanisms such as capabilities, but those belong in a later security lab.

For now:

```text
UID 0
→ root-level superuser identity
```

---

# 👥 Linux Has More Users Than Humans

Run:

```bash
cut -d: -f1 /etc/passwd
```

You may see accounts such as:

```text
root
daemon
bin
sys
mail
www-data
nobody
```

That does not mean ten people secretly live inside the server 😂.

Linux commonly creates **system/service accounts**.

A service might run under something like:

```text
www-data
```

instead of:

```text
root
```

This matters for security.

---

# 🛡️ Least Privilege

Imagine a web server.

Bad design:

```text
web server
↓
runs as root
↓
web server gets compromised
↓
attacker may inherit extremely powerful access
```

Safer design:

```text
web server
↓
dedicated restricted account
↓
only receives access it actually needs
```

This is the principle of:

```text
LEAST PRIVILEGE
```

Meaning:

> Give a user, process or service only the privileges required to perform its job.

No more than necessary.

---

# 📄 `/etc/passwd`

Linux user account information is stored in:

```text
/etc/passwd
```

Inspect:

```bash
head /etc/passwd
```

A line may look like:

```text
labuser:x:1000:1000:Lab User:/home/labuser:/bin/bash
```

There are seven colon-separated fields.

```text
labuser:x:1000:1000:Lab User:/home/labuser:/bin/bash
   │    │   │    │       │          │          │
   │    │   │    │       │          │          └── login shell
   │    │   │    │       │          └───────────── home directory
   │    │   │    │       └──────────────────────── comment / GECOS
   │    │   │    └──────────────────────────────── primary GID
   │    │   └───────────────────────────────────── UID
   │    └───────────────────────────────────────── password placeholder
   └────────────────────────────────────────────── username
```

Shortcut:

```text
username:x:UID:GID:comment:home:shell
```

---

# Why `/etc/passwd` Is Readable

Check:

```bash
ls -l /etc/passwd
```

Then:

```bash
cat /etc/passwd
```

Normal users can commonly read this file.

At first that sounds strange because the filename says:

```text
passwd
```

But modern Linux systems normally do **not** store password hashes directly in `/etc/passwd`.

The:

```text
x
```

field usually indicates that protected password information is stored elsewhere.

That place is normally:

```text
/etc/shadow
```

---

# 🔐 `/etc/shadow`

`/etc/shadow` stores sensitive authentication information.

Inspect its permissions:

```bash
ls -l /etc/shadow
```

Now try:

```bash
head /etc/shadow
```

A normal user will normally encounter:

```text
Permission denied
```

That failure is useful.

It demonstrates a real security boundary.

---

# What `/etc/shadow` Contains

It can contain information relating to:

```text
password hashes
password aging
last password change
minimum password age
maximum password age
warning periods
account expiry
```

A line has several colon-separated fields.

The exact values and hash format depend on system configuration.

The important distinction for this lab is:

```text
/etc/passwd
→ general account information


/etc/shadow
→ protected authentication information
```

---

# Reading `/etc/shadow` as Administrator

In a controlled lab:

```bash
sudo head /etc/shadow
```

may work.

That demonstrates:

```text
normal user
→ insufficient permission


sudo-authorized command
→ elevated access
```

Do not publish `/etc/shadow` contents.

Even though password hashes are not plaintext passwords, they are sensitive authentication material.

---

# `/etc/passwd` vs `/etc/shadow`

Quick reference:

```text
/etc/passwd

username
UID
GID
home
shell
account description
```

versus:

```text
/etc/shadow

password hash
password aging
expiry information
authentication-related metadata
```

---

# 👥 Groups

Groups make permissions manageable.

Imagine:

```text
alice
bob
charlie
david
```

Alice and Bob are developers.

Charlie and David are finance staff.

Instead of assigning project access individually:

```text
alice → project
bob   → project
```

Linux can use:

```text
developers group
```

Conceptually:

```text
             developers
            /          \
         alice         bob
```

Then a file can simply belong to:

```text
developers
```

and group permissions determine what members can do.

---

# GID — Group Identifier

Just as users have UIDs, groups have:

```text
GID
```

Meaning:

```text
Group Identifier
```

Conceptually:

```text
developers
    ↓
 GID 1005
```

Remember:

```text
UID
→ identifies user


GID
→ identifies group
```

---

# `/etc/group`

Group information is stored in:

```text
/etc/group
```

Inspect:

```bash
head /etc/group
```

A line may resemble:

```text
developers:x:1005:alice,bob
```

Format:

```text
group:x:GID:members
```

Breakdown:

```text
developers
→ group name


x
→ placeholder


1005
→ GID


alice,bob
→ listed supplementary members
```

---

# User Database Mental Map

At this point:

```text
/etc/passwd
→ users


/etc/group
→ groups


/etc/shadow
→ protected authentication data
```

That trio is worth remembering.

---

# Primary Groups

Every normal user has a:

```text
primary group
```

Check:

```bash
id
```

Look for:

```text
gid=
```

Example:

```text
gid=1000(labuser)
```

That represents the user's primary group.

A user's newly created files commonly receive this group as their group owner, although things such as directory configuration and setgid behaviour can influence this.

---

# Supplementary Groups

Users can belong to additional groups.

Example:

```text
labuser
│
├── primary group
│   └── labuser
│
└── supplementary groups
    ├── sudo
    ├── developers
    └── labteam
```

Check:

```bash
groups
```

or:

```bash
id
```

Supplementary group membership allows the user to receive permissions assigned to those groups.

---

# Primary vs Supplementary Groups

A simple mental model:

```text
PRIMARY GROUP
→ default/main group


SUPPLEMENTARY GROUPS
→ additional memberships
```

Important:

```text
UID
≠
group membership
```

The UID identifies the user.

GIDs identify groups.

The operating system tracks the relationship between them.

---

# Modifying Supplementary Groups

A common command:

```bash
sudo usermod -aG labteam labguest
```

Breakdown:

```text
sudo
→ execute with administrative authorization


usermod
→ modify existing user


-a
→ append


-G
→ supplementary groups


labteam
→ group to add


labguest
→ user being modified
```

So:

```bash
sudo usermod -aG labteam labguest
```

means:

> Add `labguest` to the supplementary group `labteam` while preserving existing supplementary memberships.

---

# Why `-aG` Matters

This:

```bash
sudo usermod -G labteam labguest
```

is not the same thing.

Without:

```text
-a
```

you may replace the user's supplementary group list.

Therefore:

```text
-a
→ append


-G
→ supplementary groups


-aG
→ append supplementary group membership
```

This is one of those flags worth understanding instead of copy-pasting.

---

# 🔑 `sudo`

A mistake I wanted to avoid was thinking:

```text
sudo
=
become root permanently
```

That's not what it means.

A better mental model:

```text
sudo
→ execute an authorized command as another identity
```

By default, that identity is commonly root.

Example:

```bash
sudo whoami
```

may return:

```text
root
```

But afterward:

```bash
whoami
```

still returns your normal user.

Your shell did not permanently become root.

---

# `sudo -u`

`sudo` can also run commands as another non-root user.

Example:

```bash
sudo -u labguest whoami
```

Breakdown:

```text
sudo
→ use sudo authorization


-u labguest
→ target user is labguest


whoami
→ report execution identity
```

Expected:

```text
labguest
```

Important correction:

```text
sudo -u labguest command
```

does **not** mean:

> Run as privileged labguest.

It means:

> Run the command using `labguest`'s identity and permissions.

---

# `sudo whoami` vs `sudo -u labguest whoami`

```bash
sudo whoami
```

typically:

```text
root
```

Whereas:

```bash
sudo -u labguest whoami
```

returns:

```text
labguest
```

So:

```text
sudo command
→ normally execute as root


sudo -u USER command
→ execute as specified USER
```

---

# `su`

`su` means:

```text
switch user
```

Example:

```bash
su - alice
```

attempts to start a login environment as Alice.

Without a username:

```bash
su -
```

traditionally attempts to switch to root.

Authentication rules depend on system configuration.

---

# `sudo` vs `su`

Mental distinction:

```text
sudo
→ execute an authorized command as another user


su
→ switch user/session identity
```

Another commonly used command is:

```bash
sudo -i
```

which starts a root login-style shell when permitted.

A root shell should be used deliberately rather than as the default working environment.

---

# 📦 Ownership

Every normal filesystem object has:

```text
user owner
group owner
```

Create:

```bash
touch ownership-test.txt
```

Inspect:

```bash
ls -l ownership-test.txt
```

A result may resemble:

```text
-rw-r--r-- 1 labuser labuser 0 Sep 10 14:00 ownership-test.txt
```

The important section:

```text
labuser labuser
   │       │
   │       └── group owner
   │
   └────────── user owner
```

So:

```text
FILE
│
├── user owner
└── group owner
```

---

# 🧱 Permission Classes

Linux's basic permission model separates access into:

```text
OWNER
GROUP
OTHERS
```

This is important.

It is not simply:

```text
every permission gets added together
```

Linux determines which permission class applies to the user making the request.

Conceptually:

```text
Are you the owner?
→ use owner permissions

otherwise:

Are you in the matching group?
→ use group permissions

otherwise:
→ use other permissions
```

The classes are not normally accumulated together.

---

# Reading Permission Strings

Example:

```text
-rw-r-----
```

Break it into:

```text
- | rw- | r-- | ---
    owner group others
```

The first character describes the object type.

The next nine characters are permissions.

---

# Object Type

First character:

```text
-
→ regular file


d
→ directory


c
→ character device


b
→ block device
```

Examples:

```text
-rw-r--r--
→ regular file


drwxr-xr-x
→ directory


crw-rw-rw-
→ character device
```

The last two connect back to Lab 002 and `/dev`.

---

# Permission Characters

The basic permission characters are:

```text
r
→ read


w
→ write


x
→ execute / traverse


-
→ permission absent
```

---

# Example: `-rw-r-----`

Split:

```text
-
rw-
r--
---
```

Meaning:

```text
-
→ regular file


rw-
→ owner can read + write


r--
→ group can read


---
→ others receive no permission
```

So:

```text
-rw-r-----
```

means:

```text
OWNER
read      ✅
write     ✅
execute   ❌


GROUP
read      ✅
write     ❌
execute   ❌


OTHERS
read      ❌
write     ❌
execute   ❌
```

---

# Example: `-rwxr-x---`

Split:

```text
- | rwx | r-x | ---
```

Meaning:

```text
regular file

owner
→ read
→ write
→ execute


group
→ read
→ execute


others
→ nothing
```

Once I started reading permission strings as:

```text
TYPE | OWNER | GROUP | OTHERS
```

they became much easier.

---

# 🛠️ `chmod`

`chmod` means:

```text
change mode
```

It changes permission bits.

Example:

```bash
chmod u+x hello.sh
```

means:

```text
u
→ user / owner


+
→ add


x
→ execute
```

So:

> Add execute permission for the file owner.

---

# Executable Script Example

Create:

```bash
cat > hello.sh
```

Enter:

```bash
echo "Hello from Lab 003"
```

Press:

```text
Ctrl + D
```

Inspect:

```bash
ls -l hello.sh
```

It may initially resemble:

```text
-rw-r--r--
```

Try:

```bash
./hello.sh
```

A likely result:

```text
Permission denied
```

Why?

Because the owner has:

```text
rw-
```

not:

```text
rwx
```

No execute permission exists.

---

# Fixing the Actual Problem

Instead of:

```bash
sudo ./hello.sh
```

or:

```bash
chmod 777 hello.sh
```

fix the missing permission:

```bash
chmod u+x hello.sh
```

Then:

```bash
ls -l hello.sh
```

may show:

```text
-rwxr--r--
```

Now:

```bash
./hello.sh
```

works.

Troubleshooting path:

```text
Permission denied
↓
inspect permissions
↓
execute bit missing
↓
add only execute permission
↓
verify
```

This is much better than throwing privileges at the problem.

---

# Symbolic `chmod`

Symbolic mode uses letters.

Permission classes:

```text
u
→ user / owner


g
→ group


o
→ others


a
→ all
```

Operations:

```text
+
→ add


-
→ remove


=
→ set exactly
```

Permissions:

```text
r
→ read


w
→ write


x
→ execute/traverse
```

---

# Symbolic Examples

Add owner execute:

```bash
chmod u+x script.sh
```

Add group write:

```bash
chmod g+w report.txt
```

Remove other read:

```bash
chmod o-r report.txt
```

Give everyone read:

```bash
chmod a+r report.txt
```

Set exact permissions:

```bash
chmod u=rw,g=r,o= report.txt
```

Meaning:

```text
owner
→ read + write


group
→ read


others
→ nothing
```

---

# 🔢 Numeric Permissions

Linux permissions can also be expressed numerically.

Values:

```text
r = 4

w = 2

x = 1
```

Add the values together.

---

# Permission Math

```text
rwx

4 + 2 + 1
=
7
```

```text
rw-

4 + 2
=
6
```

```text
r-x

4 + 1
=
5
```

```text
r--

4
=
4
```

```text
-wx

2 + 1
=
3
```

```text
-w-

2
=
2
```

```text
--x

1
=
1
```

```text
---

0
=
0
```

Quick map:

```text
7 → rwx
6 → rw-
5 → r-x
4 → r--
3 → -wx
2 → -w-
1 → --x
0 → ---
```

---

# Three Numbers

A command such as:

```bash
chmod 640 report.txt
```

represents:

```text
OWNER GROUP OTHERS
  6     4      0
```

Decode:

```text
6
→ 4 + 2
→ rw-


4
→ r--


0
→ ---
```

Final result:

```text
rw-r-----
```

So:

```text
owner
→ read + write


group
→ read


others
→ nothing
```

---

# Common Numeric Modes

These are useful to recognize, but understanding is more important than memorizing.

```text
600
→ rw-------
```

Owner only, read/write.

```text
640
→ rw-r-----
```

Owner read/write, group read.

```text
644
→ rw-r--r--
```

Owner read/write, everyone else read.

```text
700
→ rwx------
```

Owner only.

```text
750
→ rwxr-x---
```

Owner full, group read/traverse or execute.

```text
755
→ rwxr-xr-x
```

Owner full, others read/execute.

```text
770
→ rwxrwx---
```

Owner and group full, others none.

```text
777
→ rwxrwxrwx
```

Everyone receives all three basic permission bits.

---

# Why `chmod 777` Should Make Me Nervous

A common beginner response to:

```text
Permission denied
```

is:

```bash
chmod 777 something
```

That effectively says:

```text
OWNER
rwx


GROUP
rwx


OTHERS
rwx
```

Instead of understanding the access problem, it removes most of the basic restriction.

Better:

```text
Permission denied
↓
understand identity
↓
understand ownership
↓
understand permissions
↓
change only what is necessary
```

Least privilege applies to filesystem permissions too.

---

# 👑 `chown`

`chown` means:

```text
change owner
```

Basic syntax:

```bash
chown USER FILE
```

Example:

```bash
sudo chown alice report.txt
```

Changes the user owner.

---

# Owner and Group Together

Syntax:

```bash
chown USER:GROUP FILE
```

Example:

```bash
sudo chown alice:developers project.txt
```

Meaning:

```text
user owner
→ alice


group owner
→ developers
```

Important:

```text
chown
≠
privilege escalation
```

Changing ownership does not automatically make Alice a more privileged user globally.

It changes who owns that filesystem object.

---

# `chmod` vs `chown`

This distinction should become automatic:

```text
chmod
→ WHAT can they do?


chown
→ WHO owns it?
```

Example:

```bash
chmod 640 project.txt
```

changes:

```text
permissions
```

Whereas:

```bash
chown alice:developers project.txt
```

changes:

```text
ownership
```

---

# 📄 File Permissions

For regular files:

```text
r
→ read file contents


w
→ modify file contents


x
→ execute the file
```

Example:

```text
-rwxr-x---
```

Owner:

```text
read
write
execute
```

Group:

```text
read
execute
```

Others:

```text
nothing
```

---

# 📁 Directory Permissions

Directories behave differently.

For a directory:

```text
r
→ list directory entry names


w
→ create/delete/rename entries


x
→ traverse the directory / access entries through it
```

This was an important correction for me.

For directories:

```text
x
```

does **not** mean:

> run the directory.

It means:

> traverse through it.

---

# Directory `r`

Read permission on a directory allows names inside it to be listed.

Conceptually:

```bash
ls directory
```

depends heavily on directory read permission.

---

# Directory `w`

Write permission controls changes to directory entries.

This includes actions such as:

```text
create files
create directories
rename entries
delete entries
```

One slightly surprising point:

Deleting a file is strongly influenced by the **parent directory permissions**, because deleting a file means removing its directory entry.

This is why file write permission and file deletion are not exactly the same concept.

---

# Directory `x`

Execute permission on a directory means:

```text
traverse
```

It allows access through the directory to entries whose names are known, subject to permissions on those entries and parent directories.

This permission is involved when doing things such as:

```bash
cd directory
```

or accessing:

```text
directory/file.txt
```

---

# File vs Directory Permissions

Future-me shortcut:

```text
REGULAR FILE

r → read contents
w → modify contents
x → execute
```

```text
DIRECTORY

r → list names
w → create/delete/rename entries
x → traverse/access entries
```

This difference matters a lot.

---

# Example Directory

Consider:

```text
drwxr-x---
```

Split:

```text
d | rwx | r-x | ---
```

Meaning:

```text
d
→ directory
```

Owner:

```text
rwx
→ list
→ modify entries
→ traverse
```

Group:

```text
r-x
→ list
→ traverse
→ cannot modify directory entries
```

Others:

```text
---
→ no basic permission
```

---

# 🧪 Practical — Creating Safe Lab Identities

Create a temporary user:

```bash
sudo adduser --disabled-password --gecos "" labguest
```

Verify:

```bash
id labguest
```

And:

```bash
getent passwd labguest
```

A result may resemble:

```text
labguest:x:1001:1001::/home/labguest:/bin/bash
```

Exact IDs vary.

---

# Create a Group

```bash
sudo groupadd labteam
```

Verify:

```bash
getent group labteam
```

Add the user:

```bash
sudo usermod -aG labteam labguest
```

Verify:

```bash
id labguest
```

The output should show membership in:

```text
labteam
```

---

# Running a Command as the Lab User

```bash
sudo -u labguest whoami
```

Expected:

```text
labguest
```

Then:

```bash
sudo -u labguest id
```

This lets me test permissions from another identity without changing my entire shell.

---

# 🧪 Shared Directory Practical

Create:

```bash
mkdir /tmp/lab003-share
```

Create a file:

```bash
echo "Only the right identities should be able to change this." > /tmp/lab003-share/team.txt
```

Inspect:

```bash
ls -ld /tmp/lab003-share
ls -l /tmp/lab003-share/team.txt
```

---

# Change Group Ownership

Set the directory:

```bash
sudo chown "$USER":labteam /tmp/lab003-share
```

Set the file:

```bash
sudo chown "$USER":labteam /tmp/lab003-share/team.txt
```

Verify:

```bash
ls -ld /tmp/lab003-share
ls -l /tmp/lab003-share/team.txt
```

---

# Restrict the Directory

```bash
chmod 750 /tmp/lab003-share
```

Decode:

```text
7
→ owner rwx


5
→ group r-x


0
→ others ---
```

For a directory:

```text
owner
→ list, modify entries, traverse


group
→ list + traverse


others
→ nothing
```

---

# Restrict the File

```bash
chmod 640 /tmp/lab003-share/team.txt
```

Meaning:

```text
owner
→ read + write


group
→ read


others
→ nothing
```

Now test:

```bash
sudo -u labguest cat /tmp/lab003-share/team.txt
```

This should work because:

```text
labguest
↓
member of labteam
↓
file group = labteam
↓
group has read permission
```

---

# Test Group Write

Try:

```bash
sudo -u labguest sh -c 'echo "labguest was here" >> /tmp/lab003-share/team.txt'
```

With file mode:

```text
640
```

this should fail.

Why?

```text
group
→ r--
```

No:

```text
w
```

permission.

---

# Make the Smallest Fix

Instead of:

```bash
chmod 777 /tmp/lab003-share/team.txt
```

add only what is needed:

```bash
chmod g+w /tmp/lab003-share/team.txt
```

Now permissions become:

```text
-rw-rw----
```

Retry:

```bash
sudo -u labguest sh -c 'echo "labguest was here" >> /tmp/lab003-share/team.txt'
```

Now it should work.

This is the pattern I want to keep:

```text
identify problem
↓
identify missing permission
↓
change only that permission
↓
verify
```

---

# 🧪 Directory Write Test

Suppose the directory is:

```text
drwxr-x---
```

The group has:

```text
r-x
```

Try:

```bash
sudo -u labguest touch /tmp/lab003-share/guest-file.txt
```

This should fail because the group does not have directory:

```text
w
```

permission.

Change:

```bash
chmod 770 /tmp/lab003-share
```

Now:

```text
group
→ rwx
```

Retry:

```bash
sudo -u labguest touch /tmp/lab003-share/guest-file.txt
```

It should succeed.

That proves directory write permission controls creation of entries inside that directory.

---

# 🧨 Ownership Troubleshooting Incident

Create a deliberately root-owned file:

```bash
sudo sh -c 'echo "production=false" > /tmp/lab003-incident.conf'
```

Inspect:

```bash
ls -l /tmp/lab003-incident.conf
```

It should be owned by:

```text
root root
```

Now:

```bash
sudo chmod 600 /tmp/lab003-incident.conf
```

Permissions:

```text
rw-------
```

Meaning:

```text
owner
→ read + write


everyone else
→ nothing
```

Try:

```bash
cat /tmp/lab003-incident.conf
```

Normal user:

```text
Permission denied
```

---

# Diagnose Before Fixing

First:

```bash
whoami
```

Then:

```bash
id
```

Then:

```bash
ls -l /tmp/lab003-incident.conf
```

Reasoning:

```text
current user
≠
root


file owner
=
root


permissions
=
600


others
=
---
```

Therefore the failure makes sense.

---

# Fix Ownership Instead of Weakening Permissions

If the file is supposed to belong to the current user:

```bash
sudo chown "$USER":"$USER" /tmp/lab003-incident.conf
```

Then:

```bash
ls -l /tmp/lab003-incident.conf
```

and:

```bash
cat /tmp/lab003-incident.conf
```

Now the user can access it while mode:

```text
600
```

remains restrictive.

Important lesson:

```text
Permission denied
```

does not automatically mean:

```text
chmod problem
```

It could be:

```text
wrong owner
wrong group
missing permission
parent directory restriction
wrong identity
```

---

# 🔍 Permission Troubleshooting Workflow

This is probably the most important section for future reference.

When Linux says:

```text
Permission denied
```

start here:

```text
WHO AM I?
↓
whoami
id

↓

WHAT AM I TRYING TO ACCESS?
↓
file?
directory?
script?
device?

↓

WHO OWNS IT?
↓
ls -l FILE
ls -ld DIRECTORY

↓

WHICH PERMISSION CLASS APPLIES?
↓
owner?
group?
others?

↓

WHAT ACTION AM I ATTEMPTING?
↓
read?
write?
execute?
traverse?
create?
delete?

↓

WHICH PERMISSION IS MISSING?

↓

IS OWNERSHIP WRONG INSTEAD?

↓

MAKE THE SMALLEST CORRECT CHANGE

↓

VERIFY
```

---

# Useful Troubleshooting Commands

```bash
whoami
```

Current execution user.

```bash
id
```

UID, primary group and supplementary groups.

```bash
ls -l file
```

File ownership and permissions.

```bash
ls -ld directory
```

Directory ownership and permissions.

```bash
getent passwd username
```

Query user information through the system's account database mechanisms.

```bash
getent group groupname
```

Query group information.

```bash
groups username
```

Show group memberships.

---

# 🧠 Example Permission Decision

Suppose:

```text
-rw-r----- alice developers report.txt
```

Bob is:

```text
not alice
```

but belongs to:

```text
developers
```

Linux uses:

```text
group permissions
```

which are:

```text
r--
```

Therefore Bob can:

```text
read ✅
write ❌
execute ❌
```

---

# Another Example

Same file:

```text
-rw-r----- alice developers report.txt
```

Charlie is:

```text
not alice
```

and:

```text
not in developers
```

Linux falls through to:

```text
others
```

which are:

```text
---
```

Therefore Charlie gets no access through the basic permission bits.

---

# Important: Permissions Are Not Cumulative

Suppose Alice owns this file:

```text
----rw-r-- alice developers strange.txt
```

Even if Alice belongs to:

```text
developers
```

Linux does not normally say:

```text
owner has nothing
+
group has rw-
=
Alice gets rw-
```

Because Alice matches the **owner class**, the owner permissions apply.

This is worth remembering:

```text
owner match
→ owner bits


else group match
→ group bits


else
→ others bits
```

The classes are not simply added together.

---

# 🧠 `chmod 640` From Memory

Instead of memorizing:

```text
640
```

calculate:

```text
6
→ 4 + 2
→ rw-


4
→ r--


0
→ ---
```

Therefore:

```text
rw-r-----
```

This approach scales to any basic permission number.

---

# 🧠 Directory Permission Shortcut

For future me:

```text
FILE

r = READ IT
w = CHANGE IT
x = RUN IT
```

```text
DIRECTORY

r = SEE NAMES
w = CHANGE ENTRIES
x = GO THROUGH IT
```

Not perfect wording, but easy to remember.

---

# 🛑 Permission Mistakes I Corrected

A few things I got wrong or nearly mixed up during this lab:

### `chown` is not privilege escalation

Wrong idea:

```text
chown alice:developers file
→ escalate Alice's privileges
```

Correct:

```text
chown
→ change filesystem ownership
```

---

### `sudo -u user` does not give that user root privilege

Wrong idea:

```text
sudo -u labguest command
→ privileged labguest
```

Correct:

```text
sudo uses authorization to start the command as labguest

the command then runs using labguest's identity/permissions
```

---

### `su` is unrelated to file execute permission

This:

```bash
chmod u+x script.sh
```

changes a file permission.

This:

```bash
su - alice
```

switches user identity.

Completely different jobs.

---

### Directory `x` does not mean "run directory"

For directories:

```text
x
→ traverse
```

---

### UID does not identify groups

```text
UID
→ user


GID
→ group
```

---

### `/etc/passwd` is not where modern password hashes normally live

```text
/etc/passwd
→ general account information


/etc/shadow
→ protected authentication data
```

---

# 🧹 Lab Cleanup

After verifying the practical, remove the temporary lab account and group.

First remove temporary files/directories:

```bash
rm -rf /tmp/lab003-share
rm -f /tmp/lab003-incident.conf
```

Then remove the lab user:

```bash
sudo userdel -r labguest
```

Then remove the group if it still exists:

```bash
sudo groupdel labteam
```

Verify:

```bash
getent passwd labguest
```

and:

```bash
getent group labteam
```

No matching output means the temporary identities are gone.

Be careful with:

```bash
userdel -r
```

because:

```text
-r
```

also removes the user's home directory and mail spool where applicable.

Only use it for the disposable lab account created for this exercise.

---

# 🔐 Public Repository Safety

Do not publish raw output from commands such as:

```bash
cat /etc/shadow
```

Do not expose:

```text
real usernames
real hostname
employee/workstation identifiers
UID/GID mappings tied to real infrastructure
password hashes
SSH keys
internal group names
corporate usernames
internal paths
tokens
credentials
```

Examples in this README are deliberately generic.

---

# 📚 Command Reference

```bash
# Current user
whoami


# UID, GID and groups
id


# Inspect another user
id username


# Group memberships
groups
groups username


# User account database
getent passwd username


# Group database
getent group groupname


# Inspect passwd file
head /etc/passwd


# Inspect group file
head /etc/group


# Inspect shadow permissions
ls -l /etc/shadow


# Run command with sudo
sudo command


# Run command as another user
sudo -u username command


# Switch user
su - username


# Create user
sudo adduser username


# Create group
sudo groupadd groupname


# Append user to supplementary group
sudo usermod -aG groupname username


# Change permissions
chmod MODE file


# Add owner execute
chmod u+x script.sh


# Add group write
chmod g+w file


# Remove others read
chmod o-r file


# Numeric mode
chmod 640 file


# Change owner
sudo chown user file


# Change owner and group
sudo chown user:group file


# Inspect file permissions
ls -l file


# Inspect directory itself
ls -ld directory
```

---

# 🧪 Knowledge Check

By the end of this lab I should be able to explain these without looking them up.

```text
UID
→ numeric user identifier


GID
→ numeric group identifier


UID 0
→ root/superuser identity


/etc/passwd
→ general user account information


/etc/group
→ group information


/etc/shadow
→ protected authentication/password information


primary group
→ user's main/default group


supplementary groups
→ additional group memberships


sudo
→ execute authorized command as another identity


su
→ switch user


chown
→ change ownership


chmod
→ change permission mode


r
→ read


w
→ write


x on a file
→ execute


x on a directory
→ traverse
```

And:

```text
-rwxr-x---
```

should immediately read as:

```text
regular file

owner
→ rwx

group
→ r-x

others
→ ---
```

While:

```text
chmod 640 file
```

should immediately translate to:

```text
rw-r-----
```

---

# 🧠 The Bigger Mental Model

Everything from the first three labs is beginning to connect.

```text
WHO AM I?
↓
whoami / id


WHERE AM I?
↓
pwd


WHAT IS HERE?
↓
ls


WHO OWNS IT?
↓
ls -l


WHAT CAN THEY DO?
↓
rwx


WHO ELSE IS IN THE GROUP?
↓
id / groups


WHY WAS I DENIED?
↓
identity + ownership + permissions
```

This is where Linux is starting to feel less like memorizing commands and more like reasoning about a system.

---

# 🧩 What Changed After This Lab

Before this lab:

```text
-rwxr-x---
```

looked like:

```text
Linux password-looking nonsense 😂
```

Now I read it as:

```text
type
│
├── owner permissions
├── group permissions
└── others permissions
```

Before:

```text
Permission denied
↓
maybe sudo?
```

Now:

```text
Permission denied
↓
whoami
↓
id
↓
ls -l / ls -ld
↓
owner?
group?
others?
↓
what permission is required?
↓
fix the smallest thing necessary
```

That's probably the biggest result of this lab.

---

# ✅ Lab Result

```text
LAB 003 — Users, Groups & Permissions

Theory                 : COMPLETE
Users & UIDs            : COMPLETE
Groups & GIDs           : COMPLETE
passwd/group/shadow     : COMPLETE
sudo & su               : COMPLETE
Ownership               : COMPLETE
Symbolic chmod           : COMPLETE
Numeric chmod            : COMPLETE
Directory permissions   : COMPLETE
Troubleshooting         : COMPLETE
Hands-on practical      : COMPLETE
Live knowledge check    : PASSED WITH CORRECTIONS
Corrections reviewed    : COMPLETE

STATUS                   : COMPLETE ✅
```

---

# 🛣️ Linux Administration Progress

```text
Linux Administration
│
├── Lab 001 — Linux Orientation
│   └── COMPLETE ✅
│
├── Lab 002 — Linux Filesystem & Structure
│   └── COMPLETE ✅
│
├── Lab 003 — Users, Groups & Permissions
│   └── COMPLETE ✅
│
└── Lab 004 — Processes
    └── NEXT
```

---

# Next — Lab 004: Processes

Next we move from:

```text
who is allowed to do what?
```

to:

```text
what is actually running?
```

Lab 004 will cover concepts such as:

```text
processes
PIDs
parent processes
PPIDs
foreground/background processes
ps
ps aux
top
htop
pgrep
kill
signals
jobs
bg
fg
nice
renice
/proc/<PID>
zombies
process troubleshooting
resource usage
```

This is where commands we briefly touched earlier, such as:

```bash
ps
```

and:

```bash
top
```

finally get understood properly.

---

> Lab 001 taught me how to move around Linux.
>
> Lab 002 taught me where things live.
>
> Lab 003 taught me why Linux sometimes tells me **no**.

**Lab 003 complete. 🐧**
