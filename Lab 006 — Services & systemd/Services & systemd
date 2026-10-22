# Lab 006 — Linux Services & systemd

> **Linux Administration Lab · Day 31/90 · 22 October 2026**  
> **Environment:** Debian running under Windows Subsystem for Linux (WSL), with systemd enabled  
> **Status:** **COMPLETE**  
> **Lab theme:** Discover, configure, operate, observe, break, recover, and safely retire a Linux service.

---

## 1. Why this lab matters

An infrastructure engineer does more than launch programs. Servers run long-lived processes—web servers, databases, schedulers, monitoring agents, authentication services—and those processes must be started, supervised, logged, restarted after certain failures, and shut down predictably. Linux commonly uses **systemd** for these responsibilities.

This lab takes an intentionally simple Bash heartbeat program and turns it into a genuine systemd-managed service. Because the application does almost nothing, we can isolate the service-management concepts without confusing them with application complexity.

**Central lesson:** A process being alive, a service being enabled at startup, and an application actually being healthy are three different things.

### Objectives

By the end of the lab, I aimed to:

1. Confirm that systemd is PID 1 in my Debian WSL instance.
2. Discover active services and interpret systemd unit states.
3. Inspect a real service's configuration and runtime information.
4. Create and test an executable application before integrating it with systemd.
5. Write a custom `.service` unit using a non-root service account.
6. Reload systemd configuration and control the application's lifecycle.
7. Verify process ownership and inspect journal logs.
8. Simulate a genuine process failure and verify automatic recovery.
9. Understand the difference between starting/stopping and enabling/disabling.
10. Finish with the test service **disabled and stopped**.

### Scope and safety

- All disruptive experiments were performed against **my own `lab006.service`**, not `cron.service` or another system service.
- The heartbeat program performs no writes, network operations, or privileged tasks.
- The service was configured to run as an ordinary Linux user, **not root**.
- The service was disabled and stopped at the end of the exercise.
- Hostnames and Windows account paths are omitted from this public-friendly record. Some timestamps and PIDs are retained because they are useful experimental evidence.

---

## 2. Starting mental model: processes versus services

A **process** is a running instance of a program. It has a process ID (PID), a parent process ID (PPID), an owning user, and other operating-system attributes.

A **service** is a workload that a service manager supervises according to a configuration. In our case, systemd is the service manager and a Bash script is the workload.

```text
systemd (PID 1)
   |
   +-- lab006.service  [unit configuration / lifecycle management]
           |
           +-- lab-service.sh  [main Bash process]
                   |
                   +-- sleep 10  [short-lived child command]
```

A service unit does not replace the application. It tells systemd **what to launch, as whom, when to restart it, and how it fits into the boot process**.

### Three independent service concepts

| Concept | Example output | Meaning |
|---|---|---|
| Loaded | `Loaded: loaded` | systemd has read and recognizes the unit definition. |
| Active | `Active: active (running)` | The service is currently running. |
| Enabled | `enabled` | Startup relationships have been installed so the unit is normally pulled in by its configured target. |

Possible combinations include **loaded + running + disabled**, or **loaded + inactive + enabled**. An enabled unit is not guaranteed to be running at any given moment.

---

## 3. Exercise 01 — Identify PID 1

### Command

```bash
ps -p 1 -o pid,ppid,comm,args
```

### Observed result

The PID 1 process was `systemd`, running as `/sbin/init`, with PPID `0`.

### Interpretation

- `-p 1` selects PID 1.
- `-o` chooses output columns.
- `pid` is the process identifier.
- `ppid` is the parent process identifier.
- `comm` is the command name.
- `args` is the command line.

PID 1 is the first userspace process of this Linux environment and plays a special role in service management and process supervision. Calling it the system's *starting point* is a useful analogy, but not every process is necessarily a direct child of PID 1.

### Why it matters

Before assuming `systemctl` works, verify what is actually running as init. This is particularly relevant in WSL and container environments, where PID 1 might be something other than systemd.

### Troubleshooting

If PID 1 is not systemd, `systemctl` may fail because there is no systemd system manager available. Inspect the actual init environment before changing service configuration.

---

## 4. Exercise 02 — Check overall systemd state

### Command

```bash
systemctl is-system-running
```

### Observed result

```text
running
```

### Interpretation

`running` indicates that systemd considers the system operational. Another possible result is `degraded`, which commonly means one or more units are in a failed state. It does **not** mean that the system is old or that its processes have been running too long.

### Why it matters

This is a quick health signal for the service manager, not a complete health check for every application.

### Troubleshooting

If the result is `degraded`, inspect failed units with `systemctl --failed`, then investigate the specific units rather than restarting unrelated services.

---

## 5. Exercise 03 — Discover running services

### Command

```bash
systemctl list-units --type=service --state=running
```

### Observed result

Seven running service units were reported:

```text
cron.service
 dbus.service
 getty@tty1.service
 systemd-journald.service
 systemd-logind.service
 systemd-udevd.service
 user@1000.service
```

All appeared as **loaded**, **active**, and **running**.

### Interpretation

| Column | Meaning |
|---|---|
| `LOAD` | Whether systemd loaded the unit definition. |
| `ACTIVE` | High-level activation state. |
| `SUB` | More specific state; for a service this can include `running`, `exited`, or `dead`. |
| `DESCRIPTION` | Human-readable purpose of the unit. |

The number of running services is **not** the number of running processes. A service may have several processes; some processes are not managed as standalone services.

### Why it matters

An administrator needs to inventory managed workloads before diagnosing outages or changing configuration.

### Troubleshooting

If an expected service is absent, check its status directly. It may be inactive, failed, or named differently; the `--state=running` filter deliberately excludes non-running units.

---

## 6. Exercise 04 — Inspect a real service: cron

### Command

```bash
systemctl status cron.service --no-pager
```

### Selected observed output

```text
● cron.service - Regular background program processing daemon
     Loaded: loaded (/usr/lib/systemd/system/cron.service; enabled; preset: enabled)
     Active: active (running) since Thu 2026-10-22 09:38:01 EAT
   Main PID: 106 (cron)
      Tasks: 1 (limit: 9365)
     Memory: 1.3M (peak: 3M)
        CPU: 65ms
     CGroup: /system.slice/cron.service
             └─106 /usr/sbin/cron -f

Oct 22 10:17:01 ... CRON[455]: (root) CMD (cd / && run-parts --report /etc/cron.hourly)
```

### Interpretation

- `--no-pager` prints the result directly rather than opening a pager.
- The unit file came from `/usr/lib/systemd/system/`.
- `enabled` indicates startup configuration.
- `active (running)` indicates a live service.
- `Main PID: 106` identifies the primary cron daemon process.
- `CGroup` shows the control group used to organize and manage the service's processes.
- The journal entry shows cron invoking scripts under `/etc/cron.hourly` as root.

The `CRON[455]` log entry refers to a cron activity process, not a replacement for the main daemon PID 106. It was not evidence that our earlier package installation caused cron to run.

### Why it matters

`systemctl status` provides a useful first snapshot of unit configuration, runtime state, resources, process tree, and recent journal entries.

### Troubleshooting

A service can report `active (running)` even while the work it performs is failing. Follow up with application logs and functional tests.

---

## 7. Exercises 05–08 — Separate startup configuration from runtime state

### Hypothetical case: loaded, disabled, but running

```text
Loaded: loaded (...; disabled)
Active: active (running)
```

This is valid. **Disabled** does not mean unloaded, broken, or forbidden from running. The unit might have been manually started or started as a dependency.

### Exercise 06: Is cron enabled?

```bash
systemctl is-enabled cron.service
```

Observed:

```text
enabled
```

This checks startup enablement, **not** whether cron is currently running.

### Exercise 07: Is cron active?

```bash
systemctl is-active cron.service
```

Observed:

```text
active
```

This checks runtime state, **not** whether cron is enabled at boot.

### Exercise 08: Are any units failed?

```bash
systemctl --failed
```

Observed:

```text
0 loaded units listed.
```

This was a useful baseline. It does not prove that every application is healthy; application-level failures may not change the systemd unit state.

### Common misunderstanding corrected

Initially, it was tempting to treat `disabled` as meaning a service must be manually *loaded*. The more precise distinction is:

- **Loaded:** systemd has the unit definition.
- **Started:** the unit has been activated now.
- **Enabled:** the unit is wired into a startup target.

---

## 8. Exercises 09–10 — Read and reason about a service unit

### Command

```bash
systemctl cat cron.service
```

### Observed unit configuration

```ini
# /usr/lib/systemd/system/cron.service
[Unit]
Description=Regular background program processing daemon
Documentation=man:cron(8)
After=remote-fs.target nss-user-lookup.target

[Service]
EnvironmentFile=-/etc/default/cron
ExecStart=/usr/sbin/cron -f $EXTRA_OPTS
IgnoreSIGPIPE=false
KillMode=process
Restart=on-failure
SyslogFacility=cron

[Install]
WantedBy=multi-user.target
```

### Understanding the sections

#### `[Unit]` — Identity and relationships

- `Description=` gives a readable name.
- `Documentation=` points to reference material.
- `After=` expresses ordering relative to other units. **Ordering alone does not automatically create a requirement dependency.**

#### `[Service]` — How the workload runs

- `EnvironmentFile=-/etc/default/cron` reads optional environment settings. The leading `-` means a missing file is tolerated.
- `ExecStart=` identifies the executable and its arguments.
- `Restart=on-failure` requests restart after qualifying unexpected failures, subject to other systemd policies.
- `KillMode=process` determines which processes systemd targets when stopping the service.

#### `[Install]` — How enabling works

- `WantedBy=multi-user.target` tells `systemctl enable` which target should acquire a `Wants=` relationship to this unit.

### Hypothetical failure exercise

Suppose a unit says:

```ini
ExecStart=/usr/bin/python3 /opt/myapp/server.py
Restart=on-failure
```

If `/opt/myapp/server.py` does not exist, the Python interpreter will fail to open the script. systemd may attempt restarts, but **restarting cannot repair a missing file**. The correct fix is to repair the path, deployment, permissions, or other root cause.

### Why it matters

Service configuration is an operational contract. Understanding it lets you predict what systemd will do before issuing potentially disruptive commands.

### Troubleshooting

Use `systemctl cat` to inspect the effective unit text and any drop-ins. For a failing service, pair configuration inspection with `systemctl status` and `journalctl -u`.

---

## 9. Exercise 11 — Prepare the lab workspace

### Command

```bash
mkdir -p ~/linux-administration-lab/06-systemd-services
```

### Observed result

The directory was created successfully. The shell returned without an error.

### Interpretation

- `~` expands to the current user's home directory in the shell.
- `-p` creates parent directories as necessary and does not complain if the directory already exists.

### Why it matters

A dedicated lab directory keeps experimental application files separate from operating-system unit files.

### Troubleshooting

If creation fails, inspect the parent directory's permissions and available storage. Do not use `sudo` reflexively for files intended to be owned by the regular user.

---

## 10. Exercise 12 — Write a heartbeat application

### Command

```bash
nano ~/linux-administration-lab/06-systemd-services/lab-service.sh
```

### Script created

```bash
#!/bin/bash

echo "Lab 006 service started. PID: $$"

while true; do
    echo "Lab 006 heartbeat: $(date)"
    sleep 10
done
```

### Line-by-line explanation

| Line | Meaning |
|---|---|
| `#!/bin/bash` | Shebang specifying Bash as the script interpreter. |
| `echo ... $$` | Prints a startup message and the current shell process ID. |
| `while true; do` | Starts an indefinite loop. |
| `$(date)` | Runs `date` and substitutes its output into the message. |
| `sleep 10` | Waits approximately ten seconds between iterations. |
| `done` | Ends the loop body. |

