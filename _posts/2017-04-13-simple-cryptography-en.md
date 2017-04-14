---
layout: post
title: Web Security(1) - Basic cryptography
comments: True 
subtitle: Basic cryptography
tags: security cryptography
author: jyt0532
---


This article is about Basic knowledge of web security and so many protocol built on top of it. You will learn a lot after reading this article.
This will not have too many mathematical details, just imagine you are watching a fun story.

### Basic cryptography

There are two kind of encryption/decryption methods in cryptography. 
One is **Symmetric Encryption** and the other is **Asymmetric Encryption.** 
The difference is very simple: 
Symmetric Encryption means we use the same key for both encryption and decryption. 
As long as A and B have the same key, A encrypted the data by the key and B decrypted by the same key. 
This way, A and B can talk to each other without letting anyone knowing the content. 
Even though someone is eavesdropping, the person cannot decode their message as long as the person does not know their key.

What can go wrong here? Before you communicate to each other secretly, 
you need to at least once tell others the common key you are about to use. 
What if the first communication is eavesdropped? 
A man in the middle will know your key and can decrypt anytime. 
Worst of all, the person can decrypt and modified the content, then encrypt again without letting you know. 

What can we do?

### Asymmetric Encryption

Here comes the hero!

Asymmetric Encryption, which mean a key pair have two keys. A public key and a private key.

You can use public key to encrypt and private key to decrypt. You can also use private key to encrypt and public key to decrypt.

You can use public key to encrypt and private key to decrypt. You can also use private key to encrypt and public key to decrypt.

You can use public key to encrypt and private key to decrypt. You can also use private key to encrypt and public key to decrypt.

This is the most important part of this article so it worth to mentioning three times.

Before communicate, A and B both generate a public/private key pair.
A send A's public to B, B send B's public key to A.
So now, A have A's private key and B's public key. B have B's private key and A's public key.
**A use B's public key to encrypt before sending to B, B use B's private key to decrypt**

Period! Even if the communication is eavesdropped, nothing will be leaked as long as B's private key is not known by anyone else.

Main idea is to use the attribute that we can use **public key to encrypt.** 
And you **have to** use private key to decrypt if you encryped by public key. 
So I have nothing to worry even if my public key is leaked. As long as I keep my private key safe(In this case, private is never transmitted), no one can intercept the message.

Why this approch solved previous problem? Because our first communication only exchange the public key, so even if public key is intercepted by man in the middle, he can do nothing with the public key.

Everything looks fantastic.

![Alt text]({{ site.url }}/public/car_accident-iloveimg-cropped.gif)

It is a foreshadowing. I will come back to this later. It is first time in your life you saw a foreshadowing is indicated itself a foreshadowing. 
That is the reason this article is unusual and outstanding.

### Digital Signature

The story continues, since everyone has B's public key, everyone can use B's public key to send secret message to B. 
How can B knows who sent the message to him? 
If A claimed A sent the message, how can B know it is indeed sent by A not others? Now we introduce digital signature.

Before A send the message to B, A use A's private key to encrypt and send it to B. 
B then use A's public key to check whether it is signed by A(In fact, it is sign to the hash of the content)
We use the second sentence in "You can use public key to encrypt and private key to decrypt. You can also use private key to encrypt and public key to decrypt."

We do not care that everyone has public key. Even though all of the world knows the message is sent by A, no one can see the content as long as no one knows B's private key. 
### Conclusion

Time to conclude everything, if A needs to send message to B, how to make **this message seen only by B and B can ensure it is written by A**?

==================================================================

First: A send A's public key to B, B send B's public key to A

Second: Before A send to B, A use B's public key to encrypt and sign with A's private key
B use A's public key to check signature and use B's private key to see content

==================================================================

Perfect! Next article is about a common application - ssh

### How about the foreshadowing?

What if there is a "god in the middle"? The Asymmetric Encryption can break!

1.When A is sending A's public key to B, if C intercept, 
C can take his public key and create another public/private key pair, give public key(C1) to B and told him it is A's public key.
![Alt text]({{ site.url }}/public/cryptography-step1.png)
2.When B is sendind B's public key to A, if C intercept, 
C can take his public key and create another public/private key pair, give public key(C2) to A and told him it is B's public key.

![Alt text]({{ site.url }}/public/cryptography-step2.png)
3.A and B thought they hold each other's public key.

![Alt text]({{ site.url }}/public/cryptography-step3.png)
4.Today, say, A wants to send message to B. He used B's public key to encrypt(Actually, it is public key C2) and signed with A's private key.
He wants to send to B but it is intercepted by C

![Alt text]({{ site.url }}/public/cryptography-step4.png)
5.C use A's public key to verify and use private C2 to see the content, then modify the content as he want.

6.Then, use B's public key to encrypt and use private C1 to sign(pretend to be A)

![Alt text]({{ site.url }}/public/cryptography-step6.png)
7.When B got the message, he used A's public key to verify(Actually, it is public C1), then use B's private key to see the content

8.Abracadabra

![Alt text]({{ site.url }}/public/change_face.gif)

We will discuss how to prevent man in the middle attack in next article.


