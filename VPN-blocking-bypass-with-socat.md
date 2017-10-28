# __VPN blocking bypass with socat__

This is a **quick** and __dirty__ way to bypass VPN restrictions in some of the countries that block VPN connections.

## Requirements of the solution:

1. Free or cheap solution (no one wants to pay any money :smiley: ).  

The idea is simple, we want to make a tunnel of a white listed protocol (for ex: SSL/TLS) which is hard to block or do DPI on it. Then encapsulating our VPN traffic inside it.

While searching I came cross a Reddit [discussion](https://redd.it/73zc61) which had a lot of ideas and solutions to the problem but it didn’t suit me nor my needs.

## What you will need is:

1. AWS instance (any VPS)
2. SOCAT
3. Patience


I used to play on [HackTheBox](https://www.hackthebox.eu/), so this will be the example I'm gonna talk about.

1. For the AWS server you can find a lot of tutorials explaining and walking you through the process which costed me 1 $ (till now).

2. From the man page of socat:

> Socat is a command line based utility that establishes two bidirectional byte streams and transfers data between them. Because the streams can be constructed from a large set of different types of data sinks and sources (see address types), and because lots of address options may be applied to the streams, socat can be used for many different purposes.

What I really liked about socat apart from its great ability to manipulate sockets is the ability to handle SSL/TLS traffic which will talk about it later.

---

Enough said, let’s start circumventing the measures made to block the VPN.

We have the following topology:

insert image here (still .. too lazy to fix libreoffice .. maybe later)

Host=Kali
VPS=Server
VPN=HTB

We need to implement the following:

__Kali__ <==== SSL/TLS encrypted VPN traffic =====> __Server__ <===== VPN traffic =====> __HTB__

The changes done on the __Kali__ box:

Make a self-signed certificate and copy it to the Server.
``` #> openssl req -x509 -nodes -days 365 -newkey rsa:2048 -out enegma.crt -keyout enegma.key ```

![openssl cert](https://3n39m4.github.com/images/vpn/openssl cert.png)


Edit the connection pack file to point to your localhost (in my case called enegma.ovpn).

![editing](https://3n39m4.github.com/images/vpn/enegma.ovpn-edit.png)


Set up a socat listener and forwarder.
``` socat UDP-LISTEN:1337 OPENSSL:########.eu-west-1.compute.amazonaws.com:443,reuseaddr,pf=ip4,fork,cert=/root/tmp/enegma.pem,cafile=/root/tmp/enegma.pem,verify=0 ```

Changes done on the Server:

Set up a socat listener
``` socat OPENSSL-LISTEN:443,reuseaddr,pf=ip4,fork,cert=/root/enegma.pem,cafile=/root/enegma.pem,verify=0 UDP:edge-eu-free-1.hackthebox.eu:1337 ```

From the Kali host we make a connection to HTB as usual.

![proof 1](https://3n39m4.github.com/images/vpn/proof1-sec.png)

![proof 2](https://3n39m4.github.com/images/vpn/proof2.png)

Now have fun :smiley:

Remember this is a quick and dirty way to do, but I’ll update this post later with the best practice regarding encryption and certificate creation and use.

I’d like to here your feedback. Any advice would be very appreciated.
