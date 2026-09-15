## What is Brute Forcing?

A trial-and-error method used to crack passwords, login credentials, or encryption keys by systematically trying every possible combination of characters.

### Factors Influencing Brute Force Attacks

- Complexity of the password or key
- Computational power available to the attacker
- Security measures in place

### How Brute Forcing Works

1. Start: The attacker initiates the brute force process.
2. Generate Possible Combination: The software generates a potential password or key combination.
3. Apply Combination: The generated combination is attempted against the target system.
4. Check if Successful: The system evaluates the attempted combination.
5. Access Granted (if successful): The attacker gains unauthorized access.
6. End (if unsuccessful): The process repeats until the correct combination is found or the attacker gives up.

### Types of Brute Forcing

|Attack Type|Description|Best Used When|
|---|---|---|
|Simple Brute Force|Tries every possible character combination in a set (e.g., lowercase, uppercase, numbers, symbols).|When there is no prior information about the password.|
|Dictionary Attack|Uses a pre-compiled list of common passwords.|When the password is likely weak or follows common patterns.|
|Hybrid Attack|Combines brute force and dictionary attacks, adding numbers or symbols to dictionary words.|When the target uses slightly modified versions of common passwords.|
|Credential Stuffing|Uses leaked credentials from other breaches to access different services where users may have reused passwords.|When you have a set of leaked credentials, and the target may reuse passwords.|
|Password Spraying|Attempts common passwords across many accounts to avoid detection.|When account lockout policies are in place.|
|Rainbow Table Attack|Uses precomputed tables of password hashes to reverse them into plaintext passwords.|When a large number of password hashes need cracking, and storage for tables is available.|
|Reverse Brute Force|Targets a known password against multiple usernames.|When there’s a suspicion of password reuse across multiple accounts.|
|Distributed Brute Force|Distributes brute force attempts across multiple machines to speed up the process.|When the password is highly complex, and a single machine isn't powerful enough.|

## Default Credentials


|Device|Username|Password|
|---|---|---|
|Linksys Router|admin|admin|
|Netgear Router|admin|password|
|TP-Link Router|admin|admin|
|Cisco Router|cisco|cisco|
|Ubiquiti UniFi AP|ubnt|ubnt|


# Brute-Forcing Tools

## Custom Wordlists

Username Anarchy generates potential usernames based on a target's name.

|Command|Description|
|---|---|
|`username-anarchy Jane Smith`|Generate possible usernames for "Jane Smith"|
|`username-anarchy -i names.txt`|Use a file (`names.txt`) with names for input. Can handle space, CSV, or TAB delimited names.|
|`username-anarchy -a --country us`|Automatically generate usernames using common names from the US dataset.|
|`username-anarchy -l`|List available username format plugins.|
|`username-anarchy -f format1,format2`|Use specific format plugins for username generation (comma-separated).|
|`username-anarchy -@ example.com`|Append `@example.com` as a suffix to each username.|
|`username-anarchy --case-insensitive`|Generate usernames in case-insensitive (lowercase) format.|

CUPP (Common User Passwords Profiler) creates personalized password wordlists based on gathered intelligence.

|Command|Description|
|---|---|
|`cupp -i`|Generate wordlist based on personal information (interactive mode).|
|`cupp -w profiles.txt`|Generate a wordlist from a predefined profile file.|
|`cupp -l`|Download popular password lists like `rockyou.txt`.|


## Password Policy Filtering

Password policies often dictate specific requirements for password strength, such as minimum length, inclusion of certain character types, or exclusion of common patterns. `grep` combined with regular expressions can be a powerful tool for filtering wordlists to identify passwords that adhere to a given policy. Below is a table summarizing common password policy requirements and the corresponding `grep` regex patterns to apply:

| Policy Requirement                         | Grep Regex Pattern                                                                                                                        | Explanation                                                                                                                                                                                                                                                                                                                                                                                                               |
| ------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Minimum Length (e.g., 8 characters)        | `grep -E '^.{8,}$' wordlist.txt`                                                                                                          | `^` matches the start of the line, `.` matches any character, `{8,}` matches 8 or more occurrences, `$` matches the end of the line.                                                                                                                                                                                                                                                                                      |
| At Least One Uppercase Letter              | `grep -E '[A-Z]' wordlist.txt`                                                                                                            | `[A-Z]` matches any uppercase letter.                                                                                                                                                                                                                                                                                                                                                                                     |
| At Least One Lowercase Letter              | `grep -E '[a-z]' wordlist.txt`                                                                                                            | `[a-z]` matches any lowercase letter.                                                                                                                                                                                                                                                                                                                                                                                     |
| At Least One Digit                         | `grep -E '[0-9]' wordlist.txt`                                                                                                            | `[0-9]` matches any digit.                                                                                                                                                                                                                                                                                                                                                                                                |
| At Least One Special Character             | `grep -E '[!@#$%^&*()_+-=[]{};':"\,.<>/?]' wordlist.txt`                                                                                  | `[!@#$%^&*()_+-=[]{};':"\,.<>/?]` matches any special character (symbol).                                                                                                                                                                                                                                                                                                                                                 |
| No Consecutive Repeated Characters         | `grep -E '(.)\1' wordlist.txt`                                                                                                            | `(.)` captures any character, `\1` matches the previously captured character. This pattern will match any line with consecutive repeated characters. Use `grep -v` to invert the match.                                                                                                                                                                                                                                   |
| Exclude Common Patterns (e.g., "password") | `grep -v -i 'password' wordlist.txt`                                                                                                      | `-v` inverts the match, `-i` makes the search case-insensitive. This pattern will exclude any line containing "password" (or "Password", "PASSWORD", etc.).                                                                                                                                                                                                                                                               |
| Exclude Dictionary Words                   | `grep -v -f dictionary.txt wordlist.txt`                                                                                                  | `-f` reads patterns from a file. `dictionary.txt` should contain a list of common dictionary words, one per line.                                                                                                                                                                                                                                                                                                         |
| Combination of Requirements                | `grep -E '^.{6,}$' jane.txt \| grep -E '[A-Z]' \| grep -E '[a-z]' \| grep -E '[0-9]' \| grep -E '([!@#$%^&*].*){2,}' > jane-filtered.txt` | This command filters a wordlist to meet multiple password policy requirements. It first ensures that each word has a minimum length of 8 characters (`grep -E '^.{8,}$'`), and then it pipes the result into a second `grep` command to match only words that contain at least one uppercase letter (`grep -E '[A-Z]'`). This approach ensures the filtered passwords meet both the length and uppercase letter criteria. |

