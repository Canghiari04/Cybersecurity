# Assignment 05: SQL Injection
### Question 1: Does that indicate that the MySQL server is protected against cyber attacks?
Answer: no

### Question 2: How could that protection look like and what would it protect against? [one sentence each]
1. The MySQL server on Metasploitable virtual machine seems be configured to reject any possible external connection or handshake, recognizing IP addresses that are not in the same local network as Metasploitable virtual machine.
2. By hiding the software version, an attacker cannot search exploit databases to identify already-known and recognized vulnerabilities for that specific service.

### Question 3: Please shortly discuss your opinion of this web server's configuration concerning directly listings. [One short paragraph]
In case an attacker or someone has a full access to the entire directory listing, we can safely state that the web server configuration is poor. Usually, when we visit a web site, the web server automatically directs us to a default page. If this default file is missing, it will show us the directory listings instead, which means a bad configuration.

### Question 4: What type of SQLi attack works? Can you explain why?
There’s only a particular type of SQL injection that is actually working, specifically using a single quote (‘) rather than double quote (“), combined with a condition that is always verified, like (OR 1=1). The example used in order to get useful information was: SELECT * WHERE ‘ OR 1=1#. The reason why only a single quote works lies in two factors. On one hand, the SQL injection strictly depends from the MySQL version, the database server version, and the web application. On the other hand, another reason could be the way the developer wrote the PHP code. Likely, the web application takes into account only the single quote because the developer also used the same character to wrap the input in the code. Understanding these differences allows the attacker to gather much more details about the victim.

### Question 5: In Exercise 03, the MySQL server on port 3306 was reported as "restricted". In Exercise 05 you still got every row of the payroll database and then root access. Reconstruct the chain from the directory listing to sudo -s. Place each step in the assessment phases (gathering information, scanning, vulnerability analysis, exploitation), and say which part of the CIA triad it violates. Then explain why a port-based assessment could not show this exposure, and what that says about what an assessment can and can't guarantee. </br> [Try to stay with one sentence per part]
Given that we already know the IP address of the target machine and its active ports (information obtained by the nmap scanning tool), the attack chain is composed as follows:
1. **Scanning**: using nmap scanning tool to identify the active HTTP service on port 80. (No CIA violation).
2. **Gathering** information: access and discover the directory listing, which contains the payroll_app.php file. (Minor violation of **Confidentiality**).
3. **Vulnerability** analysis: access the payroll_app.php file and test the input fields to discover possible vulnerabilities by SQL injections. (No CIA violation).
4. **Exploitation**: type SQL injection in one of the fields, obtaining a complete dump of the database, including username and password of each user. (Violations of **Confidentiality** and **Integrity**).
5. **Exploitation**: execute a remote login via SSH to the target IP address using the stolen username and password. (Violations of **Confidentiality** and **Integrity**).
6. **Exploitation**: run sudo -s command (once obtained the remote access to the target machine via SSH) typing the same password as before to gain root access (Violates the **whole** CIA triad, since the attacker has a complete root access of the target machine, making them able to read, modify or destroy anything in the system).
A port-assessment cannot reveal these types of vulnerabilities because it only provides network-level information as open ports, firewalls and active services. Therefore, this type of assessment it’s really useful for scanning networks, not for investigating applications running on them. Simply, a port-based assessment can guarantee the security of the network, but it cannot guarantee that applications or services running on those allowed ports are secure from attacks. 

### Question 6: Include four relevant username/password combinations in your report. What is the issue with the passwords in the data base and what could be done to secure them? [One sentence]
Providing a list of four relevant username/password combinations:
1. [Username]: leia_organa, [Password]: help_me_obiwan
2. [Username]: luke_skywalker, [Password]: like_my_father_beforeme
3. [Username]: han_solo, [Password]: nerf_herder
4. [Username]: artoo_detoo, [Password]: b00p_b33p
The main issue is that passwords are saved in the database as plain text. To secure them, they should be stored as cryptographic hashes using hash functions or hashing algorithms like bcrypt, Argon2 or PBKDF2, making brute-force attacks impractical.

### Question 7: Is sudo necessary? What do we gain by using it? [One sentence]
The sudo command is mainly used in order to elevate our privileges to the highest level possible (root access). In this particular case, it is necessary for executing restricted system commands, such as find / -iname “\*payroll\*”. Additionally, by using sudo, we gain complete, unrestricted control over the entire system. 

### Question 8: Are there other ways to search for a file? Which do you know? [Short listing]
All the following commands, executed with root privileges, ensure a complete search without restrictions:
- locate: searches over a pre-built database of file paths. 
- which: identifies the exact position or location of executables.
- grep -r: searches for specific text patterns inside files across directories.  
- whereis: locates binary, source and manual page files for system commands.

### Question 9: How would you suggest to fix the SQLi problem? (Do some online research about SQL injections solutions.)
To avoid SQL injections, developers need to:
- Stop writing **dynamic queries** with string concatenation.
- Prevent malicious SQL input from being included in executed queries.
Possible defenses against SQL injections are:
- **Prepared statement**
  - Prepared statements are features provided by most database management systems that allow you to execute SQL queries with parameters. If queries follow this style, the database will always distinguish data from code. In this way, prepared statements ensure that an attacker cannot change the intent of a query.
- **Stored procedure**
  - A stored procedure is precompiled SQL code that can be saved and reused. Stored procedures can also be defined following the prepared statement design, keeping data apart from code. Furthermore, administrators can define exactly which users have the privilege to run specific procedures, rather than giving anyone the same permission.
- **Principle of least privilege**
  - The principle of least privilege is a security concept in which databases and applications should access data strictly only for the purpose of the task they are meant to perform.

### Question 10: Draft a shortly and crisply, the relevant parts of a policy trying to prevent these issues.
To prevent these issues, we could write down the following policy:
- Never use root system credentials inside web applications.
- Properly configure web servers, avoiding direct directory listings.
- Users should access only specific data and features for their designed tasks, preventing privilege escalation.
- Don’t save plain text passwords. Before storing them, use hash functions of hashing algorithms, and then save them inside the database.
- Avoid string concatenation for performing database queries. Instead, use prepared statements or combining them with stored procedures, keeping data apart from code.

### Question 11: