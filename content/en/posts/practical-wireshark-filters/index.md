---
title: Practical Wireshark 🦈 Filters 
date: 2020-03-25
tags: [wireshark filters, wireshark filtreleri, wireshark for network analysis, wireshark for infected traffic, wireshark ctf]
description: I've listed the 20 WireShark filters that will be most useful for you. Enjoy your reading...
toc: false
---

We all know Wireshark, the popular and best packet analysis tool in its field, used by malware analysts to detect errors in C&C servers, Network and System Administrators. Some of us examine 100,000 packets one by one, while some of us make direct pinpointing with the legendary feature called FILTER. Of course, we did not write this article for the first option 🙂 I have compiled **20 Wireshark Filters** for you to find what you are looking for while doing network packet analysis.

1) `ip.addr == 192.168.1.1`
This filter presents us the packets with the corresponding IP in Source or Destination.

 

2) `ip.addr == 192.168.1.1 && ip.addr == 10.0.0.3`

It  has connected these two IP addresses with the AND conjunction. So Source or Destination has to have these two. It doesn't matter which is Destination and which is Source.

 

3) `ip.addr == 192.168.0.0/24`

If one of the 255 IP addresses at 192.168.0.x is present in the packets, it brings the relevant packet in front of us.

 

4)` ip.src == 192.168.1.16 && ip.dest == 10.0.0.12`

Filters packets with Source at 192.168.1.16 and Destination at 10.0.0.0.12.

 

5) `ip.addr != 192.168.1.16`

Filters all packets without the corresponding IP address.

 

6) `tcp.port == 8080`

Filters packets connecting to port 8080.

 

7) `tcp.port in {443 80 8443}`

It is a useful filter if you are looking for packets to which multiple TCP ports are connected.

 

8) `tcp.flags & 0x02`

Filters packets with TP flag 0x02. 0x02 corresponds to the SYN bit. Likewise, the filter tcp.flags.syn == 1 has the same meaning. Also this filter will show both SYN and SYN/ACK packets. If you want to show only SYN packets, you need to use the filter tcp.flags.syn == 1 && tcp.flags.ack == 0.

 

9) `tcp contains FLAG`

Filters the packet with FLAG string in TCP packets. It is obvious that it will be useful especially in CTF competitions 🙂

 

10)` http.request.uri == “https://fatihsensoy.com/”`

It allows you to filter packets sent or received to a specific website.


 

11) `http.host matches “microsoft\.(org|com|net|tr)”`

Filters if there is an outgoing or incoming package related to the com, org, net or tr domains of the Microsoft name.

 

12) `http.request.method == “POST”`

Filters all packets in a POST request.

 

13) `http.host == “www.google.com”`

It presents us the packages where the specified URL is hosted. There is one point to pay attention to. The URL must be written in full. In other words, if there is www at the beginning of the website, if it is left incomplete, correct filtering will not be done.

 

14) `http.response.code == 200`

It filters the packets with Response code 200 (i.e. successful connection) returned as a result of the request. You can also filter all HTTP response codes such as 404 (not found), 403 (unauthorized access/prohibited).

 

15) `http.content.type == “audio/mpeg”`

Filters HTTP packets that contain audio files of the mpeg type. You can easily filter other content as well. For example "image/jpeg".

 

16)` udp.port == 69`

Filters packets sent or received on the corresponding UDP port.

 

17) `wlan.ssid == Isyeri_Wifi`

It provides us with the packets sent and received to and from the SSID of the respective WIFI.

 

18) `wlan.addr == 00:1c:12:dd:ac:4d`

Filters packets to and from the corresponding WLAN MAC address.

 

19) `icmp.type == 8 && imcp.type == 0`

This filter, set to ICMP echo 8, echo reply 0, only filters ICMP Requests.

 

20) `icmp`

There are also single uses like this. It directly filters all packets with the relevant protocol without going into too much detail. As you can guess, there are also single filters such as http, dns, tcp, udp.


I have compiled a short list of Wireshark filters that I think can be useful for you. If there is a filter that you consider important, you can share it in the comments and help me update the article 🙂