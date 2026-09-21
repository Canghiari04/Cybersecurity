# Exercise 03: General Assessment

## 1 Finding Information with whois
### 1. Trying to gather information on SDU with whois
`whois` is a widely used IP protocol to query databases storing information about domain names and IP addresses. This command queries the registration database, not **DNS** (Domain Name System).

We would like to gather information about **SDU** (University of Southern Denmark). Therefore, we need to find out which is the domain name of this entity.

> Note: we are looking for the **domain name**, not **host name**. 

```bash
kali㉿kali$ whois sdu.dk
Domain:               sdu.dk
DNS:                  sdu.dk
Registered:           1997-10-09
Expires:              2028-12-31
Registrar:            dns.services ApS
Registration period:  1 year
VID:                  no
DNSSEC:               Signed delegation
Status:               Active

Registrant
Handle:               DATA REDACTED
Name:                 Syddansk Universitet (University of Southern Denmark)
Address:              Campusvej 55
Postalcode:           5230
City:                 Odense M
Country:              DK
Email:                hostmaster@sdu.dk
ID status:            ID verified via electronic ID

Nameservers
Hostname:             ns1.sdu.dk
Hostname:             ns2.sdu.dk
Hostname:             ns3.sdu.dk
```

We got some useful information, one of those is about `Nameservers`: SDU has three active servers, waiting for incoming connections. Probably these servers are used in order to provide some internal services.

### 2. Try whois on the IP address of www.sdu.dk
As we already know, `whois` is a tool to gather information from registration databases, asking about domain names and IP addresses. Therefore, we can't use it with a **host name**, such `www.sdu.dk`.

First of all, we have to resolve the **host name** into an **IP address** and this is possible through the command `dig +[options] [domain]`. It will provide us the IP address associated to the host name `www.sdu.dk`.

Once we have obtained the IP address we can proceed to retrieve information using `whois <IP address>` command.

```bash
kali㉿kali$ dig +short www.sdu.dk
sdu.dk.
52.233.201.66
```

```bash
kali㉿kali$ whois 52.233.201.66
NetRange:       52.224.0.0 - 52.255.255.255
CIDR:           52.224.0.0/11
NetName:        MSFT
NetHandle:      NET-52-224-0-0-1
Parent:         NET52 (NET-52-0-0-0-0)
NetType:        Direct Allocation
OriginAS:       
Organization:   Microsoft Corporation (MSFT)
RegDate:        2015-11-24
Updated:        2021-12-14
Ref:            https://rdap.arin.net/registry/ip/52.224.0.0

OrgName:        Microsoft Corporation
OrgId:          MSFT
Address:        One Microsoft Way
City:           Redmond
StateProv:      WA
PostalCode:     98052
Country:        US
RegDate:        1998-07-10
Updated:        2025-06-10
Comment:        To report suspected security issues specific to traffic emanating from Microsoft online services, including the distribution of malicious content or other illicit or illegal material through a Microsoft online service, please submit reports to:
Comment:        * https://cert.microsoft.com.  
Comment:        
Comment:        For SPAM and other abuse issues, such as Microsoft Accounts, please contact:
Comment:        * abuse@microsoft.com.  
Comment:        
Comment:        To report security vulnerabilities in Microsoft products and services, please contact:
Comment:        * secure@microsoft.com.  
Comment:        
Comment:        For legal and law enforcement-related requests, please contact:
Comment:        * msndcc@microsoft.com
Comment:        
Comment:        For routing, peering or DNS issues, please 
Comment:        contact:
Comment:        * IOC@microsoft.com
Ref:            https://rdap.arin.net/registry/entity/MSFT

OrgTechHandle: BEDAR6-ARIN
OrgTechName:   Bedard, Dawn 
OrgTechPhone:  +1-425-538-6637 
OrgTechEmail:  dabedard@microsoft.com
OrgTechRef:    https://rdap.arin.net/registry/entity/BEDAR6-ARIN

OrgTechHandle: MRPD-ARIN
OrgTechName:   Microsoft Routing, Peering, and DNS
OrgTechPhone:  +1-425-882-8080 
OrgTechEmail:  IOC@microsoft.com
OrgTechRef:    https://rdap.arin.net/registry/entity/MRPD-ARIN

OrgAbuseHandle: MAC74-ARIN
OrgAbuseName:   Microsoft Abuse Contact
OrgAbusePhone:  +1-425-882-8080 
OrgAbuseEmail:  abuse@microsoft.com
OrgAbuseRef:    https://rdap.arin.net/registry/entity/MAC74-ARIN

OrgTechHandle: SINGH683-ARIN
OrgTechName:   Singh, Prachi 
OrgTechPhone:  +1-425-707-5601 
OrgTechEmail:  pracsin@microsoft.com
OrgTechRef:    https://rdap.arin.net/registry/entity/SINGH683-ARIN

OrgTechHandle: IPHOS5-ARIN
OrgTechName:   IPHostmaster, IPHostmaster 
OrgTechPhone:  +1-425-538-6637 
OrgTechEmail:  iphostmaster@microsoft.com
OrgTechRef:    https://rdap.arin.net/registry/entity/IPHOS5-ARIN

OrgRoutingHandle: CHATU3-ARIN
OrgRoutingName:   Chaturmohta, Somesh 
OrgRoutingPhone:  +1-425-882-8080 
OrgRoutingEmail:  someshch@microsoft.com
OrgRoutingRef:    https://rdap.arin.net/registry/entity/CHATU3-ARIN
```

