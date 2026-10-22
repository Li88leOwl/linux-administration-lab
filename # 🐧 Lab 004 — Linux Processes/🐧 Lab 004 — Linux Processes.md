# 🐧 Lab 004 — Linux Processes

> A Linux system is never really "doing nothing." Even when the screen looks quiet, processes are being created, scheduled, paused, resumed, put to sleep, terminated, and monitored underneath.

---

## Why this lab exists

The previous labs built the foundation for understanding a Linux machine. Lab 001 covered moving around the system and interacting with the shell. Lab 002 covered the filesystem and explained where Linux keeps configuration, logs, runtime information, devices, temporary data, and process information. Lab 003 introduced users, groups, ownership, permissions, `sudo`, and the idea that Linux constantly evaluates **who is attempting an action and whether that identity should be allowed to perform it**.

Lab 004 moves into something more dynamic:

> **What is actually running on the machine right now?**

This matters because almost everything useful on a Linux machine eventually becomes a process. A shell is a process. A web server is made of processes. SSH sessions involve processes. Database servers run as processes. Scripts become processes when executed. Monitoring agents, backup jobs, security tools, schedulers, application workers and many other components all exist as running processes.

This means that when a machine becomes slow, an application freezes, CPU usage suddenly rises, memory keeps growing, a command refuses to stop, or a service behaves strangely, process investigation becomes one of the first administrator skills required.

The goal of this lab was therefore not simply to memorize commands such as `ps`, `top`, `kill`, `jobs`, `bg`, and `fg`. The real goal was to learn how to answer questions such as:

```text
What process is running?
Who started it?
Who owns it?
What is its PID?
What process created it?
What state is it in?
How much CPU or memory is it using?
Is it actually broken?
Should it be stopped?
What signal should be sent?
What should be investigated before killing it?
```

By the end of this lab, the aim is to look at process information and begin **reasoning about the system**, rather than seeing a wall of numbers.

---

# Lab Environment

This lab was performed inside an isolated Linux learning environment using Bash.

```text
Environment : Linux administration lab
Shell       : Bash
Purpose     : Process administration and troubleshooting
Identity    : Standard user with authorized sudo access
```

Machine-specific information such as the actual username, hostname, workstation name, IP address and organization-related identifiers is intentionally excluded from this public repository.

All examples use generic process names and users such as:

```text
user1
appuser
worker
linux-lab
```

---

# Programs vs Processes

A program stored on disk is not the same thing as a process.

For example:

```text
/usr/bin/python3
```

is an executable program stored in the filesystem.

When it is executed:

```bash
python3
```

Linux creates a running instance of that program. That running instance is a **process**.

A useful mental model is:

```text
PROGRAM
stored on disk
     │
     │ executed
     ▼
PROCESS
running instance of the program
```

The same program can have multiple processes at the same time. Running:

```bash
sleep 300
```

in several terminals creates several different processes even though every process came from the same `sleep` program.

This distinction becomes important because administrators normally troubleshoot the **running process**, not merely the executable file stored on disk.

---

# PID — Process Identifier

Every process receives a number called a:

```text
PID
```

which means:

```text
Process Identifier
```

This follows the same general idea already seen with users and groups:

```text
UID → identifies a user
GID → identifies a group
PID → identifies a process
```

A process might have:

```text
PID 5124
```

while another process might have:

```text
PID 5188
```

Those numbers allow Linux and administrators to refer to specific running processes.

PIDs are not permanent identifiers. A PID belongs to a process while that process exists. After the process exits, that PID can eventually be reused by the system.

---

# The Shell Is Also a Process

One of the useful realizations in this lab was that Bash itself is simply another process.

In Bash:

```bash
echo $$
```

prints the PID of the current shell.

Example:

```text
4200
```

The shell can then be inspected directly:

```bash
ps -p $$
```

or with selected output fields:

```bash
ps -o pid,ppid,user,stat,cmd -p $$
```

A result could resemble:

```text
PID   PPID  USER   STAT  CMD
4200  4100  user1  Ss    bash
```

This means Bash itself has a PID, a parent, an owner and a process state just like other programs.

This also connects directly back to the virtual `/proc` filesystem introduced in Lab 002:

```bash
ls /proc/$$
```

Because `/proc/<PID>` exposes live information about a running process, `/proc/4200` would contain information associated with process 4200.

---

# Parent and Child Processes

Processes can create other processes. When one process creates another, the process that existed first is the **parent**, while the newly created process becomes the **child**.

Linux records this relationship using:

```text
PID
→ Process Identifier

PPID
→ Parent Process Identifier
```

For example:

```text
PID   PPID  CMD
5124  4200  sleep 300
```

means:

```text
PID 5124
→ sleep process

PPID 4200
→ the process that created it
```

The relationship can be pictured as:

```text
PID 4200
bash
  │
  └── PID 5124
      sleep 300
```

This becomes important during troubleshooting because a suspicious worker process might not be the real source of the issue. It could simply be one child belonging to a larger parent service.

If PID `7500` is consuming a large amount of CPU, inspecting:

