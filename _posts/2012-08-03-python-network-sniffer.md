---
layout: default
---

To extend my knowledge of network protocols I wrote a simple sniffer in Python
which uses Unix RAW Sockets to capture all types of Ethernet Frames.
It could decode Ethernet Frames, IP Packages, ICMP Packages and
TCP/UDP Segments.
It also generates a Pcap capture file so the whole traffic could be viewed with
some external tools later. Have a look at the
[Source Code](https://github.com/koehlma/snippets/blob/master/python/network/sniffer.py)
if you want to know more.