> Note: there's a huge difference between this result and the previous one. Before we got only registration details. Right now we have some information about `Netrange`, `Orgname`, `Orgid` and so on.

### 3. What do you learn about SDU's network? In the protocol, note the IP range.
A bunch of details are coming out from the previous section, such as:
1. `Netrange`: 52.224.0.0 - 52.255.255.255
2. `CIDR`: 52.224.0.0/11
3. `Organization`: Microsoft Corporate (MSFT), so Microsoft owns the address space defined before

> Note: the fact that Microsoft owns the address space, it doesn't mean that it's actually running `www.sdu.dk`.

### 4. Are there other Networking-Services @SDU which you could try?
We could divide services provided by SDU into:
- **Internal services**:
  - `nextcloud.sdu.dk`
  - `selvbprod.sdu.dk`
  - `sprint.sdu.dk`
  
  All of these services are self-hosted: their IP addresses fall in the same `NetRange` and they show the same `org-name` "**Syddansk Universitet, IT-service**".
- **External services**:
  - `sdu.itslearning.com`

### 5. What is the whois information for nextcloud.sdu.dk?
`nextcloud.sdu.dk` is a host name: we have to resolve it in an IP address and then run the `whois <IP address>` command.

```bash
kali㉿kali$ dig +short nextcloud.sdu.dk
130.225.156.61
```

```bash 
kali㉿kali$ whois 130.225.156.61
NetRange:       130.225.0.0 - 130.244.255.255
CIDR:           130.226.0.0/15, 130.228.0.0/14, 130.232.0.0/13, 130.225.0.0/16, 130.240.0.0/14, 130.244.0.0/16
NetName:        RIPE-ERX-130-225-0-0
NetHandle:      NET-130-225-0-0-1
Parent:         NET130 (NET-130-0-0-0-0)
NetType:        Early Registrations, Transferred to RIPE NCC
OriginAS:       
Organization:   RIPE Network Coordination Centre (RIPE)
RegDate:        2003-11-12
Updated:        2025-02-10
Comment:        These addresses have been further assigned to users in the RIPE NCC region. Please note that the organization and point of contact details listed below are those of the RIPE NCC not the current address holder. ** You can find user contact information for the current address holder in the RIPE database at http://www.ripe.net/whois.
Ref:            https://rdap.arin.net/registry/ip/130.225.0.0

ResourceLink:  https://apps.db.ripe.net/db-web-ui/query
ResourceLink:  whois.ripe.net

OrgName:        RIPE Network Coordination Centre
OrgId:          RIPE
Address:        P.O. Box 10096
City:           Amsterdam
StateProv:      
PostalCode:     1001EB
Country:        NL
RegDate:        
Updated:        2013-07-29
Ref:            https://rdap.arin.net/registry/entity/RIPE

ReferralServer:  whois.ripe.net
ResourceLink:  https://apps.db.ripe.net/db-web-ui/query

OrgAbuseHandle: ABUSE3850-ARIN
OrgAbuseName:   Abuse Contact
OrgAbusePhone:  +31205354444 
OrgAbuseEmail:  abuse@ripe.net
OrgAbuseRef:    https://rdap.arin.net/registry/entity/ABUSE3850-ARIN

OrgTechHandle: RNO29-ARIN
OrgTechName:   RIPE NCC Operations
OrgTechPhone:  +31 20 535 4444 
OrgTechEmail:  hostmaster@ripe.net
OrgTechRef:    https://rdap.arin.net/registry/entity/RNO29-ARIN
```

