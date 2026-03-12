
# Types of Penetration Testing

##### Blackbox: 
Only the essential information is provided such as IP addresses and domains.
##### Greybox: 
Additional information is provided such as specific URL's, hostnames, subnets, etc.
##### Whitebox: 
Everything is provided here by the client from the entire network structure to sometimes specific configurations, credentials, source code for web apps, etc.
##### Red-Teaming:
May include physical testing and social engineering, among other things. Can be combined with any of the above types.
##### Purple-Teaming: 
 It can be combined with any of the above types. However, it focuses on working closely with the defenders.

# Processes

##### Processes: 
A term for the directed sequence of events. In operational and organizational context are referred to as work, business, production or value creation processes. Processes are another name for programs running in a computer system.
##### Deterministic: 
A deterministic process is one which each state  is casually dependent on and determined by other previous states of events.
##### Stochastic: 
A stochastic process is one in which one state follows from other states only with a certain probability.

**Relation:** A penetration testing process is defined by successive steps and events performed by the penetration tester to find a path to the predefined objective.

# Steps of Engagement

##### 1: Pre-Engagement 
The first step is to create all the necessary documents in the pre-engagement phase, discuss the assessment objectives, and clarify any questions.
##### 2: Information Gathering 
Once the pre-engagement activities are complete, we investigate the company's existing website we have been assigned to assess. We identify the technologies in use and learn how the web application functions.
##### 3: Vulnerability Assessment
With this information, we can look for known vulnerabilities and investigate questionable features that may allow for unintended actions.
##### 4: Exploitation
Once we have found potential vulnerabilities, we prepare our exploit code, tools, and environment and test the webserver for these potential vulnerabilities.
##### 5: Post-Exploitation
Once we have successfully exploited the target, we jump into information gathering and examine the webserver from the inside. If we find sensitive information during this stage, we try to escalate our privileges (depending on the system and configurations).
##### 6: Lateral Movement
If other servers and hosts in the internal network are in scope, we then try to move through the network and access other hosts and servers using the information we have gathered.
##### 7: Proof-of-Concept
We create a proof-of-concept that proves that these vulnerabilities exist and potentially even automate the individual steps that trigger these vulnerabilities.
##### 8: Post-Engagement
Finally, the documentation is completed and presented to our client as a formal report deliverable. Afterward, we may hold a report walkthrough meeting to clarify anything about our testing or results and provide any needed support to personnel tasked with remediating our findings.


# Pre-Engagement
### Documents to be Prepared
| **Document**                                                         | **Timing for Creation**                             |
| -------------------------------------------------------------------- | --------------------------------------------------- |
| `1. Non-Disclosure Agreement` (`NDA`)                                | `After` Initial Contact                             |
| `2. Scoping Questionnaire`                                           | `Before` the Pre-Engagement Meeting                 |
| `3. Scoping Document`                                                | `During` the Pre-Engagement Meeting                 |
| `4. Penetration Testing Proposal` (`Contract/Scope of Work` (`SoW`)) | `During` the Pre-engagement Meeting                 |
| `5. Rules of Engagement` (`RoE`)                                     | `Before` the Kick-Off Meeting                       |
| `6. Contractors Agreement` (Physical Assessments)                    | `Before` the Kick-Off Meeting                       |
| `7. Reports`                                                         | `During` and `after` the conducted Penetration Test |
### Types of NDA's (Non-Disclosure Agreement)

