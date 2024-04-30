# Boiler CFT
link to the machine: https://tryhackme.com/r/room/boilerctf2

let's start to enumerate the machine open ports
~~~
nmap -sV -Pn 10.10.70.176 -p-
~~~
output:
~~~
PORT      STATE SERVICE VERSION
21/tcp    open  ftp     vsftpd 3.0.3
80/tcp    open  http    Apache httpd 2.4.18 ((Ubuntu))
10000/tcp open  http    MiniServ 1.930 (Webmin httpd)
55007/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
~~~
we found various services, let's try first to find Let's first try to log in with anonymous user to ftp service and search for some interesting files
~~~
ftp anonymous@10.10.70.176 
Connected to 10.10.70.176.
220 (vsFTPd 3.0.3)
230 Login successful.
~~~
but nothing inside:
~~~
ftp> dir
200 EPRT command successful. Consider using EPSV.
150 Here comes the directory listing.
226 Directory send OK.
~~~
Ok now we start enumerate Apache server with gobuster:
~~~
gobuster dir -u http://10.10.70.176 -w /usr/share/wordlists/dirbuster/directory-list-1.0.txt -t 50
~~~

