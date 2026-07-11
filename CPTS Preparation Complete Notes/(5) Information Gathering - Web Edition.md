
# Section 1 - Introduction

| Category                   | Tools                                                           |
| -------------------------- | --------------------------------------------------------------- |
| **Port Scanning**          | Nmap, Masscan, Unicornscan                                      |
| **Vulnerability Scanning** | Nessus, OpenVAS, Nikto                                          |
| **Network Mapping**        | Traceroute, Nmap                                                |
| **Banner Grabbing**        | Netcat, curl                                                    |
| **OS Fingerprinting**      | Nmap, Xprobe2                                                   |
| **Service Enumeration**    | Nmap (`-sV`)                                                    |
| **Web Crawling**           | Burp Spider, OWASP ZAP Spider, Scrapy, ReconSpider              |
| **Search Engines (OSINT)** | Google, DuckDuckGo, Bing, Shodan                                |
| **Domain**                 | whois                                                           |
| **DNS Enumeration**        | dig, nslookup, host, dnsenum, fierce, dnsrecon                  |
| **Web Archives**           | Wayback Machine                                                 |
| **Social Media**           | LinkedIn, Twitter/X, Facebook                                   |
| **Code Repositories**      | GitHub, GitLab                                                  |
| Automated                  | FinalRecon, Recon-ng, theHarvester, SpiderFoot, OSINT Framework |

# Section 2-3 - WHOIS

WHOIS is a query and response protocol used to retrieve information about domain names, IP addresses, and other internet resources. It's essentially a directory service that details who owns a domain, when it was registered, contact information, and more. In the context of web reconnaissance, WHOIS lookups can be a valuable source of information, potentially revealing the identity of the website owner, their contact information, and other details that could be used for further investigation or social engineering attacks.

`whois example.com`

However, it's important to note that WHOIS data can be inaccurate or intentionally obscured, so it's always wise to verify the information from multiple sources. Privacy services can also mask the true owner of a domain, making it more difficult to obtain accurate information through WHOIS.

# Section 4 - DNS

The Domain Name System (DNS) functions as the internet's GPS, translating user-friendly domain names into the numerical IP addresses computers use to communicate. Like GPS converting a destination's name into coordinates, DNS ensures your browser reaches the correct website by matching its name with its IP address. This eliminates memorizing complex numerical addresses, making web navigation seamless and efficient. When searching for something online:

1. **Local Cache** – My computer checks its DNS cache for the IP.
2. **DNS Resolver** – If not found, it queries the ISP's DNS resolver. (A server that translates domain names into IP addresses.)
3. **Root Server** (There are 13 root servers worldwide, named A-M) – Directs the resolver to the correct **TLD** server (.com, .org, etc.). 
4. **TLD Server** (.com, .org etc) – Points to the domain's **Authoritative DNS** server.
5. **Authoritative Server** – Returns the correct IP address.
6. **Resolver Caches** – Sends the IP back and caches it for future requests.
7. **Connection** – My computer connects directly to the web server using the IP.


| Tool                                                 | Purpose                                |
| ---------------------------------------------------- | -------------------------------------- |
| **dig**                                              | Detailed DNS queries & troubleshooting |
| **nslookup**                                         | Basic DNS lookups                      |
| **host**                                             | Quick DNS resolution                   |
| **dnsenum, fuff, gobuster, Feroxbuster**             | Automated DNS enumeration              |
| **fierce**                                           | Subdomain reconnaissance               |
| **dnsrecon, amass, assetfinder, puredns, sublist3r** | Comprehensive DNS enumeration          |
| **theHarvester**                                     | OSINT (emails, domains, employees)     |
| **Online DNS Lookup**                                | Quick browser-based DNS lookups        |

DNS servers store various types of records, each serving a specific purpose:

|Record Type|Description|
|---|---|
|A|Maps a hostname to an IPv4 address.|
|AAAA|Maps a hostname to an IPv6 address.|
|CNAME|Creates an alias for a hostname, pointing it to another hostname.|
|MX|Specifies mail servers responsible for handling email for the domain.|
|NS|Delegates a DNS zone to a specific authoritative name server.|
|TXT|Stores arbitrary text information.|
|SOA|Contains administrative information about a DNS zone.|

# Section 6-7 - Subdomains

Subdomains are essentially extensions of a primary domain name, often used to organize different sections or services within a website. For example, a company might use `mail.example.com` for their email server or `blog.example.com` for their blog.

From a reconnaissance perspective, subdomains can expose additional attack surfaces, reveal hidden services, and provide clues about the internal structure of a target's network. Subdomains might host development servers, staging environments, or even forgotten applications that haven't been properly secured.

