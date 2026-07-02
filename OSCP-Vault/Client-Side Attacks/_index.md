---
tags:
  - client-side-attacks
  - phase/initial-access
---

# Client-Side Attacks

%% moc %%
> [!abstract] Map of Content
> Getting code execution by exploiting the *client* rather than a server: recon the target's software stack, weaponize Microsoft Office, and abuse Windows library files.
>
> ⬆ Up: [[🏠 Home]]

## Overview

Traditional perimeter exploitation (attacking exposed services) is increasingly rare — a Verizon report ranks phishing as the **#2 attack vector** for breaching a perimeter, second only to credential attacks. **Client-side attacks** are the technical companion to phishing: instead of attacking a service, deliver a malicious file directly to a user and exploit weaknesses in their *local* software (browser, OS components, Office).

> [!warning] Ethical boundary
> The goal is code execution — never blackmail, impersonating law enforcement, or similar overreach. Client-side attacks exploit human trust; that comes with a firm ethical line penetration testers must not cross.

Why this vector is so effective, and hard to defend:
- Client machines on an internal network usually **aren't directly reachable or externally exposed** — standard perimeter defenses don't apply the same way.
- Payloads typically land on a **non-routable internal network**, giving a foothold somewhere external scanning can't reach.
- Email delivery specifically keeps getting harder — spam filters, firewalls, and scanning increasingly catch links/attachments (see [[Understanding the role of inbound email filters]]).

**Recon always comes first** — the payload has to match what the target can actually run:

| Target has... | Usable vector |
|---|---|
| Windows OS | JScript via Windows Script Host, or `.lnk` shortcut files |
| Microsoft Office installed | Malicious macro documents |

Delivery mechanisms: email attachments, malicious links, USB dropping, watering hole attacks.

## Subsections
- 📁 [[Client-Side Attacks/Target reconnaissance/_index|Target reconnaissance]] — info gathering & client fingerprinting
- 📁 [[Client-Side Attacks/Exploiting Microsoft Office/_index|Exploiting Microsoft Office]] — prep, install, weaponized macros
- 📁 [[Client-Side Attacks/Abusing Windows library files/_index|Abusing Windows library files]] — `.library-ms` code execution

## Wrapping up
<!-- 12.4 summary goes here once the section is written -->

## Related Sections
- [[Phishing Basics/_index|Phishing Basics]] — the delivery mechanism for these payloads
- [[Assess threats from malicious files]]
- [[Identifying risks of malicious Office macros]]
