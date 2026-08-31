### IMPORTANT

In penetration testing, always focus on both " what can you see and  what can you NOT see".  Question yourself, get familiar with that system, understand the system you will be penetrating. 

'**WHAT CAN YOU SEE, HOW CAN YOU SEE, WHY CAN YOU SEE, WHAT DO YOU GAIN FROM IT**' 

And also,

'**WHAT CAN YOU NOT SEE, HOW CAN YOU NOT SEE, WHY CAN YOU NOT SEE, WHAT DO YOU GAIN FROM IT**' 

The whole enumeration process is divided into three different levels:

|`Infrastructure-based enumeration`|`Host-based enumeration`|`OS-based enumeration`|
|---|---|---|
These layers are designed as follows:

|**Layer**|**Description**|**Information Categories**|
|---|---|---|
|`1. Internet Presence`|Identification of internet presence and externally accessible infrastructure.|Domains, Subdomains, vHosts, ASN, Netblocks, IP Addresses, Cloud Instances, Security Measures|
|`2. Gateway`|Identify the possible security measures to protect the company's external and internal infrastructure.|Firewalls, DMZ, IPS/IDS, EDR, Proxies, NAC, Network Segmentation, VPN, Cloudflare|
|`3. Accessible Services`|Identify accessible interfaces and services that are hosted externally or internally.|Service Type, Functionality, Configuration, Port, Version, Interface|
|`4. Processes`|Identify the internal processes, sources, and destinations associated with the services.|PID, Processed Data, Tasks, Source, Destination|
|`5. Privileges`|Identification of the internal permissions and privileges to the accessible services.|Groups, Users, Permissions, Restrictions, Environment|
|`6. OS Setup`|Identification of the internal components and systems setup.|OS Type, Patch Level, Network config, OS Environment, Configuration files, sensitive private files


# Infrastructure-based Enumeration

##### Tools

1. **crt.sh**: This one shows  Certificate Transparency logs. 
2.  **domain.glass**, **GrayHatWarfare**: infrastructure finding

| **Command**                                                                                                                                    | **Description**                              |
| ---------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------- |
| `curl -s https://crt.sh/\?q\=<target-domain>\&output\=json \| jq .`                                                                            | Certificate transparency.                    |
| `curl -s  link\&output\=json\| jq . \| grep name \| cut -d":" -f2 \| grep -v "CN=" \| cut -d'"' -f2 \| awk '{gsub(/\\n/,"\n");}1;' \| sort -u` | filtered by the unique subdomains            |
| `for i in $(cat subdomainlist);do host $i \| grep "has address" \| grep inlanefreight.com \| cut -d" " -f1,4;done`                             | To find Hosted Servers                       |
| `for i in $(cat subdomainlist);do host $i \| grep "has address" \| grep inlanefreight.com \| cut -d" " -f1,4 >> ip-addresses.txt;done`         | To save it in a file                         |
| `for i in $(cat ip-addresses.txt);do shodan host $i;done`                                                                                      | Scan each IP address in a list using Shodan. |


# Section 6 - FTP

**FTP**: It uses two channels such as port 20 for control and port 21 is for data. It has both active and passive mode, and respond in status code. Requires authentication, but anonymous FTP doesn't. 

**TFTP:**  Doesn't require authentication, uses UDP.

**vsFTPd:** One of the most used FTP servers on Linux-based distributions. The default configuration of vsFTPd can be found in `/etc/vsftpd.conf`. `/etc/ftpusers` is used to deny certain users access to the FTP service. The server show us more information with commands `debug` and `trace`. `ls_recurse_enable=YES` often set on the vsFTPd server to allows us to see all the visible content at once with `ls -R`.
##### FTP

| **Command**                                               | **Description**                                                         |
| --------------------------------------------------------- | ----------------------------------------------------------------------- |
| `ftp <FQDN/IP>`                                           | Interact with the FTP service on the target.                            |
| `nc -nv <FQDN/IP> 21`                                     | Interact with the FTP service on the target.                            |
| `telnet <FQDN/IP> 21`                                     | Interact with the FTP service on the target.                            |
| `openssl s_client -connect <FQDN/IP>:21 -starttls ftp`    | Interact with the FTP service on the target using encrypted connection. |
| `wget -m --no-passive ftp://anonymous:anonymous@<target>` | Download all available files on the target FTP server.                  |

## Extra
1. A daemon (e.g., `sshd`, `apache2`) runs continuously and listens for connections, whereas `inetd` acts as a super-server that listens on behalf of services and starts them only when a connection is received.


# Section 7 - SMB

| Feature             | SMB          | FTP           |
| ------------------- | ------------ | ------------- |
| Purpose             | File sharing | File transfer |
| Port                | 445          | 21            |
| Authentication      | Yes          | Yes           |
| Browse folders      | Yes          | Limited       |
| Network drives      | Yes          | No            |
| Windows integration | Excellent    | Minimal       |
### NetBIOS vs nmbd vs smbd  
  
| Component   | Purpose                          | Ports                     | What It Does                                              | Pentest Interest                           |
| ----------- | -------------------------------- | ------------------------- | --------------------------------------------------------- | ------------------------------------------ |
| **NetBIOS** | Naming & session layer           | 137/UDP, 138/UDP, 139/TCP | Lets hosts find each other by name and establish sessions | Hostnames, domains, user/share enumeration |
| **nmbd**    | NetBIOS service (Samba)          | 137/UDP, 138/UDP          | Handles NetBIOS name resolution and network browsing      | Enumerate hostnames, workgroups, domains   |
| **smbd**    | SMB file-sharing service (Samba) | 445/TCP, 139/TCP          | Serves files, printers, authentication, and shares        | Shares, credentials, sensitive files, RCEs |
Check in the configuration file and look for which services are allowed or not by
`cat /etc/samba/smb.conf | grep -v "#\|\;"` . If possible, then create a share, look at the man pages for Samba and configure it ourselves and experiment with the settings. Once we have adjusted `/etc/samba/smb.conf` to our needs, we have to restart the service on the server by `sudo systemctl restart smbd`. 

Smbclient also allows us to execute local system commands using an exclamation mark at the beginning (`!<cmd>`) without interrupting the connection.

##### SMB

| **Command**                                                                                                                                                  | **Description**                                                                       |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------- |
| `smbclient -N -L //<FQDN/IP>`                                                                                                                                | Null session authentication on SMB.                                                   |
| `smbclient //<FQDN/IP>/<share>`                                                                                                                              | Connect to a specific SMB share.                                                      |
| `rpcclient -U "" <FQDN/IP>`                                                                                                                                  | Interaction with the target using RPC.                                                |
| `for i in $(seq 500 1100);do rpcclient -N -U "" 10.129.14.128 -c "queryuser 0x$(printf '%x\n' $i)" \| grep "User Name\|user_rid\|group_rid" && echo "";done` | Brute Forcing User RIDs                                                               |
| `samrdump.py <FQDN/IP>`                                                                                                                                      | Username enumeration using Impacket scripts. (Alternative to Brute Forcing User RIDs) |
| `smbmap -H <FQDN/IP>`                                                                                                                                        | Enumerating SMB shares.                                                               |
| `crackmapexec smb <FQDN/IP> --shares -u '' -p ''`                                                                                                            | Enumerating SMB shares using null session authentication.                             |
| `enum4linux-ng.py <FQDN/IP> -A`                                                                                                                              | SMB enumeration using enum4linux.                                                     |
Always check for all the functions through Man or help page of a service, such as for rpcclient. Some examples are for rpcclient:

| **Query**                 | **Description**                                                    |
| ------------------------- | ------------------------------------------------------------------ |
| `srvinfo`                 | Server information.                                                |
| `enumdomains`             | Enumerate all domains that are deployed in the network.            |
| `querydominfo`            | Provides domain, server, and user information of deployed domains. |
| `netshareenumall`         | Enumerates all available shares.                                   |
| `netsharegetinfo <share>` | Provides information about a specific share.                       |
| `enumdomusers`            | Enumerates all domain users.                                       |
| `queryuser <RID>`         | Provides information about a specific user.                        |


# Section 8 NFS