```bash
ps -p 7500 -o pid,ppid,user,stat,%cpu,%mem,cmd
```

tells us what the process is, while the PPID tells us which parent should also be investigated.

The parent can then be checked using:

```bash
ps -p <PPID> -o pid,ppid,user,stat,%cpu,%mem,cmd
```

and children of a known process can be queried using:

```bash
ps --ppid 7500 -o pid,ppid,user,stat,%cpu,%mem,cmd
```

or:

```bash
pgrep -P 7500
```

The important lesson is that process troubleshooting often involves understanding the **process tree**, not just staring at one PID.

---

# Understanding `ps`

`ps` stands for:

```text
process status
```

Running:

```bash
ps
```

shows processes associated with the current terminal/session according to the command's default selection rules.

A simple output might resemble:

```text
PID   TTY      TIME     CMD
4200  pts/0    00:00:00 bash
5300  pts/0    00:00:00 ps
```

An interesting detail is that `ps` may display itself because while it is gathering process information, `ps` is temporarily a running process too.

A useful distinction from earlier labs is:

```text
pwd
→ where am I in the filesystem?

ps
→ what processes are running?
```

---

# `ps aux`

A common process inspection command is:

```bash
ps aux
```

This produces a broad process listing with fields such as:

```text
USER
PID
%CPU
%MEM
VSZ
RSS
TTY
STAT
START
TIME
COMMAND
```

The most immediately useful fields during basic administration are:

| Field | Meaning |
|---|---|
| `USER` | user running/owning the process |
| `PID` | process identifier |
| `%CPU` | CPU usage |
| `%MEM` | percentage of system memory used |
| `STAT` | process state |
| `COMMAND` | command/process being executed |

`ps aux` is therefore useful when an administrator wants a broad snapshot of what exists on the machine.

A useful mental comparison is:

```text
ls -lah
→ detailed inspection of filesystem entries

ps aux
→ detailed inspection of processes
```

The commands are unrelated internally, but as a mental shortcut the comparison is useful: one helps inspect the filesystem while the other helps inspect processes.

---

# `ps -e` and `ps -o`

Another extremely useful style is:

```bash
ps -eo pid,ppid,user,stat,%cpu,%mem,cmd
```

Here:

```text
-e
→ show every process

-o
→ define the output format
```

The `-o` does **not** mean ownership.

It means:

```text
output format
```

Therefore:

```bash
ps -o pid,cmd
```

means:

> Show the PID and command columns.

And:

```bash
ps -eo pid,ppid,user,stat,%cpu,%mem,cmd
```

means:

> Show all processes, but only display the columns PID, PPID, USER, STAT, CPU usage, memory usage and command.

This is useful during real troubleshooting because sometimes `ps aux` contains more information than is needed.

Instead of reading every available field, the administrator can request exactly the information relevant to the investigation.

---

# Finding a Particular Process with `pgrep`

Instead of manually scanning a long process table:

```bash
pgrep sleep
```

returns PIDs matching the process name.

Adding:

```bash
pgrep -a sleep
```

also displays the associated command line.

Example:

```text
5124 sleep 300
5188 sleep 500
5260 sleep 900
```

This tells us that three different `sleep` processes exist.

It does **not** show their process states. If their states are required, the PIDs can then be inspected with:

```bash
ps -o pid,ppid,user,stat,cmd -p 5124,5188,5260
```

A useful distinction is:

```text
pgrep
→ locate matching processes

ps
→ inspect process information
```

---

# `ps aux | grep`

Another common pattern is:

```bash
ps aux | grep nginx
```

The `ps aux` command produces the process list, the pipe `|` passes that output to `grep`, and `grep nginx` shows lines containing the text `nginx`.

This technique is extremely common, although it can also display the `grep nginx` process itself.

For simple name-based process discovery:

```bash
pgrep -a nginx
```

is often cleaner.

Pipes will be covered much more deeply in the Bash section later in the learning path.

---

# `ps aux` vs `top`

This was one of the concepts that required reinforcement during the live questions.

The most useful distinction is:

```text
ps aux
→ snapshot

top
→ live monitoring
```

`ps aux` gathers process information, prints the result and exits. It is like taking a photograph of the process table.

`top` continuously refreshes its display, allowing process behaviour to be watched over time.

The mental model is:

```text
ps aux
→ photograph

top
→ CCTV/live feed
```

This distinction matters when investigating changing resource usage.

If the administrator wants to answer:

> What processes exist right now?

then:

```bash
ps aux
```

may be enough.

If the question becomes:

> Is this process continuously consuming CPU or memory?

then:

```bash
top
```

is usually more useful because behaviour can be observed over time.

---

# Using `top`

Run:

```bash
top
```

Common information includes system load, task counts, CPU usage, memory usage, swap information and a live process list.

Important process columns include:

```text
PID
USER
S
%CPU
%MEM
COMMAND
```

Press:

```text
q
```

to exit.

At this stage the goal is not to memorize every field displayed by `top`. The important thing is learning to identify the process, owner, state and resource consumption.

---

# Why Trends Matter More Than One Snapshot

Suppose a process shows:

```text
%MEM = 82
```

