---
layout: post
title: Why Whitebox?
subtitle: Why Whitebox is a Smarter Choice
mathjax: true
tags: [Rants]
author: M411K
---

This is meant to be more of a rant than an essay, but I’ll try to be as articulate as I can.

# Definitions

Whitebox: From now on, whenever I say "whitebox," what I mean is giving partial (critical parts) or complete access to the source code (e.g., of the application) to the infosec professional you're hiring for a "pentest". I’m aware that what I’m referring to as "whitebox" might be called "greybox" in some contexts, but I’ve chosen to continue calling it "whitebox" here, so in this case, the terms are interchangeable.

Blackbox: This is the contrary of whitebox, e.g not giving you're hiree the source code of the application.

Application: Due to my bad english, and lack of eloquence, I ended up mostly saying application in the rest of the blog, while am actually referring to any form of software, firmware, site(s), Android/IOS application that's pertinent to you.

Hiree: This will be used interchangeably with infosec person, pentester, security researcher, although I prefer the latter one, and loathe the penultimate for some reason.

# Rational

My rationale for this can be stated as follows:

## The real attacker >> Your security guy

What a lot of managers often miss when trying to choose between whitebox and blackbox is that they put that attacker and the pentester on the same scale, and that's absolutely misleading because they're missing that the attacker can be assumed to have infinite time and resources, think of it in the worst case scenario: each person on earth can be the feared attacker, and each one of them can have the biggest nation computing resources and knowledge, so putting them on the same situation (blackbox) is really unjust, because you're comparing a hiree with limited time, resources and knowledge, with an enormous number of hackers with certainly more knowledge and resources.

This is where whitebox comes in, you're desperate - still necessary - solution for breaching this gap, where you give your hiree a bit of ground and competitive advantage by providing them with the application source code.

## You're just procrastinating

A more grave misconception is that these two approaches are two separate roads, where as they are actually the same road, with "Blackbox" being the first long-ish part the "Whitebox" being the second part of this road, and you're actually just procrastinating you're security researcher from getting sooner to the second part of the road, because sooner or later (assuming you go with a blackbox approach) the infosec person will get to a level of understanding and familiarity with the system as if you gave em the code from the first off, so you're just actually wasting a large portion of the engagement with a work that could've been avoided with a single attachment.

If you're doubtful of this idea, that giving enough time, you're application's source code is practically exposed, and any sufficiently complex system will eventually be exposed to its edge cases, cf. [Leaky abstraction](https://en.wikipedia.org/wiki/Leaky_abstraction), [Murphy's law](https://en.wikipedia.org/wiki/Murphy%27s_law).

## Conclusion

In conclusion, what you want from your hiree is to find the largest portion and most important number of vulnerabilities for you to patch them in a limited time, and a whitebox approach can help greatly with that. So if you're a project manager reading this, I hope I convinced you of choosing a whitebox approach over a blackbox-ed one, and of course there is always an exception to the rule, so take my advice with a grain of salt by addapting it to you're own situation

Thanks for @nyly for proof-reading this. Et voila!
