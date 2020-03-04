# Information Security Engineering Review

## 1.Maicious Software

##### S1.what are three broad mechanisms that malware can use to propagate?

- Infection of existing content by viruses that is then spread to other systems.  
- Exploit of software vulnerabilities by worms or drive-by-downloads to replicate malware 
- social engineering attacks that convince users to bypass security mechanisms to install trojans or to respond to phishing attacks.

##### S2.What are four broad categories of payloads that malware may carry?

- corruption of system or data files;  
- theft of service in order to make the system a zombie agent of attack as part of a botnet;  
- theft of information from the system, especially of logins, password or other personal details.
- stealthing where the malware hides it presence on the system from attempts to detect and block it.

##### Q1.Suppose you receive a letter from a finance company stating that your loan payments are in arrears, and that action is required to correct this. However, as far as you know, you have never applied for, or received, a loan from this company! 

##### What may have occurred that led to this loan being created? What type of malware, and on which computer systems, might have provided the necessary information to an attacker that enabled them to successfully obtain this loan?

Such a letter strongly suggest that **an attacker has called sufficient personal details** about you in order to satisfy the finance company that they are you for the purpose of establishing such a loan.

This was most likely done using either **a phishing attack**, perhaps persuading you to complete and return some forms with the needed personal details; Or by using **spyware installed on your personal coumputer system by a worm or trojan horse malware**, that then collected the neccessary details from files on the system, or by monitoring your access to sensitive sites, such as banking sites.

##### Q2.Assume you receive an e-mail, which appears to come from your bank, includes your bank logo in it, and with the following contents: “Dear Customer, Our records show that your Internet Banking access has been blocked due to too many login attempts with invalid information such as incorrect access number, password, or security number. We urge you to restore your account access immediately, and avoid permanent closure of your account, by clicking on this link to restore your account. Thank you from your customer service team.” 

##### What form of attack is this e-mail attempting? What is the most likely mechanism used to distribute this e-mail? How should you respond to such e-mails?

This email is attempting **a general phishing attack**, being sent to very large number of people, in the hope that a sufficient number both use the named bank, and are fooled into divulging their sensitive login credentials to the attacker.

The most likely mechanism is **via a botnet** using large number of compromised systems **to generate the neccessary high volumes of spam emails**.

You should **never ever follow such a link** in an email and supply the requested details. It should just be deleted.

## 2.Dos Attack

##### S3.What defenses are possible to prevent an organization's systems being used as internediaries in a broadcast amplification attack?

The best defense is to block the use of IP directed broadcasts. This can be done either by ISP, or by any organization whose system could potentially be used as an intermediary.

##### S4.What type of packets are commonly used for flooding attacks?

Virtually any type of network packet can be used in a flooding attack, through common flooding attacks use ICMP, UDM or TCP SYN packet types.

## 3.Intrusion Detection

##### S5.What are some motivations for using a distributed or hybrid IDS?

A distributed or hybrid IDS combines in a central IDS, **the complementary information sources** used by HIDS with host-based process and data details, and NIDS with network events and data, and in a central analyzer that is able to **better identify and respond to intrusion activity**.

##### S6.What is the difference between anomaly detection and signature or heuristic intrusion detection?
- Anomaly detection: 
  - Involves **the collection of data relating to the behaviour of legitimate users** over a period of time. Then current observed behavior is analyzed to determine whether this behaviour is that of a legitimate user or that of an intruder(Threshold detection, profile based).
- Signature/Heuristic detection: 
  - Uses **a set of known malicious data patterns (signatures) or attack rules (heuristics)** that are compared with current behavior to decide if it is of an intruder. 
  - Also known as **misuse detection**. **Only can identify known attacks** for which it has patterns or rules.


##### Q3.List and briefly describe the steps typically used by intruders when attacking a system.
The steps typically used by intruders when attacking a system are:

- **Target Acquisition and Information Gathering**: where the attacker **identifies and characterizes the target systems** using publicly available information, both technical and non-technical, and the use network exploration tools to **map target resources**;
- **Initial Access**: typically by exploiting a remote network vulnerability, by guessing weak authentication credentials used in a remote service, or **via the installation of malware on the system using some form of social engineering or drive-by-download attack**;
- **Privilege Escalation**: are actions taken on the system, typically **via a local access vulnerability to increase the privileges available to the attacker** to enable their desired goals on the target system;
- **Information Gathering or System Exploit**: are actions by the attacker to **access or modify information or resources on the system, or to navigate to another target system**;
- **Maintaining Access**: are actions such as **the installation of backdoors or other malicious software**, or the addition of covert authentication credentials or other configuration changes to the system, **to enable continued access by the attacker** after the initial attack.
- **Covering Tracks**: where the attacker **disables or edits audit logs to remove evidence of attack activity**, and uses rootkits.


