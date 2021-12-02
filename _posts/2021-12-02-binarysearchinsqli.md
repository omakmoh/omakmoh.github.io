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
![BinarySearchmiddle](/assets/img/BinarysearchMiddle.png)

Now we should compare the middle(79) with the target value(91)
The target value is greater than 79.
So our target value in the second half of the array We will ignore the first half
![BinarySearch3](/assets/img/BinarySearch3.png)