The script intentionally does not implement complex business logic, dependencies, networking, or file writes. Its predictable output makes it ideal for observing lifecycle behaviour.

### Why test our own service?

Changing or killing an existing system service such as cron could interfere with scheduled work. A self-contained test service allows deliberate failure experiments without disrupting unrelated workloads.

### Troubleshooting

- If Nano does not save, verify the destination path and write permissions.
- If the script reports a syntax error, inspect the shebang, `while`/`do`/`done` structure, and quote matching.
- If the script is run indefinitely in the foreground, use `Ctrl+C` to interrupt it.

---

## 11. Exercises 13–14 — Inspect and grant execute permission

### Initial inspection

```bash
ls -l ~/linux-administration-lab/06-systemd-services/lab-service.sh
```

The exercise established that the file initially lacked execute permission for the owner. The exact initial `ls -l` output was **not preserved** in the chat record; the illustrative starting mode discussed was `-rw-r--r--`.

### Command

```bash
chmod u+x ~/linux-administration-lab/06-systemd-services/lab-service.sh
```

### Verified result

```text
-rwxr--r-- 1 wndboto wndboto 125 Oct 22 11:36 /home/wndboto/linux-administration-lab/06-systemd-services/lab-service.sh
```

### Interpretation

```text
- rwx r-- r--
|  |   |   |
|  |   |   +-- Others: read only
|  |   +------ Group: read only
|  +---------- Owner: read, write, execute
+------------- Regular file
```

`chmod u+x` adds execute permission **only for the owning user**. It does not grant group or others permission to execute the file.

### Why it matters

Our unit will run the script as `wndboto`, so that user must be able to execute the script. Granting only the necessary permissions follows the principle of least privilege.

### Troubleshooting

A script may exist and be readable but still fail with `Permission denied` when launched directly if execute permission is missing. Also check that parent directories are traversable and that the interpreter named in the shebang exists.

---

## 12. Exercise 15 — Prove the application works without systemd

### Command

```bash
~/linux-administration-lab/06-systemd-services/lab-service.sh
```

### Actual output

```text
Lab 006 service started. PID: 790
Lab 006 heartbeat: Thu Oct 22 11:42:47 AM EAT 2026
Lab 006 heartbeat: Thu Oct 22 11:42:57 AM EAT 2026
Lab 006 heartbeat: Thu Oct 22 11:43:07 AM EAT 2026
^C
```

### Interpretation

- The program launched successfully as PID **790**.
- Heartbeats appeared at approximately ten-second intervals.
- `Ctrl+C` interrupted the foreground process.
- The terminal normally sends **SIGINT** when `Ctrl+C` is pressed in this context; this is not the same as SIGTERM.

### Why it matters

**Verify an application independently before asking a service manager to supervise it.** If a script cannot run correctly by itself, wrapping it in systemd does not fix its syntax, path, permission, or dependency problems.

### Troubleshooting

If the script fails here, troubleshoot the script first. Check executable permissions, interpreter availability, and command output before writing a systemd unit.

---

## 13. Exercises 16–17 — Inspect the system unit directory

### Command

```bash
ls -ld /etc/systemd/system
```

### Actual output

```text
drwxr-xr-x 7 root root 4096 May 16 16:32 /etc/systemd/system
```

### Interpretation

- `d` identifies a directory.
- Owner `root` has `rwx`.
- Group `root` has `r-x`.
- Others have `r-x`.
- The ordinary user does **not** have directory write permission under this mode.
- `ls -ld` reports the directory itself rather than listing its contents.

### Why it matters

`/etc/systemd/system` is the standard location for administrator-managed system service units. Writing a file there normally requires elevated privileges.

### Troubleshooting

If a normal user cannot save a unit in this directory, that is expected. Use an appropriate administrative workflow, rather than changing the directory's permissions broadly.

---

## 14. Exercise 18 — Create a custom systemd service unit

### Command

```bash
sudo nano /etc/systemd/system/lab006.service
```

### Unit file created

```ini
[Unit]
Description=Lab 006 - Linux Administration Test Service

[Service]
Type=simple
User=wndboto
ExecStart=/home/wndboto/linux-administration-lab/06-systemd-services/lab-service.sh
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

### Explanation of every directive

| Directive | What it does | Why it matters |
|---|---|---|
| `[Unit]` | Opens general unit metadata. | Organizes identity and relationships. |
| `Description=` | Sets the display name. | Makes status and logs understandable. |
| `[Service]` | Opens service-specific configuration. | Defines runtime behaviour. |
| `Type=simple` | Uses a straightforward foreground-process service model. | Appropriate for our long-running Bash loop. |
| `User=wndboto` | Runs the service process as `wndboto`. | Avoids unnecessary root privileges. |
| `ExecStart=` | Supplies the executable's absolute path. | systemd knows exactly what to launch. |
| `Restart=on-failure` | Requests a restart after qualifying failures. | Supports automatic recovery. |
| `RestartSec=5` | Adds a five-second delay between restart attempts. | Avoids immediate rapid retries. |
| `[Install]` | Opens enablement configuration. | Controls target integration. |
| `WantedBy=multi-user.target` | Connects the unit to the multi-user target when enabled. | Supports startup activation. |

### Important details

**Absolute path:** The unit uses `/home/wndboto/...` rather than shell shorthand such as `~/...`. systemd service commands do not run through an interactive shell that expands `~` the same way our terminal does.

**Least privilege:** systemd is responsible for managing the service, but `User=wndboto` determines the credentials under which the application executes. A root-managed service does **not** have to be a root-running application.

**Restart semantics:** `Restart=on-failure` does not mean systemd should undo a deliberate administrator stop. Nor does it guarantee recovery from a persistent configuration or application defect.

**WSL caveat:** The systemd unit exists inside Debian WSL. Enabling a unit affects startup of that Linux instance; it does not make the service run when WSL itself is shut down.

### Troubleshooting

- A misspelled `User=` may prevent service startup.
- An incorrect `ExecStart=` path can cause an execution failure.
- Missing execute permission can cause permission errors.
- After editing a unit, reload systemd's configuration before relying on the change.

---

## 15. Exercise 19 — Reload systemd's unit configuration

### Command

```bash
sudo systemctl daemon-reload
```

### Observed result

The command completed without a reported error.

### Interpretation

`daemon-reload` makes the systemd manager reread unit definitions and rebuild relevant dependency information. It **does not start, stop, or restart** the heartbeat application.

### Why it matters

Creating or editing a unit file is not the same as telling the running systemd manager to reprocess that configuration.

### Troubleshooting

If a changed unit appears to be ignored, confirm the file was saved and run `daemon-reload`. Then inspect the unit with `systemctl cat` or `systemctl status`.

---

## 16. Exercise 20 — Inspect the new service before starting it

### Command

```bash
systemctl status lab006.service --no-pager
```

### Actual output

```text
○ lab006.service - Lab 006 - Linux Administration Test Service
     Loaded: loaded (/etc/systemd/system/lab006.service; disabled; preset: enabled)
     Active: inactive (dead)