##### Q4.List some desirable characteristics of an IDS.
- **Run continually with minimal human supervision.**
- **Be fault tolerent** in sense that it must be able to recover from system crashes and reinitializations.
- **Resist subversion**, the IDS must be able to monitor itself and detect if it has been modified by attachers.
- **Impose a minimal overhead** on the system where it is running.
- Be able to **be configured according to the security policies** of the system that is being monitored.
- Be able to **adapt to the changes** of the system and user behavior over time.
- Be able to **scale to monitor a large number of hosts**.
- **provide gracesful degradation of service** in the sense that if some componets of the IDS stop working for any reason, the rest of them should be affected as little as possible.
- **Allow dynamic reconfiguration**: that is, the ability to configure the IDS without having to restart it.


## 4. Firewall

##### S7.What is the difference between an internal and an external firewall?

- External: 
  - placed at the edge of a local or enterprise network, just inside the boundary router that connects to the Internet or some wide area netwok(WAN)
  - provides a measure of access control and protection for the DMZ systems consistent with their need for external connectivity.
- Internal: 
  - Has stricter filtering rules in order to protect enterprise servers from external attacks.
  - Multiple internel can be used to protect portions of the internal network from each other.

##### S8.List different types of firewall?

1. Packet filtering firewalls: **compare each packet received to a set of established criteria** before being either dropped or forwarded.
2. Circuit level gateways: **Monitors the TCP handshaking going on between the local and remote hosts** to determine whether the session being initiated is legitimate -- whether the remote system is considered "trusted."
3. Stateful inspection firewalls: **Examines each packet, and also keeps track** of whether or not that packet is part of an established TCP session.
4. Application level gateways:They filter packets **not only according to the service** for which they are intended (as specified by the destination port), **but also by certain other characteristics** such as HTTP request string. 

##### Q5.Table shows a sample of a packet filter firewall ruleset for an imaginary network of IP address that range from 192.168.1.0 to 192.168.1.254. Describe the effect of each rule.

Sample Packet Filter Firewall Ruleset

|      | Source Addr | Source Port |  Dest Addr  | Dest Port | Action |
| :--: | :---------: | :---------: | :---------: | :-------: | :----: |
|  1   |     Any     |     Any     | 192.168.1.0 |   >1023   | Allow  |
|  2   | 192.168.1.1 |     Any     |     Any     |    Any    |  Deny  |
|  3   |     Any     |     Any     | 192.168.1.1 |    Any    |  Deny  |
|  4   | 192.168.1.0 |     Any     |     Any     |    Any    | Allow  |
|  5   |     Any     |     Any     | 192.168.1.2 |   SMTP    | Allow  |
|  6   |     Any     |     Any     | 192.168.1.3 |   HTTP    | Allow  |
|  7   |     Any     |     Any     |     Any     |    Any    |  Deny  |

1. Allow return TCP Connections to internal subnet.
2. Prevent Firewall system itself from directly connecting to anything.
3. Prevent External users from directly accessing the Firewall system.
4. Internal Users can access External serves.
5. Allow External users to send email in.
6. Allow External users to access WWW server.
7. Everthing not previously allowed is explicity denied.

##### Q6.Consider the threat of “theft/breach of proprietary or confidential information held in key data files on the system.” One method by which such a breach might occur is the accidental/deliberate e-mailing of information to a user outside of the organization. A possible countermeasure to this is to require all external e-mail to be given a sensitivity tag (classification if you like) in its subject and for external e-mail to have the lowest sensitivity tag. Discuss how this measure could be implemented in a firewall and what components and architecture would be needed to do this.

 At its simplest a policy can just require user’s to always include such a tag in email messages. Alternatively with suitable email agent programs it may be possible to enforce the prompting for and inclusion of such a tag on message creation. Then, when external email is being relayed through the firewall, the mail relay server must check that the correct tag value is present in the Subject header, and refuse to forward the email outside the organization if not, and notify the user of its rejection.

## 5.Buffer Overflow

##### S9.What are the two key elements that must be identified in order to implement a buffer overflow?

- **To indentify a buffer overflow vulnerability in some program** that can be triggered using externally sourced data under the attacker's control;
- **To understand how the buffer is stored in memory** and hence the potential for corrupting adjacent memory locations and potentially altering the flow of execution of the program.

##### S10.What are the possible consequences of a buffer overflow occurring?

- corruption of data used by the program
- unexpected transfer of control in the program
- memory access violations
- eventual program termination

## 6.Software Security

##### S11.List some problems that may result from a program sending unvalidated input from one user to another user.

- the content is not adequately sanitized by the program, then **an attack from one user to another is possible** (like XSS). 
- Textual terminals used special character sequences to send messages or edit the text style which **made classic command injection attacks possible**. 
- different character sets allow different encodings of meta characters, which may **change the interpretation of what is valid output**. If the display program is unaware if the specific encoding, unexpected problems will occur.

##### S12.Describe the advantage and disadvantage of the fuzzing.

- Advantage:
  - Range of input is very large
  - Simple, free of assumptions, cheap 
  - Assists with reliability as well as security
- Disadvantages:
  - bugs triggered by other forms of input would be missed
  - Combination of approaches is needed for reasonably comprehensive coverage of the inputs.

