---
layout: post
title: Cyberdefender | seized
published: true
---



## [](#header-2)Foreword. 
This is a writeup of the seized challenge. It comes from the CTF sharkCTF of 2019, and has been slightly modified. I would like to thank the creator of this challenge (2phi and Nofix). 

### [](#header-3)Introductions + Custom Image.
The objective of this challenge is to analyze a RAM dump, based on a CentOS system. 
The tools used will be the following 
- volatility V2 (https://github.com/volatilityfoundation/volatility)
- cyberchef (https://gchq.github.io/CyberChef/)
- grep (bmV2ZXIgZ29ubmEgZ2l2ZSB5b3UgdXA=)

As explained above we are going to use volatility on a CentOS system. Unfortunately the Centos "profile" does not exist by default and we need a profile to be able to use volatiliy. 

```
this is a comand in a CLI
```

![this is a pisture](/assets/Q3-seized.png)