### Scanning Options 

| **Nmap Option**      | **Description**                                                        |
| -------------------- | ---------------------------------------------------------------------- |
| -iL                  | Performs defined scans against targets in provided 'hosts.lst' list.   |
| cut -d" " -f5        | space as delimiter and  selects the 5th field                          |
| `10.10.10.0/24`      | Target network range.                                                  |
| `-sn`                | Disables port scanning.                                                |
| `-Pn`                | Disables ICMP Echo Requests                                            |
| `-n`                 | Disables DNS Resolution.                                               |
| `-PE`                | Performs the ping scan by using ICMP Echo Requests against the target. |
| `--packet-trace`     | Shows all packets sent and received.                                   |
| `--reason`           | Displays the reason for a specific result.                             |
| `--disable-arp-ping` | Disables ARP Ping Requests.                                            |
| `--top-ports=<num>`  | Scans the specified top ports that have been defined as most frequent. |
| `-p-`                | Scan all ports.                                                        |
| `-p22-110`           | Scan all ports between 22 and 110.                                     |
| `-p22,25`            | Scans only the specified ports 22 and 25.                              |
| `-F`                 | Scans top 100 ports.                                                   |
| `-sS`                | Performs an TCP SYN-Scan.                                              |
| `-sA`                | Performs an TCP ACK-Scan.                                              |
| `-sU`                | Performs an UDP Scan.                                                  |
| `-sV`                | Scans the discovered services for their versions.                      |
| `-sC`                | Perform a Script Scan with scripts that are categorized as "default".  |
| `--script <script>`  | Performs a Script Scan by using the specified scripts.                 |
| `-O`                 | Performs an OS Detection Scan to determine the OS of the target.       |
| `-A`                 | Performs OS Detection, Service Detection, and traceroute scans.        |
| `-D RND:5`           | Sets the number of random Decoys that will be used to scan the target. |
| `-e`                 | Specifies the network interface that is used for the scan.             |
| `-S 10.10.10.200`    | Specifies the source IP address for the scan.                          |
| `-g`                 | Specifies the source port for the scan.                                |
| `--dns-server <ns>`  | DNS resolution is performed by using a specified name server.          |

## Output Options

|**Nmap Option**|**Description**|
|---|---|
|`-oA filename`|Stores the results in all available formats starting with the name of "filename".|
|`-oN filename`|Stores the results in normal format with the name "filename".|
|`-oG filename`|Stores the results in "grepable" format with the name of "filename".|
|`-oX filename`|Stores the results in XML format with the name of "filename".|

## Performance Options

|**Nmap Option**|**Description**|
|---|---|
|`--max-retries <num>`|Sets the number of retries for scans of specific ports.|
|`--stats-every=5s`|Displays scan's status every 5 seconds.|
|`-v/-vv`|Displays verbose output during the scan.|
|`--initial-rtt-timeout 50ms`|Sets the specified time value as initial RTT timeout.|
|`--max-rtt-timeout 100ms`|Sets the specified time value as maximum RTT timeout.|
|`--min-rate 300`|Sets the number of packets that will be sent simultaneously.|
|`-T <0-5>`|Specifies the specific timing template.|

# Host Discovery (Section -3)

##### TTL and OS Detection

The TTL (Time To Live) value in ICMP replies can help identify the target operating system during reconnaissance.

Common default TTL values:

- `64` → Linux / Unix
- `128` → Windows
- `255` → Network devices / routers
- `32` → Older operating systems
- `60` → Some macOS/Linux variants

Examples:

```text
ttl=128 → likely Windows
ttl=64  → likely Linux
ttl=255 → likely router/network appliance
```

TTL decreases by 1 each time a packet passes through a router, so the observed value may be slightly lower than the original.

Example:

```text
ttl=127 → likely Windows
ttl=63  → likely Linux
ttl=254 → likely router/network appliance
```

# Section 4

When comparing a normal TCP packet like:

```text
ttl=56 id=57322 iplen=44 seq=1699105818 win=1024 mss=1460
```

with:

```text
ttl=64 id=0 iplen=40 seq=0 win=0
```

the second one is likely a TCP RST packet. Since it is only used to reject/reset the connection, values like `seq=0` and `win=0` are minimal because no real TCP session is being maintained.

##### For information, 

``` -sT ``` creates logs on most systems and is easily detected by modern IDS/IPS solutions, ```-sS``` is considered more stealthy because they do not complete the full handshake, leaving the connection incomplete after sending the initial SYN packet.

# Saving the Results

- Normal output (`-oN`) with the `.nmap` file extension
- Grepable output (`-oG`) with the `.gnmap` file extension
- XML output (`-oX`) with the `.xml` file extension
- (`-oA`) to save the results in all formats. 

```xsltproc target.xml -o target.html```  to presents our results in a detailed and clear way in html format. 

#### ==Extra Knowledge== 

Nmap showed that port 25 (SMTP) was open, but when connecting manually with `nc`, the server revealed additional information:

```text
nc -nv ip port
220 inlane ESMTP Postfix (Ubuntu)
```

This message is called a **banner**. Many services automatically send a banner after a successful TCP connection to identify themselves, sometimes leaking useful information such as the software name, version, or operating system.

The network communication looked like this:

```text
1. SYN      → Client: "Can we connect?"
2. SYN-ACK  → Server: "Yes, let's connect."
3. ACK      → Client: "Connection established."
4. PSH-ACK  → Server: "Here is some data (the banner)."
5. ACK      → Client: "I received the data."
6. RST      → Reset/reject connection
7. FIN      → Gracefully close connection
```
