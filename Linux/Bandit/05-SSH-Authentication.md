# SSH Authentication

## Related Levels

- **Level 0** → Connecting to a remote server using SSH with username, hostname, and custom port
- **Level 13** → Authenticating via SSH public-key authentication using a private key available at this level
- **Level 18** → Bypassing restrictive `.bashrc` behavior by executing remote commands non-interactively

## Core Idea

SSH (Secure Shell) provides encrypted remote access to Linux systems through multiple authentication methods and execution modes. Beyond basic password login, SSH supports public-key authentication for credential-based access and non-interactive command execution for automation or bypassing shell restrictions.

A critical security concept demonstrated in Bandit Level 18 is the difference between interactive and non-interactive shells. When a user's `.bashrc` is configured to exit an interactive shell, standard SSH login may successfully authenticate but fail to provide a usable interactive session. Executing a remote command avoids opening the interactive session that triggers the logout behavior in this challenge, enabling command execution despite the restriction.

## Key Commands & Patterns

### 1. Basic SSH Connection with Custom Port

```bash
ssh -p 2220 bandit0@bandit.labs.overthewire.org
```

- `ssh` → Initiates an SSH client connection to a remote host.
- `-p 2220` → Port: specifies the remote SSH server port. Bandit uses a non-default port for its SSH service.
- `bandit0@host` → Username and hostname combined; `@` separates the username from the remote host.

### 2. Public-Key Authentication

```bash
ssh -i sshkey.private -p 2220 bandit14@bandit.labs.overthewire.org
```

- `-i sshkey.private` → Identity File: specifies the private key file used for public-key authentication.
- Public-key authentication proves identity cryptographically without transmitting the private key or a password. The corresponding public key must be configured on the server.
- SSH clients may refuse to use private keys with overly permissive file permissions. A common secure setting is `600` (owner read/write only).

### 3. Non-Interactive Remote Command Execution

```bash
ssh -p 2220 bandit18@bandit.labs.overthewire.org "cat readme"
```

- `"cat readme"` → Command string executed on the remote system without allocating an interactive TTY by default.
- This mode avoids opening the interactive login session whose `.bashrc` behavior terminates the shell, enabling command execution when normal interactive access is blocked.
- Output is returned directly to the local terminal; no persistent interactive session is established.

## How They Work Together

Bandit levels demonstrate SSH as both an access mechanism and a security boundary:

1. **Connection Foundation (Level 0):** Establishes basic SSH knowledge, including custom ports and `user@host` syntax within a lab environment.
2. **Credential Chain (Levels 12→13→14):** Level 12 provides the credentials needed to access Level 13, where a private SSH key is available for authenticating as Level 14. This links [Encoding & Compression](./04-Encoding-Compression.md) skills to authentication workflows.
3. **Shell Behavior Exploitation (Level 18):** Shows that successful authentication does not guarantee interactive access. Understanding shell initialization and the interactive/non-interactive distinction is essential for both offensive testing and defensive hardening.

These three aspects reflect real-world scenarios: accessing infrastructure through non-default ports, managing key-based authentication, and troubleshooting restricted environments where traditional interactive login fails despite valid credentials.

## Important Notes

- SSH encrypts traffic between the client and server, protecting credentials and commands from network eavesdropping.
- `-i` stands for Identity File. It specifies the path to the private key used for authentication.
- Private key files should have restrictive permissions (typically `600`). SSH clients may reject keys readable by other users as a security precaution.
- Non-interactive execution does not allocate a pseudo-terminal (TTY) by default. Commands requiring terminal interaction may fail unless `-t` is explicitly specified.
- Public-key authentication can provide stronger security than passwords when properly configured, but its effectiveness depends on key strength, private key protection, passphrase usage, and server-side settings.
- Custom SSH ports do not provide security through obscurity, but they may reduce noise from automated scanners targeting the default port 22.

## Related Concepts

- [Encoding & Compression](./04-Encoding-Compression.md) — Recovering the credentials needed to access the level containing the SSH private key.
- [Networking Services](./06-Networking-Services.md) — Understanding TCP connections and ports underlying SSH transport.
- [Permissions & SUID](./07-Permissions-SUID.md) — File permission requirements for SSH identity files and their security implications.
- [Shell Scripting, Cron & Shell Escape](./08-Shell-Scripting-Cron-Escape.md) — Shell initialization behavior and escaping restricted environments.
