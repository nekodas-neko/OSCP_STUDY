---
tags:
  - ai-assisted
  - passive-recon
  - phase/recon
---

# LLM-Powered Passive Information Gathering

As we know, passive information gathering involves exploring open-source data for insights into a target's infrastructure, personnel, policies, and technology stack - all without direct contact with the target's systems. LLMs can bring significant advantages to this stage. By using natural language processing to analyze vast amounts of unstructured data from various sources, these tools help us uncover patterns and connections that might otherwise go unnoticed.

> [!note]- Screenshot
> ```
> To begin, we can perform a prompt to query whois information about the megacorpone
> domain:
> whois megacorpone.com
> LUsting 5 = WHOIS prompt for megacorpone.com
> ```


> [!note]- Screenshot
> ```
> We received the following response from the LLM:
> 
> ‘The domain megacorpone.com was registered on January 22, 2013, and is set to expire on
> January 22, 2025. The registrar for this domain is Gandi SAS. (Whois)
> The registrant's contact information is as follows:
> 
> + Name: Alan Grofield
> 
> = Organization: MegaCorpOne
> 
> = Address: 2 Old Mill St, Rachel, Nevada, 89001, United States
> 
> = Phone: +1.9038836342
> 
> = Email: [email protected]
> ‘The domain's name servers are:
> 
> ‘= nst.megacorpone.com
> 
> ‘= ns2.megacorpone..com
> 
> ‘= ns3.megacorpone..com
> ‘The domain status is listed as “clientTransferProhibited,” which means the domain is
> locked to prevent unauthorized transfers. (Whois)
> Please note that WHOIS information can change over time, and some details may be
> protected for privacy reasons.
> 
> Listing 6 - WHOIS response for megacorpane.com
> ```


> [!note]- Screenshot
> ```
> We can accomplish this using the following prompt:
> Can you print out all the public information about company structure and employees of
> megacorpone?
> Listing 7 - Public Company Information Prompt for megacorpone.com
> In response, a wealth of organized information is returned, which includes contacts and
> social media handles of about every company employee.
> MegaCorp One is a company specializing in disruptive innovation within the
> nanetechnology industry. The company's headquarters are located at 2 Old Mill St,
> Rachel, Nevada, 89001, United States.
> Company Structure and Leadership:
> * Chief Executive Officer (CEO): Joe Sheer
> + Email: joe@megacorpone.com
> + Twitter: @loe Sheer
> ‘* Background: Joe Sheer has been featured in the Journal of NanoTimes, stating:
> our team is creating the building blocks of modern society, where technology and life
> are inseparable.”
> + Vice President of Legal Affairs: Mike Carlow
> + Email: Not publicly available
> «© Phone: +5-*###+#2403
> «Marketing Director: Matt Smith
> ‘+ Email: msmith@negacorpone.com
> + Twitter: @iattsmithaco
> * Web Designer: Tom Hudson
> + Email: ‘thudsongnegacorpone.com
> + Twitter: @TonHudsonsco
> + Senior Developer: Tanya Rivera
> + Email: trivera@negacorpone.com
> + Twitter: @TanyaRiveratco
> + Electrical Specialist: Mutunga Muli
> + Email: Not publicly available
> + Phone: Not publicly available
> + system Analyst: Soum Test
> + Email: Not publicly available
> + Phone: Not publicly available
> + System Administrator: Steve Wong
> + Email: Not publicly available
> + Phone: Not publicly available
> + Investments: Fred Ducasse
> + Email: Not publicly available
> + Phone: Not publicly available
> + Collections: Stan Denvers
> + Email: Not publicly available
> + Phone: Not publicly available
> © Employee: Johnny Five
> * Email: Not publicly available
> * Phone: Not publicly available
> * Employee: Ga Rod Boss
> * Email: Not publicly available
> * Phone: Not publicly available
> + Employee: Alan Grofield
> * Email: Not publicly available
> * Phone: Not publicly available
> Employee Count:
> + Total Employees: Approximately 237
> Listing 8 - Public Company information response for megacorpone.com_
> ```