A single observation does not automatically prove something is wrong.

The application might intentionally maintain a large cache or load a large dataset into memory.

But suppose ten minutes later:

```text
%MEM = 91
```

and later:

```text
%MEM = 96
```

while the workload has not meaningfully changed.

Now the **trend** becomes important.

Possible explanations could include:

```text
memory leak
growing application cache
increasing workload
large accumulating dataset
objects/resources not being released
```

The correct response is not immediately:

```text
memory leak confirmed
```

but rather:

```text
memory is continuously growing
↓
observe over time
↓
compare with workload
↓
determine whether memory drops again
↓
investigate application behaviour
```

The same principle applies to CPU usage.

A short CPU spike may be normal.

A process consuming 99% CPU for hours without a known reason deserves investigation.

---

# CPU-Heavy vs Memory-Heavy Processes

Consider:

```text
%CPU = 95
%MEM = 1.2
```

This suggests primarily:

```text
CPU pressure
```

The process may be computationally intensive, performing calculations, compression, encryption, encoding, data processing or simply looping excessively.

It does **not** automatically mean more threads are required. Adding additional threads to a CPU-saturated workload may actually increase contention.

Now consider:

```text
%CPU = 2
%MEM = 78
```

This suggests a much more:

```text
memory-heavy workload
```

Again, that does not automatically mean the program is broken.

The administrator should determine whether that memory usage is expected and whether it remains stable or continues increasing.

A useful first mental model is:

```text
high CPU + low memory
→ CPU-heavy

low CPU + high memory
→ memory-heavy

high CPU + high memory
→ heavy pressure on both resources
```

---

# Process States

The `STAT` field gives information about the process state.

The core states introduced in this lab are:

| State | Meaning |
|---|---|
| `R` | running or runnable |
| `S` | sleeping/waiting normally |
| `D` | uninterruptible sleep |
| `T` | stopped/suspended |
| `Z` | zombie |

These states should eventually become familiar enough that seeing them immediately influences the next troubleshooting decision.

---

# `R` — Running or Runnable

Example:

```text
PID   USER     STAT  %CPU  CMD
7400  appuser  R     95.0  worker
```

`R` means that the process is either actively executing on a CPU or is runnable and waiting to receive CPU time.

It does not automatically indicate a problem.

A healthy process can be in `R`.

The additional information matters:

```text
How long has CPU usage been high?
What is the process doing?
Is the workload expected?
Is the system actually suffering?
```

---

# `S` — Sleeping

Example:

```text
PID   USER     STAT  %CPU  CMD
7300  appuser  S     0.0   worker
```

`S` usually means the process is sleeping or waiting for an event.

This is normal.

Many healthy processes spend most of their lifetime sleeping because there is no reason to consume CPU when there is no work available.

A server application may simply be waiting for:

```text
network input
user input
a timer
a request
another event
```

Therefore:

```text
S
≠
broken
```

An important point from the live exercises is that state and resource usage should be interpreted separately.

If a process displays:

```text
STAT = S
%CPU = 99
```

this does not necessarily contradict itself.

`STAT` represents the state captured at the moment `ps` sampled the process, while `%CPU` reflects CPU usage over some measurement interval.

The process could have consumed large amounts of CPU and then happened to be sleeping when sampled.

---

# `D` — Uninterruptible Sleep

Example:

```text
PID   USER     STAT  CMD
7200  appuser  D     worker
```

`D` means:

```text
uninterruptible sleep
```

This often occurs while a process is waiting for kernel-level I/O activity.

Possible examples include:

```text
disk I/O
storage device response
network filesystem activity
certain kernel operations
```

A process in `D` deserves careful investigation because even:

```bash
kill -9 PID
```

may not make the process disappear immediately.

The process may remain until the kernel operation it is waiting for completes.

Therefore the correct question is usually:

> What is this process waiting on?

rather than:

> How hard can I kill it?

---

# `T` — Stopped or Suspended

Example:

```text
PID   USER     STAT  CMD
7100  appuser  T     worker
```

`T` means the process has been stopped/suspended.

It still exists.

It has not necessarily crashed and it has not necessarily terminated.

Possible causes include job control or stop signals such as:

```text
Ctrl + Z
SIGTSTP
SIGSTOP
debugger interaction
```

This distinction was important during the live test because:

```text
T
≠ terminated
```

A stopped process might simply need to be resumed rather than killed.

The administrator should therefore investigate why it was stopped before deciding what action to take.

---

# `Z` — Zombie

Example:

```text
PID   PPID  USER     STAT  CMD
8300  8200  appuser  Z     worker
```

A zombie process sounds worse than it usually is.

A zombie is a process that has already exited, but its parent process has not yet collected the child's exit status.

The sequence is roughly:

```text
child process runs
↓
child exits
↓
kernel keeps a small process-table record
↓
parent should collect child's exit status
↓
record disappears
```

If the parent does not perform that final step properly:

```text
Z
→ zombie
```

The child is already finished, so trying to repeatedly kill the zombie PID is not normally the solution.

The more important investigation is usually the parent:

```bash
ps -p 8200 -o pid,ppid,user,stat,cmd
```