Currently, there are some differences between the whois-information collected from `www.sdu.dk` and those of `nextcloud.sdu.dk`, which are:
1. `NetRange`: they belong to different IP address blocks 
   1. `nextcloud.sdu.dk` $\rightarrow$ `130.255.0.0` - `130.244.255.255`
   2. `www.sdu.dk` $\rightarrow$ `52.224.0.0` - `52.225.255.255`
2. `NetName`: `nextcloud.sdu.dk` IP address block does not belong to Microsoft Corporation, which already leads us to assume that it's a self-hosted service
   1. `nextcloud.sdu.dk` $\rightarrow$ `RIPE-ERX-130-225-0-0`
   2. `www.sdu.dk` $\rightarrow$ `MSFT`
   
To gather more details than before, we could run also another type of `whois` command. By `whois -h whois.ripe.net <IP address>` we send a query to the **RIPE NCC's whois server** and it returns the RIPE record of that IP. In this way, we are just choosing which registration database to query, in order to get more specific information.

```bash 
kali㉿kali$ whois -h whois.ripe.net 130.225.156.61
inetnum:        130.225.128.0 - 130.225.159.255
netname:        SDU-v4-POOL-01
country:        DK
geofeed:        https://info.net.deic.dk/deic-geofeed.csv
org:            ORG-SUI1-RIPE
admin-c:        UN61-RIPE
tech-c:         UN61-RIPE
status:         ASSIGNED PA
remarks:        Generated by DeiC on 2022-07-28 for more information contact netdrift@deic.dk
mnt-by:         DEIC-MNT
mnt-by:         AS1835-MNT
created:        2015-12-10T10:05:14Z
last-modified:  2022-07-28T11:50:21Z
source:         RIPE

organisation:   ORG-SUI1-RIPE
org-name:       Syddansk Universitet, IT-service
org-type:       other
address:        Campusvej 55
address:        5230 Odense M
address:        DK
mnt-ref:        AS1835-MNT
mnt-by:         AS1835-MNT
mnt-by:         DEIC-MNT
created:        2012-05-03T10:51:17Z
last-modified:  2022-01-28T14:00:25Z
source:         RIPE

role:           DeiC Netdrift
address:        DeiC
address:        DTU Building 304
address:        2800 Lyngby
address:        Denmark
phone:          +45 35 888 222
fax-no:         +45 35 888 201
admin-c:        AMD2-RIPE
tech-c:         AMD2-RIPE
tech-c:         JF6044-RIPE
tech-c:         HUB10-RIPE
nic-hdl:        UN61-RIPE
mnt-by:         AS1835-MNT
mnt-by:         DEIC-MNT
created:        2008-11-24T13:12:55Z
last-modified:  2022-01-28T14:00:26Z
source:         RIPE
abuse-mailbox:  abuse@cert.dk

% Information related to '130.225.0.0/16AS1835'

route:          130.225.0.0/16
descr:          Forskningsnettet-130.225
origin:         AS1835
mnt-by:         AS1835-MNT
mnt-by:         DEIC-MNT
created:        1970-01-01T00:00:00Z
last-modified:  2022-01-28T14:00:18Z
source:         RIPE
```

Finally, thanks to the final command presented (`whois -h whois.ripe.net <IP address>`), we can confirm that `nextcloud.sdu.dk` is a self-hosted service, owned by `Syddansk Universitet, IT-service`.

## 2 Question: nmap
### 1. Send packets with specified ip options
> nmap --ip-options \<options> {target specification}

### 3. Spoof your MAC address
> nmap --spoof-mac \<MAC> {target specification}