```

### Interpretation

- `loaded`: systemd recognizes the unit file.
- `disabled`: the unit was not yet configured for automatic startup.
- `preset: enabled`: systemd's preset policy recommends enabling it; this does **not** override the actual `disabled` state.
- `inactive (dead)`: the program has not yet been started.

### Why it matters

This is the baseline against which we can compare the service after starting it.

### Troubleshooting

If the unit is not found, verify the filename, location, and whether `daemon-reload` completed. If it is loaded but inactive, that alone is not an error for a service that has never been started.

---

## 17. Exercise 21 — Start the service under systemd

### Command

```bash
sudo systemctl start lab006.service
```

The command completed without a reported error.

### Follow-up command

```bash
systemctl status lab006.service --no-pager
```

### Actual output

```text
● lab006.service - Lab 006 - Linux Administration Test Service
     Loaded: loaded (/etc/systemd/system/lab006.service; disabled; preset: enabled)
     Active: active (running) since Thu 2026-10-22 11:57:41 EAT; 4s ago
 Invocation: fccddf7a330e44a4a5279830d02efb63
   Main PID: 894 (lab-service.sh)
      Tasks: 2 (limit: 9365)
     Memory: 1.1M (peak: 1.7M)
        CPU: 69ms
     CGroup: /system.slice/lab006.service
             ├─894 /bin/bash /home/wndboto/linux-administration-lab/06-systemd-services/lab-service.sh
             └─896 sleep 10