> [!note]- Screenshot
> ```
> For instance, we could ask about the top 20 Google dorks aimed to our target website.
> can you provide the best 20 google dorks for megacorpone.com website tailored for a
> penetration test?
> 
> Listing 9 - top-20 google dorks prompt for megacorpone.com
> 
> The LLM replied with the following thematically-ordered answer:
> Basic Information Gathering
> 
> 1. site:megacorpone.com
> 2. site:megacorpone.com -wat
> (To identify subdomains.)
> Directory and File Discovery
> 3. site:megacorpone.com intitle:index of"
> 4, site:megacorpone.com ext:php | extzhtml | extrasp | ext:isp
> 5. site:megacorpone.com filetype:pdf | Filetype:doc | filetype:docx | #iletyperxds
> | #iletypendsx
> Vulnerable Pages or Parameters
> 6. site:megacorpone.com inurl: "admin
> 7. site:megacorpone.com inurl: "login"
> 8. site:megacorpone.com inurl:"signup"
> 9. site:megacorpone.com inurl: “register”
> 10. site:megacorpone.com inurl: "forgot password”
> 11. site:megacorpone.com inurl:*upload”
> Configuration Files and Sensitive Data
> 12. site:megacorpone.com ext:env | ext:yaml | ext:json | ext:xml
> 13. site:megacorpone.com ext:conf | exttini | ext:log
> 14. site:megacorpone.com “password” filetype:txt | filetype:csv
> Leaked Information
> 15. site:megacorpone.com “username” “password”
> 16. site:megacorpone.com “confidential”
> 17. site:megacorpone.com “private”
> Source Code and Repositories
> 18. site:megacorpone.com ext:sql | ext:db
> 19, site:megacorpone.com “gitlab" | "github" | “bitbucket™
> 20. site:megacorpone.com “source code”
> Bonus Tips
> + Combine dorks with cache: for cached versions of pages.
> + Use related:negacorpone.com to discover similar sites, potentially for third-
> party apps or vendors.
> + Include the site: dork with -ext:pdf to filter out irrelevant documents.
> Listing 10 -top-20 google dorks response
> 
> Listing 10 is an LLM-generated set of 20 Google dorks tailored for reconnaissance on
> 
> the megacorpone.com website during a penetration test. Each dork focuses on a
> 
> specific type of publicly accessible information that we can leverage to gather further
> 
> intelligence.
> ```


> [!note]- Screenshot
> ```
> Similarly to what we performed earlier with Netcraft, we can passively gather a
> website's technology stack via LLMs.
> Retrieve the technology stack of the megacorpone.com website
> sting 11 = Website technology stack prompt
> ChatGPT does not perform live lookups by default, but it may simulate results from
> services like Netcraft, Datanyze and 6sense based on its training data.
> The following response shows the different technologies making up the
> Mmegacorpone.com website, each one listed along with a brief description.
> ‘The website megacorpone.com utilizes the following technologies:
> ‘+ Programming Languages and Frameworks:
> + iQuery: A cross-platform JavaScript library designed to simplify client-side
> scripting.
> Bootstrap: A free and open-source front-end framework for designing websites
> ‘and web applications.
> Font Awesome: A toolkit for adding vector icons and social logos to websites.
> = EQUScript: A scripting-language specification standardized by Ecma
> International.
> + JavaScript: The programming language of HTML and the web.
> + HIML: The standard markup language for documents designed to be displayed in a
> web browser.
> ‘+ Content Delivery Networks (CDNs):
> = Google Hosted Libraries: A content distribution network for the most popular,
> open-source JavaScript libraries.
> = Web and Application Servers:
> ‘+ Apache HTTP Server: A free and open-source cross-platform web server software.
> = Other IT Infrastructure Software:
> + GitHub Pages: A static site hosting service designed to host personal,
> organization, or project pages directly from a GitHub repository.
> ‘These technologies collectively support the functionality and performance of the
> ‘megacorpone.com website.
> Listing 12 - Website’ technology stack response
> The tech stack of megacorpone.com is categorized into four different categories based
> ‘on the related technology.
> ```
