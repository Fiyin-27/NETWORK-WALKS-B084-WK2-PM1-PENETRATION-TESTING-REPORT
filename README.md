<div align="center">

# 🔐 Cybersecurity - Foot-printing and Network Scanning

Practical footprinting and network scanning project built with Kali Linux, Nmap, and Zenmap.

</div>

---

## 📌 Project Overview
This project summarizes the practical work I carried out in Week 2 of my Cybersecurity and Ethical Hacking internship with Networkwalks. 

The project covered two major areas: 

- Foot-printing and Reconnaissance using Kali Linux 

- Network Scanning and Host Discovery using Zenmap 

For the foot-printing phase, I collected publicly available information about the authorized domain, networkwalks.com. I used whois, whatweb, nslookup, curl, wafwoof and DNSRecon to gather registration details, web technologies, IP addresses, HTTP headers, firewall indicators and DNS records. 

For the scanning phase, I used Zenmap on my Windows PC to find my local IP address and subnet, detect the live devices on my network, note their IP and MAC addresses, and produce a topology map. 

These exercises showed me how security professionals build a picture of an environment before any deeper testing begins, and why this must only be done with permission. 

## 🎯 Objectives
The goal of this practical was to learn how a security tester collects information about a target and checks what is reachable. Specifically, I set out to: 

- Understand why foot-printing and reconnaissance come before any testing. 

- Gather information about networkwalks.com using Kali Linux tools, including its ownership, IP address, web technologies, headers, firewall and DNS records. 

- Use Zenmap to find my local IP address and subnet. 

- Discover the live devices on my network and note each IP and MAC address. 

- Create a network topology map from the scan. 

- Document everything in a clear, professional report.

## 🛠️ Tools and Purposes

## Tools Used

| Tool | Purpose |
|------|---------|
| **Kali Linux** | Main operating system used for all footprinting and testing work |
| **whois** | Looked up who registered the domain and when |
| **whatweb** | Identified the technologies the website is built on |
| **nslookup** | Found the IP address behind the domain name |
| **curl** | Fetched the website's HTTP headers to see how the server responds |
| **wafw00f** | Checked whether a Web Application Firewall protects the site |
| **dnsrecon** | Listed the domain's DNS records |
| **Nmap** | Scanning engine that runs underneath Zenmap |
| **Zenmap** | Graphical front end for scanning my network and drawing its topology |
| **Windows Command Prompt** | Ran `ipconfig` to check my PC's IP address and network details, so I could work out my subnet before scanning |

## ⚙️ Lab Configuration

| Component | Configuration |
|-----------|---------------|
| Host operating system | Windows 10 |
| Hypervisor | Oracle VirtualBox 7.2 |
| Security OS | Kali Linux |
| RAM | 8 GB |
| Processor | Intel Core i7 |
| Network environment | Virtual network |
| Primary target | `networkwalks.com` |
| Secondary target | `scanme.nmap.org` |
| Secondary target IP | 45.33.32.156 |

<div align="center">
  
## Part 1 - Foot-printing and Reconnaissance

</div>

Footprinting means collecting information about a target before any deeper security testing. Here, I used Kali Linux tools to learn about the publicly visible side of `networkwalks.com`, from its registration details to its DNS records.

---

### 1. whois Lookup

| | |
|---|---|
| **Tool** | whois |
| **Target** | `networkwalks.com` |

#### Command

```bash
whois networkwalks.com
```

#### Purpose
To collect the public registration record of the target domain, including the registrar, key dates, and the authoritative name servers.

#### Findings

| Item | Result |
|------|--------|
| Registrar | GoDaddy.com, LLC |
| Creation date | 2019-11-06 |
| Expiry date | 2027-11-06 |
| Name servers | NS6135.HOSTGATOR.COM, NS6136.HOSTGATOR.COM |
| Hosting | HostGator |
| Domain status | Locked (clientDeleteProhibited, clientTransferProhibited, etc.) |

#### Observations

- The domain was registered through GoDaddy in November 2019 and is registered until November 2027.
- The name servers belong to HostGator, which suggests the site's DNS and hosting are managed there, separately from the registrar.
- The `client...Prohibited` statuses show the domain is locked against unauthorized deletion, transfer, and updates.
- No registrant details appear in the registry WHOIS output, only the registrar's abuse contact.


#### Screenshot

![whois lookup result](screenshots/whois.png)

### 2.  whatweb: Web Technology Fingerprinting

| | |
|---|---|
| **Tool** | whatweb |
| **Target** | `networkwalks.com` |

#### Command

```bash
whatweb networkwalks.com
```

#### Purpose

