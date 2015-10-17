---
layout: post
title: "Crypto magic - zero knowledge"
date: 2015-05-05 09:23:11 +0300
comments: true
categories: [crypto]
---

####There are some applications of cryptography that can only be classified as magical. It is only fitting that my first crypto post begins with one of them - zero knowledge proofs.
####I will try and explain it without all the rather complicated mathematical stuff.

<!--more-->

{% img /images/Zero-Knowledge.png 200 200 %}

####Let's start with a simple definition. 

>####This is where one party (called the Prover) convinces a second party (called the Verifier) that they know the answer to some question. The catch is that the Verifier never gains any knowledge of what the actual answer is.

####I'll start with a crude example to explain further (don't worry, I'll give a detailed one later).
####Let's say I want to sell a super secret formula about how to cure a disease to a powerful company. A problem arises where the company needs proof that I indeed know the super secret formula yet I cannot tell them what it is (my payday depends on it). The company should gain no information of what the super secret formula is from my proof, so I cannot let them test it. In the real world we would use a third party to hold both the payment and formula, and do some independent tests of the formula, but where's the magic in that :). In any case I don't trust anybody with the formula till I get paid. And so arises our dilemma.

#So, are zero knowledge proofs possible? Can I have my cake and eat it?
####The answer is yes and let me explain with the detailed example I promised :).
####Let's say I have two identical balls, one is red and one is blue. I want to prove to a color blind guy that they are of different colors.
####What I'll do is ask him to hold one ball in each hand so that he has a red ball in one hand and a blue one in the other. I can tell the color of each but he cannot (color blindness). Next I tell him to put his hands behind his back so that I cannot see them. Then he can decide whether to switch the balls or not to. After which he shows me what ball he's holding in each hand. He then asks me to tell him if he switched them or not. I obviously will get the answer correct as I know what color he was holding in each hand before. 
####Now here is the interesting part, the only way I can tell whether he switched the balls behind his back is if they are of different colors. So I easily prove to him they are of different colors if I consistently can tell if he switched them (which I can). I will have to get the answer correct a number of times to convince him am not guessing (Probability and Maths come in which I promised to avoid in this post). He is color blind, so at the end of it all he still doesn't get to know which ball is red and which one is blue. And so the zero knowledge proof is complete. I prove that the balls are of different colors while he gains no knowledge at all which of them is blue and which is red.


#Can this be applied in the information Security world?
####Yes it can :)
####Let's say I want to authenticate with a server. I however don't want to reveal my password to the server. I just want to prove to the server that I know the password to be authenticated. I also don't want the server to gain any knowledge of what my password is. In the real world, the server stores passwords in a hashed form. You send over your plain password and the server calculates its hash value and compares it with what it has stored. This fails our zero knowledge proof as the server still sees your password before conversion. So we need a different implementation. I will explain it in detail in a different post - there's mathematics involved and I want to avoid information overload here :).

##I'll leave you with the following fun zero knowledge proofs for now.
***
####See how a person can prove to another that they know the solution to a sudoku puzzle without actually reavealing the solution. Check it out <a href="http://www.wisdom.weizmann.ac.il/~naor/PAPERS/SUDOKU_DEMO/" target="_blank">**here**</a>.
***
####Also check out this cool explanation with a modified version of <a href="http://pages.cs.wisc.edu/~mkowalcz/628.pdf" target="_blank">**Ali Baba and the forty thieves**</a>.
***
####In part two of Crypto Magic, I'll talk about homomorphic encryption :).