|Approach|Description|Examples|
|---|---|---|
|`Active Enumeration`|Directly interacts with the target's DNS servers or utilizes tools to probe for subdomains.|Brute-forcing, DNS zone transfers|
|`Passive Enumeration`|Collects information about subdomains without directly interacting with the target, relying on public sources.|Certificate Transparency (CT) logs, search engine queries|

`Active enumeration` can be more thorough but carries a higher risk of detection. Conversely, `passive enumeration` is stealthier but may not uncover all subdomains. 

1. `dnsenum --enum inlanefreight.com -f /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt -r`

`-r`: This option enables recursive subdomain brute-forcing, meaning that if `dnsenum` finds a subdomain, it will then try to enumerate subdomains of that subdomain.

2. `sublist3r -d inlanefreight.com | grep inlanefreight.com`


# Section 8 - Zone Transfers

DNS zone transfers, also known as AXFR (Asynchronous Full Transfer) requests, offer a potential goldmine of information for web reconnaissance. A zone transfer is a mechanism for replicating DNS data across servers. When a zone transfer is successful, it provides all the domain's subdomains, their associated IP addresses, mail server configurations, and other DNS records. 

To attempt a zone transfer, you can use the `dig` command with the `axfr` (full zone transfer) option. For example, to request a zone transfer from the DNS server `ns1.example.com` for the domain `example.com`, you would execute:

