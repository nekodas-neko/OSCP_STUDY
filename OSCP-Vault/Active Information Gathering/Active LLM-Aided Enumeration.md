---
tags:
  - ai-assisted
  - phase/enumeration
---

# Active LLM-Aided Enumeration

As we learned in the previous section about DNS enumeration, one of our first steps is to create a robust wordlist. In this section, we'll learn how to use LLMs to create effective wordlists for DNS enumeration. Building a strong wordlist is crucial to uncovering subdomains, services, or directories associated with a target domain. Traditionally, this process required us to manually sift through public information, but modern LLMs can streamline and enhance this step, helping us uncover patterns and generate better results.

Let's take megacorpone.com as an example again. We can start by asking ChatGPT to fetch any company's public data and filter that output to create a list of likely DNS subdomains that will make up our wordlist.

> [!note]- Screenshot
> ```
> Using public data from MegacorpOne’s website and any information that can be inferred
> about its organizational structure, products, or services, generate a comprehensive
> List of potential subdomain names.
> 
> ‘+ Incorporate common patterns used for subdomains, such as:
> 
> + Infrastructure-related terms (e.g., “api”, "dev", "test", staging").
> 
> + Service-specific terms (e.g., "mail", "auth", “cdn", “status").
> 
> + Departmental or functional terms (e.g., "hr", “sales”, “support™).
> 
> ‘+ Regional or country-specific terms (e.g., "us", "eu", “asia®).
> 
> + Factor in industry norms and frequently used terms relevant to NegacorpOne's
> sector.
> Finally, compile the generated terms into a structured wordlist of 1000 words,
> optimized for subdomain brute-forcing against megacorpone.com
> Ensure the output is in a clean, lowercase format with no duplicates, no bulletpoints
> and ready to be copied and pasted.
> Make sure the list contains 1000 unique entries.
> 
> Listing 67 - LLM's prompt to generate a DNS subdomain wordlist
> ```

Using public data from MegacorpOne's website and any information that can be inferred about its organizational structure, products, or services, generate a comprehensive list of potential subdomain names.
	•	Incorporate common patterns used for subdomains, such as:
	•	Infrastructure-related terms (e.g., "api", "dev", "test", "staging").
	•	Service-specific terms (e.g., "mail", "auth", "cdn", "status").
	•	Departmental or functional terms (e.g., "hr", "sales", "support").
	•	Regional or country-specific terms (e.g., "us", "eu", "asia").
	•	Factor in industry norms and frequently used terms relevant to MegacorpOne's sector.

Finally, compile the generated terms into a structured wordlist of 1000  words, optimized for subdomain brute-forcing against megacorpone.com

Ensure the output is in a clean, lowercase format with no duplicates, no bulletpoints and ready to be copied and pasted.
Make sure the list contains 1000 unique entries.



In order to speed up our DNS enumeration, we can install gobuster on our Kali machine, which is an open-source, command-line tool designed for fast and efficient brute-forcing and enumeration of different kind of resources.



With gobuster installation out of the way, we will issue gobuster dns -d megacorpone.com -w wordlist -t 10 to perform a DNS brute-forcing operation on the domain megacorpone.com.

The dns option will specify that we are utilizing Gobuster's DNS module. By including -d megacorpone.com, we will direct the tool to target this specific domain for our enumeration.

The -w wordlist.txt argument will allow us to supply the custom LLM-generated wordlist. Finally, the -t 10 parameter will define the level of concurrency, setting Gobuster to use 10 threads simultaneously in order to improve efficiency.

> [!note]- Screenshot
> ```
> Here, we specified in the LLM-prompt the output format and the wordlist size as well.
> The answer we receive from the LLM is quite verbose, but we can focus on the most
> relevant bit of it.
> T have generated a structured 10@0-word subdomain wordlist optimized for brute-forcing
> against megacorpone.com. You can download the file for use directly:
> Download Subdomain Wordlist
> Listing 68 - LLM’ response with the DNS subdomain wordlist
> ```


> [!note]- Screenshot
> ```
> In order to speed up our DNS enumeration, we can install gobuster on our Kali machine,
> which is an open-source, command-line tool designed for fast and efficient brute-
> forcing and enumeration of different kind of resources.
> Before running our DNS brutefoce enumeration, we need to install it on Kali, as follows:
> sudo apt update
> sudo apt install gobuster
> Listing 69 Instaling gobuster on Kali
> ```


> [!note]- Screenshot
> ```
> | kaligeali:~ gobuster dns -d megacorpone.com -w wordlist.txt -t 10 a)
> | obuster v3.6 i
> | by 03 Reeves (@TheColonial) & Christian Mehlmauer (@firefart) H
> | [+] Domain: © megacorpone.com i
> | tel threads: 10 i
> | efrimecetsS /
> | [el wordlist: wordlist i
> | starting gobuster in DNS enumeration mode :
> | (InFo} [-] Unable to validate base domain: megacorpone.com (Jookup megacorpone.com on!
> | 8.8.8.8:53: no such host) i
> { Found: admin.megacorpone.com H
> | oe ee /
> | Found: mail.negacorpone.com i
> | Found: support megacorpone.com i
> | eee /
> sea ig ro ara Galeries sitciraal arcraecsior wit oir Aeparecsied worst aa
> 
> © Info
> 
> Gobuster > 3.6 uses the --do flag for domains instead. We can always doublecheck
> 
> the help page when in doubt about a specific tool usage.
> ```


```sh
gobuster dns -d megacorpone.com -w wordlist.txt -t 10
```