To discover the software and technologies behind the target website.

#### Findings

| Item | Result |
|------|--------|
| Web server | Apache |
| CMS | WordPress 7.1.4 |
| Plugin | WordPress Download Manager 3.3.76 |
| Server IP | 192.232.216.135 |
| Redirect | HTTP redirects (301) to HTTPS |

#### Observations

- The site runs on **WordPress** with the **WordPress Download Manager** plugin, served by **Apache**.
- HTTP requests are redirected to HTTPS.
- The site exposes software version numbers, which helps a tester match them against known vulnerabilities.

#### Screenshot

![whatweb result](screenshots/whatweb.png)    

### 3. Nslookup: DNS Resolution

| | |
|---|---|
| **Tool** | nslookup |
| **Target** | `networkwalks.com` |

#### Command

```bash
nslookup networkwalks.com
```

#### Purpose

To find the IP address that the domain name points to.

#### Findings

| Item | Result |
|------|--------|
| DNS server used | 8.8.8.8 (Google Public DNS) |
| Domain | networkwalks.com |
| IP address | 192.232.216.135 |
| Answer type | Non-authoritative |

#### Observations

- The domain name resolved to the IP address `192.232.216.135`.
- This is the same IP address WhatWeb reported for the site, so both tools agree.
- The answer was non-authoritative, meaning it came from Google's DNS server and not from the domain's own name servers.
- Resolving a domain to its IP address is a basic reconnaissance step, because it shows which server to look at next.

#### Screenshot

![nslookup result](screenshots/nslookup.png)

---
### 4. Curl: HTTP Response Headers

| | |
|---|---|
| **Tool** | Curl |
| **Target** | `https://networkwalks.com` |

#### Command

```bash
curl -I https://networkwalks.com
```

#### Purpose

To check how the web server responds to a request by reading its HTTP headers.

#### Findings

| Item | Result |
|------|--------|
| Response code | HTTP/2 200 (OK) |
| Web server | Apache |
| Cache header | `x-nginx-cache: WordPress` |
| Cookie set | `__wpdm_client` (Secure, HttpOnly) |
| Referrer policy | `no-referrer-when-downgrade` |
| API link | WordPress REST API (`/wp-json/`) listed in the Link header |

#### Observations

- The `-I` option fetched only the headers, so the full page was not downloaded.
- The headers confirmed the Apache web server found earlier with WhatWeb.
- The Link header points to the WordPress REST API, which further confirms the site runs on WordPress.
- The session cookie is marked Secure and HttpOnly, which are good cookie security settings.

#### Screenshot

![curl result](screenshots/curl-I.png)

---

### 5. wafw00f: Web Application Firewall Detection

| | |
|---|---|
| **Tool** | wafw00f |
| **Target** | `networkwalks.com` |

#### Command

```bash
wafw00f networkwalks.com
```

#### Purpose

To find out if the website sits behind a Web Application Firewall (WAF) and, if so, which one.

#### Findings

| Item | Result |
|------|--------|
| WAF detected | Yes |
| WAF name | ModSecurity (SpiderLabs) |
| Requests sent | 2 |

#### Observations

- The site is protected by a WAF, identified as ModSecurity (SpiderLabs).
- Knowing a WAF is present helps a tester plan how to approach later authorized testing.
- Detecting a WAF does not show how well it is configured or how effective the protection is.

#### Screenshot

![wafw00f result](screenshots/wafw00f.png)

---

### 6. DNSRecon: DNS Record Enumeration

| | |
|---|---|
| **Tool** | DNSRecon |
| **Target** | `networkwalks.com` |

#### Command

```bash
dnsrecon -d networkwalks.com
```

#### Purpose

To list the DNS records of the domain and see how its name resolution is set up.

#### Findings

| Record | Result |
|--------|--------|
| A | networkwalks.com → 192.232.216.135 |
| NS | ns6135.hostgator.com (50.87.144.87), ns6136.hostgator.com (192.232.216.131) |
| MX | mail.networkwalks.com → 192.232.216.135 |
| TXT | SPF record and Google site verification record |
| SRV | 8 records found (cPanel email autodiscovery) |
| DNSSEC | No answer to the DNSSEC query |

#### Observations

- The A record and the MX record both point to `192.232.216.135`, so the website and mail share the same server.
- The name servers belong to HostGator, which matches the WHOIS result.
- Both name servers reported their BIND version (`9.16.23-RH`), which is a small version disclosure.
- The SRV records show the domain uses cPanel email autodiscovery.
- The SPF record shows which servers are allowed to send email for the domain.

#### Screenshot

![dnsrecon result](screenshots/dnsrecon.png)

