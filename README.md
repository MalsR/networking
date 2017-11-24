Small project I created while I was learning about networking, i.e. fundamentals, programs you can use to diagnose 
networks and understanding the layers.  

### Setting up [vagrant](https://www.vagrantup.com/downloads.html) box
a safe black box environment to change and playaround.
```
vagrant init ubuntu/trusty64
vagrant up
vagrant ssh
```
### Networking Tools
Bunch of handy tools to diagnose those packets over rusty wires!

#### *ip*
ip addr
ip addr show eth0
ip route help
ip route list

#### *ping*
ping -c 3 8.8.8.8
ping -c 7 google.com

#### *host*
host -t aaaa google.com 

host 8.8.8.8 #resolve google dns
`8.8.8.8.in-addr.arpa domain name pointer google-public-dns-a.google.com.`

host -t mx udacity.com

#### lsof *L*i*s*t *O*pen *F*iles
List open files and in this case show services that are listening or connected to. 

### *traceroute*
combining `ping` and `host` as a network diagnostic tool.
traceroute www.google.com #by default 30 hops
traceroute --max-hops=50 www.google.com #change max hops to 50 with each hop sending 3 packets by default

#### *tcptraceroute* 

#### *tcpdump*
sudo tcpdump -n -c5 -i eth0 port 22 #5 packets captured

- Capturing ICMP packets by using ping to ping a google server `8.8.8.8`. Using tcpdump to list traffic for a host
 ```
 $ ping -c 3 8.8.8.8
 PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
 64 bytes from 8.8.8.8: icmp_seq=1 ttl=60 time=25.9 ms
 64 bytes from 8.8.8.8: icmp_seq=2 ttl=60 time=28.1 ms
 64 bytes from 8.8.8.8: icmp_seq=3 ttl=60 time=25.5 ms

 //Using `-n` so it does not convert the ip address to host name 
 $ sudo tcpdump -n host 8.8.8.8
 tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
 listening on wlp3s0, link-type EN10MB (Ethernet), capture size 262144 bytes
 22:59:28.655066 IP 192.168.0.3 > 8.8.8.8: ICMP echo request, id 10095, seq 1, length 64
 22:59:28.681005 IP 8.8.8.8 > 192.168.0.3: ICMP echo reply, id 10095, seq 1, length 64
 22:59:29.656265 IP 192.168.0.3 > 8.8.8.8: ICMP echo request, id 10095, seq 2, length 64
 22:59:29.684321 IP 8.8.8.8 > 192.168.0.3: ICMP echo reply, id 10095, seq 2, length 64
 22:59:30.657713 IP 192.168.0.3 > 8.8.8.8: ICMP echo request, id 10095, seq 3, length 64
 22:59:30.683113 IP 8.8.8.8 > 192.168.0.3: ICMP echo reply, id 10095, seq 3, length 64

 //Without -n and resolves ip address to a name
 $ sudo tcpdump host 8.8.8.8
 tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
 listening on wlp3s0, link-type EN10MB (Ethernet), capture size 262144 bytes
 22:59:07.694527 IP Boeing777ER > google-public-dns-a.google.com: ICMP echo request, id 9348, seq 1, length 64
 22:59:07.721384 IP google-public-dns-a.google.com > Boeing777ER: ICMP echo reply, id 9348, seq 1, length 64
 22:59:08.695659 IP Boeing777ER > google-public-dns-a.google.com: ICMP echo request, id 9348, seq 2, length 64
 22:59:08.721207 IP google-public-dns-a.google.com > Boeing777ER: ICMP echo reply, id 9348, seq 2, length 64
 22:59:09.697549 IP Boeing777ER > google-public-dns-a.google.com: ICMP echo request, id 9348, seq 3, length 64
 22:59:09.724304 IP google-public-dns-a.google.com > Boeing777ER: ICMP echo reply, id 9348, seq 3, length 64

```

- Questions, Running the same results above on one VM resolved in IPv6 hostnames while on another resolved to using IPv4

#### [netcat](https://linux.die.net/man/1/nc)
Simple program to talk to internet services, forms a thin wrapper around tcp. Lower layer tool than curl for 
e.g. http://nc110.sourceforge.net/

##### Examples and tricks

- Using netcat to talk to google and get an HTTP response back.
  Separationg of Layers. Netcat does not know anything about talking to a server in HTTP or making an HTTP request but 
  by using `printf` you can make a HTTP request and this gets sent as a string over the wire to an internet service. 
  The server at the receiving end will send a response (HTTP) as the request looks similar to or looks like a HTTP request. 
  
