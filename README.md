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