Oct 22 11:57:41 ... systemd[1]: Started lab006.service - Lab 006 - Linux Administration Test Service.
Oct 22 11:57:41 ... lab-service.sh[894]: Lab 006 service started. PID: 894
Oct 22 11:57:41 ... lab-service.sh[894]: Lab 006 heartbeat: Thu Oct 22 11:57:41 AM EAT 2026
```

### Interpretation

- The main application now ran as PID **894**.
- The service was `active (running)` but still `disabled` for startup.
- `sleep 10` appeared as a child process under the same service cgroup.
- The service was running independently of our foreground terminal session.

### Why it matters

This demonstrates the transition from a manually launched process to a supervised service.

### Troubleshooting

If `start` fails, inspect `systemctl status lab006.service --no-pager` and `journalctl -u lab006.service --no-pager`. Check `ExecStart`, permissions, user credentials, and application behaviour.

---

## 18. Exercise 22 — Verify the service runs as the intended user

### Command

```bash
ps -o user,pid,ppid,comm -p 894
```

### Actual output

```text
USER         PID    PPID COMMAND
wndboto      894       1 lab-service.sh
```

### Interpretation

- `USER=wndboto`: the application runs under the non-root account configured in the unit.
- `PID=894`: the current application process ID.
- `PPID=1`: systemd is the parent of this service process.

### Critical distinction

**Who manages the process** and **whose permissions it uses** are different concepts. PID 1 can manage a service while the service process executes with a regular user's credentials.

### Why it matters

This reduces the impact of bugs or compromises in the application. A heartbeat script has no legitimate reason to run as root.

### Troubleshooting

If the process owner is unexpected, inspect `User=` and reload/restart the service as appropriate. Never assume a configured user is being applied—verify it.

---

## 19. Exercise 23 — Read application logs through journald

### Command

```bash
journalctl -u lab006.service -n 10 --no-pager
```

### Actual output (hostnames shortened)

```text
Oct 22 12:03:31 ... lab-service.sh[894]: Lab 006 heartbeat: Thu Oct 22 12:03:31 PM EAT 2026
Oct 22 12:03:41 ... lab-service.sh[894]: Lab 006 heartbeat: Thu Oct 22 12:03:41 PM EAT 2026
Oct 22 12:03:51 ... lab-service.sh[894]: Lab 006 heartbeat: Thu Oct 22 12:03:51 PM EAT 2026
Oct 22 12:04:01 ... lab-service.sh[894]: Lab 006 heartbeat: Thu Oct 22 12:04:01 PM EAT 2026
Oct 22 12:04:11 ... lab-service.sh[894]: Lab 006 heartbeat: Thu Oct 22 12:04:11 PM EAT 2026
Oct 22 12:04:21 ... lab-service.sh[894]: Lab 006 heartbeat: Thu Oct 22 12:04:21 PM EAT 2026
Oct 22 12:04:31 ... lab-service.sh[894]: Lab 006 heartbeat: Thu Oct 22 12:04:31 PM EAT 2026
Oct 22 12:04:41 ... lab-service.sh[894]: Lab 006 heartbeat: Thu Oct 22 12:04:41 PM EAT 2026
Oct 22 12:04:51 ... lab-service.sh[894]: Lab 006 heartbeat: Thu Oct 22 12:04:51 PM EAT 2026
Oct 22 12:05:01 ... lab-service.sh[894]: Lab 006 heartbeat: Thu Oct 22 12:05:01 PM EAT 2026
```

### Interpretation

- `-u lab006.service` filters entries associated with this unit.
- `-n 10` requests the latest ten entries.
- `--no-pager` prints the result without an interactive pager.
- The journal confirms repeated heartbeats from PID **894**, approximately ten seconds apart.

### Why it matters

`active (running)` only tells us that the process is alive. Logs can reveal failures inside a still-running application—for example, failed database connections, dependency timeouts, or application errors—**if the application records those events**.

### Troubleshooting

If expected logs are absent, check that the service actually started and that the application writes to stdout/stderr or another configured logging destination. A lack of logs does not automatically mean the application is healthy.

---

## 20. Exercises 24–25 — Stop the service and verify inactivity

### Command

```bash
sudo systemctl stop lab006.service
```

The command returned without an error.

### Journal inspection

```bash
journalctl -u lab006.service -n 10 --no-pager
```

### Actual shutdown evidence

```text
Oct 22 12:09:21 ... lab-service.sh[894]: Lab 006 heartbeat: Thu Oct 22 12:09:21 PM EAT 2026
Oct 22 12:09:23 ... systemd[1]: Stopping lab006.service - Lab 006 - Linux Administration Test Service...
Oct 22 12:09:23 ... systemd[1]: lab006.service: Deactivated successfully.
Oct 22 12:09:23 ... systemd[1]: Stopped lab006.service - Lab 006 - Linux Administration Test Service.
```

### Verification command

```bash
systemctl is-active lab006.service
```

### Observed result

```text
inactive
```

### Interpretation

A deliberate stop requests an orderly shutdown. With `Restart=on-failure`, systemd does **not** treat an administrator-issued `systemctl stop` as a reason to immediately restart the service.

`inactive` means the application is not currently running; the unit file has not been deleted. `systemctl is-active` can return a nonzero exit status for an inactive service, which is normal in this context.

### Why it matters

Operators must be able to stop a service intentionally for maintenance without fighting the restart policy.

### Troubleshooting

If the service continues running unexpectedly, inspect its state, process tree, and restart policy. Distinguish a deliberate stop from a process that exited on its own.

---

## 21. Exercise 26 — Start the service for a failure-recovery test

### Command

```bash
sudo systemctl start lab006.service
```

### Observed result

The command completed without an error. The subsequent journal shows the service's main process was **PID 1135** before the failure test.

### Why it matters

To test crash recovery, the service must be running and then terminate unexpectedly. The previous `systemctl stop` was a deliberate shutdown and therefore did not exercise the restart-on-failure condition.

---

## 22. Exercise 27 — Simulate an unexpected process failure

### Command

```bash
sudo systemctl kill --kill-whom=main --signal=SIGKILL lab006.service
```

### Interpretation

- `systemctl kill` sends a signal to processes belonging to a unit.
- `--kill-whom=main` targets the main service process rather than indiscriminately targeting every process in the cgroup.
- `--signal=SIGKILL` uses signal 9, which cannot be caught, blocked, or ignored by the target process.
- `lab006.service` explicitly restricts the experiment to our test unit.

**Safety warning:** This is a destructive test of the target process. Do not use it on an important production service without a justified, approved maintenance or test procedure.

### Why it matters

Unlike a deliberate `systemctl stop`, an unexpected main-process termination should qualify for `Restart=on-failure` in this configuration.

### Troubleshooting

If recovery does not occur, inspect the unit's restart settings, systemd's rate limiting, the application's exit result, and journal entries. Restart policy is not a substitute for repairing the cause of repeated crashes.

---

## 23. Exercise 28 — Prove systemd restarted the crashed application

### Command

```bash
journalctl -u lab006.service -n 20 --no-pager
```

### Actual evidence (relevant excerpt)

```text
Oct 22 12:19:49 ... lab-service.sh[1135]: Lab 006 heartbeat: Thu Oct 22 12:19:49 PM EAT 2026
Oct 22 12:19:59 ... lab-service.sh[1135]: Lab 006 heartbeat: Thu Oct 22 12:19:59 PM EAT 2026
Oct 22 12:20:10 ... lab-service.sh[1135]: Lab 006 heartbeat: Thu Oct 22 12:20:10 PM EAT 2026
Oct 22 12:20:19 ... systemd[1]: lab006.service: Sent signal SIGKILL to main process 1135 (lab-service.sh) on client request.
Oct 22 12:20:19 ... systemd[1]: lab006.service: Main process exited, code=killed, status=9/KILL
Oct 22 12:20:19 ... systemd[1]: lab006.service: Failed with result 'signal'.
Oct 22 12:20:24 ... systemd[1]: lab006.service: Scheduled restart job, restart counter is at 1.
Oct 22 12:20:24 ... systemd[1]: Started lab006.service - Lab 006 - Linux Administration Test Service.
Oct 22 12:20:24 ... lab-service.sh[1205]: Lab 006 service started. PID: 1205
Oct 22 12:20:24 ... lab-service.sh[1205]: Lab 006 heartbeat: Thu Oct 22 12:20:24 PM EAT 2026
Oct 22 12:20:34 ... lab-service.sh[1205]: Lab 006 heartbeat: Thu Oct 22 12:20:34 PM EAT 2026
Oct 22 12:20:44 ... lab-service.sh[1205]: Lab 006 heartbeat: Thu Oct 22 12:20:44 PM EAT 2026
Oct 22 12:20:54 ... lab-service.sh[1205]: Lab 006 heartbeat: Thu Oct 22 12:20:54 PM EAT 2026
Oct 22 12:21:04 ... lab-service.sh[1205]: Lab 006 heartbeat: Thu Oct 22 12:21:04 PM EAT 2026
Oct 22 12:21:14 ... lab-service.sh[1205]: Lab 006 heartbeat: Thu Oct 22 12:21:14 PM EAT 2026
Oct 22 12:21:24 ... lab-service.sh[1205]: Lab 006 heartbeat: Thu Oct 22 12:21:24 PM EAT 2026
Oct 22 12:21:34 ... lab-service.sh[1205]: Lab 006 heartbeat: Thu Oct 22 12:21:34 PM EAT 2026
Oct 22 12:21:44 ... lab-service.sh[1205]: Lab 006 heartbeat: Thu Oct 22 12:21:44 PM EAT 2026
Oct 22 12:21:54 ... lab-service.sh[1205]: Lab 006 heartbeat: Thu Oct 22 12:21:54 PM EAT 2026
Oct 22 12:22:04 ... lab-service.sh[1205]: Lab 006 heartbeat: Thu Oct 22 12:22:04 PM EAT 2026
```

### Failure-and-recovery timeline

| Time (EAT) | Event | Evidence |
|---|---|---|
| 12:20:10 | Last recorded heartbeat before failure | PID 1135 |
| 12:20:19 | Main process killed | `status=9/KILL` |
| 12:20:19 | systemd marks unit failed | `Failed with result 'signal'` |
| 12:20:24 | Restart scheduled and executed | Restart counter `1` |
| 12:20:24 | New application process launched | PID **1205** |
| 12:20:34 onward | Heartbeats continue | New PID 1205 |

### What this proves

1. The application **did not survive SIGKILL**.
2. systemd detected the main process's unexpected exit.
3. systemd applied `Restart=on-failure`.
4. The observed delay was approximately **five seconds**, consistent with `RestartSec=5`.
5. The replacement process received a **new PID: 1205**, rather than retaining PID 1135.
6. Heartbeat output resumed, demonstrating successful recovery in this controlled scenario.

### What this does *not* prove

- It does not prove that systemd can fix missing dependencies or broken application logic.
- It does not prove that the service can survive unlimited repeated failures; start-rate limits may apply.
- It does not prove full application health for a real production workload.

### Why it matters

This is the strongest evidence in the lab: a **deliberately induced fault**, a **recorded failure**, a **timed recovery**, and **verified continued operation**. The experiment moves beyond merely assuming the configuration works.

---

## 24. Exercises 29–30 — Enable automatic startup

### Command

```bash
sudo systemctl enable lab006.service
```

### Actual output

```text
Created symlink '/etc/systemd/system/multi-user.target.wants/lab006.service' → '/etc/systemd/system/lab006.service'.
```

### Interpretation

`enable` created a **symbolic link** in the `multi-user.target.wants` directory pointing to our unit file. It did not duplicate the unit file.

```text
/etc/systemd/system/multi-user.target.wants/lab006.service
                       |
                       +--> /etc/systemd/system/lab006.service
                                      |
                                      +--> ExecStart=/home/wndboto/.../lab-service.sh