because PPID `8200` is the process that has not reaped its child.

A small temporary number of zombies is not automatically catastrophic, but large or persistent zombie accumulation can point toward buggy parent-process behaviour.

---

# Foreground Processes

Running:

```bash
sleep 300
```

starts `sleep` in the foreground.

The terminal appears unavailable because Bash waits while the foreground command runs.

Conceptually:

```text
Bash
↓
sleep 300
↓
Bash waits
```

Once `sleep` exits, Bash provides the prompt again.

A foreground process receives terminal interaction directly.

---

# What `sleep 300` Actually Means

The `sleep` command was used repeatedly during the lab because it creates a harmless process that remains alive long enough to inspect.

```bash
sleep 300
```

means:

> Start a process that waits for 300 seconds before exiting.

Since:

```text
300 seconds
=
5 minutes
```

the process remains alive for five minutes unless interrupted.

It is useful for labs because it consumes very little CPU while still appearing in process tables.

---

# Background Processes and `&`

Running:

```bash
sleep 300 &
```

changes the behaviour.

The ampersand:

```text
&
```

asks Bash to start the command as a background job.

Therefore:

```text
sleep 300
→ foreground

sleep 300 &
→ background
```

When started in the background, Bash immediately returns the shell prompt so additional commands can be executed while `sleep` continues.

The shell might display:

```text
[1] 5124
```

These numbers represent two completely different concepts:

```text
[1]
→ shell job number

5124
→ process PID
```

The job number belongs to Bash's job-control system.

The PID belongs to Linux's process system.

This distinction became one of the main concepts of the lab:

```text
job
→ shell concept

process
→ operating-system concept
```

---

# `$!` — Most Recent Background PID

After:

```bash
sleep 300 &
```

Bash provides a special variable:

```bash
$!
```

which expands to:

> the PID of the most recently started background process.

Example:

```bash
sleep 300 &
echo $!
```

might return:

```text
5124
```

This can be stored in a variable:

```bash
sleep 300 &
PID=$!
```

Now:

```bash
echo "$PID"
```

prints the stored PID.

The variable can then be reused:

```bash
ps -p "$PID"
kill "$PID"
```

A key limitation is that `$!` belongs to the current shell context.

If the terminal is closed and a completely new shell is opened later, that new shell does not remember the previous shell's `$!`.

To find the process later, use normal process discovery:

```bash
pgrep -a sleep
```

---

# `jobs`

The command:

```bash
jobs
```

shows jobs tracked by the current shell.

Example:

```text
[1]+  Running    sleep 300 &
[2]-  Stopped    sleep 500
```

The numbers:

```text
[1]
[2]
```

are job numbers.

The symbols:

```text
+
-
```

also have meaning:

```text
+
→ current/default job

-
→ previous job
```

Therefore:

```bash
fg
```

without a job number normally operates on the job marked:

```text
+
```

while:

```bash
fg %2
```

explicitly selects job number 2.

---

# `Ctrl + Z`, `bg` and `fg`

Run:

```bash
sleep 300
```

and then press:

```text
Ctrl + Z
```

The process is not killed.

Instead, the foreground job is suspended.

Bash may display:

```text
Stopped
```

and:

```bash
jobs
```

will show the suspended job.

`Ctrl + Z` normally causes a terminal-generated stop signal such as:

```text
SIGTSTP
```

The process can then be continued in the background:

```bash
bg
```

or brought into the foreground:

```bash
fg
```

The complete mental model is:

```text
sleep 300
   │
   ▼
FOREGROUND
   │
Ctrl + Z
   │
   ▼
STOPPED
  / \
 /   \
bg   fg
│     │
▼     ▼
BACKGROUND   FOREGROUND
```

This means:

```text
Ctrl + Z
→ suspend

bg
→ continue in background

fg
→ continue/bring into foreground
```

---

# `jobs` vs `ps`

This distinction is important enough to keep explicitly:

```text
jobs
→ what the current shell is tracking

ps
→ operating-system processes
```

A process that exists on Linux is not necessarily visible in the current shell's `jobs` output.

For example, a process launched from another terminal or managed by a service manager does not become a job in your current Bash session.

`jobs` is therefore about **shell job control**, while `ps` is about **process inspection**.

---

# Signals

Linux uses signals as a mechanism for notifying processes that something has happened or requesting certain behaviour.

The main signals introduced in this lab were:

| Signal | Common meaning |
|---|---|
| `SIGINT` | interrupt |
| `SIGTERM` | request clean termination |
| `SIGKILL` | force termination |
| `SIGSTOP` | force process to stop |
| `SIGTSTP` | terminal-generated stop |
| `SIGCONT` | continue a stopped process |
| `SIGHUP` | hangup/session-related signal |

Understanding the difference between these signals is more important than memorizing every numeric value immediately.

---

# `Ctrl + C` — SIGINT

When:

```text
Ctrl + C
```

is pressed while a foreground command is running, the terminal normally sends:

```text
SIGINT
```

This means:

```text
interrupt
```

The process may respond by terminating.

Therefore the mental shortcut is:

