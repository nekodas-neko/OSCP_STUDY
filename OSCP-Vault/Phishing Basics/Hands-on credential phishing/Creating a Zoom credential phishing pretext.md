---
tags:
  - phishing
  - credential-harvesting
  - pretexting
  - llm
  - hands-on-lab
  - phase/initial-access
---

# Creating a Zoom credential phishing pretext

> [!tip] Quick Reference
> | Step | Action | Purpose |
> |------|--------|---------|
> | 1 | Use leaked/breached creds to access a low-priv mailbox | Establish an internal foothold and vantage point |
> | 2 | Read the account's **Sent** folder | Find real internal correspondence to imitate |
> | 3 | Feed a real email + instructions to an LLM | Generate a new pretext matching the org's actual tone |
> | 4 | Review/tweak the LLM output | Confirm structure and voice before sending |

## Visual Flow

```mermaid
flowchart TD
    A["Leaked/breached low-priv email account"] --> B["Log into the webmail portal"]
    B --> C["Recon: read the Sent folder"]
    C --> D["Find a real internal email<br/>with a distinct tone/style"]
    D --> E["Feed the original email to an LLM<br/>ask it to write in the same style"]
    E --> F["LLM generates a new pretext email<br/>with a clickable login link"]
    F --> G["Next: build the credential-stealing clone site"]
```

## The scenario

Assume extensive OSINT is already done, and the attacker has **low-privileged access to a real email account** — `helpdesk@mail.corp.com` at `CORP.com` — obtained from a **public password leak**. This is exactly the "compromised legitimate account" advantage from [[Email phishing]]: the account is shared/internal-only in scope, and its password isn't reused anywhere else, so it can't be pivoted further directly. Its value here is entirely as a **recon and pretext source**, not a lateral-movement foothold.

> [!example] Lab access pattern
> Webmail portal reached at `http://<MAILER-IP>/mail/`, logging in with `helpdesk@mail.corp.com` and the leaked password. In this training lab, the inbox itself was empty — all the useful material was in **Sent**.

## Step 1 — Recon inside the mailbox

Rather than reading incoming mail, check what the account has **sent** — that reveals its real, established voice. In this case, a recent email to the Sales department explained that Zoom licenses were being audited, and unused accounts would be downgraded within two weeks unless the employee logged in and scheduled a meeting.

> [!example] The real internal email found in Sent
> ```
> Subject: Zoom License Inventory Refresh
>
> Hello Sales department,
>
> Hope you're knocking it out of the park this week! We're trying to redo
> our inventory of Zoom licenses as we seem to have a large number which
> aren't being used at the moment. Rather than having everyone reply to
> the email, in order to keep your Zoom license, please just ensure that
> you login to your account and schedule a meeting within the next two
> weeks. Any accounts which don't do this within the time frame will be
> transitioned to a free license.
>
> Thank you very much for your cooperation and apologies for the hassle!
> ```
> Distinct, casual-but-professional voice — friendly opener, soft urgency, an apologetic close. This is the exact tone to imitate.

## Step 2 — Use an LLM to extend that voice

Rather than trying to freehand a matching tone, feed the **real email** to an LLM along with a request to write a similar one — a direct application of [[LLMs, generative AI and deepfakes]] to pretext-building.

> [!example] Prompt sent to ChatGPT
> *"Looking at the following email: [pasted original] — Write another email in the same style as this, and include a reminder for employees to login to Zoom. Include a hyperlink that can be clicked and directs people to the appropriate page."*

> [!example] LLM-generated pretext
> ```
> Subject: Reminder: Please Log In to Keep Your Zoom License!
>
> Hello Sales department,
>
> Just a quick reminder—hope everything's going smoothly on your end!
> We're still working on updating our Zoom license inventory and noticed
> that some accounts haven't yet logged in to schedule a meeting. To make
> sure your account remains on a full license, please click here to log
> in and schedule a meeting within the next week.
>
> If no meeting is scheduled by the deadline, any inactive accounts will
> be moved to a free license.
>
> Thanks again for your cooperation, and sorry for the added task! Let us
> know if you have any questions.
>
> Best regards,
> [Your Company] Helpdesk Team
> ```
> Same voice, same structure, same soft-urgency close — just refreshed and carrying the malicious hyperlink. Minor tweaks may still be needed to fully match the original's structure.

> [!warning] LLM guardrails can refuse the request outright
> Mainstream LLMs (ChatGPT, Claude, Gemini) commonly refuse prompts that explicitly name themselves as phishing, malware, or social-engineering content. Notice the prompt above never uses the word "phishing" — it just asks for a similarly-styled reminder email with a login link, which reads as ordinary business correspondence. If a request gets refused:
> - Strip any wording that reads as malicious intent (without misrepresenting the actual use in a way that would violate your engagement's own rules).
> - Frame the ask as tone-matching/copywriting rather than "write me a scam email."
> - As a fallback for authorized engagements, a locally-hosted or uncensored model avoids provider-side guardrails entirely — see [[LLMs, generative AI and deepfakes]].

## Why this pretext is so strong

Every earlier concept converges here:
- **Real sender** ([[Email phishing]]) — a genuinely compromised internal account, not a look-alike domain, so it clears reputation/domain-age checks in [[Understanding the role of inbound email filters]] trivially.
- **Matched department tone** ([[Enhancing phishing through social engineering]]) — not invented, *lifted* from a real internal email, so it can't fail a tone/expectation check.
- **AI-accelerated drafting** ([[LLMs, generative AI and deepfakes]]) — the LLM only needs to extend an established voice, not invent one from scratch.

> [!success] What makes this pretext land
> The recipients have almost certainly seen this exact sender and this exact topic before — the new email isn't asking them to trust something new, just to repeat an action they may already expect.

> [!danger] Common pitfalls
> - Assuming a leaked password unlocks more than the one mailbox it was leaked for — don't over-extend a low-value credential.
> - Sending LLM output verbatim without a final human read-through — small mismatches with the original structure are still detectable.
> - Overusing a shared/internal account carelessly — real users sharing that mailbox may notice unusual activity.

> [!tip] Beginner note
> This is **pretext-by-imitation**: instead of inventing a believable internal voice, find a *real* one already trusted inside the organization and extend it. It's far more reliable than trying to guess what "sounds right" for a department you've never worked in.

## Resources
- [ChatGPT](https://chatgpt.com/)

---
%% graph-links %%
## Related
- [[Email phishing]]
- [[Enhancing phishing through social engineering]]
- [[LLMs, generative AI and deepfakes]]
- [[Cloning a legitimate website]]

> [!info] Navigation
> Section: [[Phishing Basics/Hands-on credential phishing/_index|Hands-on credential phishing]] · Home: [[🏠 Home]]