```

This relationship lets the multi-user target pull in our service when that target is activated.

### Verification

```bash
systemctl is-enabled lab006.service
```

Observed:

```text
enabled
```

### Why it matters

**Start** controls what happens now; **enable** configures startup integration. Neither operation should be confused with the other.

### WSL-specific consideration

WSL must itself be running for the Debian service to run. A service enabled inside WSL does not continue operating after its Linux instance has fully shut down.

---

## 25. Exercises 31–32 — Disable startup and stop the test service

### Disable command

```bash
sudo systemctl disable lab006.service
```

The user confirmed this operation was completed. The exact `disable` command output was not preserved in the conversation, so no removal message is fabricated here.

### Meaning

Disabling removes the startup relationship created by `enable`. It does **not** automatically stop a currently running service.

### Stop command

```bash
sudo systemctl stop lab006.service
```

The final stop completed without a reported error.

### Why it matters

A service can be disabled for the next startup while continuing to run now. To finish the lab safely, we performed both actions: **disable** and **stop**.

---

## 26. Exercise 33 — Final audit and clean shutdown

### Command

```bash
systemctl status lab006.service --no-pager
```

### Actual final output (hostnames shortened)

```text
○ lab006.service - Lab 006 - Linux Administration Test Service
     Loaded: loaded (/etc/systemd/system/lab006.service; disabled; preset: enabled)
     Active: inactive (dead)