`dig @ns1.example.com example.com axfr` (@ indicates where you would transfer, and the primary ones's ip and dns should be enlisted in `/etc/hosts`)

However, zone transfers are not always permitted. Many DNS servers are configured to restrict zone transfers to authorized secondary servers only. 

# Section 9 - Virtual Hosts

Virtual hosting is a technique that allows multiple websites to share a single IP address. Each website is associated with a unique hostname, which is used to direct incoming requests to the correct site. This can be a cost-effective way for organizations to host multiple websites on a single server, but it can also create a challenge for web reconnaissance.

#### How it Works  

1. I visit a website (e.g., `http://admin.example.com`).
2. My browser sends the domain in the **Host** header.
   - Example:
     ```http
     GET / HTTP/1.1
     Host: admin.example.com
     ```
3. The web server reads the **Host** header.
4. It matches the request to the correct virtual host.
   - Example:
     - `admin.example.com` → Admin site
     - `blog.example.com` → Blog site
     - `shop.example.com` → Shop site
1. The server returns the correct website content to my browser.
  
### Types of Virtual Hosting  
  
| Type           | Simple Explanation                                                                                    |     |
| -------------- | ----------------------------------------------------------------------------------------------------- | --- |
| **Name-Based** | Multiple websites share **one IP**. The server uses the **Host** header to decide which site to show. |     |
| **IP-Based**   | Each website has its **own IP address**.                                                              |     |
| **Port-Based** | Multiple websites share one IP but use **different ports** (e.g., `:80`, `:8080`).                    |     |

Gobuster is a versatile tool that can be used for various types of brute-forcing, including virtual host discovery. Its `vhost` mode is designed to enumerate virtual hosts.

1. `gobuster vhost -u http://<target_IP_address> -w <wordlist_file> --append-domain`

The --append-domain flag  ensures that Gobuster correctly constructs the full virtual hostnames, which is essential for the accurate enumeration of potential subdomains. There are a couple of other arguments that are worth knowing:

- Consider using the `-t` flag to increase the number of threads for faster scanning.
- The `-k` flag can ignore SSL/TLS certificate errors.

2. `ffuf -u http://94.237.57.155:42456/ -H "Host: FUZZ.inlanefreight.htb" -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-110000.txt`

`FUZZ` is simply ffuf's variable that gets replaced by every line in the wordlist.


# Section 10 - Certificate Transparency (CT) Logs

Certificate Transparency (CT) logs offer a treasure trove of subdomain information for passive reconnaissance. These publicly accessible logs record SSL/TLS certificates issued for domains and their subdomains, serving as a security measure to prevent fraudulent certificates. 

#### Certificate Transparency (CT) Logs

**What are CT Logs?**  
A **public database** of every SSL/TLS certificate issued. They help detect fake or unauthorized certificates.

1. Website requests an SSL/TLS certificate from a **Certificate Authority (CA)**.
2. The CA submits the certificate to **CT Logs**.
3. The CT Log returns an **SCT (Signed Certificate Timestamp)** as proof it was logged.
4. The CA issues the final certificate with the SCT included.
5. Browsers verify the SCT before trusting the certificate.
6. Security researchers and domain owners monitor CT Logs for suspicious certificates.

##### Merkle Tree (Simple)

- A **Merkle Tree** is a tree of **hashes** used to protect the integrity of CT Logs.
- Every certificate is hashed.
- Hashes are combined until one final hash is created, called the **Merkle Root**.
- If **any certificate changes**, the Merkle Root also changes, making tampering obvious.

The `crt.sh` and Censys websites provides a searchable interface for CT logs. To efficiently extract subdomains using `crt.sh` within your terminal, you can use a command like this:

`curl -s "https://crt.sh/?q=%25.example.com&output=json" | jq -r '.[].name_value' | sed 's/\*\.//g' | sort -u`

This command fetches JSON-formatted data from `crt.sh` for `example.com` (the `%` is a wildcard), extracts domain names using `jq`, removes any wildcard prefixes (`*.`) with `sed`, and finally sorts and deduplicates the results.

# Section 11 - Fingerprinting


Fingerprinting is used to identify the technologies running behind a website (web server, CMS, WAF, frameworks, OS, etc.). This helps me choose the right exploits and understand the target's attack surface.

| Tool | Purpose | Example Command |
|------|---------|-----------------|
| `curl` | Grab HTTP headers (banner grabbing) | `curl -I http://target.com` |
| `curl` | Follow redirects & inspect headers | `curl -I -L http://target.com` |
| `Wappalyzer` | Detect website technologies (Browser Extension) | Browser Extension |
| `BuiltWith` | Online technology fingerprinting | `https://builtwith.com/target.com` |
| `WhatWeb` | Identify web technologies | `whatweb http://target.com` |
| `Nmap` | Detect services & versions | `nmap -sV target.com` |
| `Nmap NSE` | Advanced web fingerprinting | `nmap --script http-enum,http-title target.com` |
| `Netcraft` | Hosting & infrastructure information | `https://sitereport.netcraft.com` |
| `wafw00f` | Detect Web Application Firewall (WAF) | `wafw00f target.com` |
| `Nikto` | Fingerprint + basic web vulnerability scan | `nikto -h target.com -Tuning b` |

#### Common HTTP Headers I Should Check

| Header           | What it tells me                                                              |
| ---------------- | ----------------------------------------------------------------------------- |
| `Server`         | Web server (Apache, Nginx, IIS, etc.)                                         |
| `X-Powered-By`   | Backend language/framework (PHP, ASP.NET, Express, etc.)                      |
| `X-Redirect-By`  | What performed the redirect (often WordPress)                                 |
| `Link`           | API endpoints (e.g., `wp-json`)                                               |
| `Location`       | Redirect destination                                                          |
| security headers | If a security header is present, it usually means that protection is enabled. |

### Quick note on 

**Security headers:**

- **HSTS** → HTTPS only
- **CSP** → Blocks malicious scripts (XSS)
- **X-Frame-Options** → No clickjacking
- **X-Content-Type-Options** → No MIME sniffing
- **Referrer-Policy** → Hide URL information
- **Permissions-Policy** → Disable browser APIs
- **CORP / COEP / COOP** → Cross-origin protection

**Hostname resolution error:** 
> If a tool says **"Cannot resolve hostname"**, it means the domain cannot be translated into an IP address. Fix it by ensuring the hostname exists in **DNS** or by adding it to **`/etc/hosts`**.

## Questions 1-3

1. First scan the Ip with `whatweb http://ip`
2. First add those two vhosts to /etc/hosts by `sudo nano /etc/hosts`
3. then add 10.129.49.190 app.inlanefreight.local dev.inlanefreight.local
4. Then run `nikto -h http://app.inlanefreight.local -Tuning b`

or skip 2 and 3, run `nikto -h 10.129.XX.XX -vhost app.inlanefreight.local`

Check robots.txt for question 2.


# Section 12 -15 -  Web Crawling

Web crawling is the automated exploration of a website's structure. A web crawler, or spider, systematically navigates through web pages by following links, mimicking a user's browsing behavior. This process maps out the site's architecture and gathers valuable information embedded within the pages.

A crucial file that guides web crawlers is `robots.txt`, `sitemap.xml`.

`.well-known` is a **standard directory (URI path)** used to store important website configuration and metadata. A **URI** identifies a specific resource or endpoint on a website. During reconnaissance, checking `/.well-known/` can reveal useful files like `security.txt` (security contact), `openid-configuration` (OAuth/OpenID endpoints (OAuth gives permission, while OpenID Connect confirms who the user is)), `change-password` (password reset page), `assetlinks.json` (app-domain verification), and `mta-sts.txt` (email security policy).

`Scrapy` is a powerful and efficient Python framework for large-scale web crawling and scraping projects. 

1. pip3 install scrapy
2. wget -O ReconSpider.zip https://academy.hackthebox.com/storage/modules/144/ReconSpider.v1.2.zip
3. unzip ReconSpider.zip
4. python3 ReconSpider.py http://inlanefreight.com
5. cat result.json


`import scrapy class ExampleSpider(scrapy.Spider):     name = "example"    start_urls = ['http://example.com/']     def parse(self, response):        for link in response.css('a::attr(href)').getall():            if any(link.endswith(ext) for ext in self.interesting_extensions):                yield {"file": link}            elif not link.startswith("#") and not link.startswith("mailto:"):                yield response.follow(link, callback=self.parse)`

After running the Scrapy spider, you'll have a file containing scraped data (e.g., `example_data.json`). You can analyze these results using standard command-line tools. For instance, to extract all links:


`jq -r '.[] | select(.file != null) | .file' example_data.json | sort -u`

This command uses `jq` to extract links, `awk` to isolate file extensions, `sort` to order them, and `uniq -c` to count their occurrences. By scrutinizing the extracted data, you can identify patterns, anomalies, or sensitive files that might be of interest for further investigation.

# Section 16 -  Search Engine Discovery

| Operator              | Purpose                      | Example                          |
| --------------------- | ---------------------------- | -------------------------------- |
| `site:`               | Search specific domain       | `site:example.com`               |
| `inurl:`              | Find text in URL             | `inurl:login`                    |
| `filetype:`           | Search file types            | `filetype:pdf`                   |
| `intitle:`            | Find text in page title      | `intitle:"admin"`                |
| `intext:` / `inbody:` | Find text in page body       | `intext:"password"`              |
| `cache:`              | View cached page             | `cache:example.com`              |
| `link:`               | Find pages linking to target | `link:example.com`               |
| `related:`            | Find similar websites        | `related:example.com`            |
| `info:`               | Basic page information       | `info:example.com`               |
| `define:`             | Get word definition          | `define:phishing`                |
| `numrange:`           | Search number range          | `numrange:1000-2000`             |
| `allintext:`          | All words in body            | `allintext:admin password`       |
| `allinurl:`           | All words in URL             | `allinurl:admin panel`           |
| `allintitle:`         | All words in title           | `allintitle:confidential report` |
| `AND`                 | Both conditions              | `site:example.com AND login`     |
| `OR`                  | Either condition             | `linux OR ubuntu`                |
| `NOT`                 | Exclude term                 | `site:bank.com NOT login`        |
| `*`                   | Wildcard                     | `user* manual`                   |
| `..`                  | Numeric range                | `price 100..500`                 |
| `" "`                 | Exact phrase                 | `"information security"`         |
| `-`                   | Exclude keyword              | `site:news.com -sports`          |

# Section 17 -  Web Archives

Web archives are digital repositories that store snapshots of websites across time, providing a historical record of their evolution. Among these archives, the Wayback Machine is the most comprehensive and accessible resource for web reconnaissance.

The Wayback Machine, a project by the Internet Archive, has been archiving the web for over two decades, capturing billions of web pages from across the globe. This massive historical data collection can be an invaluable resource for security researchers and investigators.

|Feature|Description|Use Case in Reconnaissance|
|---|---|---|
|`Historical Snapshots`|View past versions of websites, including pages, content, and design changes.|Identify past website content or functionality that is no longer available.|
|`Hidden Directories`|Explore directories and files that may have been removed or hidden from the current version of the website.|Discover sensitive information or backups that were inadvertently left accessible in previous versions.|
|`Content Changes`|Track changes in website content, including text, images, and links.|Identify patterns in content updates and assess the evolution of a website's security posture.|

### Automated tools setup

**FinalRecon**

1. git clone https://github.com/thewhiteh4t/FinalRecon.git 
2. cd FinalRecon
3. pip3 install -r requirements.txt
4. chmod +x ./finalrecon.py
5. /finalrecon.py --headers --whois --url http://inlanefreight.com

# Final Question 

At first, i was like how to find the actual full domain out of that vhost and ip given. I had tried nmap, nmap for port 53 -sU, sublist3r, nslookup, nikto but found nothing. Although whatweb and curl  gave small information for **question 1,2**.  So far, no luck for the next ones.

`ffuf -u http://154.57.164.83:3038/ -H "Host: FUZZ.inlanefreight.htb" -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-110000.txt -o output.txt` this although gave a lot of information i didnt quite understand.

after a whole conversation with chatgpt to extract and filter out only the valid hosts from that `output.txt`, with 

```
DEFAULT=$(jq -r '.results[].length' output.txt | sort -n | uniq -c | sort -nr | head -1 | awk '{print $2}')

jq -r --argjson d "$DEFAULT" '
.results[]
| select(.length != $d)
| "\(.input.FUZZ).inlanefreight.htb"
' output.txt

```

it gave me `web1337.inlanefreight.htb` which i added to hosts. Then:

1. `curl -I web1337.inlanefreight.htb:31539/robots.txt` the page works! By manually reviewing the robots.txt page, got the admin page. and there, found the key for **question 3**.
2.  `gobuster vhost -u http://web1337.inlanefreight.htb:30381 -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt -t 500 --append-domain` this gave me another subdomain., added that to hosts. 
3. The ran `python3 ReconSpider.py http://dev.web1337.inlanefreight.htb:30381`
4.  Then `cat results.json`.