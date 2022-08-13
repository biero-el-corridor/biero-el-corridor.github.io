---
layout: post
title: Cyberdefender | Acoustic Writeup
published: true
---

## [](#header-2)Foreword. 
This is a writeup of the Acoustic challenge. It come from the collective mind of the [Honeynet Project](https://www.honeynet.org/). 

## [](#header-2)Introductions.
The objective is to challenge and analyze the network logs (in addition to the logs of a tool that we will see later) of a [VoIP](https://fr.wikipedia.org/wiki/Voix_sur_IP) telephone conversation

The tools used will be the following 
- [Wireshark](https://www.wireshark.org/)
- Notepad++ (d2VsY29tZSB0byB0aGUgcmljZWZpZWxkIG1m)

## [](#header-2) Q1 : What is the transport protocol being used?

To answer this question let's take a look at wireshark and see which protocols are used and which are inherent to VoIP.

We have RTP and SIP. 

These two protocols are based on the UDP transport protocol, as explained in their [wikipedia] page (https://fr.wikipedia.org/wiki/Real-time_Transport_Protocol). 

Awnser: UDP

## [](#header-2) Q2 : The attacker used a bunch of scanning tools that belong to the same suite. Provide the name of the suite.

For this one, I wasn't very creative, I just started reading articles about VoIP hacking and the protocols named in question 1, and I came across this [article](https://hakin9.org/voip-hacking-techniques/)

## [](#header-2) Q3 : What is the User-Agent of the victim system? 

this question is a bit more complex. we already talk about User-Agent. 

So in your head it's ""HTTP"", yes but no. 
by reading the wikipedia page of [SIP](https://fr.wikipedia.org/wiki/Session_Initiation_Protocol), we realize that SIP used HTTP to build its logics.
So SIP takes the error code, and the naming of HTTP. 

SIP has also user agents. And as we are in a VoIP challenge we don't talk about HTTP user agent, but about SIP user agent. 

Taking this into account we look for a user agent in the UDP protocol, then we see that the victim is 172.25.105.40 because it is the one who receives the imput. 

Awnser: Asterisk PBX 1.6.0.10-FONCORE-r40

## [](#header-2) Q4 : Which tool was only used against the following extensions: 100,101,102,103, and 111?

Start by listing the different scripts of the tool we have defined as the one used on the victim [SIPvicious](https://github.com/EnableSecurity/sipvicious/tree/master/sipvicious). 

There are 5 in all. 

This is not svreport.py because we are talking about specific extensions and report all extensions. 

It's not svmap.py either for the same reason as svreport.py (and svcrash.py depends on svmap, so it's not him).

And svwar.py scans PaBXs SIPs, and we don't see any on the logs, nor in the network capture. 

Awnser: svcrack.py 

## [](#header-2) Q5 : Which extension on the honeypot does NOT require authentication?

Using the concept of SIP based on the HTTP protocol, we know that the OK access code is 200 and we are looking for an OK access for a USER who has not registered a password.

So we apply the filter "udp contains 200" in wireshark. 

We forward the UDP flow of the first one and by chance we see that it's the one that receives an access without putting anything, not even a User-Agent. 

![the auth free sip](/assets/acoustic/Q5cap1.jpg)

Awnser: 100

## [](#header-2) Q6 : How many extensions were scanned in total?

For this question I tried to be pragmatic. we know that the attacker has used an enumeration tool. we have the log of this enumeration. we also know that he enumerates one user at a time with the request REGISTER sip:.

So the thing to do is to count the number of times he makes this request (removing duplicates or empty registers).

Here is the python code. 
```py
with open("log.txt", "r") as file: 
    with open("registe.txt", "a") as to_write:
        lines = file.readlines()
        line = ""
        for line in lines: 
            if "REGISTER sip:" in line:
                print(line,end="")
                to_write.write(line)
```
we remove the obvious mistakes like. 

REGISTER sip:honey.pot.IP.removed SIP/2.0

REGISTER sip:honey.pot.IP.removed;transport=UDP SIP/2.0

And tadaaaa count the number of line. 
Awnser: 2652

 	

## [](#header-2) Q7 : There is a trace for a real SIP client. What is the corresponding user-agent? (two words, once space in between)

Here we are talking about the "log.txt" file. 
If we look at the file we can see that there is a user agent on it. 

"User-Agent: friendly-scanner

Looking a little bit more towards the end of the file we can see another user agent. 

"User-Agent: Zoiper rev.6751"

Awnser: Zoiper rev.6751

## [](#header-2) Q8 : Multiple real-world phone numbers were dialed. Provide the first 11 digits of the number dialed from extension 101?

The 1st free hint for this challenge will help me. 
Google "SIP requests name
This [site](https://www.3cx.com/pbx/sip-methods/) explains the different methods used to send instructions. 

Here we realize that it is the INVITE option that allows to establish a session and therefore to register the phone number of the sesisons. 

once this in mind we can look at the RFC that talks about dialing numbers
https://datatracker.ietf.org/doc/html/rfc4967
this RFC tells us that the dialed numbers are preceded by the command "sip:". 

By cumulating the information we will look for the command "INVITE sip:" in the log file.

Awnser: 00112524021


## [](#header-2) Q9 : What are the default credentials used in the attempted basic authentication? (format is username:password) 

We will return here in a dimensions a little more web in this questions.

The fact is that basic auth are generally encoded in base 64 and as we are talking about autentifications, we are talking about POST request. and the field of such request is in Auto

we only have to filter in wireshark these criteria.  

[wireshark auth filter](/assets/acoustic/Q9cap1.jpg)

them apply follow TCP on the first packet and you will see the awnser

Awnser: maint:password

## [](#header-2) Q10 : Which codec does the RTP stream use? (3 words, 2 spaces in between) 

The complicated part of this exercise is understanding that the codec corresponds to the payload-type field in rtp packets that carry information. 

This can be understood between the lines by carefully reading the RTP protocol wikipedia page

then wireshark filter by rtp and go find that payload-type field. 

Awnser:  ITU-T G.711 PCMU 

## [](#header-2) Q12 : How long is the sampling time (in milliseconds)?

We are looking for the sample rate (SR) of this communication. If we know the SR, we can divide 1 by the SR and find the sampling level in seconds.

By searching on the internet we can see that the SR can ve found in the following wireshark options field. 

wireshark -> telephonie -> VoIP Call -> play stream

Awnser: 0.125

## [](#header-2) Q12 :What was the password for the account with username 555?


we return to the HTTP world with this question. we look for the password of extension 555. 

For this we can apply the following filtering method in wireshark. 

wireshark -> tcp contains 555 -> 200 OK -> search for 555 -> found secret

Awnser: 1234

## [](#header-2) Q13  :Which RTP packet header field can be used to reorder out of sync RTP packets in the correct sequence?

By doing some google dorks || RTP packet header "timestamp" || whe found this [website](https://www.cs.columbia.edu/~hgs/rtp/faq.html). by searching the word "synchronized" we found that. 

```txt
RFC 3550 specifies one media-timestamp in the RTP data header and a mapping between such timestamp and a globally synchronized clock, carried as RTCP timestamp mappings.
```

Awnser: timestamp

## [](#header-2) Q13  : The trace includes a secret hidden message. Can you hear it?

by reading the wireshark documentations about how to listen a VoIP call we found this options path. 

wireshark -> telephonie -> VoIP Call -> play stream -> play button

Awnser: mexico