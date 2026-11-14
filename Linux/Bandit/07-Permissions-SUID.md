# Permissions & SUID

## Related Levels

- **Level 19** → Executing a SUID binary to run commands with the file owner's effective privileges
- **Level 20** → Using a SUID-enabled program (`suconnect`) to communicate with a local TCP service

## Core Idea

Linux file permissions control access to files and determine who can read, write, or execute them.

The SUID (Set User ID) bit is a special permission that changes the effective user ID (EUID) of a process when a SUID-enabled executable is launched under normal execution conditions. The process retains the real user ID (RUID) of the user who started it but performs most permission checks using the file owner's identity.

This allows programs to perform specific operations with their owner's privileges without granting those privileges permanently to the user.

## Key Commands & Patterns

### 1. Inspecting File Permissions

```bash
ls -l filename
```

- `ls -l` → Long listing format: displays file permissions and ownership.
- `s` in the owner's execute position indicates that SUID is enabled.
- `S` indicates that SUID is set but the owner's execute permission is disabled.
- The owner column matters as much as the `s`: it determines which identity the program will run as.

### 2. Executing a SUID Binary

```bash
./bandit20-do cat /etc/bandit_pass/bandit20
```

- `bandit20-do` → A SUID-enabled executable owned by `bandit20`.
- The executable runs with `bandit20` as its effective user ID.
- `cat` → Runs as a child of the SUID process and inherits its effective privileges; `cat` itself has no SUID bit.

### 3. Inspecting User Identity

```bash
id
id -u
id -ur
```

- `id` → Displays user and group identity information, including real and effective IDs when they differ, and supplementary group memberships.
- `-u` (`--user`) → Prints only the effective user ID.
- `-r` (`--real`) → Prints the real ID instead of the effective one; used with `-u`, `-g`, or `-G`.
- **RUID (Real UID)** → Identifies the user who started the process.
- **EUID (Effective UID)** → Determines the identity generally used for permission checks.

## How They Work Together

Bandit demonstrates two uses of SUID:

1. **Level 19:** The `bandit20-do` executable allows commands to run with the effective privileges of its owner, `bandit20`.
2. **Level 20:** The `suconnect` executable is owned by `bandit21` and runs with that effective identity. This allows it to access the next level's password while communicating with a local TCP service.

Both levels demonstrate how SUID allows a program to perform operations beyond the invoking user's normal permissions. The elevated identity comes from the executable's owner, not from the level number or the logged-in user.

## Important Notes

- SUID does not automatically grant root privileges. A process receives the effective user ID of the executable's owner.
- SUID changes the effective UID, while the real UID remains associated with the user who launched the process.
- The `s` or `S` character appears in the owner's permission position in `ls -l` output.
- Elevated identity is inherited by child processes, which is why a SUID wrapper can make an ordinary command read a protected file.
- Vulnerable or improperly designed SUID binaries can create privilege-escalation risks.
- Linux may ignore the SUID bit under certain conditions, such as when a filesystem is mounted with the `nosuid` option.

## Related Concepts

- [Networking Services](./06-Networking-Services.md) — Local TCP communication involving the SUID-enabled `suconnect` binary in Level 20.
- [Shell Scripting, Cron & Shell Escape](./08-Shell-Scripting-Cron-Escape.md) — Understanding command execution and shell behavior in relation to process privileges.