```
printf 'HEAD / HTTP/1.1\r\nHost: www.google.co.uk\r\n\r\n' | nc www.google.com 80

HTTP/1.1 200 OK
Date: Thu, 02 Nov 2017 21:54:49 GMT
Expires: -1
Cache-Control: private, max-age=0
Content-Type: text/html; charset=ISO-8859-1
P3P: CP="This is not a P3P policy! See g.co/p3phelp for more info."
Server: gws
X-XSS-Protection: 1; mode=block
X-Frame-Options: SAMEORIGIN
Set-Cookie: 1P_JAR=2017-11-02-21; expires=Thu, 09-Nov-2017 21:54:49 GMT; path=/; domain=.google.co.uk
Set-Cookie: NID=116=hag44b-d9Ig_A7Fc7Hilf7gWUaN1Kdhzb5-lEfzytD3vssn4leF9pe0VWQZd9q7SI3vI84Xi3rvJntiw4cDX8ml1tGOVigWlR_roldo55hmTIHiFLS01pfwHUE2vKbha; expires=Fri, 04-May-2018 21:54:49 GMT; path=/; domain=.google.co.uk; HttpOnly
Transfer-Encoding: chunked
Accept-Ranges: none
Vary: Accept-Encoding
```

Below command gives us a [302](https://en.wikipedia.org/wiki/HTTP_302) since google is telling us to redirect to 
www.google.co.uk (in localtion). A Browser will make a subsequent call to the new location provided in the response. 

```
printf 'HEAD / HTTP/1.1\r\nHost: www.google.com\r\n\r\n' | nc www.google.com 80

HTTP/1.1 302 Found
Cache-Control: private
Content-Type: text/html; charset=UTF-8
Referrer-Policy: no-referrer
Location: http://www.google.co.uk/?gfe_rd=cr&dcr=0&ei=G5T7WfzIIK-GtgfEwp24Bg
Content-Length: 271
Date: Thu, 02 Nov 2017 21:54:35 GMT
```
Make a second call using the new location. 

```
printf 'GET /?gfe_rd=cr&dcr=0&ei=G5T7WfzIIK-GtgfEwp24Bg HTTP/1.1\r\nHost: www.google.co.uk\r\n\r\n' | nc www.google.com 80

HTTP/1.1 200 OK
Date: Tue, 07 Nov 2017 12:49:53 GMT
Expires: -1
Cache-Control: private, max-age=0
Content-Type: text/html; charset=ISO-8859-1
P3P: CP="This is not a P3P policy! See g.co/p3phelp for more info."
Server: gws
X-XSS-Protection: 1; mode=block
X-Frame-Options: SAMEORIGIN
Set-Cookie: 1P_JAR=2017-11-07-12; expires=Thu, 07-Dec-2017 12:49:53 GMT; path=/; domain=.google.co.uk
Set-Cookie: NID=116=qEXpwFMtu8-TfzzuBTMwe704__TAFV1KKDAUUnnRtS3IRj8V357WfupapRAeH7heLywkP7P6KpWgow6oPLLHVCRT6N9MerULWXu9hKDXcxzK9kxUaQ_voO6DdRhuoLZg; expires=Wed, 09-May-2018 12:49:53 GMT; path=/; domain=.google.co.uk; HttpOnly
Accept-Ranges: none
Vary: Accept-Encoding
Transfer-Encoding: chunked

7a52
<!doctype html><html itemscope="" itemtype="http://schema.org/WebPage" lang="en-GB"><head><meta content="text/html; charset=UTF-8" http-equiv="Content-Type"><meta content="/logos/doodles/2017/celebrating-pad-thai-5712000859504640-l.png" itemprop="image"><meta content="Celebrating Pad Thai" property="twitter:title"><meta content="Celebrating Pad Thai! #GoogleDoodle" property="twitter:description"><meta content="Celebrating Pad Thai! #GoogleDoodle" property="og:description"><meta content="summary_large_image" property="twitter:card"><meta content="@GoogleDoodles" property="twitter:site"><meta content="https://www.google.com/logos/doodles/2017/celebrating-pad-thai-5712000859504640-l.png" property="twitter:image"><meta content="https://www.google.com/logos/doodles/2017/celebrating-pad-thai-5712000859504640-l.png" property="og:image"><meta content="500

```
- Using an undefined HTTP verb or request when talking to an internet service

```
printf 'HEAD-MALSR / HTTP/1.1\r\nHost: www.google.co.uk\r\n\r\n' | nc www.google.co.uk 80

HTTP/1.1 405 Method Not Allowed
Content-Type: text/html; charset=UTF-8
Referrer-Policy: no-referrer
Content-Length: 1595
Date: Sat, 11 Nov 2017 12:21:37 GMT

<!DOCTYPE html>
<html lang=en>
  <meta charset=utf-8>
  <meta name=viewport content="initial-scale=1, minimum-scale=1, width=device-width">
  <title>Error 405 (Method Not Allowed)!!1</title>
  <style>
    *{margin:0;padding:0}html,code{font:15px/22px arial,sans-serif}html{background:#fff;color:#222;padding:15px}body{margin:7% auto 0;max-width:390px;min-height:180px;padding:30px 0 15px}* > body{background:url(//www.google.com/images/errors/robot.png) 100% 5px no-repeat;padding-right:205px}p{margin:11px 0 22px;overflow:hidden}ins{color:#777;text-decoration:none}a img{border:0}@media screen and (max-width:772px){body{background:none;margin-top:0;max-width:none;padding-right:0}}#logo{background:url(//www.google.com/images/branding/googlelogo/1x/googlelogo_color_150x54dp.png) no-repeat;margin-left:-5px}@media only screen and (min-resolution:192dpi){#logo{background:url(//www.google.com/images/branding/googlelogo/2x/googlelogo_color_150x54dp.png) no-repeat 0% 0%/100% 100%;-moz-border-image:url(//www.google.com/images/branding/googlelogo/2x/googlelogo_color_150x54dp.png) 0}}@media only screen and (-webkit-min-device-pixel-ratio:2){#logo{background:url(//www.google.com/images/branding/googlelogo/2x/googlelogo_color_150x54dp.png) no-repeat;-webkit-background-size:100% 100%}}#logo{display:inline-block;height:54px;width:150px}
  </style>
  <a href=//www.google.com/><span id=logo aria-label=Google></span></a>
  <p><b>405.</b> <ins>That’s an error.</ins>
  <p>The request method <code>HEAD-MALSR</code> is inappropriate for the URL <code>/</code>.  <ins>That’s all we know.</ins>  
```
- Using to scan ports. Ports upto 1023 reserved for superuser/root while 1024-65535 available for any other programm. 

```
nc -l localhost 1024 /listen on port 1024 on different terminal
nc -zv localhost 1024-1028                                                             

Connection to localhost 1024 port [tcp/*] succeeded!
nc: connect to localhost port 1025 (tcp) failed: Connection refused
nc: connect to localhost port 1026 (tcp) failed: Connection refused
nc: connect to localhost port 1027 (tcp) failed: Connection refused
nc: connect to localhost port 1028 (tcp) failed: Connection refused

```

- Acting as a webserver. Using netcat to act as a webserver and responding with a HTTP response.

`printf 'HTTP/1.1 302 Moved\r\nLocation: https://www.bbc.co.uk' | nc -l 4445`

```
curl -v localhost:4445
...
HTTP/1.1 302 Moved

//Once response received by nc it quits
GET / HTTP/1.1
Host: localhost:6665
User-Agent: curl/7.47.0
Accept: */*
```
If I use a browser then the any modern browser will redirect back to www.bbc.co.uk.


### DNS (Domain Name System)
Distributed directory that holds information about internet services. 

- Used as a security mechanism for HTTP including SSL encryption and cookie privacy
- Role of resolver, essentially DNS client in your operating system that tells which server to use when performing DNS lookups. 

Reads on DNS
- http://compsec101.antibozo.net/papers/dnssec/dnssec.html

#### Examples
- Resolving a host name using ping
```
$ ping -c 3 yahoo.com
PING yahoo.com (206.190.39.42) 56(84) bytes of data.
64 bytes from yahoo.com (206.190.39.42): icmp_seq=1 ttl=51 time=162 ms
64 bytes from yahoo.com (206.190.39.42): icmp_seq=2 ttl=51 time=163 ms
64 bytes from yahoo.com (206.190.39.42): icmp_seq=3 ttl=51 time=164 ms

--- yahoo.com ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 162.903/163.802/164.688/0.865 ms
```
```
$ sudo tcpdump -n port 53
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on wlp3s0, link-type EN10MB (Ethernet), capture size 262144 bytes
23:06:58.142917 IP 192.168.0.3.47690 > 192.168.0.1.53: 13199+ A? yahoo.com. (27)
23:06:58.176361 IP 192.168.0.1.53 > 192.168.0.3.47690: 13199 3/0/0 A 206.190.39.42, A 98.139.180.180, A 98.138.252.38 (75)
23:06:58.341031 IP 192.168.0.3.47690 > 192.168.0.1.53: 50292+ PTR? 42.39.190.206.in-addr.arpa. (44)
23:06:58.344163 IP 192.168.0.1.53 > 192.168.0.3.47690: 50292- 1/0/0 PTR yahoo.com. (67)
23:06:58.871361 IP 192.168.0.3.47690 > 192.168.0.1.53: 8666+ A? daisy.ubuntu.com. (34)
```

#### Types of DNS records

#### Host
Simple service for looking up records in DNS.

```
//Resolve host google.com and get A record for google.com
host -ta google.com
```

#### IPv4
Dotted quads, represented as 4 bytes or 4 octets with each quad range of `0-255`.
- So a v4 IP address has a bit width of 32 bits (4 bytes) or 32 bits wide i.e. 32 bits are used to represent this value in a packet.
- So why is the highest TCP port number 65535? The space allocated or the bit width for a port number in a packet is 16 
bits or 2 bytes therefore the max port number we can fit in a packet is 65535!
- We write it as dotted quads purely as a convention since it's easy to read as opposed to reading a 32-bit numerical value.

##### Adress exhaustion
NAT - Network Address Translation, configuration in the router that maps internal private network addresses to internet services/public IP addreses. 
A workaround as a result of ipv4 address exhaustion. So does this mean with ipv6 we do not need NAT?

Reads
- https://en.wikipedia.org/wiki/IPv4
- Private address allocation https://tools.ietf.org/html/rfc1918
- Reserved IP addresses https://en.m.wikipedia.org/wiki/Reserved_IP_addresses

  
Lookup
- demonstrate HTTP verbs with netcat
- server algorithms for handling multiple requests
- reset packet/RST
- network link
- multi-link path (include and check bits per second)
