---
tags:
  - osint
  - passive-recon
  - phase/recon
  - shodan
---

# Shodan

Shodan is a search engine that crawls devices connected to the internet, including the servers that run websites, as well as devices like routers and IoT devices.







We can review the ports, services, and technologies used by the server on this page. Shodan will also reveal if there are any published vulnerabilities for any of the identified services or technologies running on the same host. This information is invaluable when determining where to start when we move to active testing.

> [!note]- Screenshot
> ```
> Before using Shodan, we must register a free account, which provides limited access.
> Let's start by using Shodan to search for hostname:megacorpone.com.
> 8 1 oP SS
> srereon ona ane
> Figura 12: Searcing Maya Corp One's comes with Shodan
> ```


> [!note]- Screenshot
> ```
> In this case, Shodan lists the IPs, services, and banner information. All of this is
> gathered passively, avoiding interacting with the client's web site.
> This information gives us a snapshot of our target's internet footprint. For example,
> there are four servers running SSH. We can drill down to refine our results by clicking on
> SSH under Top Ports on the left pane.
> 
> ‘igure 13: MegaCorp One servers running SSH
> ```


> [!note]- Screenshot
> ```
> Based on Shodan's results, we know exactly which version of OpenSSH is running on
> 
> each server. if we click on an IP address, we can retrieve a summary of the host.
> 
> 40 5624487 aS lie AS od —
> ‘Figure 14: Shodan Host Summary
> ```
