# OSCP-Playbook
_"Good pentesters don't build their careers running tools. They build their career on explaining risk and why the findings matter. Connecting technical details to business risk. The question you should be asking yourself is whether your value comes from operating tools or understanding business risk. - Dr. Mic Merritt"_

## Active Information Gathering
**Perform Port Scanning with Netcat and Nmap**
**Enumerate DNS, SMB, SMTP, and SNMP Servers**


**Apply Living Off the Land Techniques**

Living Off the Land Binaries (LOLBins) and Living Off the Land Binaries and Scripts (LOLBAS). 

Leveraging pre-installed and trusted Windows binaries to perform post-compromise analysis.
Tools like whoami.exe, ping.exe, and netstat.exe are considered LOLBAS because they are included with Windows and can be repurposed for enumeration without triggering security alerts.

## NSE vulnerability scripts (kali)
```
# navigating the Nmap script database
cd /usr/share/nmap/scripts/
cat script.db  | grep "\"vuln\""

# runs all vuln category NSE scripts against port 443
sudo nmap -sV -p 443 --script "vuln" <ip>

# Lookup, copy, and update the NSE script nad the script.db database
# search bar
CVE-2021-41773 nse

sudo cp /home/kali/Downloads/http-vuln-cve-2021-41773.nse /usr/share/nmap/scripts/http-vuln-cve2021-41773.nse

sudo nmap --script-updatedb

# how to specifically utilize NSE script
sudo nmap -sV -p 443 --script "http-vuln-cve2021-41773" 192.168.50.124
```

## Web App Assessment Methodology
```
# Discover web server version 
sudo nmap -p80 -sV <ip>

# Running nmap NSE http enumeration scripts
sudo nmap -p80 --script=http-enum <ip>

# Web enuneration for publicly accessible files and directories
gobuster dir -u <ip> -w /usr/share/wordlists/dirb/common.txt -t 5

Context clues can be found in the source of the web page.
Web apps can also include sitemap files to help search engine bots crawl and index their sites as well as directives of URLs not to crawl. 
```

Application Programming Interface (API) - responsible for interacting with the back-end logic and providing a solid backbone of functions to the web application. 

Representational State Transfer (REST) is a type of API used for a variety of purposes, including authentication.

POST and PUT methods, if permitted on a specific API, could allow us to override the user credentials. 

### XSS 
Unsanitized data allows an attacker to inject and potentially execute malicious code. 
XSS is a vuln that exploits a user's trust in a website by dynamically injecting content into the page rendered by the user's browser. 
Allows for client-side script(s) injection like Javascipt into web pages visited by other users. 

**Stored, Persistent** - payload is stored in a database or cached by a server. Web app retrieves this payload and displays it to anyone who visits the vulnerable page. 
Seen in forum software - comment sections, product reviews, or whatever user content can be stored and reviewed later. 

**Reflected** - web app takes the payload that is crafted in the request or link and places it into the page content. Only attacks the person submitting the request or visiting the link. 
Often found in search fields amd results as well as anywhere user input is included in error messages. 

Stored or reflected can be manifested as client (browser) or server-side as well as DOM-based.

DOM-based XSS takes place soley within the page's Document Object Model (DOM). Browsers parse page's HTML content and then generate an internal DOM representation. When a page's DOM is modified with user-controlled values - DOM XSS occurs. Can be reflected or stored. DOM-based XSS attacks occur when a browser parses the page's content and inserted JavaScript is executed.

The user's browser - NOT the web app executes the XSS payload. This attack can escalate to sesssion hijacking, force redirection to malicious pages, execution of local applications as that user or even trojanized web app. 






