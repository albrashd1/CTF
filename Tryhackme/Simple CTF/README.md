**1. How many services are running under port 1000?**

Let's start with enumerationg the machine using NMAP and save the result to txt file:
`nmap -sS -sV -A <IP> -oN nmap.txt`
```
snip ...
PORT     STATE SERVICE VERSION
21/tcp   open  ftp     vsftpd 3.0.3
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.6.36.242
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 4
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: TIMEOUT
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
| http-robots.txt: 2 disallowed entries 
|_/ /openemr-5_0_1_3 
|_http-server-header: Apache/2.4.18 (Ubuntu)
2222/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
snip ...
```
We can see there are three open ports and two of them are under 1000. 


**2. What is running on the higher port? **
ssh is running on port 2222


Now, we need to enumerate a little bit to find out our CVE to use. Our NMAP showing us that there is an apache server is running on port 80, let's go and open it using our IP. 
