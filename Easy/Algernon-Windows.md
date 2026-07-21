# Algernon - Windows - OffSec - Proven Grounds

Completed: July 21, 2026 ~ 42:14 mins
References: 

## Workflow
### Inital Recon
```
# Inital scan 
nmap -Pn -n -sS -p- -sV --min-hostgroup 255 --min-rtt-timeout 25ms --max-rtt-timeout 100ms --max-retries 1 --max-scan-delay 0 --min-rate 1000 -oA Full-Ports-Scan -vvv --open 192.168.229.65


# Ips and Ports
cat Full-Ports-Scan.nmap| awk '/^Nmap scan report/{ip=$NF} /(\/tcp|\/udp)/ && /open/{split($1,p,"/"); print ip ":" p[1]}' | sort -u > discovery_scan_ip_ports.txt

# Only ports
cat discovery_scan_ip_ports.txt | cut -d: -f2 | sort -u > only-ports-list.txt

# Detailed information on open ports
nmap -Pn -sV -sC --open -p "$(paste -sd, only-ports-list.txt)" 192.168.205.40 -oA detailed-ports-results
```
#### Enumeration
```
nmap -Pn -sV -sC --open -p "$(paste -sd, only-ports-list.txt)" 192.168.229.65 -oA detailed-ports-results

Starting Nmap 7.99 ( https://nmap.org ) at 2026-07-21 14:02 -0400
Nmap scan report for 192.168.229.65
Host is up (0.024s latency).
Not shown: 1 closed tcp port (reset)
PORT      STATE SERVICE       VERSION
21/tcp    open  ftp           Microsoft ftpd
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| 04-29-20  10:31PM       <DIR>          ImapRetrieval
| 07-21-26  10:54AM       <DIR>          Logs
| 04-29-20  10:31PM       <DIR>          PopRetrieval
|_04-29-20  10:32PM       <DIR>          Spool
| ftp-syst: 
|_  SYST: Windows_NT
80/tcp    open  http          Microsoft IIS httpd 10.0
|_http-title: IIS Windows
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds?
5040/tcp  open  unknown
9998/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-IIS/10.0
| http-title: Site doesn't have a title (text/html; charset=utf-8).
|_Requested resource was /interface/root
| uptime-agent-info: HTTP/1.1 400 Bad Request\x0D
| Content-Type: text/html; charset=us-ascii\x0D
| Server: Microsoft-HTTPAPI/2.0\x0D
| Date: Tue, 21 Jul 2026 18:04:48 GMT\x0D
| Connection: close\x0D
| Content-Length: 326\x0D
| \x0D
| <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN""http://www.w3.org/TR/html4/strict.dtd">\x0D
| <HTML><HEAD><TITLE>Bad Request</TITLE>\x0D
| <META HTTP-EQUIV="Content-Type" Content="text/html; charset=us-ascii"></HEAD>\x0D
| <BODY><h2>Bad Request - Invalid Verb</h2>\x0D
| <hr><p>HTTP Error 400. The request verb is invalid.</p>\x0D
|_</BODY></HTML>\x0D
17001/tcp open  remoting      MS .NET Remoting services
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
49669/tcp open  msrpc         Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2026-07-21T18:04:50
|_  start_date: N/A

```
#### Flow
Ok so we have some interesting ports open, right out of the gate I am seeing `Anonymous FTP login allowed` and I'm just poking around to see what is available and in the logs it has entries for `administrator` but I didn't see any references to password. 

I tried reaching `http://192.168.229.65:9998` and it landed on a login page for SmarterMail and tried default creds and nothing was going through. 
So I went back to the nmap results and tried searching to see what was on port 17001 `MS .NET Remoting services 17001` and in the results I see that htere is Exploit-DB for RCE but it was only for Windows and then right above that search entry I saw a GitHub markdown page `smatermail_rce.md` 

<img width="941" height="399" alt="image" src="https://github.com/user-attachments/assets/1d7f01ed-1b4f-49e9-a471-ce0c635b18e3" />

Taking a look at the markdown file contents, it seem to be describing this target's usecase. 
I keep scrolling and see an example scenario to use this on:
https://github.com/rapid7/metasploit-framework/blob/master/documentation/modules/exploit/windows/http/smartermail_rce.md#scenarios

I literally do the same thing, it does use metasploit and get the proof.txt flag. 