Oct 22 12:32:25 ... lab-service.sh[1205]: Lab 006 heartbeat: Thu Oct 22 12:32:25 PM EAT 2026
Oct 22 12:32:35 ... lab-service.sh[1205]: Lab 006 heartbeat: Thu Oct 22 12:32:35 PM EAT 2026
Oct 22 12:32:45 ... lab-service.sh[1205]: Lab 006 heartbeat: Thu Oct 22 12:32:45 PM EAT 2026
Oct 22 12:32:55 ... lab-service.sh[1205]: Lab 006 heartbeat: Thu Oct 22 12:32:55 PM EAT 2026
Oct 22 12:33:05 ... lab-service.sh[1205]: Lab 006 heartbeat: Thu Oct 22 12:33:05 PM EAT 2026
Oct 22 12:33:15 ... lab-service.sh[1205]: Lab 006 heartbeat: Thu Oct 22 12:33:15 PM EAT 2026
Oct 22 12:33:25 ... lab-service.sh[1205]: Lab 006 heartbeat: Thu Oct 22 12:33:25 PM EAT 2026
Oct 22 12:33:27 ... systemd[1]: Stopping lab006.service - Lab 006 - Linux Administration Test Service...
Oct 22 12:33:27 ... systemd[1]: lab006.service: Deactivated successfully.
Oct 22 12:33:27 ... systemd[1]: Stopped lab006.service - Lab 006 - Linux Administration Test Service.
```

### Final verified state

| Property | Result | Meaning |
|---|---|---|
| Unit configuration | `loaded` | The unit file still exists and is recognized. |
| Startup integration | `disabled` | It is not configured to start automatically through normal enablement. |
| Runtime | `inactive (dead)` | The heartbeat application is not running. |
| Shutdown | `Deactivated successfully` | systemd completed the deliberate stop. |

The output also contained a note that some journal lines were **ellipsized**. This is a display-formatting notice, **not a service failure**. `systemctl status -l` can be used when full lines are required.

**Lab exit condition satisfied:** the test service remains defined but is **stopped and disabled**.

---

## 27. Troubleshooting playbook

Use this approach when a systemd-managed application is not behaving as expected.

### A. The service is not found

```bash
systemctl status lab006.service --no-pager
systemctl cat lab006.service
```

Check the unit filename and whether it is located in a valid unit search directory. After creating or changing the unit:

```bash
sudo systemctl daemon-reload
```

### B. The unit loads but the application fails to start

```bash
systemctl status lab006.service --no-pager
journalctl -u lab006.service -n 30 --no-pager
```

Check:

- Whether `ExecStart=` points to the correct absolute path.
- Whether the script exists.
- Whether the script is executable by the configured user.
- Whether the interpreter in the shebang exists.
- Whether the service account can traverse all parent directories.
- Whether the application works when run manually as the intended user.

### C. The service is active but not functioning correctly

`active (running)` is a **process-state signal**, not proof of application health. Inspect journal messages, dependencies, external connectivity, and actual functional behaviour.

### D. The service keeps restarting

Inspect `Restart=`, `RestartSec=`, failure codes, and systemd's start-rate limiting. Repeated automatic restarts can conceal a persistent defect rather than solve it.

### E. The service is disabled but still running

This is expected if it was already started. Use `stop` if the intention is to terminate it now; use `disable` if the intention is to remove normal startup activation.

### F. The service is enabled but inactive

Enablement does not itself guarantee that the service has been started. Check status and logs. Depending on the unit and environment, a service may be enabled but intentionally inactive or may have failed during startup.

### G. The service is not running after Windows restarts

Remember that this lab runs in **Debian WSL**. Inspect whether the WSL instance started, whether systemd is enabled in that instance, and whether the service is enabled. Do not assume WSL behaves identically to an always-on physical Linux server.

### H. Why `sudo` for some commands but not others?

Reading status or logs may be permitted to a regular user, depending on local policy. Modifying system unit files and controlling system services typically requires administrative authorization. Apply `sudo` only where needed.

---

## 28. Command reference

The following commands were used in the lab unless explicitly labeled as additional reference material.

| Command | Purpose |
|---|---|
| `ps -p 1 -o pid,ppid,comm,args` | Identify PID 1 and its command. |
| `systemctl is-system-running` | Check systemd's overall state. |
| `systemctl list-units --type=service --state=running` | List currently running service units. |
| `systemctl status cron.service --no-pager` | Inspect an existing service. |
| `systemctl is-enabled cron.service` | Check cron startup enablement. |
| `systemctl is-active cron.service` | Check cron runtime state. |
| `systemctl --failed` | List failed units. |
| `systemctl cat cron.service` | Read cron's unit configuration. |
| `mkdir -p ~/linux-administration-lab/06-systemd-services` | Create the lab directory. |
| `nano ~/linux-administration-lab/06-systemd-services/lab-service.sh` | Write the test script. |
| `ls -l ~/linux-administration-lab/06-systemd-services/lab-service.sh` | Inspect script permissions. |
| `chmod u+x ~/linux-administration-lab/06-systemd-services/lab-service.sh` | Grant owner execute permission. |
| `~/linux-administration-lab/06-systemd-services/lab-service.sh` | Test the script manually. |
| `ls -ld /etc/systemd/system` | Inspect system unit directory permissions. |
| `sudo nano /etc/systemd/system/lab006.service` | Create the custom unit. |
| `sudo systemctl daemon-reload` | Reload systemd unit configuration. |
| `systemctl status lab006.service --no-pager` | Inspect our service state. |
| `sudo systemctl start lab006.service` | Start the service now. |
| `ps -o user,pid,ppid,comm -p 894` | Verify process ownership and parent. |
| `journalctl -u lab006.service -n 10 --no-pager` | Read recent service logs. |
| `sudo systemctl stop lab006.service` | Deliberately stop the service. |
| `systemctl is-active lab006.service` | Check whether the service is active. |
| `sudo systemctl kill --kill-whom=main --signal=SIGKILL lab006.service` | Simulate unexpected main-process death. |
| `journalctl -u lab006.service -n 20 --no-pager` | Examine crash and recovery evidence. |
| `sudo systemctl enable lab006.service` | Configure automatic startup. |
| `systemctl is-enabled lab006.service` | Verify startup enablement. |
| `sudo systemctl disable lab006.service` | Remove automatic startup integration. |

### Useful additional commands (reference only; not claimed as performed)

```bash
# Show full, untruncated status lines
systemctl status lab006.service --no-pager -l

# Follow new service logs live
journalctl -u lab006.service -f

# Show the service's full unit definition
systemctl cat lab006.service

# Restart the service deliberately
sudo systemctl restart lab006.service

# Display failure details and state
systemctl show lab006.service -p ActiveState -p SubState -p Result -p NRestarts

