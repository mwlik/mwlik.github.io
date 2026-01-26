---
layout: post
title: Tiktok Open Redirect
subtitle: Watch your Regexes!
mathjax: true
share-img: https://github.com/user-attachments/assets/b0372bec-de72-489c-941e-1571c3768e11
cover-img: https://github.com/user-attachments/assets/b0372bec-de72-489c-941e-1571c3768e11
tags: [Web, Rants]
author: m411k
---

Quick lesson why you should mind your regexes.

<center><em>Note this was reported (in collaboration with nyly and s0ng0ku) 8 months ago to Tiktok with no apparent fix published, beware not to be spammed by this bug!</em></center>


Apparently Tiktok's login page uses a query parameter `redirect_url` to save where it should return after a successful login attempt, alack a check is implemented to `redirect_url` to make sure it's a trusted origin and thus prevent [open redirect attacks](https://cheatsheetseries.owasp.org/cheatsheets/Unvalidated_Redirects_and_Forwards_Cheat_Sheet.html), the check goes as follows (the code is minified/obfuscated since it's in production a *common practice*):

```js
// ...
u =
  d.match(location.origin) &&
  "/" === (null === (r = new URL(d)) || void 0 === r ? void 0 : r.pathname)),
  (p =
    [
      /^https?:\/\/([\w\-.]+?\.)?tiktok\.(com|in)(\/.*)?$/,
      /^https:\/\/seller\.tiktokglobalshop\.com\//,
      /^https:\/\/business\.tiktokshop\.com(\/.*)?/,
    ].some((e) => new RegExp(e).test(d)) ||
    (t &&
      [
        /^https:\/\/seller-boe\.byteintl\.net\//,
        /^https?:\/\/([\w\-.]+?\.)?bytedance\.net(\/.*)?$/,
      ].some((e) => new RegExp(e).test(d)))));
// ...
```

The regexes seems to be sufficient, alas the third regex (`/^https:\/\/business\.tiktokshop\.com(\/.*)?/`) is particulary interesting since it doesn't mandate a trailing slash (`/`), nor does it mandate it to finish by `.com` i.e. by adding a trailing dollar sign (`$`), this opens the door for a textbook trick known by most bug hunters/CTF players, which is that a URI (as defined in the [RFC](https://datatracker.ietf.org/doc/html/rfc3986#section-3.2)) can contain a `userinfo` part which is seperated from the `host` by an at (`@`), so a browser that parses this url for example `https://business.tiktokshop.com@example.com` will process `example.com` as the host, thus bypassing the intended checks.

# PoC

<center>
<br />
<video height="400" controls style="max-width: 90vw;">
  <source src="/assets/img/PoC.mov" type="video/mp4">
</video>
<br />
<code>https://www.tiktok.com/login?redirect_url=https://business.tiktokshop.com@example.com</code>
</center>