##### Unilateral NDA
This type of NDA obligates only one party to maintain confidentiality and allows the other party to share the information received with third parties.
##### Bilateral NDA
In this type, both parties are obligated to keep the resulting and acquired information confidential. This is the most common type of NDA that protects the work of penetration testers.
##### Multilateral NDA
Multilateral NDA is a commitment to confidentiality by more than two parties. If we conduct a penetration test for a cooperative network, all parties responsible and involved must sign this document.
### Scoping Questionnaire
After the initial contact with the client we send a scoping questionnaire to get a full understanding of the services they are seeking. They will be given a list of our services and they will check off what they want from a list. Typically looking like this:
```
☐ Internal Vulnerability Assessment
☐ External Vulnerability Assessment|
☐ Internal Penetration Test
☐ External Penetration Test
☐ Wireless Security Assessment
☐ Application Security Assessment
☐ Physical Security Assessment
☐ Social Engineering Assessment
☐ Red Team Assessment
☐ Web Application Security Assessment
```
With options for the clients to elaborate further if needed. There will also be other important information we should know about what we can and can't touch. What is in and out of scope and whether they would like us to test evasive or non-evasive methods etc. 
### Pre-Engagement Meeting
After receiving the scoping questionnaire back from the client we will then organize a pre-engagement meeting. Here is our opportunity to cover contract details in depth to make sure both us and the client are clear on the operation.
##### Contract Checklist
| **Checkpoint**                       | **Description**                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| ------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `☐ NDA`                              | Non-Disclosure Agreement (NDA) refers to a secrecy contract between the client and the contractor regarding all written or verbal information concerning an order/project. The contractor agrees to treat all confidential information brought to its attention as strictly confidential, even after the order/project is completed. Furthermore, any exceptions to confidentiality, the transferability of rights and obligations, and contractual penalties shall be stipulated in the agreement. The NDA should be signed before the kick-off meeting or at the latest during the meeting before any information is discussed in detail. |
| `☐ Goals`                            | Goals are milestones that must be achieved during the order/project. In this process, goal setting is started with the significant goals and continued with fine-grained and small ones.                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| `☐ Scope`                            | The individual components to be tested are discussed and defined. These may include domains, IP ranges, individual hosts, specific accounts, security systems, etc. Our customers may expect us to find out one or the other point by ourselves. However, the legal basis for testing the individual components has the highest priority here.                                                                                                                                                                                                                                                                                              |
| `☐ Penetration Testing Type`         | When choosing the type of penetration test, we present the individual options and explain the advantages and disadvantages. Since we already know the goals and scope of our customers, we can and should also make a recommendation on what we advise and justify our recommendation accordingly. Which type is used in the end is the client's decision.                                                                                                                                                                                                                                                                                  |
| `☐ Methodologies`                    | Examples: OSSTMM, OWASP, automated and manual unauthenticated analysis of the internal and external network components, vulnerability assessments of network components and web applications, vulnerability threat vectorization, verification and exploitation, and exploit development to facilitate evasion techniques.                                                                                                                                                                                                                                                                                                                  |
| `☐ Penetration Testing Locations`    | External: Remote (via secure VPN) and/or Internal: Internal or Remote (via secure VPN)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| `☐ Time Estimation`                  | For the time estimation, we need the start and end dates for the penetration test. This provides a precise time window to perform the test and helps us plan our procedure. It is also vital to explicitly determine the duration of the time windows for each phase of the attack, such as Exploitation, Post-Exploitation, and Lateral Movement. These can be carried out during or outside regular working hours. When testing outside regular working hours, the focus is more on the security solutions and systems that should withstand our attacks.                                                                                 |
| `☐ Third Parties`                    | For the third parties, it must be determined via which third-party providers our customer obtains services. These can be cloud providers, ISPs, and other hosting providers. Our client must obtain written consent from these providers describing that they agree and are aware that certain parts of their service will be subject to a simulated hacking attack. It is also highly advisable to require the contractor to forward the third-party permission sent to us so that we have actual confirmation that this permission has indeed been obtained.                                                                              |
| `☐ Evasive Testing`                  | Evasive testing is the test of evading and passing security traffic and security systems in the customer's infrastructure. We look for techniques that allow us to find out information about the internal components and attack them. It depends on whether our contractor wants us to use such techniques or not.                                                                                                                                                                                                                                                                                                                         |
| `☐ Risks`                            | We must also inform our client about the risks involved in the tests and the possible consequences. Based on the risks and their potential severity, we can then set the limitations together and take certain precautions.                                                                                                                                                                                                                                                                                                                                                                                                                 |
| `☐ Scope Limitations & Restrictions` | It is also essential to determine which servers, workstations, or other network components are essential for the client's proper functioning and its customers. We will have to avoid these and must not influence them any further, as this could lead to critical technical errors that could also affect our client's customers in production.                                                                                                                                                                                                                                                                                           |
| `☐ Information Handling`             | HIPAA, PCI, HITRUST, FISMA/NIST, etc.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| `☐ Contact Information`              | For the contact information, we need to create a list of each person's name, title, job title, e-mail address, phone number, office phone number, and an escalation priority order.                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| `☐ Lines of Communication`           | It should also be documented which communication channels are used to exchange information between the customer and us. This may involve e-mail correspondence, telephone calls, or personal meetings.                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| `☐ Reporting`                        | Apart from the report's structure, any customer-specific requirements the report should contain are also discussed. In addition, we clarify how the reporting is to take place and whether a presentation of the results is desired.                                                                                                                                                                                                                                                                                                                                                                                                        |
| `☐ Payment Terms`                    | Finally, prices and the terms of payment are explained.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
##### Rules of Engagement Checklist
| **Checkpoint**                              | **Contents**                                                                                          |
| ------------------------------------------- | ----------------------------------------------------------------------------------------------------- |
| `☐ Introduction`                            | Description of this document.                                                                         |
| `☐ Contractor`                              | Company name, contractor full name, job title.                                                        |
| `☐ Penetration Testers`                     | Company name, pentesters full name.                                                                   |
| `☐ Contact Information`                     | Mailing addresses, e-mail addresses, and phone numbers of all client parties and penetration testers. |
| `☐ Purpose`                                 | Description of the purpose for the conducted penetration test.                                        |
| `☐ Goals`                                   | Description of the goals that should be achieved with the penetration test.                           |
| `☐ Scope`                                   | All IPs, domain names, URLs, or CIDR ranges.                                                          |
| `☐ Lines of Communication`                  | Online conferences or phone calls or face-to-face meetings, or via e-mail.                            |
| `☐ Time Estimation`                         | Start and end dates.                                                                                  |
| `☐ Time of the Day to Test`                 | Times of the day to test.                                                                             |
| `☐ Penetration Testing Type`                | External/Internal Penetration Test/Vulnerability Assessments/Social Engineering.                      |
| `☐ Penetration Testing Locations`           | Description of how the connection to the client network is established.                               |
| `☐ Methodologies`                           | OSSTMM, PTES, OWASP, and others.                                                                      |
| `☐ Objectives / Flags`                      | Users, specific files, specific information, and others.                                              |
| `☐ Evidence Handling`                       | Encryption, secure protocols                                                                          |
| `☐ System Backups`                          | Configuration files, databases, and others.                                                           |
| `☐ Information Handling`                    | Strong data encryption                                                                                |
| `☐ Incident Handling and Reporting`         | Cases for contact, pentest interruptions, type of reports                                             |
| `☐ Status Meetings`                         | Frequency of meetings, dates, times, included parties                                                 |
| `☐ Reporting`                               | Type, target readers, focus                                                                           |
| `☐ Retesting`                               | Start and end dates                                                                                   |
| `☐ Disclaimers and Limitation of Liability` | System damage, data loss                                                                              |
| `☐ Permission to Test`                      | Signed contract, contractors agreement                                                                |
### Kick-Off Meeting
This meeting will usually take place in-person and after signing all necessary documents.
The meeting will usually include all points of contact personnel:
- Internal Audit
- Information Security
- IT
- Governance & Risk
- Developers
- sysadmins
- Network Engineers
- Practical Leads
- Sales Account Executives