# Validate a unit file's configuration
systemd-analyze verify /etc/systemd/system/lab006.service
```

**Note:** The commands above are optional references for future practice. They are not presented as actions completed during Lab 006.

---

## 29. Mistakes, corrections, and lessons learned

### 1. Confusing `disabled` with `unloaded`

**Initial idea:** A disabled service would need to be manually loaded to work.

**Correction:** `disabled` describes startup enablement. A unit can already be loaded and running while disabled.

### 2. Treating Ctrl+C as generic termination

**Initial idea:** Ctrl+C simply terminates a foreground script.

**Correction:** In the normal terminal scenario, Ctrl+C generates **SIGINT** for the foreground process group. SIGINT is distinct from SIGTERM and SIGKILL.

### 3. Assuming systemd-managed processes inherit root privileges

**Initial analogy:** The root-managed system is the 'haystack' and the service is a smaller 'needle'.

**Correction:** The manager's privileges and the application's execution credentials are separate. `User=wndboto` ensured the script ran as `wndboto`, verified using `ps`.

### 4. Assuming `daemon-reload` starts services

**Correction:** It reloads unit configuration; `systemctl start` activates a service.

### 5. Assuming an automatic restart means automatic repair

**Correction:** systemd can restart a crashed program, but it cannot fix an invalid path, missing dependency, or defective application logic by restarting alone.

### 6. Assuming `disable` immediately stops a service

**Correction:** Disabling affects startup integration. Stopping affects the currently running instance. We performed both during cleanup.

### 7. Reading too much into `systemctl status`

**Correction:** A green `active (running)` state is useful, but journal inspection and application-level verification are necessary to assess actual behaviour.

### 8. Learning to distinguish evidence from assumptions

This README retains actual terminal output where it was provided and describes confirmed actions without inventing missing terminal output. The initial permission listing and exact `disable` output were not recorded, so they are not fabricated.

---

## 30. Security and reliability takeaways

**Least privilege:** A service should run with only the permissions it requires. The test service did not need root.

**Controlled failure testing:** Failure experiments should target isolated lab services, not production workloads or shared system infrastructure.

**Observability:** Logs, process metadata, and service state provide complementary views. None alone gives the whole story.

**Resilience:** `Restart=on-failure` can reduce downtime after unexpected process exits. Recovery should be verified, not assumed.

**Change management:** Reloading configuration, starting a service, and enabling startup are distinct operations with different effects.

**Operational cleanup:** A successful lab includes leaving the environment in a known safe state.

**Evidence-based troubleshooting:** Use timestamps, exit reasons, PIDs, restart counters, and application output to establish what actually happened.

---

## 31. Knowledge check

1. What does PID 1 do in this Debian WSL environment?
2. Can a service be active while disabled? Explain.
3. What is the difference between `systemctl is-active` and `systemctl is-enabled`?
4. What does `systemctl daemon-reload` change—and what does it not change?
5. Why did we make the Bash script executable before putting it under systemd?
6. Why was `User=wndboto` preferable to running the heartbeat as root?
7. What did PPID 1 tell us about PID 894?
8. Why are journal logs useful even when a service is active?
9. Why did `systemctl stop` not trigger `Restart=on-failure`?
10. What did `status=9/KILL` tell us about the failure test?
11. How did we prove a new process was launched after the crash?
12. What role did `RestartSec=5` play?
13. What did `systemctl enable` create on disk?
14. Why did we need both `disable` and `stop` to finish the lab?
15. What additional checks would you perform if a real web service were active but users still could not reach it?

### Answers / self-check

1. It runs as the system init and service manager.
2. Yes; active is runtime state, disabled is startup configuration.
3. One checks current runtime state, the other startup enablement.
4. It reloads unit definitions; it does not itself start or restart the service.
5. The configured service user needed permission to execute the script directly.
6. The application had no privileged requirements, so non-root execution reduced unnecessary risk.
7. systemd was the process's parent, while `wndboto` was its owner.
8. They can reveal errors and internal activity not captured by the process-alive state.
9. It was an intentional administrative stop, not an unexpected failure.
10. The main process was killed by signal 9, SIGKILL.
11. PID changed from **1135** to **1205**, and the new PID emitted fresh heartbeats.
12. It introduced the observed approximately five-second restart delay.
13. A symbolic link under `multi-user.target.wants` pointing to the service unit.
14. Disable removes startup integration; stop terminates the current process.
15. Inspect application logs, listening sockets, network/firewall rules, dependencies, configuration, and functional health checks.

---

## 32. Lab completion record

| Item | Result |
|---|---|
| Date | 22 October 2026 |
| Streak | Day 31 of 90 |
| Lab | 006 — Services & systemd |
| Exercises | 01–33 completed |
| Test application | Bash heartbeat every ~10 seconds |
| Custom unit | `/etc/systemd/system/lab006.service` |
| Service account | `wndboto` (non-root) |
| Initial managed PID | 894 |
| PID before induced failure | 1135 |
| PID after automatic recovery | 1205 |
| Observed recovery delay | ~5 seconds |
| Automatic recovery test | **PASS** |
| Final loaded state | `loaded` |
| Final startup state | `disabled` |
| Final runtime state | `inactive (dead)` |
| Final shutdown | **Successful** |

### Final reflection

In this lab I moved beyond checking whether a process exists. I built an application, placed it under systemd supervision, verified its execution identity, observed its journal output, deliberately killed its main process, and used timestamps and process IDs to prove that systemd restarted it after five seconds. I also learned why starting, stopping, enabling, disabling, and reloading are separate administrative operations.

The biggest takeaway is that **reliable service administration depends on understanding the configuration, observing real behaviour, and testing failure handling**. A service that looks healthy in `systemctl status` may still be failing at its actual job; a restart policy may recover a process without fixing its root cause; and a service manager can supervise a workload without granting it root privileges.

**Lab 006 complete. Environment verified clean: `loaded`, `disabled`, `inactive (dead)`.**

---

## 33. Roadmap

| Lab | Topic | Status |
|---|---|---|
| 001 | Meet the Linux System | Complete |
| 002 | Filesystem Navigation | Complete |
| 003 | Users & Groups | Complete |
| 004 | Permissions & Ownership | Complete |
| 005 | Processes | Complete |
| **006** | **Services & systemd** | **Complete** |
| 007 | Package Management | Next |
| 008 | Storage & Filesystems | Planned |
| 009 | Networking | Planned |
| 010 | SSH | Planned |
| 011 | Logs & Monitoring | Planned |
| 012 | Bash Automation | Planned |
| 013 | Cron & Scheduled Jobs | Planned |
| 014 | Firewalls & Hardening | Planned |
| 015 | Troubleshooting | Planned |
| 016 | Backup & Recovery | Planned |
| Capstone | Provision, configure, secure, monitor, back up, break, troubleshoot and recover a Linux server | Planned |

---

*Documented from the actual Lab 006 terminal evidence and exercise discussions. Examples marked as hypothetical or reference-only are not represented as executed commands.*