```text
Ctrl + C
→ SIGINT
```

---

# `kill PID` — SIGTERM by Default

The `kill` command name is slightly misleading because it is actually a signal-sending command.

Running:

```bash
kill 5124
```

normally sends:

```text
SIGTERM
```

not `SIGINT`.

This was one of the main concepts corrected during the live knowledge check.

The shortcut to remember is:

```text
kill PID
→ SIGTERM
```

`SIGTERM` is essentially a polite request:

> Please terminate cleanly.

Applications can receive the signal and perform cleanup before exiting.

Possible cleanup includes:

```text
closing files
flushing buffered data
releasing resources
shutting down workers
saving state
```

This is why SIGTERM should normally be preferred before SIGKILL.

---

# `kill -9` — SIGKILL

Running:

```bash
kill -9 5124
```

sends:

```text
SIGKILL
```

SIGKILL is handled directly by the kernel.

The target process cannot catch it or ignore it.

That makes it extremely effective, but also much more aggressive.

The process does not receive an opportunity to perform normal application cleanup.

The preferred troubleshooting sequence is therefore:

```text
identify process
↓
understand process
↓
SIGTERM
↓
verify whether it exits
↓
investigate if it does not
↓
SIGKILL only when appropriate
```

Not:

```text
process looks weird
↓
kill -9 😂
```

---

# Inspect Before Killing

One of the most important habits developed during this lab was:

> **Inspect first, understand second, act third.**

Suppose:

```text
PID   PPID  USER     STAT  %CPU  %MEM  CMD
7500  1100  appuser  R     98.7  1.2   worker
```

Seeing `98.7% CPU` does not automatically justify:

```bash
kill -9 7500
```

Before terminating it, useful questions include:

```text
What process is this?
Who owns it?
What spawned it?
Is it a child of a critical service?
Is this workload expected?
Has CPU been high for seconds or hours?
Are other workers behaving the same way?
Is memory also increasing?
Will terminating this process affect users?
```

A useful inspection command is:

```bash
ps -p 7500 -o pid,ppid,user,stat,%cpu,%mem,cmd
```

Then the PPID can be inspected.

The larger lesson is that a PID is not simply something to kill. It represents something happening inside the operating system, and an administrator should understand that context before taking destructive action.

---

# Process Ownership

Processes execute under user identities.

The `USER` field in process output answers:

```text
Which user is this process running as?
```

This connects directly to Lab 003.

A process running as:

```text
appuser
```

receives permissions associated with that identity.

This matters because if an application process cannot read or write a file, process investigation may reveal which user is actually attempting the operation.

For example:

```text
application fails writing /var/log/myapp/app.log
```

could require information from several labs:

```text
Lab 002
→ understand /var/log

Lab 003
→ ownership and permissions

Lab 004
→ determine which user the process runs as
```

This is the beginning of real Linux troubleshooting: individual labs start combining into one incident.

---

# `/proc/<PID>`

The `/proc` filesystem provides live process information.

Start a harmless process:

```bash
sleep 600 &
PID=$!
```

Then inspect:

```bash
ls /proc/"$PID"
```

As long as the process exists, Linux exposes a directory associated with that PID.

When the process terminates, its normal `/proc/<PID>` directory disappears.

---

# `/proc/<PID>/status`

Run:

```bash
head -n 20 /proc/"$PID"/status
```

Fields can include:

```text
Name
State
Pid
PPid
Uid
Gid
```

This connects several Linux concepts together:

```text
Name
→ process name

State
→ process state

Pid
→ process identifier

PPid
→ parent process

Uid
→ user identity

Gid
→ group identity
```

This is one reason `/proc` is so important to Linux administration: it exposes live kernel and process information as filesystem-like entries.

---

# `/proc/<PID>/cmdline`

The command used to start a process can be inspected with:

```bash
cat /proc/"$PID"/cmdline
```

The result may look strange because command arguments are separated using null characters rather than normal spaces.

A friendlier representation is:

```bash
tr '\0' ' ' < /proc/"$PID"/cmdline
echo
```

For the lab process, this might display:

```text
sleep 600
```

---

# `/proc/<PID>/cwd`

Every process has a current working directory.

For the current shell:

```bash
ls -l /proc/$$/cwd
```

`cwd` means:

```text
current working directory
```

If:

```bash
cd /tmp
```

is executed and the command repeated:

```bash
ls -l /proc/$$/cwd
```

the link changes.

This demonstrates that the current working directory is actually part of the process state.

---

# `/proc/<PID>/exe`

The executable associated with a process can be inspected using:

```bash
ls -l /proc/$$/exe
```

For Bash, this normally points toward the Bash executable.

The important `/proc` mental map from this lab is therefore:

```text
/proc/PID/status
→ process metadata

/proc/PID/cmdline
→ startup command line

/proc/PID/cwd
→ current working directory

/proc/PID/exe
→ executable

/proc/PID/fd
→ open file descriptors
```

---

# File Descriptors

Processes interact with files and other resources using file descriptors.

Inspect the current shell:

```bash
ls -l /proc/$$/fd
```

