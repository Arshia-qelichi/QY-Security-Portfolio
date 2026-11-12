# Networking Services

## Related Levels
- **Level 14** → Connecting to a local TCP service using `nc` to submit data and receive a response
- **Level 15** → Establishing a TLS-encrypted connection using `openssl s_client` when the service requires TLS
- **Level 16** → Scanning TCP ports with `nmap` and identifying services behind open ports
- **Level 20** → Using `nc` as a local TCP listener to receive data from another process via loopback
- **Level 24** → Automating repeated TCP requests to a service that validates credentials programmatically

## Core Idea
Network services enable communication between processes through standardized protocols. In TCP-based communication, a server listens on a specific port while clients initiate connections to exchange data. Understanding the protocol stack is essential: TLS operates *on top of* TCP, not as a replacement for it. Tools like `nc` handle raw TCP and do not implement TLS, while `openssl s_client` manages the TLS handshake and encryption/decryption before passing application data to the user.

Bandit demonstrates three fundamental networking skills: connecting to known services (`nc`), discovering unknown services (`nmap`), and handling protocol-specific requirements (`openssl`). These skills form the foundation of service enumeration, network troubleshooting, and security testing in penetration testing workflows.

## Key Commands & Patterns

### 1. Raw TCP Connection with Netcat
```bash
nc localhost 30000
```
- `nc` → Netcat: reads and writes data across TCP or UDP connections.
- `localhost` → Target hostname resolving to the loopback interface (`127.0.0.1` or `::1`).
- `30000` → Destination TCP port number.

Netcat can also operate in listen mode:
```bash
nc -l -p 30000
```
- `-l` → Listen Mode: waits for an incoming connection instead of initiating one.
- `-p 30000` → Port Specification: defines which local port to bind when listening.

### 2. TLS Connection with OpenSSL
```bash
openssl s_client -connect localhost:30001 -quiet
```
- `openssl s_client` → Starts a TLS client capable of negotiating encrypted sessions over TCP.
- `-connect localhost:30001` → Specifies the target host and port in `host:port` format.
- `-quiet` → Quiet Mode: suppresses session and certificate information. It also enables `-ign_eof`, which prevents standard input EOF from immediately closing the connection.

Unlike `nc`, this command performs the TLS handshake automatically, allowing interaction with services that require encryption. Sending plaintext through `nc` to a TLS-enabled port does not establish a valid TLS session because `nc` does not handle the TLS layer.

### 3. Port Scanning with Nmap
```bash
nmap -p 31000-32000 localhost
```
- `nmap` → Network exploration and security auditing tool.
- `-p 31000-32000` → Port Specification: scans only the specified port range instead of Nmap's default selection of commonly used ports.
- Reports port states:
  - `open` → An application is accepting connections on the port.
  - `closed` → The port is reachable, but no application is listening on it.
  - `filtered` → Nmap cannot determine whether the port is open because probes are blocked or receive no definitive response.

Port scanning identifies *where* services may exist but does not necessarily determine *what* protocol they use. Protocol identification requires subsequent connection attempts with appropriate tools.

### 4. Automating TCP Requests
```bash
echo "password 1234" | nc localhost 30002
```
- Pipes formatted input directly into a TCP connection without manual typing.
- Demonstrates how standard input can be used to automate interaction with a network service (Level 24).
- This is a single-request example. Repeated requests can be generated using shell scripts or loops, as demonstrated in Bandit Level 24.

Both `nc` and `openssl s_client` can accept standard input, making them compatible with shell pipelines and automated interactions.

### 5. Local TCP Listener for Inter-Process Communication
```bash
nc -l -p 4444
```
In Bandit Level 20, `nc` acts as a receiver for data sent by the `suconnect` program over the loopback interface. This demonstrates TCP as a mechanism for inter-process communication on the same host, where both sender and receiver reside locally but communicate through the network stack rather than shared memory or pipes.

## How They Work Together
These tools represent different stages of network service interaction:

1.  **Discovery:** Use `nmap` to identify open ports within a range. An open port indicates that an application is accepting connections but reveals nothing about its protocol or functionality.
2.  **Protocol Identification:** Connect with `nc` when testing a plain TCP service. If the response is unreadable or the service does not respond as expected, it may require a different application-layer protocol, such as TLS. Use `openssl s_client -quiet` when TLS is suspected.
3.  **Service Interaction:** Once the appropriate protocol is established, send expected input (passwords, PINs, or other data) and observe responses. Use standard input piping for automated or repeated interactions (Level 24).
4.  **Local Communication:** When another process communicates over TCP internally, `nc -l` serves as a lightweight endpoint to receive transmitted data without implementing custom socket code (Level 20).

This workflow mirrors real-world penetration testing: scan → identify protocol → interact → automate. Each tool addresses a specific part of network service interaction, and using an inappropriate tool (e.g., plain Netcat against a TLS service) can produce misleading results.

## Important Notes
-   A **port** is a logical communication endpoint associated with a network address and transport protocol. It helps the operating system deliver incoming traffic to the appropriate application.
-   TCP is connection-oriented: a three-way handshake must complete before application data can be exchanged.
-   A failed connection does not necessarily mean that no service exists. Filtering, routing problems, service availability, and protocol mismatches can all prevent successful communication.
-   `localhost` resolves to the loopback interface and does not send traffic over an external network. Services bound only to `127.0.0.1` are generally inaccessible from remote hosts.
-   Netcat transmits data without TLS encryption by default. Avoid using it for sensitive data over untrusted networks unless an external encryption layer is provided.
-   `openssl s_client -quiet` suppresses session and certificate information and enables `-ign_eof`. This can make its output easier to use in scripted interactions.
-   Nmap's `filtered` state means probes received no definitive response, possibly due to firewall rules, rate limiting, or packet loss. It does not confirm whether a service exists behind the filter.
-   Port scanning unauthorized systems may violate laws or terms of service. Always verify authorization before scanning targets outside controlled lab environments.
-   Binding to ports below 1024 typically requires root privileges on most Linux systems. Non-root users can generally bind to higher-numbered ports.
-   Netcat syntax varies between implementations (GNU Netcat, Ncat, and OpenBSD Netcat). The examples here reflect the variant available in the Bandit environment.

## Related Concepts
-   [SSH Authentication](./05-SSH-Authentication.md) — SSH as a specific TCP-based service with its own authentication layer
-   [Permissions & SUID](./07-Permissions-SUID.md) — Privileged processes communicating over local TCP in Level 20
-   [Shell Scripting, Cron & Shell Escape](./08-Shell-Scripting-Cron-Escape.md) — Automating TCP interactions and piping input to network tools
