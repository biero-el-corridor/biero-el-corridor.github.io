---
layout: post
title: Cyberdefender | seized
published: true
---



## [](#header-2)Foreword. 
This is a writeup of the seized challenge. It comes from the CTF sharkCTF of 2019, and has been slightly modified. I would like to thank the creator of this challenge (2phi and Nofix). 

## [](#header-2)Introductions.
The objective of this challenge is to analyze a RAM dump, based on a CentOS system. 
The tools used will be the following 
- volatility V2 (https://github.com/volatilityfoundation/volatility)
- cyberchef (https://gchq.github.io/CyberChef/)
- grep (bmV2ZXIgZ29ubmEgZ2l2ZSB5b3UgdXA=)



## [](#header-2) Create custom Image

As explained above we are going to use volatility on a CentOS system. Unfortunately the Centos "profile" does not exist by default and we need a profile to be able to use volatiliy. 

The creators of the challenge were kind enough to give us all the files to create a custom profile. These are the following files: 
- module.dwarf
- System.map-3.10.0-1062.el7.x86_64

A complete procedure for creating a custom profile from a file is available [here](https://tunnelix.com/linux-memory-analysis-with-lime-and-volatility/), but we can summarize the creation of the profile with the following command

```
zip .\volatility\volatility\plugins\overlays\linux\Centos7.3.10.1062.zip .\c73-EZDump\Centos7.3.10.1062\module.dwarf .\c73-EZDump\Centos7.3.10.1062\boot\System.map-3.10.0-1062.el7.x86_64
```

Great now we have our custom profiles. 

## [](#header-2) Q1 : What is the CentOS version installed on the machine?

We can see that the file name is "Centos7.3.10.1062". 
Spoiler it is not the name of the version.
The real answer is on the wikipedia page of centos we can see that the name corresponds to the version 7.7.1908 of centos 

## [](#header-2) Q2 : There is a command containing a strange message in the bash history. Will you be able to read it?

Finally, let's start digging. 

Here is the command we will use 
python2 volatility/vol.py -f c73-EZDump/dump.mem --profile=LinuxCentos7_3_10_1062x64 linux_bash

- python2 volatility/vol.py = launch volatility. 
-f c73-EZDump/dump.mem = specify the file to analyze
--profile=LinuxCentos7_3_1062x64 = the created cutom profile 
- linux_bash = the command that allows to see the history of bash commands. 

![Q2](/assets/Q2-seized.jpg)

```
this is a comand in a CLI
```

![this is a pisture](/assets/Q3-seized.png)