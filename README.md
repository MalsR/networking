## Setting up vagrant box

vagrant init ubuntu/trusty64
vagrant up
vagrant ssh

*ip*
ip addr
ip addr show eth0
ip route help
ip route list

*ping*
ping -c3 8.8.8.8

*host*
host -t aaaa google.com 

host 8.8.8.8 #resolve google dns
`8.8.8.8.in-addr.arpa domain name pointer google-public-dns-a.google.com.`

host -t mx udacity.com

*traceroute*
combining `ping` and `host` as a network diagnostic tool.
traceroute www.google.com #by default 30 hops
traceroute --max-hops=50 www.google.com #change max hops to 50 with each hop sending 3 packets by default

*tcpdump*
sudo tcpdump -n -c5 -i eth0 port 22 #5 packets captured

### netcat
Simple program to talk to internet services, forms a thin wrapper around tcp. Lower layer tool than curl for e.g. 

- Using netcat to talk to google and get an HTTP response back.

  Separationg of Layers. Netcat does not know anything about talking to a server in HTTP (forming HTTP requests) but by using `printf` you can format HTTP Header information and send this string over the wire to an internet service. The server at the receiving end will send a response (HTTP). 
  
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


printf 'HEAD / HTTP/1.1\r\nHost: www.google.com\r\n\r\n' | nc www.google.com 80
HTTP/1.1 302 Found
Cache-Control: private
Content-Type: text/html; charset=UTF-8
Referrer-Policy: no-referrer
Location: http://www.google.co.uk/?gfe_rd=cr&dcr=0&ei=G5T7WfzIIK-GtgfEwp24Bg
Content-Length: 271
Date: Thu, 02 Nov 2017 21:54:35 GMT


Lookup reset packet/RST
