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

Client-side attacks are an effective way of getting an initial foothold on a non-routable internal network precisely because they don't attack a service — they exploit weaknesses and everyday functionality in software the client already trusts. Across this module:

- [[Client-Side Attacks/Target reconnaissance/_index|Target reconnaissance]] established *what* to attack — document metadata and browser/OS fingerprinting to pick a vector that will actually run on the target.
- [[Client-Side Attacks/Exploiting Microsoft Office/_index|Exploiting Microsoft Office]] weaponized macros into a full PowerCat reverse shell, and covered the delivery obstacles (MOTW, Protected View, default macro-blocking) that have to be routed around first.
- [[Client-Side Attacks/Abusing Windows library files/_index|Abusing Windows library files]] showed a lesser-known alternative to macros — a two-stage `.Library-ms` + `.lnk` chain that never triggers Mark of the Web at all, since the payload travels over what Windows treats as a network location rather than an internet download.

> [!success] The throughline
> Every vector in this module follows the same shape: recon first to confirm the target can actually run the payload, then pick delivery/format tricks that dodge whatever control is currently in the way (spam filters, MOTW, macro-blocking, AV). The specific bypass changes; the recon-then-route-around-the-control pattern doesn't.

## Related Sections
- [[Phishing Basics/_index|Phishing Basics]] — the delivery mechanism for these payloads
- [[Assess threats from malicious files]]
- [[Identifying risks of malicious Office macros]]
