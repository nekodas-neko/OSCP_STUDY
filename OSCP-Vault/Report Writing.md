---
tags:
  - report-writing
---

# Report Writing

> [!warning] OSCP Exam Report — Key Requirements
> - Submit within **24 hours** of exam end
> - Must include: proof.txt contents + screenshot showing `hostname` / `whoami` + IP
> - Each flag needs: IP, proof.txt value, screenshot
> - Format: PDF, named `OSCP-OS-XXXXX-Exam-Report.pdf`

## Exam Flag Checklist

```
For each machine:
□ Screenshotted: whoami/hostname + ipconfig/ifconfig + proof.txt cat
□ Noted: IP address, machine name, OS
□ Documented: full attack chain (how you got in, how you escalated)
□ Saved: all commands run
```

## Resources
- [OffSec Exam Guide](https://help.offsec.com/hc/en-us/articles/360040165632)
- [OSCP Report Template (WhoisFrederik)](https://github.com/whoisflkr/OSCP-Human-Friendly-Vulnerability-Report)
- [OSCP Report Template (noraj)](https://github.com/noraj/OSCP-Exam-Report-Template-Markdown)


Application Name: This is important in a multi-application test, and a good habit to get into. The application names also lend itself to building a natural folder and file structure quite nicely.

URL: This is the exact URL that would be used to locate the vulnerability that we've detected.

Request Type: This represents both the type of request (i.e.: GET, POST, OPTIONS, etc.) that was made, as well as any manual changes we made to it. For example, we might intercept a POST request message and change the username or password before forwarding it on.

Issue Detail: This is the overview of the vulnerability that will be triggered by our actions. For example, we may point to a CVE describing the vulnerability if one exists, and/or explain the impact we observe. We may categorize the impact as denial of service, remote code execution, privilege escalation, and so on.

Proof of Concept Payload: This is a string or code block that will trigger the vulnerability. This is the most important part of the note, as it is what will drive the issue home and allow it to be replicated. It should list all of the necessary preconditions and provide the exact code or commands that would need to be used to trigger the vulnerability again.


Example: The target we tested has a web page aptly named XSSBlog.html. When we navigate to it, we can enter a blog entry.

> [!note]- Screenshot
> ```
> ; Blogging
> Figure 1: XSS Testing
> ```


> [!note]- Screenshot
> ```
> When we read back the blog entry, we get the following alert:
> a
> XSS Blogging
> oy ae tar tha no ster as acpi te commence of an enters whch yo av garded ith ich en
> Figure 2: XSS Testing Issue
> ```


## While making these requests, we keep a record of our actions, as shown below.


```bash
Testing for Cross-Site Scripting 

Testing Target: 192.168.1.52 
Application:    XSSBlog
Date Started:   31 March 2022

1.  Navigated to the application
    http://192.168.1.52/XSSBlog.html
    Result: Blog page displayed as expected
    
2.  Entered our standard XSS test data: 
    You will rejoice to hear that no disaster has accompanied the
    commencement of an enterprise which you have regarded with such
    evil forebodings.<script>alert("Your computer is infected!");</script> 
    I arrived here yesterday, and my first task is to assure my dear
    sister of my welfare and increasing confidence in the success of
    my undertaking. 

3.  Clicked Submit to post the blog entry.
    Result: Blog entry appeared to save correctly.

4.  Navigated to read the blog post
    http://192.168.1.52/XSSRead.php
    Result: The blog started to display and then the expected alert popped up.

5.  Test indicated the site is vulnerable to XSS.

PoC payload: <script>alert(‘Your computer is infected!')</script>
```


## General Structure:


## Executive Summary:

This enables senior management to understand the scope and outcomes of the testing at a sufficient level to understand the value of the test, and to approve remediation

Time Frame of the test:
This includes the length of time spent on testing, the dates, and potentially the testing hours as well.

Refer to the Rules of Engagement and reference the referee report if a referee was part of the testing team:
 If denial of service testing was allowed, or social engineering was encouraged, that should be noted here. If we followed a specific testing methodology, we should also indicate that here.
 
 Core:
  include supporting infrastructure and accounts. Using the example of a web application, if we were given user accounts by the client, include them here along with the IP addresses that the attacks came from (i.e. our testing machines). We should also note any accounts that we created so the client can confirm they have been removed. The following is an example of this high-level structure:

## Executive Summary (short Form):


```bash
- Scope: https://kali.org/login.php
- Timeframe: Jan 3 - 5, 2022
- OWASP/PCI Testing methodology was used
- Social engineering and DoS testing were not in scope
- No testing accounts were given; testing was black box from an external IP address
- All tests were run from 192.168.1.2
```


## The Executive Summary can generally be broken down as follows:


```bash
- "The Client hired OffSec to conduct a penetration test of
their kali.org web application in October of 2025. The test was conducted
from a remote IP between the hours of 9 AM and 5 PM, with no users
provided by the Client."
```


## Next, we introduce a discussion of the vulnerabilities discovered:


```bash
- "However, there were still areas of concern within the application.
OffSec was able to inject arbitrary JavaScript into the browser of
an unwitting victim that would then be run in the context of that
victim. In conjunction with the username enumeration on the login
field, there seems to be a trend of unsanitized user input compounded
by verbose error messages being returned to the user. This can lead
to some impactful issues, such as password or session stealing. It is
recommended that all input and error messages that are returned to the
user be sanitized and made generic to prevent this class of issue from
cropping up."
```


## Executive Summary should conclude with an engagement wrap-up:


```bash
"These vulnerabilities and their remediations are described in more
detail below. Should any questions arise, OffSec is happy
to provide further advice and remediation help."
```


## Testing Environment Considerations:

The first section of the full report should detail any issues that affected the testing. This is usually a small section. At times, there are mistakes or extenuating circumstances that occur during an engagement. While those directly involved will already be aware of them, we should document them in the report to demonstrate that we've been transparent.

```bash
We'll consider three potential states regarding extenuating circumstances:

Positive Outcome: 
"There were no limitations or extenuating circumstances in the engagement. 
The time allocated was sufficient to thoroughly test the environment."

Neutral Outcome: 
"There were no credentials allocated to the tester in the first two days of the test. 
However, the attack surface was much smaller than anticipated. 
Therefore, this did not have an impact on the overall test. 
OffSec recommends that communication of credentials occurs immediately before the engagement begins for future contracts, so that we can provide as much testing as possible within the allotted time."

Negative Outcome: 
"There was not enough time allocated to this engagement to conduct a thorough review of the application, and the scope became much larger than expected. 
It is recommended that more time is allocated to future engagements to provide more comprehensive coverage."
```


## Technical Summary:

The next section should be a list of all the key findings in the report, written out with a summary and recommendation for a technical person, like a security architect, to learn at a glance what needs to be done.

```bash
This section should group findings into common areas. For example, all weak account password issues that have been identified would be grouped, regardless of the testing timeline. An example of the structure of this section might be:

User and Privilege Management
Architecture
Authorization
Patch Management
Integrity and Signatures
Authentication
Access Control
Audit, Log Management and Monitoring
Traffic and Data Encryption
Security Misconfigurations
An example of a technical summary for Patch Management is as follows:

4. Patch Management

Windows and Ubuntu operating systems that are not up to date were
identified. These are shown to be vulnerable to publicly-available
exploits and could result in malicious execution of code, theft
of sensitive information, or cause denial of services which may
impact the infrastructure. Using outdated applications increases the
possibility of an intruder gaining unauthorized access by exploiting
known vulnerabilities. Patch management ought to be improved and
updates should be applied in conjunction with change management.
```


## Technical Findings and Recommendation:

This section is often presented in tabular form and provides full details of the findings. A finding might cover one vulnerability that has been identified or may cover multiple vulnerabilities of the same type.
It's important to note that there might be a need for an attack narrative. This narrative describes, in story format, exactly what happened during the test. This is typically done for a simulated threat engagement but is also useful at times to describe the more complex exploitation steps required for a regular penetration test. If it is necessary, then writing out the attack path step-by-step, with appropriate screenshots, is generally sufficient. An extended narrative could be placed in an Appendix and referenced from the findings table.

> [!note]- Screenshot
> ```
> Below are three example entries:
> 
> “at tuber anttetow =| sormentins
> Seo Pero ara none cass nee
> oom ranean est | per bet om vcd
> 
> aa by a strict policy. All accounts
> that are no longer required. The following issues | itt aay. is should
> were identified by performing an analysis of ape seeraae them. All
> 122,624 user accounts post-compromise: 722 user
> accounts should be set to
> accounts were configured to never expire; 23,142 a i
> users had never logged in; 6 users were members | ©XPI"e automatically. Accounts
> a oe ar no longer required should be
> of the domain administrator group; default initial re :
> passwords were in use for 968 accounts. or
> To prevent information
> gathering via anonymous SMB
> : sessions: Access to TCP ports
> Information enumerated through an anonymous en Beier
> 2 SMB Soar a MB ation ined | Testticted based on roles and
> was then used to gain unauthorized user access as | eduirements. Enumeration of
> ei - ‘SAM accounts should be
> detailed in Appendix E.9. i"
> disabled using the Local
> Security Policy > Local
> Policies > Security Options
> Malicious JavaScript code can be run to silently
> carry out malicious activity. A form of this is
> reflected cross-site scripting (XSS), which occurs
> when a web application accepts user input with
> embedded active code and then outputs it intoa _| Treat all user input as
> webpage that is subsequently displayed to a user. | potentially tainted and perform
> This will cause attacker-injected code to be proper sanitization through
> executed on the user's web browser. XSS attacks | special character filtering.
> can be used to achieve outcomes such as Adequately encode all user-
> unauthorized access and credential theft, which controlled output when
> can in some cases result in reputational and rendering to a page. Do not
> financial damage as a result of bad publicity or include the username in the
> fines. As shown in Appendix E.8, the [client] error message of the
> application is vulnerable to an XSS vulnerability application login.
> because the username value is displayed on the
> screen login attempt fails. A proof-of-concept
> using a maliciously crafted username is provided in
> Appendix E.
> Table 1 Findings and Recommendations
> ```

The Technical Findings and Recommendations section will likely be a major part of the report and the time and effort invested in writing it should reflect its importance.
In describing the findings, we will present the means of replicating them, either in the body of the report or in an appendix.

We need to show exactly where the application was affected, and how to trigger the vulnerability. A full set of steps to replicate the finding should be documented with screenshots. This includes steps that we take for granted (such as running with administrative privileges), as these may not be obvious to the reader.

The details should be separated into two sections:
The affected URL/endpoint
A method of triggering the vulnerability

## Appendices, Further Information, and References:

The final part of the report is the Appendices section. Things that go here typically do not fit anywhere else in the report or are too lengthy or detailed to include inline. This includes long lists of compromised users or affected areas, large proof-of-concept code blocks, expanded methodology or technical write-ups, etc. A good rule to follow is if it's necessary for the report but would break the flow of the page, put it in an appendix.