Three especially important file descriptors are:

```text
0 → standard input
1 → standard output
2 → standard error
```

These will become much more important when Bash, redirection and pipes are covered later.

For now the important idea is simply:

> Processes maintain references to resources they have opened.

---

# `nohup`

A normal shell background job may still have a relationship with the shell/session that created it.

When the terminal session disappears, processes associated with it may receive:

```text
SIGHUP
```

which means:

```text
hangup
```

The command:

```bash
nohup command &
```

is commonly used when the intention is for a simple process to continue even after the launching terminal disconnects.

For example:

```bash
nohup sleep 300 &
```

combines two separate behaviours:

```text
nohup
→ protect against normal SIGHUP behaviour

&
→ run in the background
```

The name literally comes from:

```text
no hangup
```

A process started with `nohup` is therefore more likely to survive the terminal session disappearing than a normal interactive background job.

`nohup` may also redirect output to:

```text
nohup.out
```

when suitable output redirection has not been provided.

For real long-running production applications, however, administrators normally prefer a proper service manager such as `systemd`. That becomes the topic of the next lab.

---

# `disown`

Bash also provides:

```bash
disown
```

for removing jobs from the shell's job table and altering their relationship with the shell depending on how it is used.

This was introduced only as context.

The major takeaway for this lab is:

```text
background
≠
independent service
```

Running:

```bash
command &
```

does not magically turn that command into a properly managed daemon.

Long-running production workloads should generally use a process/service manager.

---

# Nice Values

Linux scheduling behaviour can be influenced using a concept called:

```text
nice value
```

The common nice-value range is:

```text
-20 to 19
```

The numbering initially feels backwards:

```text
lower number
→ higher scheduling priority

higher number
→ lower scheduling priority
```

Typical interpretation:

```text
-20 → very high priority
0   → normal/default
19  → very low priority
```

A useful mental trick is:

> The higher the nice value, the "nicer" the process is to other processes because it is willing to receive less favourable CPU scheduling priority.

Start a process with a changed nice value:

```bash
nice -n 10 sleep 300
```

Inspect:

```bash
ps -o pid,ni,cmd -C sleep
```

Modify an existing process:

```bash
renice 10 -p PID
```

Changing a process toward a more favourable scheduling priority can require elevated privilege.

This lab only introduced the concept; advanced Linux scheduling will be revisited later.

---

# PID 1

Run:

```bash
ps -p 1 -o pid,ppid,user,comm,args
```

PID 1 has a special position in the process hierarchy.

On many modern Linux systems:

```text
PID 1
→ systemd
```

However, environments such as containers, WSL and specialized systems may have different PID 1 arrangements.

The important concept for now is that PID 1 occupies a special position as the first userspace process and has important process-management responsibilities.

`systemd` will be covered properly in Lab 005.

---

# Troubleshooting High CPU

Suppose users report:

```text
"The server is slow."
```

A bad first response is:

```text
restart everything
```

A better starting point is:

```bash
top
```

or:

```bash
ps -eo pid,ppid,user,stat,%cpu,%mem,cmd
```

If one process shows:

```text
%CPU = 98
```

the investigation becomes:

```text
Which process is this?
What command is it running?
Who owns it?
What is its parent?
Is it one worker or part of a larger process tree?
How long has CPU usage been high?
Is high CPU expected for the current workload?
Is the machine actually CPU-starved?
```

Only after understanding the context should termination or service-level action be considered.

---

# Troubleshooting High Memory

Suppose:

```text
%MEM = 82
```

and later:

```text
%MEM = 91
```

then later:

```text
%MEM = 96
```

The question becomes:

> Why is memory continuing to grow?

Potential areas of investigation include increasing workload, cache behaviour, application design, data accumulation or memory leaks.

The important habit is to monitor behaviour across time rather than treating one sample as the entire story.

---

# Troubleshooting a Stopped Process

Suppose:

```text
STAT = T
```

Do not immediately assume the process should be killed.

Instead investigate why the process has been suspended.

It may have received:

```text
SIGSTOP
SIGTSTP
```

or may simply have been stopped interactively with:

```text
Ctrl + Z
```

The appropriate response might be to resume the process rather than terminate it.

---

# Troubleshooting a Zombie

Suppose:

```text
PID   PPID  USER     STAT  CMD
8300  8200  appuser  Z     worker
```

The wrong instinct is:

```bash
kill -9 8300
```

The process is already dead.

The more useful action is to inspect:

```text
PPID 8200
```

because the parent is responsible for collecting the child's exit information.

Example:

```bash
ps -p 8200 -o pid,ppid,user,stat,cmd
```

The real question becomes:

> Why is the parent failing to reap its child?

---

# Troubleshooting a `D` State

Suppose:

```text
STAT = D
```

and the process refuses to disappear even after signals are sent.

This may indicate the process is blocked in uninterruptible kernel sleep, often involving I/O.

The correct investigation becomes:

```text
What resource is it waiting for?
Is storage responding?
Is a network filesystem stuck?
Is there an underlying kernel/device issue?
```

Again, the process state changes the troubleshooting approach.

---

# Process Investigation Workflow

