+++
title = "Crypto Magic Homomorphic Encryption"
date = "2015-05-11 13:45:52 +0300"
tags = ["cryptography"]
+++
Could Google do your search without knowing what it was looking for? Could your hospital provide you with information regarding your health records without knowing what you asked? 
<!--more-->

Can data be encrypted in such a way that computation can be done on it without decrypting it first?
Yes it can! This is what homomorphic encryption is all about. 
Our exploration of crypto magic continues :)

Let's start with two scenarios

# scenario 1: encrypted search
First, I am not talking about searching over secure protocols like https. What I am referring to is where your search term is in encrypted form before it reaches the search engine (say Google). Google therefore doesn't know what you want to search for. 
After Google receives your encrypted search term, it searches for it in an encrypted database. The results it gets are also encrypted which are then passed on to you. You can then privately decrypt them :). You'll note that Google never knew what you were looking for nor what results were returned. No decryption occurred at any point before you received the results.

# scenario 2: secure cloud processing
Would it not be nice to have some powerful hosting provider do complex computation on your data without them ever knowing the contents? A case would be like your bank having a cloud provider hosting its data. The bank would encrypt the data before uploading it, so the cloud provider never sees the plain version. The bank would then be able to perform complex computation on the encrypted data using the cloud providers powerful processing power without it ever been decrypted. The bank could also be able to request information from the encrypted data using encrypted queries. The returned answers would also be in encrypted form which only the bank can decrypt. And again no decryption takes place at the cloud provider.

# How can this be done?
This concept was first proposed over 30 years ago by Rivest, Adleman and Dertouzos and is referred to homomorphic encryption (They also suggested it was probably impossible). 
Let's say you have x and y. if you multiply the encrypted version of x and the encrypted version of y and the answer is the same as if you multiplied their plain values and encrypted the answer, then the process is said to be homomorphic with respect to multiplication.
Any computer algorithm is a series of arithmetic steps so if we can get a process that is homomorphic with respect to both addition and multiplication, any computing application would be possible on encrypted data.

# Has it been achieved?
Over the past three decades, partial solutions have been discovered. 
However, it is not until 2009 that Craig Gentry developed a fully homomorphic system. Unfortunately it requires lots and lots of computational power. For instance, an encrypted Google search would multiply the computation time to 1 trillion times that of a normal one :(.
The good news is that the theoretical barrier was broken and with lots of work a practical solution may be found :).

For anyone willing to give it a go, [**DARPA is offering $20 million**](http://www.i-programmer.info/news/112-theory/2330-darpa-spends-20-million-on-homomorphic-encryption.html) if you can come up with a scheme that only takes 100,000 times as long as unencrypted calculations :).

And we may well be on our way there, three MIT scientists developed a system that emulates fully homomorphic encryption for most of the functions of the SQL databases used in many applications. It computes only with encrypted data and takes just 15% to 26% more computing time.
The project is called CryptDB and you can check it out [**here**](https://github.com/CryptDB/cryptdb).

The application of this in information security does not even need explaining :)
