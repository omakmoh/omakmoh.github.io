---
layout: post
title: Implementing Binary Search Algorithm in Blind SQL Injection
tags: [sqli]
published: true
---
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