We also inform the clients that if any of the below are found the penetration test will pause immediately:
- A critical vulnerability is documented and needs emergency patch
- If we find evidence of illegal activity
- Evidence of a prior breach that went unnoticed
- Presence of a threat actor in the network

We must also inform customers and clients of the following:
- The test may leave log entries and alarms
- We may lock some users out accidently with brute force attacks
- They should inform us immediately if the test has negative impacts on their network.
### Contractors Agreement
Red teams typically will also be doing physical testing. For this scenario additional agreement is required since it isn't only virtual but physical intrusion. It is possible that many of the employees will be unaware of the test so having a contractors agreement it is essentially our "Get out of jail free" card.
##### Contractor Agreement - Checklist for Physical Assessment
| **Checkpoint**                    |
| --------------------------------- |
| `☐ Introduction`                  |
| `☐ Contractor`                    |
| `☐ Purpose`                       |
| `☐ Goal`                          |
| `☐ Penetration Testers`           |
| `☐ Contact Information`           |
| `☐ Physical Addresses`            |
| `☐ Building Name`                 |
| `☐ Floors`                        |
| `☐ Physical Room Identifications` |
| `☐ Physical Components`           |
| `☐ Timeline`                      |
| `☐ Notarization`                  |
| `☐ Permission to Test`            |
### Setting Up
After you have made your way through all of the prior points and we have the information we need, we plan the approach. Here we will set up our VM's, VPS and any other tooling or systems we will need to perform the test.
# Information Gathering
This is the phase of the engagement that is the cornerstone of any penetration test. We enumerate as much information as we can about our targets. This information might be relevant to us in many different ways and is pbroken down into 4 categories:
- Open-Source Intelligence (OSINT)
- Infrastructure Enumeration
- Service Enumeration
- Host Enumeration

