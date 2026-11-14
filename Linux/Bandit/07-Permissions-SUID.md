# Permissions & SUID

## Related Levels
- **Level 19** → Executing a SUID binary to run commands with the file owner's effective privileges
- **Level 20** → Using a SUID-enabled program (`suconnect`) to communicate with a local TCP service

## Core Idea

Linux file permissions control access to files and determine who can read, write, or execute them.

The SUID (Set User ID) bit is a special permission that changes the effective user ID (EUID) of a process when a SUID-enabled executable is launched. The process retains the real user ID (RUID) of the user who started it but performs most permission checks using the file owner's identity.

This allows programs to perform specific operations with their owner's privileges without granting those privileges permanently to the user.

## Key Commands & Patterns

### 1. Inspecting File Permissions

```bash
ls -l filename
```

- `ls -l` → Displays file permissions and ownership.
- `s` in the owner's execute position indicates that SUID is enabled.
- `S` indicates that SUID is set but the owner's execute permission is disabled.

### 2. Executing a SUID Binary

```bash
./bandit20-do cat /etc/bandit_pass/bandit20
```

- `bandit20-do` → A SUID-enabled executable owned by `bandit20`.
- The executable runs with `bandit20` as its effective user ID.
- `cat` → Reads the password file using the process's effective privileges.

### 3. Inspecting User Identity

```bash
id
```

- `id` → Displays the current user's identity and group information.
- **RUID (Real UID)** → Identifies the user who started the process.
- **EUID (Effective UID)** → Determines the identity generally used for permission checks.

## How They Work Together

Bandit demonstrates two uses of SUID:

1. **Level 19:** The `bandit20-do` executable allows commands to run with the effective privileges of its owner, `bandit20`.
2. **Level 20:** The `suconnect` executable runs with the effective identity of `bandit20` and uses those privileges while communicating with a local TCP service.

Both levels demonstrate how SUID allows a program to perform operations beyond the invoking user's normal permissions.

## Important Notes

- SUID does not automatically grant root privileges. A process receives the effective user ID of the executable's owner.
- SUID changes the effective UID, while the real UID remains associated with the user who launched the process.
- The `s` or `S` character appears in the owner's permission position in `ls -l` output.
- Vulnerable or improperly designed SUID binaries can create privilege-escalation risks.
- Linux may ignore the SUID bit under certain conditions, such as when a filesystem is mounted with the `nosuid` option.

## Related Concepts

- [Networking Services](./06-Networking-Services.md) — Local TCP communication involving the SUID-enabled `suconnect` binary in Level 20.
- [Shell Scripting, Cron & Shell Escape](./08-Shell-Scripting-Cron-Escape.md) — Understanding command execution and shell behavior in relation to process privileges.
