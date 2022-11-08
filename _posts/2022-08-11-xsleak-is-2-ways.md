---
layout: post
title: Solving XSleaks in 2 ways | Leaker - WiCSME 2022 CTF
tags: [xsleak,clientside,chrome,n-day,pwn,web]
published: true
---
# Introduction

Hi, In this blog post, We'll discuss how I solved Leaker in 2 ways.

As a client-side web challenge, Another way is Pwn.

Leaker challenge from [WiCSME CTF 2022](https://cybertalents.com/competitions/women-in-cyber-security-middle-east-wicsme-ctf-2022)

# Intended Solution ( As a web challenge )
Before we dive in, You should know
### What is XSleak?
Cross-site leaks are a class of side-channel vulnerabilities that abuse the features of web browsers that help websites to interact with each other.
## Application Discovery
Here is The main page
![](stb-1.png)

We Have 3 Functions

1. Login into your existing account
![](/assets/img/leakerwriteup/stb-login.png)

2. Register a New account
![](/assets/img/leakerwriteup/stb-register.png)

3. Report Link to admin

![](/assets/img/leakerwriteup/stb-report.png)

We will create normal account. 

![](/assets/img/leakerwriteup/stb-registering.png)

After registering an account, we'll find two functions
1. Create Paste
2. Search over our pastes

![](/assets/img/leakerwriteup/stb-home.png)

## Spotting the bug
I'll create a paste with the fake flag `FLAG{blablabla}`

![](/assets/img/leakerwriteup/stb-createpaste.png)

After clicking submit, the application will redirect us to our note.

![](/assets/img/leakerwriteup/stb-afterredirect.png)

As we see, the paste id is too long. We can't brute-force or guess.

When we use the `Search My pastes` Feature with a non-existing word in your pastes

![](/assets/img/leakerwriteup/stb-notexistingword.png)

The application response will be  `Not Found Sorry.`

![](/assets/img/leakerwriteup/stb-notfoundsorry.png)

Otherwise, If our query is true ( the word in our pastes ), the application will send us a file `code.res` containing the paste id.

![](/assets/img/leakerwriteup/stb-downloaded.png)

We have to get the flag from the admin account. We don't have XSS to steal the admin's cookie.

So our Goal is clear now.

Host the payload on our server, and send it to the admin. The admin visits our site, the script loops over the Charest, and sends search queries on behalf of the admin. When the window closes, the file downloaded equals the character in the paste.

![](/assets/img/leakerwriteup/final-intended.png)

You can find my code [here](https://gist.github.com/omakmoh/48ff4d2b4fa33fb99cedf3ca03a36a66)

---
# Unintended Solution ( As a pwn challenge )

While checking the bot request, I noticed the bot User-Agent is 

`Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) HeadlessChrome/93.0.4577.0 Safari/537.36` 

![](/assets/img/leakerwriteup/unintended-1.png)

The bot is using HeadlessChrome version `93.0.4577.0` Which is Vulnerable to `CVE-2021-30632` 

So I generated reverse shell shellcode using msfvenom with my IP & port
then modified the shellcode part in the exploit script from [Github](https://github.com/CrackerCat/CVE-2021-30632) 



![](/assets/img/leakerwriteup/jsshellcode.png)


Now we have to set up a listenr for our shell.

![](/assets/img/leakerwriteup/listen.png)

We will now upload the modified script to our server & Use the `Report to Admin` feature to make the bot visit our URL 

![](/assets/img/leakerwriteup/sendtoadmin.png)

Finally, Enjoy your ROOT shell

![](/assets/img/leakerwriteup/rootshell.png)
![](/assets/img/leakerwriteup/catflag.png)

Thanks for reading 😊

# References

1. https://xsleaks.dev
2. https://securitylab.github.com/research/in_the_wild_chrome_cve_2021_30632/
