---
layout: post
title: Is your password secure?
comments: True 
subtitle: Password security
tags: security
author: jyt0532
---

This article is about how good website store user's password. If you are not major in Computer Science, you can scroll down to conclustion section, I will show you how secure you password is.

Let's begin.

### Hash your password

If you are a backend developer, you need a hash function to hash the plain-text password user provide and stored in database. Whenever user login, you use the same hash function and checked whether the hashed string is the same to the string in database. Since hash function is not revertible, so even if hacker hacked your database and get the hashed value, no other user information will be leaked. (Please aware that hacker cannot login by your hashed string)

HASH(plain_text) == saved_string

### Sounds pretty good, right?

It is pretty good at beginning, so most website does not limit your password complexity ten years ago. 
As the memory getting cheaper over time, **hacker can pre-store all the result of possible plain text.** When hacker hacked the database and get the hashed string, he can just check with his pre-stored table and know your plain text password. The hash function become useless if the input space is too small. 

### Give me some math

At beginning, no website limit your password. People used simple lowercase english letter and digit. 
If the length is 7, which means all the possible combination is (26+10)^7. 
Assume we use md5 hash(128 bits for hashed string)

(1\*7 + 128 / 8)\*(26+10)^7 bytes = 1.8TB

This is the space needed to store all the mapping of possible plained text. 
(1*7 is the size of plain text, 128/8 is size of hashed string)

1.8TB storage is toooooo easy in 2017, what can we do?

### Adding Salt

It is time for add some randomness into our picture. 

When we create/update user's password, we randomly generate a string and concatenate/interleave with plaintext. Same process apply to verification, as long as the server can achieve the same random string when verifying.

HASH(plain_text + salt) == saved_string

An easier way is to use username/email as our saly string. The main idea of adding salt is to add difficulty in original plain text. Since the output space of md5 is 2^128, it is not possible to pre-store all the possible output. And of course the simplest way to add complexity is to make plain text longer. It is 36 times more combination if we add one more letter or digit in our plain text.

Another way to add complexity is to use more characters like uppercase, special characters([Password special character](https://www.owasp.org/index.php/Password_special_characters)). Base will change from 36 to 94, so of course it will be harder to guess.

That is the reason why good website require your minimum length to be 12 and have to contain at least one lowercase letter, one uppercase letter, one digit and one special character. In case a hacker got your hacked string, he still cannot know your plain text. If you do add salt before hash, basically there is no easy brute-force way to guess your password.

### Questions


Q: Since we store salt in database, so the hacker will have salt as well as your hashed string. If some day we are able to pre-store all combination of length 10, hacker can achieve your plain text by deduct your salt, right?

A: So we use salt for two purposes, first is to lengthen the string before hash. Even if user provide a short password, we can still increase the difficulty. 

For exmaple, if the user provide a password of length 10, length of salt is 5, it is hard to pre-store all the combination of 94^15 strings 

Q: String with length 15 is hard but string with length 10 is a lot easy. Hacker can build all combination of length 10 with salt added, so the combination is 94^10, right?

A: Exactly, that is the reason why salt needs to be random string for differend user.

The second purpose of salt is to minimize the lost of data leakage. **Which means hacker have to build his whole mapping for every salt.** Hacker can only hack one single user at a time. It will buy security team some time to recover their system.


That is the reason why you should always add salt when you hash your password. As Buckets effect reveals, the capacity of a bucket depends on the shortest board. *Security of your password depends on the worst website among those you use the same password*

###  Is your password secure?

Longer is better! That is the most important factor. Don't forget to interleave with uppercase letter and digit and special characters.

How secure is your password? I can provide an easy formula

(1\*n + 128 / 8)\*(26 or 36 or 62 or 94)^n

n is the length of your plain text

26 if you only have lower case letter

36 if you have lower case letter and digit

62 if you have lower and upper case letter and digit

94 if you have lower and upper case letter and digit and special character

Rule of thumb is that if your number is smaller than 10^24, then you are just like Jon Snow

![Alt text]({{ site.url }}/public/hugearmy.gif)

You should change your password!