## Hydra

When using Hydra, `-L` and `-P` specify the usernames and passwords to try, but Hydra also needs to know **how the target accepts those credentials**. For services like **SSH, FTP, SMB, or RDP**, the service is specified directly because Hydra knows how to communicate with those protocols. For HTTP, it depends on how the website handles authentication. **`http-get`** is used when a page is protected by **HTTP authentication**, such as the browser's username/password popup. **`http-post-form`** is used when a website has a normal login form where the username and password are submitted using a POST request. So, `GET` and `POST` tell Hydra **how the credentials are sent and tested**, just like `ssh` tells Hydra to use SSH and `ftp` tells it to use FTP. **In short, when the target is a web/HTTP service, an HTTP-specific Hydra module is needed to tell Hydra how the authentication works.**

##### Constructing the `params` String

For Hydra's `http-post-form`, the general structure is:

```text
<path>:<parameters>:<failure_string>
```

Example:

```text
/login:username=^USER^&password=^PASS^:Invalid credentials
```

Here:

```text
/login                 → Login page/request path
username=^USER^       → Username field
password=^PASS^       → Password field
Invalid credentials   → Message shown when login fails
```

`^USER^` and `^PASS^` are replaced by Hydra with the usernames and passwords being tested. The **structure stays the same**, but the actual parameters depend on the website's login form. For example, Website A might have:

```text
username=^USER^&password=^PASS^
```

Website B might use different field names:

```text
user=^USER^&pass=^PASS^
```

Website C might have extra fields:

```text
username=^USER^&password=^PASS^&submit=Login
```

So when building the `params` string, you need to inspect the login form.

| Hydra Service             | Description                                                                                                                                                    | Example Command                                                                                                          |
| ------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------ |
| HTTP Basic Authentication |                                                                                                                                                                | `hydra -l <username> -P <password_list> <target> http-get <path> -s <port>`                                              |
| http-get/post             | Used to brute-force login credentials for HTTP web login forms using either GET or POST requests.                                                              | `hydra -l admin -P /path/to/password_list.txt 127.0.0.1 http-post-form "/login.php:user=^USER^&pass=^PASS^:F=incorrect"` |
| ssh                       | Targeting Multiple SSH Servers                                                                                                                                 | `hydra -l root -p toor -M targets.txt ssh`<br>`                                                                          |
| ssh                       | Targets SSH services to brute-force credentials, commonly used for secure remote login to systems.                                                             | `hydra -l root -P /path/to/password_list.txt ssh://192.168.1.100`                                                        |
| ftp                       | Used to brute-force login credentials for FTP services, commonly used to transfer files over a network.                                                        | `hydra -l admin -P /path/to/password_list.txt ftp://192.168.1.100`                                                       |
| ftp                       | Testing FTP Credentials on a Non-Standard Port                                                                                                                 | `hydra -L usernames.txt -P passwords.txt -s 2121 -V ftp.example.com ftp`                                                 |
| rdp                       | You suspect the username is "administrator," and that the password consists of 6 to 8 characters, including lowercase letters, uppercase letters, and numbers. | `hydra -l administrator -x 6:8:abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789 192.168.1.100 rdp`         |

## Medusa


| Medusa Module | Description                                                                     | Example Command                                                             |
| ------------- | ------------------------------------------------------------------------------- | --------------------------------------------------------------------------- |
| ssh           | Brute force SSH login for the `admin` user.                                     | `medusa -h 192.168.1.100 -u admin -P passwords.txt -M ssh`                  |
| ftp           | Brute force FTP with multiple usernames and passwords using 5 parallel threads. | `medusa -h 192.168.1.100 -U users.txt -P passwords.txt -M ftp -t 5`         |
| rdp           | Brute force RDP login.                                                          | `medusa -h 192.168.1.100 -u admin -P passwords.txt -M rdp`                  |
| http-get      | Brute force HTTP Basic Authentication.                                          | `medusa -h www.example.com -U users.txt -P passwords.txt -M http -m GET`    |
| ssh           | Stop after the first valid SSH login is found.                                  | `medusa -h 192.168.1.100 -u admin -P passwords.txt -M ssh -f`               |
|               | Targeting Multiple Web Servers with Basic HTTP Authentication                   | `edusa -H web_servers.txt -U usernames.txt -P passwords.txt -M http -m GET` |
- Perform additional checks for empty passwords (`-e n`) and passwords matching the username (`-e s`).



---

[HTB_academy/Login_Brute_Forcing at master · bakassarinad/HTB_academy](https://github.com/bakassarinad/HTB_academy/tree/master/Login_Brute_Forcing) 