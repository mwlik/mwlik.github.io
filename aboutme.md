---
layout: page
title: About Me
subtitle: ‎0x4141414141414141
---

Offensive security engineer focused on web and browser exploitation. Passionate about uncovering, reproducing, and tinkering with intricate bugs in high-impact, widely used software.

# Work Experience

**[LEET Solutions](https://www.leetsolutions.ma/)** — Morocco — 2024–Present — *Core Pentester*
{: style="margin-bottom: 2px;"}
- Performed penetration testing on tens of web applications spanning diverse tech stacks and AI-powered solutions for high-profile Moroccan clients, including the ["Marhaba operation"](https://diasporafordevelopment.eu/cpt_practices/marhaba-operation/), [OCP Group](https://www.ocpgroup.ma/en) (Fortune 500 Arabia), and [other big Moroccan companies](https://www.leetsolutions.ma/projects).
- Findings from engagements led to multiple CVEs in widely-used open-source projects (see Security Advisories), as well as community-recognized security research: ["Revisiting ReDoS Attacks"](https://mwlik.github.io/2026-04-15-revisiting-redos-attacks/) and ["Reading your files is two flags away"](https://mwlik.github.io/2025-08-11-reading-youre-files-is-two-flags-away/).
- Utilized Burp Suite, Frida, custom discovery tooling, and security-tailored AI agents across assessments.

# Security Advisories

- [`CVE-2026-24486`](https://github.com/Kludex/python-multipart/security/advisories/GHSA-wp53-j4wj-2cfg): [python-multipart](https://github.com/Kludex/python-multipart), used by FastAPI (~tens of millions of downloads) — Arbitrary file write via a non-default configuration.
- [`CVE-2026-22256`](https://github.com/salvo-rs/salvo/security/advisories/GHSA-rjf8-2wcw-f6mp): [Salvo](https://github.com/salvo-rs/salvo) web framework — Reflected XSS allowing arbitrary JavaScript execution in victims' browsers.
- [`CVE-2026-22257`](https://github.com/salvo-rs/salvo/security/advisories/GHSA-8j5r-5j5r-5j5r): [Salvo](https://github.com/salvo-rs/salvo) web framework — Stored XSS via malicious file uploads, enabling persistent code execution.
- [`CVE-2025-58362`](https://github.com/honojs/hono/security/advisories/GHSA-9hp6-4448-45g2): [Hono](https://github.com/honojs/hono) web framework (~27M monthly downloads) — Path confusion flaw allowing bypass of proxy-level access controls (e.g. Nginx).
- [`CVE-2025-59139`](https://github.com/honojs/hono/security/advisories/GHSA-92vj-g62v-jqhh): [Hono](https://github.com/honojs/hono) web framework (~27M monthly downloads) — HTTP header parsing violation allowing bypass of body size limits.
- [`CVE-2025-53535`](https://github.com/better-auth/better-auth/security/advisories/GHSA-36rg-gfq2-3h56): [Better-Auth](https://github.com/better-auth/better-auth) library (~2.8M monthly downloads) — Open redirect affecting authentication flows across multiple routes.
- [`GHSA-hq75-xg7r-rx6c`](https://github.com/better-auth/better-call/security/advisories/GHSA-hq75-xg7r-rx6c): [Better-Call](https://github.com/Bekacru/better-call) framework (~2.7M monthly downloads) — Routing flaw leading to cache deception attacks.

# Bug Bounty <span style="font-size: 0.6em;"><small>Reported 4 critical bugs</small></span>

- Account takeover via XSS in a multi-million-user website, reported through a private program ($4,500 bounty).
- Web2/XSS in `require_payment` function in `x402.fastapi.middleware` Python package can lead to ATO or funds stealing in [Coinbase's x402 protocol](https://github.com/coinbase/x402) ($2,000 bounty).
- Web2/XSS in the basic HTML paywall in `@x402/express`, `@x402/hono`, `@x402/next` packages can lead to ATO or funds stealing in [Coinbase's x402 protocol](https://github.com/coinbase/x402) ($200).
- NULL pointer dereference in a blockchain library allowing full denial of service ($400).

# CTF Experience (Awards)

- **Cyber Odyssey 2024:** Secured first place in Cyber Odyssey 2024, the biggest CTF competition in Morocco, winning a total of 80,000 DH with my team FC2MK, focused on Web challenges.
- **MCSC National CTF 2024:** Secured second place with my team FC2MK, winning a total of 10,000 DH, focused on Web challenges.
- **NULL Hat Morocco 2025:** Secured second place with my team FC2MK focusing on Web challenges.
- **The International Days of Ethical Hacking (IDEH) v7 CTF:** Secured the third place with my team FC2MK, winning a total of 3,000 DH, focusing on Web challenges.

# Browser Exploitation Research

- [`CVE-2025-10891`](https://mwlik.github.io/2026-03-01-exploiting-chromium-140-renderer-process/): Reproduced and exploited [Issue 443765373](https://issues.chromium.org/issues/443765373), an integer truncation in Ignition — V8's interpreter — that leads to arbitrary bytecode execution ultimately leading to sandbox renderer process RCE.
- [`CVE-2023-6702`](https://github.com/mwlik/v8-ndays/tree/main/CVE-2023-6702): Reproduced [Issue 40941600](https://issues.chromium.org/issues/40941600), a type confusion vulnerability in V8 leading to memory corruption and RCE eventually.

# Education

**[1337 Coding School](https://1337.ma/en/) - University Mohammed VI Polytechnic** — Khouribga, Morocco — 2023–2025
*Software Engineering* — Completed [42](https://www.42network.org/)'s project-based curriculum focused on systems, algorithms, and practical engineering fundamentals.

# N.B.

All external resources mentioned in all of my writings are included solely for their technical content; the views, backgrounds, or actions of the creators do not reflect my endorsement.

# Public PGP key

```
-----BEGIN PGP PUBLIC KEY BLOCK-----

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
=OLNE
-----END PGP PUBLIC KEY BLOCK-----
```