##### Q7.Define an injection attack. List some examples of injection attacks. What are the general circumstances in which injection attacks are found? Then State the similarities and differences between command injection and SQL injection attacks.

An injection attack refers to **a wide variety of program flaws** related to invalid handling of input data, particularly when such input data can accidentally or deliberately **influence the flow of execution of the program**. 

Examples of injection attacks include: **command injection, SQL injection, code injection, and remote code injection**. There are a wide variety of mechanisms that can result in injection attacks. These include when **input data is passed as a parameter** to another helper program (command) or to a database system (SQL), whose output is then processed and used by the original program. Or when the **input includes either machine or script code** that is then executed/interpreted by the attacked system (code).

In both cases the unchecked input **allows the execution of arbitrary programs/SQL queries** rather than the program/query specified by the program designer. They differ in the **syntax of the respective shell/SQL metacharacters** used that allow this to occur.

##### Q8.Identify several concerns associated with the use of environment variables by shell scripts.

Environment variables are **a collection of string values** inherited by each process from its parent, that can affect the way a running process behaves. The operating system includes these in the processes memory when it is constructed. 

Well known environment variables include the variable PATH which specifie s the set of directories to search for any given command, IFS which specifies the word boundaries in a shell script, and LD_LIBRARY_PATH which specifies the list of directories to search for dynamically loadable libraries. 

**All of these have been used to attack programs, and especially privileged shell scripts.** The attacker changes the values of one or more of these, then calls a script running with other (higher) privileges, which is then “tricked” into running a program or loading a library of the attackers choice as a result.



## 7.Operating System Security

##### S13.What is the point of removing unnecessary services, applications, and protocols?

the point is to minimize the amount of software that can run, since if less software is available to run, then the risk that it may contain vulnerabilities is reduced.

##### S14.What type of access control model do Unix and Linux systems implement?

discretionary access controls to all file system resources, including not only files and dirctories, but devices, processes, memory and indeed most system resources.

##### Q9.  Some have argued that Unix/Linux systems reuse a small number of security features in many contexts across the system, while Windows systems provide a much larger number of more specifically targeted security features used in the appropriate contexts. This may be seen as a trade-off between simplicity and lack of flexibility in the Unix/Linux approach, against a better targeted but more complex and harder to correctly configure approach in Windows. Discuss this trade-off as it impacts on the security of these respective systems, and the load placed on administrators in managing their security.

Unix and Linux systems use **a small number of access rights** for subjects being owner/group/other across nearly all resources/objects – this means it is **a simple model**, but because **the same rights are used everywhere**, their meaning can differ for different objects, and sometimes **it is hard or impossible to specify** some desired complex requirements.

Windows systems use **a much larger set of access rights**, which differ for different types of objects – a **more complex model**, which can be **harder to master**, but which may **allow better specification** of some desired complex requirements. 

An increase in security features is desirable provided the system **administrator is competent** and completely understands the **purpose and use of each of the security features**.

If the administrator was not as familiar with all the complex security features then there would **be a benefit in having less security features available**. This is because having less to understand means less chance of making mistakes.

##### Q10.Suppose you operate an Apache-based Linux Web server that hosts your company’s e-commerce site. Suppose further that there is a worm called “WorminatorX,” which exploits a (fictional) buffer overflow bug in the Apache Web server package that can result in a remote root compromise. Construct a simple threat model that describes the risk this represents: attacker(s), attack-vector, vulnerability, assets, likelihood of occurrence, likely impact, and plausible mitigations.

**Attack-vector**: the worm “WorminatorX” (that multiple exploits may target the same vulnerability -- this is just the one we know about)

**Attackers**: competitors, thieves, identity thieves, website defacers(vandals), disgruntled ex-employees, the Byelorusan mob, etc. --public web sites can be prey to any Internet-connected type of attacker

**Vulnerability**: buffer-overflow in Apache 

**Assets**: website availability, system availability, local network integrity(integrity of other systems the attacker may reach via compromised web server), corporate data, customer data, ecommerce business activity (immediate revenue), company reputation(future revenue) 

**Likelihood of Occurrence**: High -- Internet worms spread very far very quickly

**Likely Impact**: High -- complete exposure/loss of any or all affected assets to any potential attacker

**Plausible Mitigations**: Patch the Apache vulnerability; if no patch is available, protect Apache with SELinux or AppArmor; or run Apache in a chroot jail (Note that Apache already runs as an unprivileged user, by default -- presumably the WorminatorX vulnerability depends on some other privilege-escalation vulnerability)

## 8.Trusted Computing and Multilevel Security

##### S15.Explain the differences among the terms security class, security level, security clearance, and security classification.

In most security models, each subject and each object is assigned a **security class**. 

In the simplest formulation, security classes form a strict hierarchy and are referred to as **security levels**.

A subject is said to have a **security clearance** of a given level; 

an object is said to have a **security classification** of a given level.

##### S16.What is the principal difference between the BLP model and the Biba model?

- The **BLP model** deals with confidentiality and is concerned with unauthorized disclosure of information.
- The **Biba models** deals with integrity and is concerned with the unauthorized modification of data.