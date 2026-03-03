
## Types of Penetration Testing

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

## Processes

##### Processes: 
A term for the directed sequence of events. In operational and organizational context are referred to as work, business, production or value creation processes. Processes are another name for programs running in a computer system.
##### Deterministic: 
A deterministic process is one which each state  is casually dependent on and determined by other previous states of events.
##### Stochastic: 
A stochastic process is one in which one state follows from other states only with a certain probability.

**Relation:** A penetration testing process is defined by successive steps and events performed by the penetration tester to find a path to the predefined objective.

## Steps of Engagement

##### 1: Pre-Engagement 
The first step is to create all the necessary documents in the pre-engagement phase, discuss the assessment objectives, and clarify any questions.
##### 2: Information Gathering 
Once the pre-engagement activities are complete, we investigate the company's existing website we have been assigned to assess. We identify the technologies in use and learn how the web application functions.
##### 5: Vulnerability Assessment
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


## Pre-Engagement
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

