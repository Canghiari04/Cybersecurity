# 4 Vulnerabilities (Highest Severity Found)

## V1
CVSS Severity: 10 (Critical)
Vulnerability: Drupal Coder RCE Vulnerability (SA-CONTRIB-2016-039) - Active Check
Summary: Drupal is prone to a remote code execution (RCE) vulnerability.
Service: HTTP
Port: 80/tcp
Detected Version: Drupal 7.5 (running on Apache 2.4.7)
CVE: [3rd party module flaw](https://www.drupal.org/node/2765575)
Implications: An attacker can run arbitrary code on the web server with the privileges of the web user, allowing full compromise of the host, data theft, and use of the machine as a pivot into the internal network. Critical severity is justified because no authentication is required, exploitation is straightforward, and the potential loss of confidentiality, integrity and availability is total.

## V2
CVSS Severity: 9.8 (Critical)
Vulnerability: SSH Brute Force Logins With Default Credentials Reporting
Summary: It was possible to login into the remote SSH server using default credentials.
Service: SSH
Port: 22/tcp
Detected Version: OpenSSH 6.6.1p1 (on Ubuntu 14.04)
CVE: There's quite a few references; this is [one](https://www.cve.org/CVERecord?id=CVE-1999-0501)
Implications: An attacker who guesses the default credentials gains a valid shell on the server, leading to full control of the system (data theft, tampering, malware deployment, lateral movement). Critical severity is warranted because default credentials are publicly known, require no exploit code, and result in complete host compromise.

## V3
CVSS Severity: 7.5 (High)
Vulnerability: SSL/TLS: Report Vulnerable Cipher Suites for HTTPS
Summary: This routine reports all SSL/TLS cipher suites accepted by a service where attack vectors exists only on HTTPS services; vulnerable to SWEET32 attacks.
Service: HTTPS (CUPS)
Port: 631/tcp
Detected Version: CUPS 1.7.2
CVE: [CVE-2016-2183](https://www.cve.org/CVERecord?id=CVE-2016-2183)
Implications: A network attacker able to capture traffic can recover plaintext from long-lived encrypted sessions such as large downloads, breaking confidentiality of the transmitted data. High severity is appropriate because it undermines TLS/SSL protection on an exposed service, though it requires a privileged network position and large amounts of traffic to succeed.

## V4
CVSS Severity: 7.5 (High)
Vulnerability: FTP Brute Force Logins With Default Credentials Reporting
Summary: It was possible to login into the remote FTP server using weak/known credentials.
Service: FTP
Port: 21/tcp
Detected Version: ProFTPD 1.3.5
CVE: [CVE-1999-0502](https://www.cve.org/CVERecord?id=CVE-1999-0502)
Implications: An attacker using weak or known credentials can upload, download, or delete files on the server, potentially replacing web content or planting malicious files for further attacks. High severity is justified because it gives unauthenticated-ish access to stored data and a foothold for further compromise, though impact is limited compared to full RCE.

