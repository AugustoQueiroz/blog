---
title: 'Encryption vs. Hashing'
subtitle: 'Why you encrypt your files and hash your passwords'
category: tech
---

Recently I came upon an interesting Reddit comment on a post about a security leak with Amazon's Ring. In the comment a user talked about how they had faced a similar situation before, and after being dismissed for not using two-factor authentication (2FA) they had discovered that their password was stored in plaintext on the servers. This led to a response where a user talked about encrypting passwords, and how easy it is to do nowadays such that it is innacceptable that it isn't done more commonly, which lead to a further response about how it is important to remember that passwords shouldn't be *encrypted* but *hashed*. As I personally used to use "encryption" do describe the operation done to the passwords for storage, I thought this was an interesting distinction that I had never really thought about.

## Encryption

From [Wikipedia]():

> In cryptography, encryption is the process of encoding information.

Ok, that's the very first sentence on the page, and probably not enough to give a good understanding what encryption is (also, it would be really low-effort if I just quoted the literal first sentence of wiki to explain something). The idea of encryption is a very old one and it comes from the need to send messages that you don't want anyone but the intended recipient to be able to read. For that you perform an operation on the message (or, more generally, on the *data* you want encrypted) using a *key* such that the person on the other side, knowing the *key* can *decrypt* it and get the original data back. A popular and famous first example of such an operation (and such a cypher) is [Caesar's Cypher](), which consists of taking an alphabetic cypher and shifting each letter on the message by a set number, which is the key often called *k* (a common k is 13, having it's own name: [rot-13]()), so 'a' with rot-13 becomes 'n', 'b' becomes 'o' and so on. Encryption has long preceeded computing, and it has had a very important place specially in times of war, but the advent of computers has completely revolutionized how encryption is thought about, because most encryption that was good enough for people trying to solve by hand became completely useless when you could have a computer go through it to try and *break* it.

Nowadays the (probably) more well known encryption system is [RSA]().

## Hashing

Also from [Wikipedia]() (for symmetry's sake):

> A hash function is any function that can be used to map data of arbitrary size to fixed-size values. 

The idea of a hashing algorithm is to map one input from one domain to another domain, that second domain having a limited size. Hashing algorithms have a big importance in datastructures, where if you ever used a python dict, for example, you've dealt with hashing without even knowing it (which is why you can't just use anything as a key for a dict). In cryptography, the hash function is developed to be very hard to reverse, such that if you pass a given *message* through the algorithm, in order for someone to discover what the original message was from the generated hash they might as well just try every single possibility until one of them gives a matching hash.

A popular hash algorithm is [SHA]() (which, not very creatively, stands for *Secure Hash Algorithm*).

## Why encrypt files and hash passwords?

So, to the point that was raised by the user in Reddit: why the distinction when talking about password storage between encryption and hashing? Well, because when you store a password you have no reason to want to know what it is (you shouldn't have, anyway!). Instead, to check that the correct password has been entered you can just hash it and compare the hashes. If they match you can be highly certain that the correct password has been entered. This is not the case, for example, when you want to make sure that you alone can read a document on your computer. In that case you'll encrypt it, usually providing a password that will serve as the encryption key.

## Other uses of encryption and hashing