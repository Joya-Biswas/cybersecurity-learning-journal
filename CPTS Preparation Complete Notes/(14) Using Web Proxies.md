# Burp Shortcuts

|**Shortcut**|**Description**|
|---|---|
|`CTRL+R`|Send to repeater|
|`CTRL+SHIFT+R`|Go to repeater|
|`CTRL+I`|Send to intruder|
|`CTRL+SHIFT+I`|Go to intruder|
|`CTRL+U`|URL encode|
|`CTRL+SHIFT+U`|URL decode|

# ZAP Shortcuts

|**Shortcut**|**Description**|
|---|---|
|`CTRL+B`|Toggle intercept on/off|
|`CTRL+R`|Go to replacer|
|`CTRL+E`|Go to encode/decode/hash|

# Firefox Shortcuts

|**Shortcut**|**Description**|
|---|---|
|`CTRL+SHIFT+R`|Force Refresh Page|

---
## What Are Web Proxies?

Web proxies are specialized tools that can be set up between a browser/mobile application and a back-end server to capture and view all the web requests being sent between both ends, essentially acting as man-in-the-middle (MITM) tools. While other `Network Sniffing` applications, like Wireshark, operate by analyzing all local traffic to see what is passing through a network, Web Proxies mainly work with web ports such as, but not limited to, `HTTP/80` and `HTTPS/443`.


# Burp Intruder

---

Both Burp and ZAP provide additional features other than the default web proxy, which are essential for web application penetration testing. Two of the most important extra features are `web fuzzers` and `web scanners`. The built-in web fuzzers are powerful tools that act as web fuzzing, enumeration, and brute-forcing tools. This may also act as an alternative for many of the CLI-based fuzzers we use, like `ffuf`, `dirbuster`, `gobuster`, `wfuzz`, among others.

Burp's web fuzzer is called `Burp Intruder`, and can be used to fuzz pages, directories, sub-domains, parameters, parameters values, and many other things. Though it is much more advanced than most CLI-based web fuzzing tools, the free `Burp Community` version is throttled at a speed of 1 request per second, making it extremely slow compared to CLI-based web fuzzing tools, which can usually read up to 10k requests per second. 

## Target

As usual, we'll start up Burp and its pre-configured browser and then visit the web application from the exercise at the end of this section. Once we do, we can go to the Proxy History, locate our request, then right-click on the request and select `Send to Intruder`, or use the shortcut `CTRL+I` to send it to `Intruder`.

## Positions

'`Positions`', is where we place the payload position pointer, which is the point where words from our wordlist will be placed and iterated over. We will be demonstrating how to fuzz web directories, which is similar to what's done by tools like `ffuf` or `gobuster`.

To check whether a web directory exists, our fuzzing should be in '`GET /DIRECTORY/`', such that existing pages would return `200 OK`, otherwise we'd get `404 NOT FOUND`. So, we will need to select `DIRECTORY` as the payload position, by either wrapping it with `§` or by selecting the word `DIRECTORY` and clicking on the `Add §` button:

**Note: Be sure to leave the extra two lines at the end of the request, otherwise we may get an error response from the server.**

Also, to set a proxy for any exploit within Metasploit, we can use the set PROXIES.

[Using Web Proxies](https://notes.maccgenics.com/02-areas/hack-the-box/htb-academy/bug-bounty-hunter/3-using-web-proxies/using-web-proxies/) 