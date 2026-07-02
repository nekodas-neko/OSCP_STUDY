---
tags:
  - client-side-attacks
  - phase/recon
---

# Target reconnaissance

%% moc %%
> [!abstract] Map of Content
> Before weaponizing anything, work out what software and OS the target is actually running.
>
> ⬆ Up: [[Client-Side Attacks/_index|Client-Side Attacks]]

## Overview

Client-side recon is fundamentally different from network recon: there's usually **no direct connection to the target** — client machines aren't reachable the way exposed services are, so this calls for a more tailored, creative approach rather than a straightforward scan.

Two goals before attacking:
1. **Identify potential users** — company website contacts, or passive OSINT/social media (see [[Passive Information Gathering/_index|Passive Information Gathering]]).
2. **Determine their OS and installed software** — the payload has to match what they can actually run.

## Notes
- [[Information gathering]]
- [[Client fingerprinting]]

## Related Sections
- [[Client-Side Attacks/Exploiting Microsoft Office/_index|Exploiting Microsoft Office]]
