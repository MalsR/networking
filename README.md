Small project I created while I was learning about networking, i.e. fundamentals, programms you can use to diagnose networks and understanding the layers.  

### Setting up [vagrant](https://www.vagrantup.com/downloads.html) box

vagrant init ubuntu/trusty64
vagrant up
vagrant ssh

### *ip*
ip addr
ip addr show eth0
ip route help
ip route list

### *ping*
ping -c3 8.8.8.8

### *host*
host -t aaaa google.com 

host 8.8.8.8 #resolve google dns
`8.8.8.8.in-addr.arpa domain name pointer google-public-dns-a.google.com.`

host -t mx udacity.com

### lsof *L*i*s*t *O*pen *F*iles
List open files and in this case show services that are listening or connected to. 

### *traceroute*
combining `ping` and `host` as a network diagnostic tool.
traceroute www.google.com #by default 30 hops
traceroute --max-hops=50 www.google.com #change max hops to 50 with each hop sending 3 packets by default

###   *tcpdump*
sudo tcpdump -n -c5 -i eth0 port 22 #5 packets captured

### [netcat](https://linux.die.net/man/1/nc)
Simple program to talk to internet services, forms a thin wrapper around tcp. Lower layer tool than curl for e.g. http://nc110.sourceforge.net/

#### Examples and tricks

- Using netcat to talk to google and get an HTTP response back.
  Separationg of Layers. Netcat does not know anything about talking to a server in HTTP or making an HTTP request but by using `printf` you can make a HTTP request and this gets sent as a string over the wire to an internet service. The server at the receiving end will send a response (HTTP) as the request looks similar to or looks like a HTTP request. 
  
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

Below command gives us a [302](https://en.wikipedia.org/wiki/HTTP_302) since google is telling us to redirect to www.google.co.uk (in localtion). A Browser will make a subsequent call to the new location provided in the response. 

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
If I use a brower then the any modern browser will redirect back to www.bbc.co.uk.




  
Lookup
- demonstrate HTTP verbs with netcat
- server algorithms for handling multiple requests
- reset packet/RST