**NFS (Network File System)** is a Linux/Unix file-sharing protocol that lets you access a remote directory as if it were a local folder on your machine. **NFSv4.1** improved NFS by adding support for faster access to files stored across multiple servers (**pNFS**) and allowing multiple network paths to the same server for better performance and reliability (**multipathing**). The RPC protocol ( port 111) handles the client data and NFS itself using port 2049 handles the files by checking UID,GID.

**Mounting an NFS share** means attaching a remote folder from another machine to a local directory on your system, so it behaves like a normal local folder. You can **mount a different NFS share into the same folder**, but only after unmounting the first one. It is important to note that if the `root_squash` option is set, we cannot edit the `backup.sh` file even as `root`.
##### NFS

| **Command**                                               | **Description**                              |
| --------------------------------------------------------- | -------------------------------------------- |
| `showmount -e <FQDN/IP>`                                  | Show available NFS shares.                   |
| `mount -t nfs <FQDN/IP>:/<share> ./target-NFS/ -o nolock` | Mount the specific NFS share to ./target-NFS |
| `umount ./target-NFS`                                     | Unmount the specific NFS share.              |
check setting with `cat /etc/exports`

root_squash     → Don't trust remote root
no_root_squash  → Trust remote root

nosuid          → Ignore SUID even if there is `s`  (SUID: For example, the `passwd` command, which modifies `/etc/passwd` and `/etc/shadow`, has the SUID bit set. This allows a regular user to change their own password even though the files are owned by root)

suid            → Respect SUID

Read-only       → View only
Writable        → Upload/edit files

 Writable + no_root_squash + suid = "Remote root can upload privileged files."

# Section 9 - DNS


FQDN                        = full address (www.mail.example.com)
TLD (top level)           = last part (.com, .net)
SLD (secondary level) = main domain (example in example.com)
Subdomain                = anything before SLD (www, mail, vpn)
Hostname                  = single machine name (web01, ns1)
Resolver                     = system that asks DNS questions for you
Domain                      = main name (example.com = SLD + TLD)

Some common DNS records:

| Record | Easy Meaning                            | Example                        |
| ------ | --------------------------------------- | ------------------------------ |
| A      | Name → IPv4 address                     | example.com → 1.2.3.4          |
| AAAA   | Name → IPv6 address                     | example.com → 2001:db8::1      |
| CNAME  | Nickname (points to another name)       | www → example.com              |
| MX     | Mail server                             | example.com → mail.example.com |
| TXT    | Extra text info (verification/security) | "google-site-verification=..." |
| NS     | Which DNS servers control domain        | ns1.example.com                |
| SOA    | Main DNS info (admin + settings)        | primary DNS + admin email      |
| PTR    | IP → name (reverse lookup)              | 1.2.3.4 → example.com          |
| SRV    | Where a service runs (host + port idea) | service → server               |
**Advanced Records**  

| Record Type | Simple Meaning               |
| ----------- | ---------------------------- |
| CAA         | Which CA can issue SSL certs |
| DNSKEY      | DNSSEC encryption keys       |
| NAPTR       | Advanced service routing     |
| TLSA        | Links TLS cert to domain     |
All DNS servers work with three different types of configuration files:

