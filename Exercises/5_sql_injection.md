# Exercise 05: SQL Injection

## 5.a Preparation
### 5.a.1 From the assessment you performed with nmap and gvm in exercise 03, notice that ports for TCP - Hypertext Transfer Protocol (HTTP) and a MySQL Server are open. What are the port numbers?
We already know that `nmap` is a scanning tool in order to get some information about target system's network such as active ports, port numbers or services running on them. To obtain such information, we should execute the following command (knowing already IP address of the target machine):
```bash
kali㉿kali$ nmap -sT 10.0.2.15
Starting Nmap 7.99 ( https://nmap.org ) at 2026-10-04 05:39 -0400
Nmap scan report for 10.0.2.15
Host is up (0.00035s latency).
Not shown: 991 filtered tcp ports (no-response)
PORT     STATE  SERVICE
21/tcp   open   ftp
22/tcp   open   ssh
80/tcp   open   http
445/tcp  open   microsoft-ds
631/tcp  open   ipp
3000/tcp closed ppp
3306/tcp open   mysql
8080/tcp open   http-proxy
8181/tcp closed intermapper
MAC Address: 08:00:27:42:51:79 (Oracle VirtualBox virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 5.30 seconds
                                                                                            
kali㉿kali$
```
Therefore, the port numbers are 80 and 3306 for HTTP ans MySQL Server respectively.

### 5.a.2 Does that indicate that the MySQL server is protected against cyber attacks?
From the previous scan we didn't get any information about the services' software version. In this case, we can still use the `nmap` scanning tool but with a different type of scan. We have to run the following command:
```bash
kali㉿kali$ nmap -sV 10.0.2.15
Starting Nmap 7.99 ( https://nmap.org ) at 2026-10-04 05:52 -0400
Nmap scan report for 10.0.2.15
Host is up (0.00026s latency).
Not shown: 991 filtered tcp ports (no-response)
PORT     STATE  SERVICE     VERSION
21/tcp   open   ftp         ProFTPD 1.3.5
22/tcp   open   ssh         OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.13 (Ubuntu Linux; protocol 2.0)
80/tcp   open   http        Apache httpd 2.4.7
445/tcp  open   netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
631/tcp  open   ipp         CUPS 1.7
3000/tcp closed ppp
3306/tcp open   mysql       MySQL (unauthorized)
8080/tcp open   http        Jetty 8.1.7.v20120910
8181/tcp closed intermapper
MAC Address: 08:00:27:42:51:79 (Oracle VirtualBox virtual NIC)
Service Info: Hosts: 127.0.1.1, UBUNTU; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 10.99 seconds
                                                                                            
kali㉿kali$
```
The answer of the original question is: no, MySQL server is not protected against cyber attacks only because the software version of it is not available. Software version is just what the service claims to be, not whether the binary code is reliable or it has no vulnerabilities.

After that, we execute an other command trying to establish a full handshake over the 3306 port with the Metasplotable virtual machine.
```bash
kali㉿kali$ mysql -h 10.0.2.15 -P 3306
WARNING: option --ssl-verify-server-cert is disabled, because of an insecure passwordless login.
ERROR 2002 (HY000): Received error packet before completion of TLS handshake. The authenticity of the following error cannot be verified: 1130 - Host '10.0.2.4' is not allowed to connect to this MySQL server
                                                                                            
kali㉿kali$
```

### 5.a.3 How could that protection look like? And what exactly would it protect against?
1. The MySQL server on Metasploitable virtual machine seems be configured to reject any possible external connection or handshake, recognizing IP addresses that are not in the same local network as Metasploitable virtual machine.
2. If the software version is hidden, an attacker cannot search exploit databases to identify already-known and recognized vulnerabilities for that specific service.

## 5.b Spying with SQL Injection
### 5.b.1 Please shortly discuss your opinion of this web server's configuration concerning directly listings.
In case an attacker or someone has a full access to the entire directory listing, we can safely state that the web server configuration is poor. Usually, when we visit a web site, the web server automatically directs us to a default page. If this default file is missing, it will show us the directory listings instead, which means a bad configuration.

### 5.b.2 What type of SQLi attack works? Can you explain why?
There’s only a particular type of SQL injection that is actually working, specifically using a single quote (‘) rather than double quote (“), combined with a condition that is always verified, like (OR 1=1). The example used in order to get useful information was: SELECT * WHERE ‘ OR 1=1#. The reason why only a single quote works lies in two factors. On one hand, the SQL injection strictly depends from the MySQL version, the database server version, and the web application. On the other hand, another reason could be the way the developer wrote the PHP code. Likely, the web application takes into account only the single quote because the developer also used the same character to wrap the input in the code. Understanding these differences allows the attacker to gather much more details about the victim.

### 5.b.3 What is the # sign for? Can we generally assume it to do the trick?
SQL is a domain specific language used to manage data. Usually, the sharp symbol (#) is used to write down comments inside the code. 

### 5.b.4 Include four relevant username/password combinations in your report. What is the issue with the passwords in the data base and what could be done to secure them?
Providing a list of four relevant username/password combinations:
- [Username]: leia_organa, [Password]: help_me_obiwan
- [Username]: luke_skywalker, [Password]: like_my_father_beforeme
- [Username]: han_solo, [Password]: nerf_herder
- [Username]: artoo_detoo, [Password]: b00p_b33p

The main issue is that passwords are saved in the database as plain text. To secure them, they should be store as cryptographic hashes using hash functions or hashing algorithms like bcrypt, Argon2 or PBKDF2, making brute-force attacks impractical.

## Elevation of Privilige
### 5.c.1 