## 3 Scanning the Metasploitable VMs
We need to provide three different types of scans via the `nmap` tool, which are:
1. `SYN scan`, and half-open scanning method that determines port states without completing the TCP handshake
2. `Connect scan`, full TCP connection scanning method that defines port states completing the TCP three-way handshake
3. `Full scan`, heavy and detailed scanning method that combines OS detection, version detection and script scanning completing every time a full TCP handshake. It provides not only port states but also what software and versions are actually running on them

First of all, we need to retrieve the IP address of the Metasploitable virtual machine and then we can run the scans listed above.

```bash
vagrant@ubuntu$ ip a sh
...
```

```bash
kali㉿kali$ nmap -sS ...
...
```

```bash
kali㉿kali$ sudo nmap -sT ...
...
```

```bash
kali㉿kali$ sudo nmap -A ...
...
```

For each of them we are gonna to describe a list of advantages and disadvantages, starting from the SYN scan.
- `SYN scan`:
    - Advantages:
        1. ...
    - Disadvantages:
        1. ...
- `Connection scan`:
    - Advantages:
        1. ...
    - Disadvantages:
        1. ...
- `Full scan`:
    - Advantages:
        1. ...
    - Disadvantages:
        1. ...

## 4 Vulnerabilities (Highest Severity Found)

### V1
CVSS Severity: 10 (Critical)
Vulnerability: Drupal Coder RCE Vulnerability (SA-CONTRIB-2016-039) - Active Check
Summary: Drupal is prone to a remote code execution (RCE) vulnerability.
Service: HTTP
Port: 80/tcp
Detected Version: Drupal 7.5 (running on Apache 2.4.7)
CVE: [3rd party module flaw](https://www.drupal.org/node/2765575)
Implications: An attacker can run arbitrary code on the web server with the privileges of the web user, allowing full compromise of the host, data theft, and use of the machine as a pivot into the internal network. Critical severity is justified because no authentication is required, exploitation is straightforward, and the potential loss of confidentiality, integrity and availability is total.

### V2
CVSS Severity: 9.8 (Critical)
Vulnerability: SSH Brute Force Logins With Default Credentials Reporting
Summary: It was possible to login into the remote SSH server using default credentials.
Service: SSH
Port: 22/tcp
Detected Version: OpenSSH 6.6.1p1 (on Ubuntu 14.04)
CVE: There's quite a few references; this is [one](https://www.cve.org/CVERecord?id=CVE-1999-0501)
Implications: An attacker who guesses the default credentials gains a valid shell on the server, leading to full control of the system (data theft, tampering, malware deployment, lateral movement). Critical severity is warranted because default credentials are publicly known, require no exploit code, and result in complete host compromise.

### V3
CVSS Severity: 7.5 (High)
Vulnerability: SSL/TLS: Report Vulnerable Cipher Suites for HTTPS
Summary: This routine reports all SSL/TLS cipher suites accepted by a service where attack vectors exists only on HTTPS services; vulnerable to SWEET32 attacks.
Service: HTTPS (CUPS)
Port: 631/tcp
Detected Version: CUPS 1.7.2
CVE: [CVE-2016-2183](https://www.cve.org/CVERecord?id=CVE-2016-2183)
Implications: A network attacker able to capture traffic can recover plaintext from long-lived encrypted sessions such as large downloads, breaking confidentiality of the transmitted data. High severity is appropriate because it undermines TLS/SSL protection on an exposed service, though it requires a privileged network position and large amounts of traffic to succeed.

### V4
CVSS Severity: 7.5 (High)
Vulnerability: FTP Brute Force Logins With Default Credentials Reporting
Summary: It was possible to login into the remote FTP server using weak/known credentials.
Service: FTP
Port: 21/tcp
Detected Version: ProFTPD 1.3.5
CVE: [CVE-1999-0502](https://www.cve.org/CVERecord?id=CVE-1999-0502)
Implications: An attacker using weak or known credentials can upload, download, or delete files on the server, potentially replacing web content or planting malicious files for further attacks. High severity is justified because it gives unauthenticated-ish access to stored data and a foothold for further compromise, though impact is limited compared to full RCE.