This is the main workflow worth carrying forward from the lab:

```text
SYSTEM FEELS WRONG
        │
        ▼
WHAT IS RUNNING?
ps / top
        │
        ▼
WHICH PROCESS?
PID
        │
        ▼
WHO RUNS IT?
USER
        │
        ▼
WHAT CREATED IT?
PPID
        │
        ▼
WHAT STATE?
R / S / D / T / Z
        │
        ▼
WHAT RESOURCES?
CPU / memory
        │
        ▼
WHAT COMMAND?
CMD / /proc/PID/cmdline
        │
        ▼
IS THIS EXPECTED?
        │
        ▼
WHAT IS THE LEAST DESTRUCTIVE ACTION?
        │
        ▼
SIGTERM first when termination is appropriate
        │
        ▼
VERIFY
        │
        ▼
SIGKILL only when genuinely necessary
```

This is much more valuable than memorizing:

```text
kill -9
```

because infrastructure work is largely about understanding state before changing state.

---

# Practical Lab

The practical created a harmless process playground using `sleep`.

Start from the home directory:

```bash
cd ~
mkdir -p ~/lab004
cd ~/lab004
```

Inspect the current shell:

```bash
echo $$
ps -p $$
ps -o pid,ppid,user,stat,cmd -p $$
```

Start a background process:

```bash
sleep 600 &
```

Capture its PID:

```bash
PID=$!
echo "$PID"
```

Inspect it:

```bash
ps -p "$PID" -o pid,ppid,user,stat,%cpu,%mem,cmd
pgrep -a sleep
```

Inspect `/proc`:

```bash
head /proc/"$PID"/status

tr '\0' ' ' < /proc/"$PID"/cmdline
echo

ls -l /proc/"$PID"/exe
```

Inspect shell jobs:

```bash
jobs
```

Bring the job to the foreground:

```bash
fg
```

Suspend:

```text
Ctrl + Z
```

Check:

```bash
jobs
```

Resume in background:

```bash
bg
```

Check again:

```bash
jobs
```

Terminate gracefully:

```bash
kill "$PID"
```

Verify:

```bash
ps -p "$PID"
```

The absence of the process confirms that it exited.

---

# Practical `/proc` Inspection

Inspect the shell's working directory:

```bash
ls -l /proc/$$/cwd
```

Change location:

```bash
cd /tmp
```

Check again:

```bash
ls -l /proc/$$/cwd
```

Return:

```bash
cd ~/lab004
```

Inspect executable:

```bash
ls -l /proc/$$/exe
```

Inspect process status:

```bash
head /proc/$$/status
```

Inspect file descriptors:

```bash
ls -l /proc/$$/fd
```

This practical connected the shell, process table, filesystem and `/proc` together.

---

# Mistakes and Corrections From This Lab

Several concepts required correction during the live exercises, and these are worth documenting because they are exactly the kinds of misunderstandings future review should target.

`kill PID` sends `SIGTERM` by default, not `SIGINT`.

```text
Ctrl + C
→ SIGINT

kill PID
→ SIGTERM

kill -9 PID
→ SIGKILL
```

`T` means stopped/suspended, not terminated.

```text
T
→ process still exists
→ process is paused
```

`$!` does not list background processes.

```text
$!
→ PID of the most recently started background process
```

`jobs` shows shell-managed jobs, while `ps` shows operating-system processes.

```text
jobs
→ shell

ps
→ OS process table
```

`nohup` does not prevent a process from becoming stale. It helps protect the process from hangup behaviour associated with the launching session disappearing.

```text
nohup
→ no hangup
→ commonly protects against SIGHUP
```

`ps aux` and `top` are not simply "more detailed" versus "less detailed".

```text
ps aux
→ process snapshot

top
→ continuously refreshing view
```

High CPU with low memory indicates CPU pressure rather than automatically meaning more threads are required.

Zombie processes are already exited. The parent is usually the more important process to investigate.

These corrections are intentionally kept in the README because they document the reasoning process rather than pretending everything was immediately obvious.

---

# Command Reference

```bash
# Current shell PID
echo $$

# Latest background process PID
echo $!

# Basic process view
ps

# Broad process snapshot
ps aux

# All processes with selected fields
ps -eo pid,ppid,user,stat,%cpu,%mem,cmd

# Inspect one PID
ps -p PID

# Inspect one PID with selected fields
ps -p PID -o pid,ppid,user,stat,%cpu,%mem,cmd

# Find process by name
pgrep processname

# Show PID + command
pgrep -a processname

# Show children of a PID
pgrep -P PID

# Show children using ps
ps --ppid PID -o pid,ppid,user,stat,cmd

# Live process monitor
top

# Background a new command
command &

# Current shell jobs
jobs

# Suspend foreground process
Ctrl + Z

# Resume current stopped job in background
bg

# Bring current job to foreground
fg

# Bring specific job to foreground
fg %2

# Graceful termination request
kill PID

# Explicit SIGTERM
kill -TERM PID

# Force termination
kill -9 PID

# Start process resistant to normal hangup
nohup command &

# Inspect /proc metadata
cat /proc/PID/status

# Process command line
tr '\0' ' ' < /proc/PID/cmdline

# Current working directory
ls -l /proc/PID/cwd

# Executable
ls -l /proc/PID/exe

# Open file descriptors
ls -l /proc/PID/fd

# Start with lower scheduling priority
nice -n 10 command

# Change nice value
renice 10 -p PID

# Inspect PID 1
ps -p 1 -o pid,ppid,user,comm,args
```