##### OSINT
Using publicly available information to gather more information about your target. Key things can be found like hashes, passwords, keys or tokens.
Great places to look:
- Misconfigured GitHub Repositories
- Developers using StackOverflow sometimes share too much
##### Infrastructure Enumeration
This is where we try to build a model of what the company infrastructure looks like. Using services like DNS to create a map of servers and hosts. Finding what security measures are in place so we can avoid them. After this process we compare it with our SOW document to make sure we aren't overstepping.
##### Service Enumeration
Here we are identifying what services we can use that will allow us to interact with the network. As well as other important information such as the service version.
##### Host Enumeration
Now you have a detailed list of the infrastructure we can look at individual hosts listed in our scoping document. Looking at things like what services are running, OS in use, what version are the services using, etc.

After exploitation we can then perform `internal` host enumeration. This is typically where we look for sensitive `files` local `services`, `scripts`, `applications`.
# Vulnerability Assessment
Here we analyze all the information we have gathered to see if we can get a foothold.

| **Analysis Type** | **Description**                                                                                                                                                                                                                                                                                                                                                         |
| ----------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Descriptive       | Descriptive analysis is essential in any data analysis. On the one hand, it describes a data set based on individual characteristics. It helps to detect possible errors in data collection or outliers in the data set.                                                                                                                                                |
| Diagnostic        | Diagnostic analysis clarifies conditions' causes, effects, and interactions. Doing so provides insights that are obtained through correlations and interpretation. We must take a backward-looking view, similar to descriptive analysis, with the subtle difference that we try to find reasons for events and developments.                                           |
| Predictive        | By evaluating historical and current data, predictive analysis creates a predictive model for future probabilities. Based on the results of descriptive and diagnostic analyses, this method of data analysis makes it possible to identify trends, detect deviations from expected values at an early stage, and predict future occurrences as accurately as possible. |
| Prescriptive      | Prescriptive analytics aims to narrow down what actions to take to eliminate or prevent a future problem or trigger a specific activity or process.                                                                                                                                                                                                                     |
We look for open ports and if any known attack vectors.