local   = list of domains (/etc/bind/named.conf.local) (suppose alex)
zone    = details per domain (/etc/bind/db.domain.com) (suppose alex's family tree)
reverse = IP lookup files (/etc/bind/db.10.129.14)
options = global rules

##### DNS

| **Command**                                                                                                   | **Description**                          |
| ------------------------------------------------------------------------------------------------------------- | ---------------------------------------- |
| `dig ns <domain.tld> @<nameserver>`                                                                           | NS request to the specific nameserver.   |
| `dig any <domain.tld> @<nameserver>`                                                                          | ANY request to the specific nameserver.  |
| `dig axfr <domain.tld> @<nameserver>`                                                                         | AXFR request to the specific nameserver. |
| `dnsenum --dnsserver <nameserver> --enum -p 0 -s 0 -o found_subdomains.txt -f ~/subdomains.list <domain.tld>` | Subdomain brute forcing.                 |
| `dig CH TXT version.bind 10.129.120.85`                                                                       | dig version query                        |

AXFR = full DNS zone copy from primary (master) server to secondary (slave) server over TCP 53. It keeps DNS servers in sync by copying ALL records (subdomains, IPs, mail, etc.) when changes happen. If misconfigured, attackers can request this transfer and get the entire DNS structure.

| AXFR                   | Entire DNS zone (everything at once) |
| ---------------------- | ------------------------------------ |
| normal DNS query (dig) | One record at a time                 |

##### for question 4 use this wordlist "/usr/share/wordlists/seclists/Discovery/DNS/fierce-hostlist.txt"

**Wordlist choice = how much you already know about the target**



# Section 10 - SMTP

SMTP = Send emails and IMAP / POP3 = Receive emails

| Port | Purpose |
|--------|---------|
| 25 | SMTP (server ↔ server) |
| 587 | SMTP + Authentication |
| 465 | SMTP over SSL/TLS |

**Components**

| Term | Meaning                               | Additional knowledge                                                                                                                                                                                                                                           |
| ---- | ------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| MUA  | Mail User Agent (email app)           |                                                                                                                                                                                                                                                                |
| MSA  | Mail Submission Agent (checks sender) | **Open Relay attack (`MSA` is also called `Relay` server)**: is like a Bad configuration (mynetworks = 0.0.0.0/0), Server sends mail without authentication. Use the `smtp-open-relay` nmap script to identify the target SMTP server as an open relay or not. |
| MTA  | Mail Transfer Agent (moves mail)      |                                                                                                                                                                                                                                                                |
| MDA  | Mail Delivery Agent (delivers mail)   |                                                                                                                                                                                                                                                                |

**Encryption**

| Command  | Purpose                             |                                                                                           |
| -------- | ----------------------------------- | ----------------------------------------------------------------------------------------- |
| EHLO     | Extended SMTP hello.                | To ask server/domain that What features do they support.                                  |
| STARTTLS | Upgrade plaintext connection to TLS | Without STARTTLS,  Username/password is visible but With STARTTLS,  Connection encrypted. |


**DNS Records Used**

| Record | Purpose            |
| ------ | ------------------ |
| MX     | Mail server        |
| SPF    | Allowed senders    |
| DKIM   | Email authenticity |

Default Configuration: `cat /etc/postfix/main.cf | grep -v "#" | sed -r "/^\s*$/d"`

##### Commands

|**Command**|**Description**|
|---|---|
|`telnet <FQDN/IP> 25`||
EHLO = Hello
AUTH = Login
VRFY = User exists?
MAIL FROM = From
RCPT TO = To
DATA = Message
QUIT = Bye

### HTB Question 2:

To get the valid usernames on that smtp service, try `smtp-user-enum -M VRFY -U f.txt -t 10.129.49.167 -v -w20` where you can use htb's wordlist (which i saved as f.txt) or any list searched online. I recommend to go through this website [SMTP User Enumeration Cheat Sheet — Email Account Discovery](https://vespersec.net/docs/network-enumeration-and-scanning/smtp-user-enumeration-cheat-sheet/) 


- **VRFY Scan**: `smtp-user-enum -M VRFY -u user -t mail.target.com`  (-u for one user, -U for many)

- **RCPT TO**: `swaks --to user@target.com --server mail.target.com --quit-after RCPT`  (RCPT enum = "Can I deliver to a local mailbox?", Open Relay = "Can I use you to deliver mail anywhere?")

- **Nmap Script**: `nmap -p25,587 --script smtp-commands,smtp-enum-users target`

- **Wordlist**: `/usr/share/wordlists/seclists/Usernames/Names/names.txt`


# Section 11- IMAP / POP3

| Protocol | Job                         | Port         | Easy Memory |
| -------- | --------------------------- | ------------ | ----------- |
| SMTP     | Send email                  | 25, 587, 465 | Send        |
| IMAP     | Read/manage email on server | 143, 993     | Sync        |
| POP3     | Download email from server  | 110, 995     | Download    |

#### IMAP vs POP3

| Feature | IMAP | POP3 |
|----------|------|------|
| Emails stay on server | ✅ | ❌ |
| Multiple devices sync | ✅ | ❌ |
| Folders supported | ✅ | ❌ |
| Download emails | Optional | Main purpose |

##### IMAP/POP3

| **Command**                                            | **Description**                         |
| ------------------------------------------------------ | --------------------------------------- |
| `curl -k 'imaps://<FQDN/IP>' --user <user>:<password>` | Log in to the IMAPS service using cURL. |
| `openssl s_client -connect <FQDN/IP>:imaps`            | Connect to the IMAPS service.           |
| `openssl s_client -connect <FQDN/IP>:pop3s`            | Connect to the POP3s service.           |


**IMAP** 

| Command | What it does |
|----------|--------------|
| LOGIN user pass | Log in to mailbox |
| LIST "" * | Show folders (INBOX, Sent, etc.) |
| SELECT INBOX | Open a mailbox |
| STATUS INBOX (MESSAGES) | Show mailbox info |
| SEARCH ALL | Find emails |
| FETCH 1 BODY[] | Read email #1 |
| FETCH 1 ENVELOPE | Show email headers |
| LOGOUT | Exit session |


**POP3** 

| Command | What it does |
|----------|--------------|
| USER name | Provide username |
| PASS pass | Login password |
| LIST | Show emails in mailbox |
| RETR 1 | Read email #1 |
| DELE 1 | Delete email #1 |
| STAT | Show number of emails |
| QUIT | Exit session |

#### question 1-4
after openssl scans, i got some server certificates and ticket ids, i was confused if they can be used. But SSL/TLS certificates cannot be used for authentication, but they often reveal hostnames, emails, and internal domains useful for enumeration and attack surface expansion.

#### Question 5-6
1. After opening connection with openssl or nc, scroll in the end and to login, type: `a login user pass`. (here a,b,c are like a command tag, you can do a login, b search etc)

2. List mail folders:   `a list "" *`

3. Select inbox:   `a select INBOX`

4. You’ll see something like: * 3 EXISTS , Meaning, There are 3 emails in this mailbox

5. List all emails (get IDs):   `a search all` Example output: * SEARCH 1 2 3 👉 These numbers are email IDs

6. Read emails: Read full email: `a fetch 1 body[]` Or full metadata + body: `a fetch 1 full`

7. Faster method (sometimes works): `a fetch 1:* body[]`



# Section 12 - SNMP

SNMP (Simple Network Management Protocol) is used to monitor and manage network devices like routers, switches, servers, and IoT devices, and it can also change configurations remotely. It mainly runs over UDP port 161 for queries/commands, while UDP port 162 is used for “traps,” which are automatic alerts sent by devices when events occur. SNMP uses a structured database called the MIB (Management Information Base), which defines what information can be accessed, and each item is identified using an OID (Object Identifier) in a numbered hierarchy. SNMP has different versions: v1 and v2c are widely used but insecure because they have no encryption and rely on community strings (like passwords) sent in plaintext, making them easy to intercept. SNMPv3 improves security by adding authentication (username/password) and encryption, but it is more complex to configure.

SNMP Daemon Config:  `cat /etc/snmp/snmpd.conf | grep -v "#" | sed -r '/^\s*$/d'`

##### SNMP

| **Command**                                       | **Description**                                     |
| ------------------------------------------------- | --------------------------------------------------- |
| `onesixtyone -c community-strings.list <FQDN/IP>` | Bruteforcing community strings of the SNMP service. |
| `snmpwalk -v2c -c <community string> <FQDN/IP>`   | Querying OIDs using snmpwalk.                       |
| `braa <community string>@<FQDN/IP>:.1.*`          | Bruteforcing SNMP service OIDs.                     |

Look for the below and more and if found, look for exploits!:  
- Hostname  
- Email/usernames  
- OS version  
- Installed software  
- Running services  
- Network info

SNMP OID = path to information (like folders); `1.3.6.1 = iso.org.dod.internet` and most useful SNMP data lives under it. `braa public@IP:.1.3.6.*` = enumerate everything under the main SNMP branch. 

Also from the scan, i saw two things repeatedly, which are "Counter = total count since boot (Only goes **up** (counts events))  
Gauge = current status/CPU usage, running processes, disk space (↑ or ↓)""

| Some Common OID        | Meaning                             |
| ---------------------- | ----------------------------------- |
| 1.3.6.1.2.1.1          | System info (hostname, contact, OS) |
| 1.3.6.1.2.1.4          | IP/network info                     |
| 1.3.6.1.2.1.6          | TCP info                            |
| 1.3.6.1.2.1.7          | UDP info                            |
| 1.3.6.1.2.1.25.1       | Host info (uptime, boot)            |
| 1.3.6.1.2.1.25.4       | Running processes                   |
| 1.3.6.1.2.1.25.6.3.1.2 | Installed software                  |
| 1.3.6.1.4.1            | Vendor-specific info                |

## Question 1-3.

Download wordlist from seclists to get community string first with onesixtyone.


# Section 13 - MySQL


Port 3306. 

One thing I learned: 

Your data
├── users
├── employees
└── products

Metadata about your data
├── information_schema (its like "database of databases")
└── sys/system schema (its like per internal database of the DBMS itself)

so, `SELECT table_name FROM information_schema.tables;` shows metadata about tables.

|**Command**|**Description**|
|---|---|
|`mysql -u <user> -p<password> -h <FQDN/IP>`|Login to the MySQL server.|

| Command                                                                                            | Purpose                       |
| -------------------------------------------------------------------------------------------------- | ----------------------------- |
| SHOW DATABASES;                                                                                    | List databases                |
| USE dbname;                                                                                        | Select database               |
| USE sys;                                                                                           | Admin/performance views       |
| USE information_schema;                                                                            | Metadata database             |
| SHOW TABLES;                                                                                       | List tables                   |
| DESCRIBE table;                                                                                    | Show columns                  |
| SHOW COLUMNS FROM table;                                                                           | Show table structure          |
| SELECT * FROM table;                                                                               | View all data                 |
| SELECT * FROM table LIMIT 10;                                                                      | View first 10 rows            |
| SELECT * FROM table WHERE column='value';                                                          | Filter rows                   |
| SELECT VERSION();                                                                                  | MySQL version                 |
| SELECT @@hostname;                                                                                 | Server hostname               |
| SELECT DATABASE();                                                                                 | Current database              |
| SELECT USER();                                                                                     | Logged-in user                |
| SELECT CURRENT_USER();                                                                             | Effective user                |
| SELECT user,host FROM mysql.user;                                                                  | List MySQL accounts           |
| SHOW GRANTS;                                                                                       | Show your privileges          |
| SHOW PROCESSLIST;                                                                                  | Active connections            |
| SHOW VARIABLES;                                                                                    | Server settings               |
| SHOW STATUS;                                                                                       | Server status                 |
| SELECT @@datadir;                                                                                  | Database file location        |
| SELECT @@secure_file_priv;                                                                         | File access restrictions      |
| SHOW VARIABLES LIKE '%file%';                                                                      | File-related settings         |
| SELECT LOAD_FILE('/etc/passwd');                                                                   | Read file (if allowed)        |
| SELECT schema_name FROM information_schema.schemata;                                               | List databases (metadata)     |
| SELECT table_schema, table_name FROM information_schema.tables;                                    | List all databases and tables |
| SELECT table_name FROM information_schema.tables WHERE table_schema='db';                          | Tables in a specific database |
| SELECT column_name FROM information_schema.columns WHERE table_name='users';                       | Columns in a table            |
| SELECT column_name FROM information_schema.columns WHERE table_schema='db' AND table_name='users'; | Columns in a specific table   |
| SELECT * FROM information_schema.tables LIMIT 10;                                                  | Table metadata                |
| SELECT * FROM information_schema.columns LIMIT 10;                                                 | Column metadata               |
| EXIT; / QUIT;                                                                                      | Exit MySQL                    |

## Question 2

mysql was not logging me in and was showing TLS/SSL error. Then i tried `mysql -u robin -probin -h 10.129.42.195 --skip-ssl` 


# Section 14 - MSSQL

Port 1433

| Topic              | Meaning                                                                                                 |
| ------------------ | ------------------------------------------------------------------------------------------------------- |
| MSSQL              | Microsoft relational database system (Windows-focused)                                                  |
| Purpose            | Store and manage application data                                                                       |
| Common stack       | IIS + ASP.NET + MSSQL                                                                                   |
| Default admin user | sa                                                                                                      |
| System databases   | master (track system info), model(track all new and old database), msdb(schedule jobs & alerts), tempdb |
| Client tools       | SSMS (GUI), impacket-mssqlclient (CLI) etc.                                                             |
| Authentication     | SQL login or Windows Authentication (AD/local)                                                          |

| **Command**                                     | **Description**                                          |
| ----------------------------------------------- | -------------------------------------------------------- |
| `mssqlclient.py <user>@<FQDN/IP> -windows-auth` | Log in to the MSSQL server using Windows authentication. |


**MSSQL Common Commands**

| Command                              | Purpose                                |
| ------------------------------------ | -------------------------------------- |
| SELECT @@version;                    | Show MSSQL version                     |
| SELECT SYSTEM_USER;                  | Current logged-in user                 |
| SELECT USER_NAME();                  | Current database user                  |
| SELECT name FROM sys.databases;      | List all databases                     |
| USE dbname;                          | Switch to a database                   |
| SELECT name FROM sys.tables;         | List tables in current DB              |
| SELECT * FROM table;                 | View all data                          |
| SELECT TOP 10 * FROM table;          | Show first 10 rows                     |
| EXEC xp_cmdshell 'whoami';           | Run system commands (if enabled)       |
| SELECT is_srvrolemember('sysadmin'); | Check admin privileges                 |
| SELECT * FROM sys.sql_logins;        | List SQL logins                        |
| SELECT @@servername;                 | Show server name                       |
| SELECT host_name();                  | Show host info                         |
| EXEC sp_helpdb;                      | Show database info                     |
| EXEC sp_who;                         | Show active sessions                   |
| EXEC sp_who2;                        | Detailed session info                  |
| GO                                   | Execute batch of commands (SQL Server) |
| EXIT                                 | Leave MSSQL session                    |
Another thing i learned, if you get named pipe like `Named pipe: \\10.129.201.248\pipe\sql\query` , it is just another way to talk to MSSQL besides normal TCP (port 1433).

## Question 1

`sudo nmap --script ms-sql-info,ms-sql-empty-password,ms-sql-xp-cmdshell,ms-sql-config,ms-sql-ntlm-info,ms-sql-tables,ms-sql-hasdbaccess,ms-sql-dac,ms-sql-dump-hashes --script-args mssql.instance-port=1433,mssql.username=sa,mssql.password=,mssql.instance-name=MSSQLSERVER -sV -p 1433 10.129.51.178`


# Section 15 - Oracle TNS

Oracle TNS is a server which is a communication protocol that facilitates communication between Oracle databases and applications over networks.

- Default port: **1521/TCP**
- Used for client ↔ Oracle DB communication
- Supports SSL/TLS encryption
- Main config files, Usually located in:   `$ORACLE_HOME/network/admin`: 
  - `tnsnames.ora` (client side)
  - `listener.ora` (server side)
- Default creds to remember:
  - `CHANGE_ON_INSTALL` (for oracle 9)
  - `dbsnmp` (for oracle DBSNMP)


SID (System Identifier) = unique name of an Oracle database instance (Instance = a running Oracle database process that manages a database.).

Connection usually requires: IP, Port (1521), SID (Examples: XE, ORCL, PROD) where Wrong SID = connection fails.

In short ,

IP = Which server?
Port = Which Oracle listener/service? (usually 1521)
Database = The actual stored data
Instance = Running Oracle process managing the database
SID = Name of the instance (e.g., ORCL, XE)
Service Name = Which database/application to connect to

Before enumerating TNS, install Odat, [GitHub - quentinhardy/odat: ODAT: Oracle Database Attacking Tool · GitHub](https://github.com/quentinhardy/odat) 
##### Oracle TNS

| **Command**                                                                                                          | **Description**                                                                                         |
| -------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------- |
| `sudo nmap -p1521 -sV 10.129.204.235 --open --script oracle-sid-brute`                                               | nmap sid brute forcing                                                                                  |
| `./odat.py all -s <FQDN/IP>`                                                                                         | Perform a variety of scans to gather information about the Oracle database services and its components. |
| `sqlplus <user>/<pass>@<FQDN/IP>/<db>`                                                                               | Log in to the Oracle database.                                                                          |
| `./odat.py utlfile -s <FQDN/IP> -d <db> -U <user> -P <pass> --sysdba --putFile C:\\insert\\path file.txt ./file.txt` | Upload a file with Oracle RDBMS.                                                                        |
Common roles:

| Role     | Meaning      |
| -------- | ------------ |
| CONNECT  | Login        |
| RESOURCE | Create stuff |
| DBA      | Admin        |
| SYSOPER  | Operate DB   |
| SYSDBA   | Super Admin  |

| Command                                           | Question it Answers                        |
| ------------------------------------------------- | ------------------------------------------ |
| `SELECT USER FROM dual;`                          | Who am I?                                  |
| `SELECT * FROM user_role_privs;`                  | What permissions/roles do I have?          |
| `SELECT username FROM all_users;`                 | What users exist?                          |
| `SELECT table_name FROM all_tables;`              | What tables can I access?                  |
| `SELECT * FROM <table> FETCH FIRST 10 ROWS ONLY;` | What's inside this table?                  |
| `SELECT banner FROM v$version;`                   | What Oracle version is running?            |
| `sqlplus user/pass@IP/XE as sysdba`               | Can I become an administrator?             |
| `SELECT name,password FROM sys.user$;`            | Can I access password hashes?              |
| `SELECT * FROM users;`                            | Are there user accounts or credentials?    |
| `SELECT * FROM employees;`                        | Is there employee information?             |
| `SELECT * FROM accounts;`                         | Are there accounts, passwords, or secrets? |
| `EXIT;`                                           | How do I leave Oracle?                     |


#### Notes

I saw this kind of (`SYS`, `SYSTEM`, `OUTLN`, etc. in `sys.user$`) name with password in sysuser and i didnt know if they are users or not and if can i login with what given. So yeah, after my understanding, you can login with system, pass @ ip after cracking the password from hashes.

## Question 1

I had encountered problems while installing odat by following there github tutorials. but then i followed htb's tutorial which is given below:

```
sudo apt-get update
sudo apt-get install -y build-essential python3-dev libaio1
cd ~
wget https://files.pythonhosted.org/packages/source/c/cx_Oracle/cx_Oracle-8.3.0.tar.gz
tar xzf cx_Oracle-8.3.0.tar.gz
cd cx_Oracle-8.3.0
python3 setup.py build
sudo python3 setup.py install
cd ~
git clone https://github.com/quentinhardy/odat.git
cd odat/
pip install python-libnmap
git submodule init
git submodule update
sudo apt-get install python3-scapy -y
sudo pip3 install colorlog termcolor passlib python-libnmap
sudo apt-get install build-essential libgmp-dev -y
pip3 install pycryptodome
pip3 install openpyxl
./odat.py -h
```

for sqlplus:

```
sudo apt install oracle-instantclient-sqlplus
```

Then i came across an error and to fix it:
```
sudo sh -c "echo /usr/lib/oracle/12.2/client64/lib > /etc/ld.so.conf.d/oracle-instantclient.conf";sudo ldconfig
```

Go with scott/tiger as it takes a huge time to scan and doesnt really provide any creds.


# Section 16 - IPMI

**IPMI (UDP 623)** is a hardware management interface that runs through a BMC (Baseboard Management Controller) and works independently of the OS. IPMI/BMC is a separate mini-computer on the motherboard. It allows admins to monitor, reboot, power on/off, access BIOS, and recover systems remotely. The server itself can be powered off (like you shut down the laptop but didnt unplug the power cable) and  so IPMI/BMC will be working allowing remote management.  Access to IPMI is nearly equivalent to physical access to the machine.

IPMI 2.0 (RAKP) can leak a salted password hash for a valid username before authentication, allowing attackers to crack the password offline and then able to SSH into many critical servers in the environment as the root user and gain access to web management consoles for various network monitoring tools.

Default creds fail → Enumerate valid user → Obtain RAKP hash → Crack with Hashcat (mode 7300) → Login to IPMI/BMC.

Some unique default passwords to keep in our cheatsheets include:

| Common BMCs     | Username      | Password                                                                                                                                                                                                                                                                          |
| --------------- | ------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Dell iDRAC      | root          | calvin                                                                                                                                                                                                                                                                            |
| HP iLO          | Administrator | randomized 8-character string consisting of numbers and uppercase letters. (we can use this Hashcat mask attack command `hashcat -m 7300 ipmi.txt -a 3 ?1?1?1?1?1?1?1?1 -1 ?d?u` which tries all combinations of upper case letters and numbers for an eight-character password.) |
| Supermicro IPMI | ADMIN         | ADMIN                                                                                                                                                                                                                                                                             |

##### IPMI

| **Command**                                     | **Description**                   |
| ----------------------------------------------- | --------------------------------- |
| `auxiliary/scanner/ipmi/ipmi_version`           | metasploit PMI version detection. |
| `auxiliary/scanner/ipmi/ipmi_dumphashes`        | Dump IPMI hashes.                 |
| `sudo nmap -sU --script ipmi-version -p 623 ip` |                                   |
Experimenting with different word lists is crucial for obtaining the password from the acquired hash. After getting the password, you can typically log in by opening `https://<IP>` in a browser or by SSH if enabled.

## Question 2

i used this wordlist [SecLists/Passwords/Common-Credentials/100k-most-used-passwords-NCSC.txt at master · danielmiessler/SecLists · GitHub](https://github.com/danielmiessler/SecLists/blob/master/Passwords/Common-Credentials/100k-most-used-passwords-NCSC.txt)

`hashcat -m 7300 -a 0 -w 3  g.txt 1.txt` where g.txt contains the hash and 1.txt is the wordlist.


# Section 17 - Linux Remote Management Protocols

### SSH 

SSH (Secure Shell) is a secure protocol used to remotely manage systems over **TCP 22**. It allows remote login, command execution, file transfer (SCP/SFTP), and port forwarding. SSH encrypts all communication between the client and server, making it safe to use over public networks. Always prefer **SSH-2**, as SSH-1 is outdated and insecure.


Authentication flow:

```text
Client creates:
  Public Key + Private Key

Public Key → copied to server
Private Key → stays on client

Server sends challenge
↓
Client solves it using Private Key
↓
Server verifies using Public Key
↓
Login succeeds
```


##### Important SSH Files

| File | Purpose |
|--------|--------|
| `/etc/ssh/sshd_config` | SSH server configuration |
| `~/.ssh/id_rsa` | Private key |
| `~/.ssh/id_rsa.pub` | Public key |
| `~/.ssh/authorized_keys` | Public keys allowed to log in |

##### Dangerous SSH Settings

- `PermitRootLogin yes` → Direct root login allowed
- `PermitEmptyPasswords yes` → Empty passwords allowed
- `PasswordAuthentication yes` → Password brute-forcing possible
- `Protocol 1` → Old insecure SSH version
- `X11Forwarding yes` → Larger attack surface
- `AllowTcpForwarding yes` → Port forwarding abuse possible
- `PermitTunnel yes` → Tunneling abuse possible


Tool: ssh-audit

| **Command**                                                 | **Description**                                       |
| ----------------------------------------------------------- | ----------------------------------------------------- |
| `ssh-audit.py <FQDN/IP>`                                    | Remote security audit against the target SSH service. |
| `ssh <user>@<FQDN/IP>`                                      | Log in to the SSH server using the SSH client.        |
| `ssh -i private.key <user>@<FQDN/IP>`                       | Log in to the SSH server using private key.           |
| `ssh <user>@<FQDN/IP> -o PreferredAuthentications=password` | Enforce password-based authentication.                |

### Rsync 

**Rsync** is a file synchronization and backup service that runs on **TCP 873**. It is commonly used to copy files between systems and keep directories synchronized. Its main feature is the **delta-transfer algorithm**, which means that after the first sync, it transfers only the changed parts of files instead of sending the entire file again. Misconfigured Rsync shares may allow anonymous users to list directories and download sensitive files such as backups, configuration files, credentials, and SSH keys.


| Command | Purpose |
|----------|----------|
| `nmap -sV -p 873 <IP>` | Check if Rsync is running |
| `nc -nv <IP> 873` | Connect to Rsync service |
| `#list` | List available shares/modules |
| `rsync -av --list-only rsync://<IP>/<share>` | List files in a share |
| `rsync -av rsync://<IP>/<share>` | Download/sync entire share |
| `rsync -av -e ssh rsync://<IP>/<share>` | Use SSH transport |
| `rsync -av -e "ssh -p2222" rsync://<IP>/<share>` | Use SSH on custom port |

### R-Services 

**R-Services** are old Unix remote-access services that existed before SSH. They allow remote login, command execution, and file copying, but **send data and credentials in plaintext (unencrypted)**, making them vulnerable to sniffing and MITM attacks. They rely heavily on trust relationships using **`/etc/hosts.equiv`** (system-wide trust) and **`.rhosts`** (per-user trust). Misconfigured trust files can allow login **without a password**, which is why SSH largely replaced R-services.


| Port | Service |
|--------|--------|
| TCP 512 | rexec |
| TCP 513 | rlogin |
| TCP 514 | rsh / rcp |


| Command | Purpose |
|----------|----------|
| `rlogin <IP> -l <user>` | Remote login |
| `rsh <IP> command` | Execute remote command |
| `rcp file user@host:/path` | Copy files remotely |
| `rwho` | Show logged-in users on network |
| `rusers -al <IP>` | Detailed user session info |
| `nmap -sV -p 512,513,514 <IP>` | Check for R-services |

##### Trust Files

| File | Purpose |
|--------|--------|
| `/etc/hosts.equiv` | Trusted hosts for all users |
| `~/.rhosts` | Trusted hosts for one user |

Example:

```text
htb-student 10.0.17.5
+           10.0.17.10
+           +
```

**`+` = wildcard (trust anything)** ← very dangerous.

##### Quick Workflow

```text
Ports 512/513/514 open
↓
Try rlogin/rsh
↓
Check for trusted hosts (.rhosts / hosts.equiv)
↓
Enumerate users with rwho/rusers
```

I wanted to check the exploit for this one, like it was said, if you find anything then always look for exploit and try it. [Penetration-Testing-Cheat-Sheet/Enumeration/Berkley-R-Services/BerkeleyR.md at master · curtishoughton/Penetration-Testing-Cheat-Sheet · GitHub](https://github.com/curtishoughton/Penetration-Testing-Cheat-Sheet/blob/master/Enumeration/Berkley-R-Services/BerkeleyR.md) 


# Section 18 - Windows Remote Management Protocols

### RDP (Remote Desktop Protocol)

RDP is Microsoft's remote desktop protocol that allows you to view and control a Windows computer's graphical desktop remotely, as if you were sitting in front of it. It commonly uses **TCP/3389** (and sometimes **UDP/3389**) and is often used by administrators for remote management. To access an RDP server from the Internet, port **3389** must be open on the firewall. If the server is behind a router using NAT (NAT hides private IPs behind a public IP, so the Internet sees the router's public IP, not the internal machine's private IP.), **port forwarding** must be configured so traffic reaching port 3389 is sent to the correct internal machine (as in to that private ip). SSH = Remote terminal, RDP = Remote desktop.

ms-wbt-server = RDP is present.
### NLA (Network Level Authentication)

NLA is an RDP security feature that requires users to authenticate **before** a remote desktop session is created. Without NLA, the server first presents a login screen and then checks credentials; with NLA, credentials are verified first, reducing resource usage and improving security. Most modern Windows systems enable NLA by default. When Nmap interacts with an RDP service, it may send the cookie `mstshash=nmap`, which can identify the connection as an Nmap scan. Security tools such as EDR, IDS, or threat hunters may detect this fingerprint and generate alerts or block the source. 

## WinRM (Windows Remote Management)

WinRM is Microsoft's remote command-line management service, similar to SSH on Linux. It allows administrators to remotely run PowerShell commands, manage Windows systems, and open remote shells without using RDP. WinRM uses port **5985 (HTTP)** or **5986 (HTTPS)** and is enabled by default on modern Windows Server versions. 

**Memory Trick:** `RDP = Remote Desktop (GUI)` | `WinRM = Remote PowerShell (CLI)`

## WMI (Windows Management Instrumentation)

WMI is Windows' built-in management system that allows administrators to view and change almost anything on a Windows machine. WMI communication always takes place on `TCP` port `135`. For example:

```
Task Manager shows running processes
↓
Task Manager gets that information from Windows
↓
WMI can give you that same information through commands
```


##### Windows Remote Management

| **Command**                                                                                        | **Description**                                    |
| -------------------------------------------------------------------------------------------------- | -------------------------------------------------- |
| `./rdp-sec-check.pl <FQDN/IP>`                                                                     | Check the security settings of the RDP service.    |
| `xfreerdp /u:<user> /p:"<password>" /v:<FQDN/IP>`                                                  | Log in to the RDP server from Linux.               |
| `evil-winrm -i <FQDN/IP> -u <user> -p <password>`                                                  | Log in to the WinRM server (by linux).             |
| `wmiexec.py <user>:"<password>"@<FQDN/IP> "<system command>"`                                      | Execute command using the WMI service.             |
| /usr/share/doc/python3-impacket/examples/wmiexec.py Cry0l1t3:"P455w0rD!"@10.129.201.248 "hostname" |                                                    |
| `Test-WsMan ip`                                                                                    | to check WinRM server running or not (by windows). |

# All ports found on this module

|  Port | Protocol / Transport Mentioned | Service / Context            | Connected Section | Purpose                                               |
| ----: | ------------------------------ | ---------------------------- | ----------------- | ----------------------------------------------------- |
|    20 | TCP                            | FTP                          | 6                 | FTP data/control channel mentioned in FTP explanation |
|    21 | TCP                            | FTP                          | 6, 7; Final Lab 1 | FTP service / FTP connection                          |
|    22 | TCP                            | SSH                          | 17; Final Lab 1   | Remote Linux shell                                    |
|    25 | TCP                            | SMTP                         | 10, 11            | SMTP server-to-server email                           |
|    53 | TCP                            | DNS / AXFR                   | 9; Final Lab 1    | DNS zone transfer                                     |
|    80 | TCP                            | HTTP                         | Module Summary    | Web server                                            |
|   110 | TCP                            | POP3                         | 11                | Email download                                        |
|   111 | Not specified                  | RPC / NFS support            | 8                 | RPC service used with NFS                             |
|   135 | TCP                            | WMI                          | 18                | Windows Management Instrumentation                    |
|   137 | UDP                            | NetBIOS / nmbd               | 7                 | NetBIOS name resolution                               |
|   138 | UDP                            | NetBIOS / nmbd               | 7                 | NetBIOS browsing / datagram service                   |
|   139 | TCP                            | NetBIOS / SMB                | 7                 | NetBIOS session / SMB                                 |
|   143 | TCP                            | IMAP                         | 11                | Read/manage email on server                           |
|   161 | UDP                            | SNMP                         | 12                | SNMP queries / commands                               |
|   162 | UDP                            | SNMP traps                   | 12                | Automatic SNMP alerts                                 |
|   443 | TCP                            | HTTPS                        | Module Summary    | Secure web server                                     |
|   445 | TCP                            | SMB / smbd                   | 7                 | SMB file sharing                                      |
|   465 | TCP                            | SMTP over SSL/TLS            | 10, 11            | Secure SMTP                                           |
|   512 | TCP                            | rexec                        | 17                | R-Services remote command execution                   |
|   513 | TCP                            | rlogin / rwho                | 17                | Remote login / user enumeration                       |
|   514 | TCP                            | rsh / rcp                    | 17                | Remote shell / remote copy                            |
|   587 | TCP                            | SMTP + Authentication        | 10, 11            | Authenticated mail submission                         |
|   623 | UDP                            | IPMI                         | 16                | Hardware/server management                            |
|   873 | TCP                            | Rsync                        | 17                | File synchronisation and backups                      |
|   993 | TCP                            | IMAPS                        | 11                | Secure IMAP                                           |
|   995 | TCP                            | POP3S                        | 11                | Secure POP3                                           |
|  1433 | TCP                            | MSSQL / Microsoft SQL Server | 14                | Microsoft SQL Server database                         |
|  1521 | TCP                            | Oracle TNS                   | 15                | Oracle database listener                              |
|  2049 | Not specified                  | NFS                          | 8                 | NFS file access                                       |
|  2121 | TCP                            | FTP                          | Final Lab 1       | Alternative FTP service found during enumeration      |
|  2222 | TCP                            | Custom SSH port              | 17                | SSH custom-port example for Rsync transport           |
|  3306 | TCP                            | MySQL                        | 13                | MySQL database                                        |
|  3389 | TCP / UDP                      | RDP                          | 18                | Windows Remote Desktop                                |
|  5432 | TCP                            | PostgreSQL                   | Module Summary    | PostgreSQL database                                   |
|  5985 | TCP                            | WinRM HTTP                   | 18                | Remote PowerShell / CLI                               |
|  5986 | TCP                            | WinRM HTTPS                  | 18                | Encrypted WinRM                                       |
| 27017 | TCP                            | MongoDB                      | Module Summary    | MongoDB database                                      |

# HTB Academy – Footprinting Final Lab 1


#### 1. Initial Enumeration

Found open ports: 21 FTP, 22 SSH, DNS, 2121 FTP.

Initially,  i tried downloading all the files from ftp (2121) but found nothing. Then after logging in with telnet connection for FTP and trying a few commands,  FTP returned `425 Unable to build data connection: Connection refused`. 

**Reason:** FTP uses **two connections**:

- **Control connection** → login and commands.
    
- **Data connection** → `LIST`, `GET`, `PUT`, etc.

Then I tried  by typing `help` i saw `pasv` command enlisted to Enable **Passive Mode** or you reconnect using `ftp -p <IP>`, then some commands ran but still could not find anything i guess. 

Then i tried the below:

Tried SSH using a password:

```bash
ssh ceil@10.129.71.96 -o PreferredAuthentications=password
```

Result:

```text
Permission denied (publickey)
```

**Meaning:** SSH only accepts **public key authentication**, not passwords.

#### 2. Enumerate FTP

Login:

```bash
ftp 10.129.71.96 21
```

After logging in:

```ftp
pwd
ls
```

Initially, `ls` showed nothing because the current directory appeared empty.

Used a detailed listing instead:

```ftp
ls -la
```

Output:

```text
.bash_history
.bashrc
.profile
.cache/
.ssh/
```


#### 3. Enumerate `.ssh`

```ftp
cd .ssh
ls -la
```

Found:

```text
authorized_keys
id_rsa
id_rsa.pub
```

#### 4. Download the Private Key

```ftp
get id_rsa
get id_rsa.pub
get authorized_keys
```

#### 5. Prepare the Key

SSH refuses keys with insecure permissions.

```bash
chmod 600 id_rsa
```

#### 6. Login via SSH

```bash
ssh -i id_rsa ceil@10.129.71.96
```

This authenticates using the **private key** instead of a password.

Then just try discovering the flag.

# HTB Academy – Footprinting Final Lab 2

### (NFS → SMB → WinRM → SQL → HTB Password)

Okay, so its gonna be a long one, but i happily learned everything included here.

**Goal:** My goal was to obtain the password for the **HTB** user. I started with normal enumeration and eventually abused an exposed NFS share, authenticated to SMB, gained Administrator access, and finally extracted the HTB password from a SQL database.

### Reconnaissance

I started with an Nmap scan.

```bash
nmap -sC -sV 10.129.202.41
```

#### Open Ports Found

| Port | Service |
|------|---------|
| 111 | rpcbind |
| 135 | MSRPC |
| 139 | NetBIOS |
| 445 | SMB |
| 2049 | NFS |
| 3389 | RDP |
| 5985 | WinRM |

The most interesting services were **SMB, NFS, WinRM, and RDP**.

---

### SMB Enumeration

I first checked for anonymous SMB access.

```bash
smbclient -N -L //10.129.202.41
```

**Result**

```
NT_STATUS_ACCESS_DENIED
```

Then I tried `rpcclient`.

```bash
rpcclient -U HTB 10.129.202.41
```

**Result**

```
NT_STATUS_LOGON_FAILURE
```

Finally, I attempted enumeration with CrackMapExec.

```bash
crackmapexec smb 10.129.202.41 --shares -u '' -p ''
```

**Result**

```
STATUS_ACCESS_DENIED
```

Anonymous SMB enumeration was not possible.

---

### NFS Enumeration

Since port **2049 (NFS)** was open, I checked the exported shares.

```bash
showmount -e 10.129.202.41
```

**Result**

```
/TechSupport (everyone)
```

This was a good sign because the share was accessible by everyone.

---

#### Mistake #1 – Mount Point Didn't Exist

I immediately tried mounting the share.

```bash
mount -t nfs 10.129.202.41:/TechSupport ./target-NFS -o nolock
```

**Error**

```
mount point ./target-NFS does not exist
```

### Fix

I created a mount point first.

```bash
mkdir x
sudo mount -t nfs 10.129.202.41:/TechSupport ./x -o nolock
```

The mount succeeded.

---

#### Mistake #2 – Permission Denied

After mounting, I tried accessing the directory.

```bash
cd x
```

**Error**

```
Permission denied
```

I also tried:

```bash
chmod 600 x
```

but still received **Permission denied**. I realized the mounted files belonged to another UID, so instead of changing permissions I simply listed the contents using sudo.

```bash
sudo ls -la x
```

This worked.

---

#### Finding Credentials

Inside the mounted share were hundreds of ticket files. Almost all were **0 bytes**, except one:

```
ticket4238791283782.txt
```

The file contained SMTP credentials.

```text
user="alex"
password="lol123!mD"
from="alex.g@web.dev.inlanefreight.htb"
```

### First Credentials Found

```
alex
lol123!mD
```

---

#### Validating Credentials

I immediately verified the credentials.

```bash
crackmapexec smb 10.129.202.41 -u alex -p 'lol123!mD'
```

**Result**

```
SUCCESS
```

---

### Enumerating SMB Shares

```bash
crackmapexec smb 10.129.202.41 -u alex -p 'lol123!mD' --shares
```

### Interesting Shares

- devshare
- Users

---

#### Accessing devshare

I connected to the share.

```bash
smbclient //10.129.202.41/devshare -U 'alex%lol123!mD'
```

After listing the files, I found:

```
important.txt
```

#### Mistake #3 – Using `cat` Inside smbclient

I tried:

```text
cat important.txt
```

**Error**

```
command not found
```

#### Fix

`smbclient` does **not** support `cat`, so I downloaded the file instead.

```text
get important.txt
```

After reading it locally, I found:

```
sa:87N1ns@slls83
```

### Second Credentials Found

```
sa
87N1ns@slls83
```

---

### Exploring the Users Share

I browsed the Users share.

```bash
smbclient //10.129.202.41/Users -U 'alex%lol123!mD'
```

I downloaded:

- NTUSER.DAT
- LOG files
- TechSupport folder

I searched the registry hive and all downloaded files for the HTB password.

```bash
strings NTUSER.DAT | grep -i htb
```

Nothing useful was found.

**Lesson:** Large registry files do not always contain credentials.

---

#### RPC Enumeration

Using Alex's credentials, I authenticated with `rpcclient`.

```bash
rpcclient -U 'alex%lol123!mD' 10.129.202.41
```

`enumprivs` worked, but `enumdomusers` disconnected the session, so nothing useful was obtained.

---

#### Trying WinRM

```bash
evil-winrm -i 10.129.202.41 -u alex -p 'lol123!mD'
```

**Result**

```
Authorization Error
```

Alex was not allowed to use WinRM.

---

#### Trying RDP

I also attempted an RDP login using `xfreerdp`, but received:

```
ERRCONNECT_LOGON_FAILURE
```

This path was not useful.

---

# Important Hint

The lab mentioned that **SQL Management Studio can edit the last 200 entries**, which strongly suggested that SQL Server was involved.

---

### Mistake #4 – Assuming `sa` Was SQL Authentication

I tried connecting directly.

```bash
mssqlclient.py sa@10.129.202.41
```

It failed, and I also noticed that **port 1433** was not reachable remotely.

**Lesson:** SQL Server may only accept **local connections**.

---

#### Realization

The credentials

```
sa
87N1ns@slls83
```

might simply be a reused password rather than SQL credentials. Since Windows always has an **Administrator** account, I tested the password.

```bash
crackmapexec smb 10.129.202.41 -u Administrator -p '87N1ns@slls83'
```

**Result**

```
Pwn3d!
```

The SQL password had been reused as the local Administrator password.

---

#### Administrator Access

I connected successfully.

```bash
evil-winrm -i 10.129.202.41 -u Administrator -p '87N1ns@slls83'
```

Administrator shell obtained.

---

#### Mistake #5 – PowerShell Paths With Spaces

I tried:

```powershell
cd SQL Server Management Studio
```

PowerShell complained because of the spaces.

#### Fix

```powershell
cd "SQL Server Management Studio"
```

A small mistake, but an easy one to forget. But in this directory i found nothing expect more directories with nothing included. The hint might say like a puzzle that check for sql services not directory.

---

### Checking SQL Services

```powershell
Get-Service *SQL*
```

I found:

```
MSSQLSERVER
```

It was running.

---

#### Connecting to SQL

Interactive `sqlcmd` did not display a prompt properly inside Evil-WinRM, so I used direct queries instead.

#### List Databases

```bash
sqlcmd -S localhost -E -Q "SELECT name FROM sys.databases"
```

The interesting database was:

```
accounts
```

---

#### Enumerating Tables

I first guessed there would be a `Users` table.

```sql
SELECT * FROM Users
```

**Result**

```
Invalid object name
```

Instead, I enumerated the tables.

```sql
SELECT TABLE_NAME,COLUMN_NAME FROM INFORMATION_SCHEMA.COLUMNS ORDER BY TABLE_NAME
```

I discovered the table:

```
devsacc
```

with the columns:

- id
- name
- password

---

#### Dumping Credentials

```bash
sqlcmd -S localhost -E -d accounts -Q "SELECT * FROM devsacc"
```

This successfully dumped all stored credentials.

---

#### Final Credential

The HTB user's password was:

```
lnch7ehrdn43i7AoqVPK4zWR
```

Mission accomplished.

---

#### Credentials Collected

| Source | Username | Password |
|---------|----------|----------|
| SMTP Ticket | alex | lol123!mD |
| important.txt | sa | 87N1ns@slls83 |
| Local Account | Administrator | 87N1ns@slls83 |
| Database | HTB | lnch7ehrdn43i7AoqVPK4zWR |

---

#### Biggest Lessons Learned

- Always enumerate NFS when port **2049** is open.
- If mounting fails, create the mount point before retrying.
- If NFS permissions appear broken, try `sudo ls` before assuming the mount failed.
- `smbclient` does not support `cat`; download files with `get`.
- Validate every discovered credential immediately with CrackMapExec.
- Password reuse is common—always test recovered passwords against other local accounts, especially **Administrator**.
- Don't assume SQL Server is remotely accessible; it may only accept local connections.
- If a guessed SQL table doesn't exist, enumerate tables using `INFORMATION_SCHEMA`.
- In PowerShell, always quote paths containing spaces.
- When `sqlcmd` behaves oddly inside Evil-WinRM, use the `-Q` option to execute queries directly.

---

#### Commands Used Most

```bash
nmap -sC -sV

showmount -e

mount -t nfs

sudo ls -la

crackmapexec smb

smbclient

rpcclient

evil-winrm

Get-Service *SQL*

sqlcmd -S localhost -E -Q

SELECT TABLE_NAME,COLUMN_NAME FROM INFORMATION_SCHEMA.COLUMNS

SELECT * FROM devsacc
```






# HTB Academy – Footprinting Final Lab 3

My goal was to recover the credentials for the **HTB** user. During this lab I enumerated multiple hosts, discovered an exposed SNMP service, recovered user credentials, accessed an IMAP mailbox, extracted two SSH private keys, obtained root access, and finally recovered the HTB user's credentials.

---

### Network Overview

At first, I kept enumerating the given machine and all the services i found.. but they were not very useful. But after doing the `nmap -sn`  scan, I identified three interesting machines.

|IP Address|Role|Interesting Services|
|---|---|---|
|**10.129.202.20**|Mail / Backup Server|SSH, POP3, IMAP, IMAPS, POP3S, SNMP|
|**10.129.202.64**|Web Server|SSH, HTTP|
|**10.129.202.149**|Windows File Server|SMB, WinRM, RDP|

---

### Initial Enumeration

I began by performing full TCP scans against every discovered host.

|Command|Purpose|Result|
|---|---|---|
|`sudo nmap -p- -T4 10.129.202.20`|Full TCP scan|SSH, POP3, IMAP discovered|
|`sudo nmap -p- -T4 10.129.202.64`|Full TCP scan|SSH and Apache HTTP|
|`sudo nmap -p- -T4 10.129.202.149`|Full TCP scan|SMB, WinRM, RDP|

I also got another host which later got closed. I tried connecting with ssh, pop3, imap with the commands i have but got no access.  Same i did with SMB (where got no access), RDP ( the do need credentials!). Since SSH was open, I collected host keys with `ssh-keyscan 10.129.202.64` and for 10.129.202.20, as sometimes host keys can reveal:  reused infrastructure, duplicated VMs, cloned hosts. After comparing them i found that both hosts had identical SSH host keys. This strongly suggested they were cloned from the same template. Although this did not directly lead to credentials, it helped me understand the environment.  Then I tried investigating  the web server.

---

### Web Enumeration

I fingerprinted the web server, tried to enumerate a new port `8080`, although i got Certificate: CN = roob.tromp.biz, and just tried logging in with roob and tromp just in case, got nothing. I tried  accessing the service. with `curl -i http://10.129.202.64:8080; curl -vk https://10.129.202.64:8080; nmap -Pn -p22,80,8080 10.129.202.64; curl -H "Host: roob.tromp.biz" http://10.129.202.64` but got nothing interesting. 

|Command|Purpose|Result|
|---|---|---|
|`whatweb http://10.129.202.64`|Identify technologies|Apache 2.4.41 (Ubuntu)|
|`nikto -h http://10.129.202.64`|Look for common web issues|Default Apache page only|
|`ffuf`|Directory fuzzing|Only default files|
|`gobuster`|Directory fuzzing|Same result as ffuf|

The web server only hosted the default Apache page. No hidden directories. No virtual hosts. No interesting applications.


---

### UDP Enumeration

After trying all the commands with dig, i tried `nslookup`, and interestingly found port 53 open for all this 3 host services. After scanning them with nmap, only one replied with an open port.

| Command                                               | Purpose       | Result                     |
| ----------------------------------------------------- | ------------- | -------------------------- |
| `sudo nmap -sU -Pn -p- --min-rate 5000 10.129.202.20` | UDP scan      | Port 161 (SNMP) discovered |
| `sudo nmap -sU -sV -p161 --script snmp-info`          | Identify SNMP | Net-SNMP v3  detected      |

Finding SNMP completely changed the direction of the attack. Before this phase, every enumerating gave me a dead end.

---

### SNMP Enumeration

My first attempt was to brute-force the community string using the all the SecLists snmp wordlists.

|Command|Result|
|---|---|
|`onesixtyone -c common-snmp-community-strings.txt`|Nothing|
|`onesixtyone -c common-snmp-community-strings-onesixtyone.txt`|Nothing|

Interestingly, none of the default community lists contained the correct value. Since the lab description repeatedly mentioned a **backup server**, I manually tested the community string **backup**.

```bash
onesixtyone 10.129.202.20 backup
```

Result:

```
10.129.202.20 [backup] Linux NIXHARD ...
```

Community string recovered:

```
backup
```

---

#### Enumerating SNMP Data

I first gathered general system information.

```bash
snmpwalk -v2c -c backup 10.129.202.20 1.3.6.1.2.1.1
```

This revealed:

- Hostname
    
- Contact information
    
- System description
    
- Location
    

Although useful, it did not reveal credentials.

I then switched to **braa**, which is much faster for dumping large OID trees.

```bash
braa backup@10.129.202.20:.1.*
```

Among the output I discovered:

```
0:tom NMds732Js2761
```

Recovered credentials:

|Username|Password|
|---|---|
|tom|NMds732Js2761|


---

### Testing the Credentials

My first instinct was SSH.

```bash
ssh tom@10.129.202.20
```

Result:

```
Permission denied (publickey)
```

This told me two important things:

- The account exists.
    
- Password authentication is disabled.

Instead of assuming the password was wrong, I remembered that the same credentials might work on another service.

---

### Enumerating IMAPS

I connected manually.

```bash
openssl s_client -connect 10.129.202.20:993 -quiet
```

Then authenticated.

```
a LOGIN tom NMds732Js2761
```

Login succeeded.

I enumerated the mailbox.

|IMAP Command|Purpose|
|---|---|
|`LIST "" *`|List mailboxes|
|`SELECT INBOX`|Open Inbox|
|`SEARCH ALL`|List message IDs|
|`FETCH 1 BODY[]`|Read email|

The only email contained an **OpenSSH private key**.

---

### First SSH Private Key

I copied the private key into a local file.

```bash
chmod 600 id_rsa
```

Then authenticated with SSH.

```bash
ssh -i id_rsa tom@10.129.202.20
```

Success. I now had an interactive shell as **tom**.

---

#### Local Enumeration

After gaining a user shell,  I inspected the home directory.

```bash
ls -la
```

Since I had already read the Inbox through IMAP, I expected the physical email file to exist under **cur/** 

```bash
cd ~/Maildir
ls -la
```

I checked the stored email files. with  

```bash
find ~/Maildir/cur -type f
cat ~/Maildir/cur/*
cp ~/Maildir/cur/* /tmp/mail.txt
awk '/BEGIN OPENSSH PRIVATE KEY/,/END OPENSSH PRIVATE KEY/' /tmp/mail.txt
```

(or just copy paste :/ but again that can cause terminal wrapping.)

Surprisingly, the mailbox stored on disk contained another OpenSSH private key. This second key was just a  different from the first one, but this second one allowed both tom and root to login by ssh (maybe cause same public key exists in both user's .ssh file, and so the same private key can authenticate as either user.).

---

### Second SSH Private Key

After saving the second key locally:

```bash
chmod 600 root_key
```

I tested it.

```bash
ssh -i root_key tom@10.129.202.20
```

Worked. Then I tried looging it with bob, tom and root :

```bash
ssh -i root_key root@10.129.202.20
```

To my surprise, it also worked. I now had a root shell. After obtaining any shell, always inspect:

- Mailboxes
    
- SSH keys
    
- Home directories
    
- Configuration files
    
- Backup files
    
- Databases

---

### Root Enumeration

As root, I listed the contents of the home directory.

```bash
ls
```

Interesting file:

```
users.sql
```

Here i found the desired final password. This SQL dump contained the credentials for the **HTB** user, completing the lab.

---

## Complete Attack Chain

```
Host Discovery
        │
        ▼
TCP Enumeration
        │
        ▼
Web Enumeration
        │
        ▼
Nothing Useful
        │
        ▼
UDP Enumeration
        │
        ▼
SNMP Found
        │
        ▼
Community = backup
        │
        ▼
braa Enumeration
        │
        ▼
Recovered:
tom : NMds732Js2761
        │
        ▼
IMAPS Login
        │
        ▼
Downloaded SSH Key #1
        │
        ▼
SSH as tom
        │
        ▼
Read Maildir
        │
        ▼
Recovered SSH Key #2
        │
        ▼
SSH as root
        │
        ▼
Read users.sql
        │
        ▼
Recovered HTB Credentials
```


---

## Commands Cheat Sheet

|Step|Command|
|---|---|
|Host discovery|`nmap -sn`|
|Full TCP scan|`nmap -p-`|
|Web fingerprinting|`whatweb`|
|Web vulnerability scan|`nikto`|
|Directory enumeration|`ffuf`, `gobuster`|
|SMB enumeration|`crackmapexec smb`|
|UDP scan|`nmap -sU`|
|SNMP detection|`snmp-info`|
|Community discovery|`onesixtyone`|
|SNMP enumeration|`snmpwalk`|
|Fast OID dump|`braa`|
|IMAPS connection|`openssl s_client`|
|SSH with private key|`ssh -i id_rsa`|
|Local mailbox inspection|`cat ~/Maildir/cur/*`|
|Root login|`ssh -i root_key root@10.129.202.20`|

---

### Credentials Recovered

|Stage|Credential|
|---|---|
|SNMP|Community: `backup`|
|SNMP|`tom : NMds732Js2761`|
|IMAP|SSH Private Key #1|
|Local Enumeration|SSH Private Key #2|
|Root|`users.sql` containing HTB credentials|
