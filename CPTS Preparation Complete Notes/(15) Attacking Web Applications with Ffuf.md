## Fuzzing

The term `fuzzing` refers to a testing technique that sends various types of user input to a certain interface to study how it would react. If we were fuzzing for SQL injection vulnerabilities, we would be sending random special characters and seeing how the server would react. If we were fuzzing for a buffer overflow, we would be sending long strings and incrementing their length to see if and when the binary would break.

Remember the `FUZZ` word, which is like a keyword to replace whatever from the wordlist and iterate.  We can always use two wordlists and have a unique keyword for each, and then do `FUZZ_1.FUZZ_2` to fuzz for both.

When we scan recursively, it automatically starts another scan under any newly identified directories that may have on their pages until it has fuzzed the main website and all of its subdirectories.

| **Command**                                                                                                                                                     | **Description**                                                                                                                                   |
| --------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| `ffuf -h`                                                                                                                                                       | ffuf help                                                                                                                                         |
| `ffuf -w wordlist.txt:FUZZ -u http://SERVER_IP:PORT/FUZZ`                                                                                                       | Directory Fuzzing                                                                                                                                 |
| `ffuf -w wordlist.txt:FUZZ -u http://SERVER_IP:PORT/indexFUZZ`                                                                                                  | Extension Fuzzing.                                                                                                                                |
| `ffuf -w wordlist.txt:FUZZ -u http://SERVER_IP:PORT/blog/FUZZ.php`                                                                                              | Page Fuzzing                                                                                                                                      |
| `ffuf -w wordlist.txt -u http://154.57.164.82:32590/blog/FUZZ -e .php,.html,.htm,.txt,.asp,.aspx,.jsp,.json,.xml,.bak,.old,.zip,.conf`                          |                                                                                                                                                   |
| `ffuf -w wordlist.txt:FUZZ -u http://SERVER_IP:PORT/FUZZ -recursion -recursion-depth 1 -e .php -v`                                                              | Recursive Fuzzing                                                                                                                                 |
| `ffuf -w wordlist.txt:FUZZ -u https://FUZZ.hackthebox.eu/`                                                                                                      | Sub-domain Fuzzing                                                                                                                                |
| `ffuf -w wordlist.txt:FUZZ -u http://academy.htb:PORT/ -H 'Host: FUZZ.academy.htb' -fs xxx`                                                                     | VHost Fuzzing (fuzzing sub-domains that do not have a public DNS record or sub-domains under websites that are not public). -fs is response size. |
| `ffuf -w wordlist.txt:FUZZ -u http://admin.academy.htb:PORT/admin/admin.php?FUZZ=key -fs xxx`                                                                   | Parameter Fuzzing - GET, `?` = URL query string. GET commonly uses it, but other HTTP methods can use it too.                                     |
| `ffuf -w wordlist.txt:FUZZ -u http://admin.academy.htb:PORT/admin/admin.php -X POST -d 'FUZZ=key' -H 'Content-Type: application/x-www-form-urlencoded' -fs xxx` | Parameter Fuzzing - POST                                                                                                                          |
| `ffuf -w ids.txt:FUZZ -u http://admin.academy.htb:PORT/admin/admin.php -X POST -d 'id=FUZZ' -H 'Content-Type: application/x-www-form-urlencoded' -fs xxx`       | Value Fuzzing                                                                                                                                     |


- **GET** → Give me (GET commonly uses `?`)
- **POST** → Take this (`POST` requests are passed in the `data` field within the HTTP request)
- **PUT** → Replace this
- **PATCH** → Change this part
- **DELETE** → Remove this
- **HEAD** → Give me information about it
- **OPTIONS** → Tell me what I can do
- **CONNECT** → Create a connection
- **TRACE** → Show me the request path

# Wordlists

| **Command**                                                               | **Description**                                                                                                                          |
| ------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| `/opt/useful/seclists/Discovery/Web-Content/directory-list-2.3-small.txt` | Directory/Page Wordlist                                                                                                                  |
| `/opt/useful/seclists/Discovery/Web-Content/web-extensions.txt`           | Extensions Wordlist. The wordlist we chose already contains a dot (.), so we will not have to add the dot after "target" in our fuzzing. |
| `/opt/useful/seclists/Discovery/DNS/subdomains-top1million-5000.txt`      | Domain Wordlist                                                                                                                          |
| `/opt/useful/seclists/Discovery/Web-Content/burp-parameter-names.txt`     | Parameters Wordlist                                                                                                                      |

Tip: Taking a look at this wordlist we will notice that it contains copyright comments at the beginning, which can be considered as part of the wordlist and clutter the results. We can use the following in `ffuf` to get rid of these lines with the `-ic` flag.
# Misc

| **Command**                                                                                                                   | **Description**          |
| ----------------------------------------------------------------------------------------------------------------------------- | ------------------------ |
| `sudo sh -c 'echo "SERVER_IP academy.htb" >> /etc/hosts'`                                                                     | Add DNS entry            |
| `for i in $(seq 1 1000); do echo $i >> ids.txt; done`                                                                         | Create Sequence Wordlist |
| `curl http://admin.academy.htb:PORT/admin/admin.php -X POST -d 'id=key' -H 'Content-Type: application/x-www-form-urlencoded'` | curl w/ POST             |


[Attacking Web Applications with Ffuf — HTB Academy Walkthrough | Mulandi's Blog](https://mulandii.github.io/posts/attacking-web-applications-with-ffuf/) 