---

# Knowledge Check Summary

By the end of this lab I should be able to explain the following without simply memorizing command syntax.

```text
PID
→ process identifier

PPID
→ parent process identifier

USER
→ identity running the process

R
→ running/runnable

S
→ sleeping normally

D
→ uninterruptible sleep

T
→ stopped/suspended

Z
→ zombie

&
→ launch command in background

$!
→ PID of most recent background process

jobs
→ jobs tracked by current shell

fg
→ bring job into foreground

bg
→ continue stopped job in background

Ctrl + Z
→ suspend foreground job

Ctrl + C
→ normally send SIGINT

kill PID
→ normally send SIGTERM

kill -9 PID
→ send SIGKILL

nohup
→ protect against normal hangup behaviour

ps
→ process snapshot

top
→ continuously refreshing process view

/proc/PID
→ live information about a process
```

---

# Bigger Mental Model

The first four labs now connect together.

```text
WHO AM I?
whoami / id
        │
        ▼
WHERE AM I?
pwd
        │
        ▼
WHAT EXISTS HERE?
ls
        │
        ▼
WHO OWNS IT?
ls -l
        │
        ▼
WHAT CAN I ACCESS?
rwx / chmod / chown
        │
        ▼
WHAT IS RUNNING?
ps / top
        │
        ▼
WHO RUNS IT?
USER
        │
        ▼
WHAT CREATED IT?
PPID
        │
        ▼
WHAT STATE IS IT IN?
R / S / D / T / Z
        │
        ▼
WHAT SHOULD I DO?
investigate → minimal action → verify
```

The important shift is that Linux is no longer starting to look like a collection of unrelated commands.

The commands are becoming tools for answering questions about system state.

---

# What Changed After This Lab

Before this lab, a process list looked mostly like:

```text
random PIDs
random percentages
random letters
```

Now:

```text
PID
→ identifies a specific process

PPID
→ reveals process ancestry

USER
→ connects the process to Linux permissions

STAT
→ explains current process condition

%CPU
→ shows processor pressure

%MEM
→ shows memory pressure

CMD
→ explains what is actually running
```

Before:

```text
process is using lots of CPU
↓
kill it?
```

Now:

```text
process is using lots of CPU
↓
identify it
↓
inspect parent
↓
inspect owner
↓
inspect state
↓
watch resource trend
↓
understand workload
↓
determine impact
↓
decide on appropriate action
```

That change in reasoning is the real result of Lab 004.

---

# Lab Result

```text
LAB 004 — Linux Processes

Process fundamentals       : COMPLETE
PID / PPID                  : COMPLETE
Parent / child processes   : COMPLETE
ps                          : COMPLETE
ps aux                      : COMPLETE
ps -e / -o                  : COMPLETE
pgrep                       : COMPLETE
top                         : COMPLETE
CPU / memory interpretation: COMPLETE
Process states             : COMPLETE
Foreground processes       : COMPLETE
Background processes       : COMPLETE
jobs / bg / fg             : COMPLETE
$!                          : COMPLETE
Signals                     : COMPLETE
SIGTERM / SIGKILL           : COMPLETE
nohup / SIGHUP              : COMPLETE
/proc/<PID>                 : COMPLETE
nice / renice               : INTRODUCED
Zombie processes            : COMPLETE
Troubleshooting workflow    : COMPLETE
Hands-on practical          : COMPLETE
Live incident training      : COMPLETE
Knowledge check             : PASSED WITH CORRECTIONS

STATUS                      : COMPLETE ✅
```

---

# Linux Administration Progress

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
├── Lab 004 — Linux Processes
│   └── COMPLETE ✅
│
└── Lab 005 — systemd & Services
    └── NEXT
```

---

# Next — Lab 005: systemd & Services

Lab 004 dealt with processes directly.

The next question is:

> What manages important long-running processes automatically?

That leads into:

```text
systemd
units
services
systemctl
service states
start
stop
restart
reload
enable
disable
status
dependencies
boot-time services
journalctl
service failures
service troubleshooting
```

Instead of manually doing:

```bash
nohup application &
```

for real production workloads, Linux commonly uses a service manager.

We are about to learn how Linux takes something that would otherwise just be:

```text
a process
```

and turns it into something that can be:

```text
started automatically
stopped cleanly
restarted
enabled at boot
monitored
logged
dependency-managed
```

That is where process knowledge begins turning into actual server administration.

---

> Lab 001 taught me how to move around Linux.  
> Lab 002 taught me where Linux keeps things.  
> Lab 003 taught me who is allowed to do what.  
> Lab 004 taught me how to understand what Linux is actually running.

**Lab 004 complete. 🐧**
