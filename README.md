Phishing Analysis Report — Case 001
Analyst: Akinola selim ishola  Case ID: PHISH-001 Verdict: Confirmed Phishing

What is this email about?
This email is impersonating Banco Bradesco Livelo, which is a well-known bank in Brazil. The subject line says the victim has 92,990 loyalty points that are expiring today and pushes them to click a link to redeem them before it is too late.
That urgency is the trick. The attacker wants the victim to panic and click without thinking. It is a classic phishing tactic.
I obtained this sample from the public phishing_pot repository on GitHub for practice and portfolio purposes.

How I analysed it
I started by uploading the raw .eml file to PhishTool. This tool reads the entire email automatically and breaks it down  into sections like headers, sender details, authentication results, and any URLs hidden inside the body. It saved me a lot of time compared to reading the raw file manually.
Once PhishTool gave me the key findings, I took each indicator and verified it using additional tools. I checked the sender IP on AbuseIPDB and ran the phishing URL through VirusTotal.
Before writing anything down, I defanged the URL using CyberChef. Defanging means converting the URL into a safe format that cannot be accidentally clicked. This is standard practice in any SOC so that malicious links cannot do harm if a report gets forwarded or opened somewhere unexpected.

What PhishTool showed me
The first thing that stood out was the sending server. PhishTool identified the email as coming from a machine called ubuntu-s-1vcpu-1gb-35gb-intel-sfo3-06 with the IP address 137.184.34.4.
That server name is actually very revealing. It is the default name that DigitalOcean, which is a  cloud hosting company, gives to a basic rented virtual machine. The sfo3 part means it is sitting in DigitalOcean's San Francisco datacenter.
Email authentication
Next I looked at the authentication results. There are three standard checks that every email goes through SPF, DKIM, and DMARC. Think of them as three different ways of verifying whether the person sending the email is who they claim to be.
All three failed on this email:
SPF came back as TempError. This means the system could not even look up whether the sending server was authorised because the sender's DNS timed out. That kind of evasion is not accidental.
DKIM was completely absent. The email had no digital signature at all, which means there was nothing to verify the sender's identity cryptographically.
DMARC also came back as TempError, which follows from the SPF failure.
When a real company like Bradesco sends an email, all three of these pass cleanly. Seeing all three fail at once is a strong signal that someone is pretending to be them.
The hidden phishing link
The body of the email was encoded in Base64 essentially scrambled text that hides the content from a quick glance. PhishTool decoded it automatically and found the link the victim is supposed to click:
Defanged URL: hXXps[://]blog1seguimentmydomaine2bra[.]me/
That domain name tells its own story. It has random words jumbled together, uses a .me extension which is commonly abused by phishers, and has absolutely no connection to Bradesco. A legitimate bank uses their own official domain.

What my other tools showed
AbuseIPDB — Sender IP check
I checked the sender IP 137.184.34.4 on AbuseIPDB, which is a community database where people report IPs involved in malicious activity. The IP was found in the database with a history of reported abuse. This confirmed that the infrastructure being used here has been used for bad activity before.
VirusTotal — URL check
I submitted the phishing URL to VirusTotal, which checks it against 95 different security engines at once. The result came back clean — zero detections.
This might sound like good news, but it is not something that clears the case. Phishing attackers deliberately register brand new domains that have no history anywhere. They know that VirusTotal and similar tools only flag things that have already been reported. A fresh domain used in a new campaign will almost always look clean at first. This is sometimes called a zero-day phishing domain.
In this case, the clean VirusTotal result simply means the domain was new. All the other evidence still points clearly to phishing.

Indicators of Compromise (IOCs)
These are the specific pieces of evidence I extracted from this email. All URLs are defanged.
Type
Value
What it tells us
Sender IP
137.184.34.4
Found on AbuseIPDB — prior abuse history
Sending server
ubuntu-s-1vcpu-1gb-35gb-intel-sfo3-06
Cheap rented VPS — not a bank server
From address
banco.bradesco@atendimento.com.br
Spoofed — not a Bradesco domain
Return-Path
root@ubuntu-s-1vcpu-1gb-35gb-intel-sfo3-06
Points back to attacker server
Phishing URL
hXXps[://]blog1seguimentmydomaine2bra[.]me/
Fake redemption page
Domain
blog1seguimentmydomaine2bra[.]me
Lookalike domain — no connection to Bradesco

Email checksum (recorded by Microsoft on arrival)
Original: 3B61F64750F88C5569DF38A496B2374685F23D8BC662A6A19B6823B2F6745D54

My verdict
This is confirmed phishing. The URL being clean on VirusTotal is the only thing that does not point to malicious activity, and that is easily explained by the domain being newly registered.
Everything else lines up: the email came from a rented cloud server, it failed every authentication check, the sender domain has nothing to do with Bradesco, the sending IP has a prior abuse history, and the phishing domain is a jumbled lookalike with no legitimate purpose.
If this arrived in a real company's inbox, my recommended actions would be:
Block the sender IP 137.184.34.4 at the email gateway immediately
Block the domain blog1seguimentmydomaine2bra[.]me at the DNS level
Search the SIEM for other employees who may have received the same email
Check proxy and firewall logs to see if anyone clicked the link
If anyone did click — escalate to the incident response team straight away
Submit the domain to VirusTotal and PhishTank so other analysts benefit
MITRE ATT&CK mapping: T1566.001 — Phishing: Spearphishing Link

Tools I used
Tool
What I used it for
PhishTool
Primary analysis — headers, authentication, URL extraction
CyberChef
Defanging the URL before writing it in this report
VirusTotal
Checking URL reputation across 87 security engines
AbuseIPDB
Checking the sender IP reputation


This analysis was done in a safe environment. No phishing links were clicked directly. All URLs are defanged. Sample sourced from phishing_pot on GitHub for educational and portfolio purposes.
AKINOLA SELIM ISHOLA SOC Analyst Portfolio 

