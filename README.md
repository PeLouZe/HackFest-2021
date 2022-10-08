# HackFest-2021
HackFest 2021 Writeup for the Wireshark Track named HellCorp.     😈😈😈

**
Initial Thoughts : 


This challenge was really interesting and I think was something original that we don't usually see in most CTF, which is fun :D!
Also integrating wireless in a CTF is a pretty hard task but I think HackFest covered this topic in a nice way over the years!!!
**

### The Challenge
![wireshark1](https://user-images.githubusercontent.com/16509773/142954250-ab81ba94-3e54-4b19-a25a-25a3772ce99c.jpg)


### Digging HellCorp

The first thing that we have to help us is a PCAP wireshark file. At first opening we see that there is a ton of packets with 802.11 Protocol.
In a wireshark context, the 802.11 protocol is for the multiple applications of Wi-FI and multiple technologies around it.
https://en.wikipedia.org/wiki/IEEE_802.11

At first I have spent a few minutes trying to make sense of the packets. Looked at the length, the patterns, the name of the network and the multiple MAC Addresses involved.
![wire1](https://user-images.githubusercontent.com/16509773/142954746-31c9f768-5ba7-4edc-87e2-99118b544975.jpg)

I knew that there was not so much coming around for 802.11 packets. It's the radio layer of the Wi-Fi and it's mostly used for channel management and timing of the protocol, there is almost
no intelligible information in those packets. After wandering around for a bit of time I have noticed that the encryption used was WEP encryption.

WEP is as easy to crack as 1-2-3. So I told myself, why not give it a try, I have nothing to lose...
If you want to see what's involved in the WLAN communication you can go to the top menu under the Wireless section and  > WLAN Traffic.
![multiple info](https://user-images.githubusercontent.com/16509773/142954990-b4f2ad87-44ed-4e90-ae10-8306b6329fcc.jpg)

### Fight hells's fire with fire and fire up Kali🔥

A nice module from Kali is the Aircrack-NG module. This suite of tools integrated within kali is your go-to module for everything wi-fi. The module is packed with everything  we need to have fun with wifi.
Cracking multiple technologies, enabling monitor mode, WPS or other fun stuff.
One of the submodule from it is Airdecap-ng which does... exactly what we need! Take a PCAP and decrypt the packets.

One of the weakness in WEP is that the IVs used have a length too small which makes it easy to crack.
Some of the protocols part are also vulnerable and you don't even need to have a 'first handshake' to find the password. So let's give it a try...

It's easy to do with Aircrack.
We just have to ask it : *aircrack-ng hellcorp1.pcap*. It will work for like 3 seconds and then... we got it!
![uncrypt](https://user-images.githubusercontent.com/16509773/142957379-6651ec76-9b9e-4162-9057-18cf4bac1239.jpg)

### Decap Hell 
Then you just need to use the found hexkey with AirDecap and it will output a nicely decrypted wireshark file.
![fun](https://user-images.githubusercontent.com/16509773/142957653-0e7d3e84-1c71-4b2f-9bec-e45ccd34e0b2.jpg)

When you open the newly created file you notice that there is no more 802.11. Only normal ethernets packed with multiple protocols and intelligble content.
Make a little search in these to find the flag.

Hint 1 : look for a password.

Hint 2 : Look for packet #5264

### Escape from Hell - Flag-2
We now arrive to flag-2. Here is the description :

![wireshark2](https://user-images.githubusercontent.com/16509773/142958417-b254e923-3678-451c-aba0-fcada42755e5.jpg)

## Reversing-Hell : 
First, after reading carefully the description, we see that we need to find a specific HTML file on the PCAP file. Also, we will need to reverse part of the RockYou that contains the words hell.
First, it's easy to reverse our part-of-rockyou file.

Just use CyberChef with the function *Reverse*. You can copy the whole provided wordlist and paste it to cyberchef.
Then use the *Reverse* utility and use *Save Output To File*. We now have the right tool to crack our PCAP.

Looking at the same way we did for flag 1. *Wireless > WLAN Traffic*, we see that WPA is used for this specific communication.
Aircrack-NG will still be helpful for this, however, there is a caveat. 

In order to crack WPA2, you need some specific part of the 802.11 communication. These packets are EAPOL Packets. These eapol packets are basically a Handshake that the wireless devices
does at the beginning of the conversations. These are like a 'key-exchange'. This in WPA can be leveraged to crack the wifi password (but good luck getting it!).
Please, note that this handshake is composed of 4-packets and you need to all the 4-packets in order to crack it.

Thank god! In this pcap we have these...
![eapol](https://user-images.githubusercontent.com/16509773/142959177-3f2452cf-6539-444a-91a5-d7fd9c57ca1c.jpg)

Fortunately, I had already tested thoroughly with these kind of things in my life (fun nights!). However, you might need to do a bit of Googling about aircrack-ng in order to do this.
In aircrack, you have a functionnality for cracking using a wordlist. Since the wordlist is provided for flag2, we might use it (but don't forget to use the reversed version made at the beginning).

Let's try : *aircrack-ng hell2.pcap -w wordlist.txt*
It will work until like 5 seconds and then give you the key!
![cracked](https://user-images.githubusercontent.com/16509773/142960112-b49e776a-ba6c-4779-b26e-93795ea29052.jpg)

## Decrypt Satan
So... we now have the key, and if we look closely, we also have the Wi-Fi SSID.
That's all we need to decrypt the whole thing.
In wireshark you can decrypt wireless trafic using this function : 
*Edit* -> *Preferences* -> (on the left side, protocols section) select *IEEE 802.11*

Make sure to check : *Enable Decryption* then select 'Add-Keys'.
You can just select WPA-PWD, and enter the key we just found and the SSID in the good format.
![decrypticsatan](https://user-images.githubusercontent.com/16509773/142960536-a5d1518e-893b-4c0a-9347-19ae5f80f9af.jpg)

Upon doing this, you will notice that some packets are still in 802.11 format. It is normal as they are used for radio controlling the network. If you don't find proper ethernet packets, just scroll down.

## Trying to get to Heaven
We now need to find heaven. According to their description we can find it from the file '*how_to_escape_from_hell.html*'. 

At this point I needed some time to figure out what the communications were and see through the hell's smoke.
I have a usual routine while doing Wireshark CTF to find flags. Which is to find anomalies against the normal trafic.
So what do we have here... after few minutes I have finally found that : 



-We have lots of DNS Request but it doesn't look to be of any utility.

-We have like 29 FTP packets which are interesting because of their low amount of packets.

-We have some TCP packets which have great lengths in size.

-Searching for "HF-" doesn't find anything (no easy flag here!).



I have started by looking at the FTP packets. 
You can filter it with the *ftp* filter or just select an FTP packet and use *Follow TCP Stream*.
It looks like a .ZIP file backup is being moved.

![FITUP](https://user-images.githubusercontent.com/16509773/142961390-d2cc6f32-1419-40ce-9a71-3c8069dabf66.jpg)

An interesting finding here is that we don't see the file being transferred, however, we have a password *autobackup*.

## Backing up until we get in heaven
We will now need to find a .zip file called autobackup. After looking around and noticing the TCP Packets we can see that one Packet has the PK.. Magic bytes in it.
Let's take a look at packet #4618. 

It Starts with PK.. which means it's the beginning of a Zip archive.
If we do again *Follow* -> *TCP Stream* on that specific packet, we will be able to extract the whole thing.
To be honest, I had hard times doing this. I messed around with CyberChef, messed around with wireshark and couldn't find a proper way to extract it.

A friend of mine told me that I could just put it in *Raw* bytes and use the *Save AS* button, from the menu in opened packets (see image).
In this context, we can also see that the Zip file contains the 'How_to_escape_from_hell.html' (before we convert it to raw bytes) file that we are looking for.
![saveas](https://user-images.githubusercontent.com/16509773/142961966-406a96ec-3413-4509-8bc2-548581396922.jpg)

Let's see if we can have the great escape.
We can see that the zip file contains multiple files and folders. However, in order to open it, we need the password.
As a password I asked myself if there could be something else but I first tried 'autobackup' (since it was the FTP password) and it did work.

We would then navigate : *app/templates/How_to_escape_from_hell.html*. The open webpage find us with a FLAG ! :) 
![cantescape](https://user-images.githubusercontent.com/16509773/142964209-7f1f1750-8230-4345-984b-4131a7f8a085.jpg)

And then it tells you that you can't Friggin' escape from it GOD HELP ME I DID ALL THIS FOR NOTHING!😡 (except a total score of 400 points from this).😈 

This was a really nice CTF, thanks to all the organizers!!!! ❤

_PeLouZe 🔥🔥🔥


