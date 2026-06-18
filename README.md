# OSCP-Playbook

## Active Information Gathering - 6.4
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
```








