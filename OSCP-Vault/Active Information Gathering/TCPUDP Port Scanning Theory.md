---
tags:
  - enumeration
  - nmap
---

# TCP/UDP Port Scanning Theory

Port scanning is the process of inspecting TCP or UDP ports on a remote machine with the intention of detecting what services are running on the target and what potential attack vectors may exist.

The simplest TCP port scanning technique, usually called CONNECT scanning, relies on the three-way TCP handshake mechanism. This mechanism is designed so that two hosts attempting to communicate can negotiate the parameters of the network TCP socket connection before transmitting any data.

In basic terms, a host sends a TCP SYN packet to a server on a destination port. If the destination port is open, the server responds with a SYN-ACK packet and the client host sends an ACK packet to complete the handshake. If the handshake completes successfully, the port is considered open.






UDP Scanning

> [!note]- Screenshot
> ```
> We can demonstrate this by running a TCP Netcat port scan on ports 3388-3390. We'll
> use the -w option to specify the connection timeout in seconds, as well as -z to specify
> zero-I/O mode, which is used for scanning and sends no data.
> 
> kaligcali:-$ nc -nvy -w 4 -2 192.168.50.152 3388-3390
> 
> (UNKNOWN) [192.168.50.152] 339@ (?) : Connection refused
> 
> (UNKNOWN) [192.168.508.152] 3389 (ms-wbt-server) open
> 
> (UNKNOWN) [192.168.50.152] 3388 (?) : Connection refused
> 
> sent @, revd 0
> Listing 28 - Using netcat to perform a TCP port scan
> ```


```sh
nc -nvv -w 1 -z 192.168.50.152 3388-3390
```


> [!note]- Screenshot
> ```
> Based on this output, we know that port 3389 is open, while connections on ports 3388
> ‘and 3390 have been refused. The screenshot below shows the Wireshark capture of
> this scan:
> 
> Lolaiesor7a3 i92.d60.s0.i52 oa.i60.3i0,2. Te 74 3900 ~ a8342 [Svm, Aon] Seqro Acke waneeso00 Le
> 
> ‘Figure 7: Wireshark cepture ofthe Netcat port scan
> 
> In this capture (Figure 17), Netcat sent several TCP SYN packets to ports 3390, 3389,
> and 3388 on packets 1, 3, and 7, respectively. Due to a variety of factors, including
> timing issues, the packets may appear out of order in Wireshark. We'll observe that the
> server sent a TCP SYN-ACK packet from port 3389 on packet 4, indicating that the port
> is open. The other ports did not reply with a similar SYN-ACK packet, and actively
> rejected the connection attempt via an RST-ACK packet. Finally, on packet 6, Netcat
> closed this connection by sending a FIN-ACK packet.
> ```


> [!note]- Screenshot
> ```
> Now that we have a good understanding of the TCP handshake and have examined how
> a TCP scan works behind the scenes, let's cover UDP scanning. Since UDP is stateless
> and does not involve a three-way handshake, the mechanism behind UDP port scanning
> is different from TCP.
> Let's run a UDP Netcat port scan against ports 120-123 on a different target. We'll use
> the only nc option we have not covered yet, -u, which indicates a UDP scan.
> 
> kaligcali:-$ nc -nv -u -z -w 1 192.168.50.149 120-123
> 
> (Uwaoun) [192.168.509.149] 123 (ntp) open
> 
> sting 29 - Using Netcat to perform a UDP port scan
> ```


```sh
nc -nv -u -z -w 1 192.168.50.149 120-123
```


> [!note]- Screenshot
> ```
> From the Wireshark capture, we'll notice that the UDP scan uses a different mechanism
> than a TCP scan.
> CT
> ‘Figura 18: Wireshark capture of a UDP Netcat port scan
> ‘As shown above, an empty UDP packet is sent to a specific port (packets 2, 3, 4, 6, and
> 8). If the destination UDP port is open, the packet will be passed to the application layer.
> The response received will depend on how the application is programmed to respond to
> ‘empty packets. In this example, the application sends no response. However, if the
> destination UDP port is closed, the target should respond with an ICMP port
> unreachable (as shown in packets 5, 7, and 9), sent by the UDP/IP stack of the target
> machine.
> ```