### Vulnerability Research and Assessment
This is where we cross reference the information we have gathered for any already known vulnerabilities using [CVE](https://www.cve.org/ResourcesSupport/FAQs), as it is likely this this vulnerability is still present.
Other potential sources:
- [CVEdetails](https://www.cvedetails.com/)
- [Exploit DB](https://www.exploit-db.com/)
- [Vulners](https://vulners.com/)
- [Packet Storm Security](https://packetstormsecurity.com/)
- [NIST](https://nvd.nist.gov/vuln/search?execution=e2s1)
Diagnostic Analysis and Predictive analysis is used here. Here we must understand the functionality of our Proof-Of-Concept Documentation.

### Assessment of Possible Attack Vectors
This includes actual testing and predictive analysis. Cross referencing historical information and combining it with information we have gathered from previous stages of the process.

### The Return
If we are unable to identify  vulnerability, we then double back and repeat the process looking for a potential exploit that will work. Rinse and repeat.

# Exploitation
Here we actually use the identified vulnerabilities and adapt them to our specific case.

### Prioritizing Possible Attacks
After we have potential attack vectors of our target system we need to then prioritize which comes first. This can depend on the following factors:
- Probability of Success
- Complexity
- Probability of Damage

It helps to have a personal point system you can use to break this down.
Example:
![[Screenshot 2026-03-04 150707.png]]
### Preparation for the Attack
Sometimes in can be hard to find quality, known PoC exploit code. In this case we will need to set up a VM that more closely resembles the target machine and see if we can modify the attack to work in our use case.

**Important:** When in doubt about an attack. It is always best to check with the client and give them the necessary data to make an informed decision.

Once we have successfully exploited the target we need make sure our notes are clear and we have clear logs for replication to show the client.

# Post-Exploitation
Here we must again consider whether or not to use Evasive Testing. Also in this phase we are looking to obtain sensitive and security relevant information from a local perspective as well as business relevant information that usually requires higher privileges. Components of this phase include:
- Evasive Testing
- Pillaging
- Privilege Escalation
- Data Exfiltration
- Information Gathering
- Vulnerability Assessment
- Persistence
### Evasive Testing
If there is a skilled administrator monitoring the systems, any change, or even a single command could trigger an alarm such as `net user` or `whoami` are usually flagged by EDR systems.
Evasion testing can be broken down into 3 categories:
- Evasive
- Hybrid Evasive
- Non-Evasive
### Information Gathering
Here again we are looking for valuable info that could help us gain a better understanding of the system and internal architecture. Now we have a new perspective on the engagement which can lead to more possibilities to access information that is relevant to us.

### Pillaging
This is where we examine the host machines role in the corporate network. We analyze the network configurations, that could include
- Interfaces
- Routing
- DNS
- ARP
- Services
- VPN
- IP Subnets
- Shares
- Network Traffic

Here we will also hunt for any sensitive data such as
- Passwords on shares
- Local machines
- In scripts
- Configuration files
- Password vaults
- Documents (Excel, Word, .txt,)
- Emails
### Persistence
This is the phase of looking for a way to keep yourself in the system even if you were disconnected or removed. It also avoids having to attack a system multiple times or let you pick up where you left off between sessions.

### Vulnerability Assessment
Performing the same vulnerability assessment just with new information from a different perspective. 

### Privilege Escalation
Here we are trying to gain the highest level of access we possibly can. On Linux machines that will be `root` level access. For windows that will be `administrator`/`local administrator`/`SYSTEM`. Sometimes we will not always be able to do it on the initial host and we might need to make lateral movement to get to another users machine.

### Data Exfiltration
Here the client may ask us to test their security systems and try to move data from that machine to our own.

# Lateral Movement
Here we want to test how far we can move manually in the entire network and see what vulnerabilities we can find from the internal perspective that might be exploited. This has several phases.
- Pivoting
- Evasive Testing
- Information Gathering
- Vulnerability Assessment
- (Privilege) Exploitation
- Post-Exploitation

### Pivoting
Some systems will not be visible publicly facing, so we may need to use the exploited machine as proxy to perform scans on the internal network. This process is called `Pivoting` or `Tunneling`.

### Evasive Testing
Here there may be new methods in place to prevent lateral movement. `Segmentation`, `Threat monitoring`, `IPS/IDS`, `EDR`, etc.
To bypass these we need to understand how they work and what they respond to.

### Information Gathering
We return to Info Gathering but this time from inside the network. First we get an overview of reachable systems, then enumerate each host individually. Some of this may already be available from the Post-Exploitation stage.

### Vulnerability Assessment
Internal VA differs from external  -  more errors occur inside a network. Group assignments and user rights play a key role. Compromising a developer account for example could grant access to most dev resources, providing crucial internal info and potential attack paths.

### (Privilege) Exploitation
Use discovered paths to access other systems. Common methods include cracking passwords/hashes, reusing credentials across systems, or using hashes directly via `pass-the-hash`. Tools like `Responder` can intercept `NTLMv2` hashes from administrators for use in pass-the-hash attacks.

### Post-Exploitation
Repeat the Post-Exploitation stage for each newly accessed system  -  collecting system info, user data, and business information as evidence. Always consider data handling rules defined in the contract. From here we move into the Proof-of-Concept phase.

# Proof-of-Concept
A PoC proves that a discovered vulnerability is real and exploitable. It serves as the decision-making basis for remediation and helps administrators and developers validate, reproduce, and test fixes for security issues.

### PoC Representations
A PoC can take multiple forms:
- **Documentation**  -  a detailed write-up of the vulnerability and how it was exploited
- **Script/Code**  -  automated exploitation that clearly demonstrates each step, making it easy for admins and devs to reproduce and understand the impact

### The Script Trap
A common pitfall  -  admins will sometimes "fight the script" by patching only what breaks it, rather than addressing the underlying vulnerability. A script is just one path to exploitation. Make sure this is explicitly discussed: fixing the script doesn't fix the problem.

### The Report
The report should paint the full picture  -  not just individual findings but how they chain together. An attack chain walkthrough is especially valuable in internal assessments. Fixing one link in the chain is good, but the remaining flaws still exist and may provide another path.
Always focus remediation advice on the **root cause**, not the symptom. Example: a Domain Admin using `Password123` is a symptom  -  the root cause is a weak password policy.

# Post-Engagement
The final phase  -  everything that must happen after active testing ends. Contractually binding in most cases.

### Cleanup
Remove all tools, scripts, and artifacts uploaded during testing. Revert any configuration changes made. Document everything  -  if something can't be cleaned up, flag it in the report appendices.

### Documentation and Reporting
Before disconnecting or sending "stop" notifications, ensure all findings are fully documented  -  command output, screenshots, affected hosts. Do not retain any PII or sensitive client data beyond what's needed for the report.

Report deliverables should include:
- Attack chain (if full compromise was achieved)
- Executive summary (non-technical audience)
- Detailed findings with risk rating, impact, remediation advice, and references
- Steps to reproduce each finding
- Near, medium, and long-term recommendations
- Appendices: scope, OSINT data, password analysis, ports/services, compromised hosts/accounts, file transfers, AD analysis, scan data

The first version sent to the client is marked **DRAFT**.

### Report Review Meeting
Walk the client through each finding  -  not word for word, but a clear explanation from your perspective. The client will ask questions and flag anything needing correction. Low-risk findings often get skipped. Be prepared to clarify and defend your findings.

### Deliverable Acceptance
Once the client reviews the DRAFT and provides feedback, issue a final version marked **FINAL**. Some audit firms will not accept a DRAFT-marked report, so keeping a consistent DRAFT → FINAL workflow is best practice.

### Post-Remediation Testing
Retest each remediated finding and issue a post-remediation report showing the before/after state. For each finding show evidence it is resolved  -  or that it is not. A simple status table works well:

| # | Severity | Finding | Status |
|---|----------|---------|--------|
| 1 | High | SQL Injection | Remediated |
| 2 | Medium | SMB Signing Not Enabled | Not Remediated |

### Role of the Pentester in Remediation
We are impartial third parties  -  we do **not** fix things ourselves. We can give general remediation advice (e.g., "sanitize user input") but not rewrite code or make configuration changes. This preserves the integrity of the assessment and avoids conflicts of interest.

### Data Retention
Retain client data (scan results, logs, credentials, screenshots) securely and encrypted for a reasonable period after the engagement in case questions arise. Wipe all data from tester systems at conclusion. Use a fresh VM for any post-remediation work. Refer to the SoW and RoE for specific retention requirements.

### Close Out
Wipe or destroy all systems used during the engagement. Store any retained artifacts encrypted per firm policy. Invoice the client. Send a post-assessment satisfaction survey  -  what went well and what can improve. This is also where follow-on work discussions often begin.


