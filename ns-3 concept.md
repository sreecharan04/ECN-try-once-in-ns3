### WHAT IS CONGESTION??

      congestion control ensures that the sender does not overflow the network. Comparing to the flow control
      technique where the flow control mechanism ensures that the source host does not overflow the destination 
      host.It ensures that the capability of the routers along the path does not become overflowed. 


### ECN (Explict Congestion Notification):

      Explicit Congestion Notification (ECN) is an extension to the Internet Protocol and to the Transmission
      Control Protocol. ECN allows end-to-end notification of network congestion without dropping packets. 
      ECN is an optional feature that may be used between two ECN-enabled endpoints when the underlying network 
      infrastructure also supports it.

      Conventionally, TCP/IP networks signal congestion by dropping packets. When ECN is successfully negotiated,
      an ECN-aware router may set a mark in the IP header instead of dropping a packet in order to signal impending 
      congestion. 
      The receiver of the packet echoes the congestion indication to the sender, which reduces its transmission 
      rate as if it detected a dropped packet.
   
      The Negotiation of ECN involves only TCP headers. It is as follows
      
![ecn-diagram-1](https://user-images.githubusercontent.com/43876863/47962282-8e9d4380-e040-11e8-93b9-c6b43ac949a0.jpg)


### ECN-aware Routers:
          
        Explicit Congestion Notification is inherently coupled with the idea of Active Queue Management.The 
        primary goal of AQM algorithms is to allow network operators simultaneously to achieve high throughput
        and low average delay by detecting incipient congestion.  This is achieved by sending appropriate 
        indications to the endpoints before the queue overflows.  However, the method of informing sources of
        congestion is not limited  to dropping  packets, as is the case with non-AQM-enabled FIFO queues.Instead, 
        AQM-enabled routers can mark packets during congestion by setting the ECN bit in the packets’ header.

### ECN operations on IP:

       ECN uses the two least significant (right-most) bits in the IPv4 or IPv6 header to encode four different 
       code points:

    00 – Non ECN-Capable Transport, Non-ECT.
    10 – ECN Capable Transport, ECT(0)
    01 – ECN Capable Transport, ECT(1)
    11 – Congestion Encountered, CE.

      When both endpoints support ECN they mark their packets with ECT(0) or ECT(1). If the packet traverses an
      active queue management (AQM) queue (e.g., a queue that uses random early detection (RED)) that is 
      experiencing congestion and the corresponding router supports ECN, it may change the code point to CE 
      instead of dropping the packet. This act is referred to as “marking” and its purpose is to inform the
      receiving endpoint of impending congestion. At the receiving endpoint, this congestion indication is 
      handled by the upper layer protocol (transport layer protocol) and needs
      to be echoed back to the transmitting node in order to signal it to reduce its transmission rate.

     Because the CE indication can only be handled effectively by an upper layer protocol that supports it, ECN 
     is only used in conjunction with upper layer protocols, such as TCP, that support congestion control and 
     have a method for echoing the CE indication to the transmitting endpoint. 
     

### Operation of ECN with TCP

       TCP supports ECN using three flags in the TCP header. The first one, the Nonce Sum (NS), is used to
       protect against accidental or malicious concealment of marked packets from the TCP sender.

        >>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
       The other two bits are used to echo back the congestion indication (i.e. signal the sender to 
       reduce the amount of information it sends) and to acknowledge that the congestion-indication echoing
       was received.
        These are the ECN-Echo (ECE) and Congestion Window Reduced (CWR) bits.
        <<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<

       Use of ECN on a TCP connection is optional, for ECN to be used, it must be negotiated at connection 
       establishment by including suitable options in the SYN and SYN-ACK segments.This is called ECN+

       When ECN has been negotiated on a TCP connection, the sender indicates that IP packets that carry 
       TCP segments of that connection are carrying traffic from an ECN Capable Transport by marking them with
       an ECT code point. This allows intermediate routers that support ECN to mark those IP packets with the 
       CE code point instead of dropping them in order to signal impending congestion.

       Upon receiving an IP packet with the Congestion Experienced code point, the TCP receiver echoes back 
       this congestion indication using the ECE flag in the TCP header. When an endpoint receives a TCP segment
       with the ECE bit it reduces its congestion window as for a packet drop. It then acknowledges the
       congestion indication by sending a segment with the CWR bit set.

       A node keeps transmitting TCP segments with the ECE bit set until it receives a segment with the CWR bit 
       set. 
       
  ![slide_10](https://user-images.githubusercontent.com/43876863/47962259-277f8f00-e040-11e8-8156-d7c8879f036a.jpg)
  
       After negotiation the transmission(ip and tcp bits) is as follows:
       
      
 ![datapackets-ecn](https://user-images.githubusercontent.com/43876863/47962410-64e51c00-e042-11e8-8365-fffa33bdf7d1.jpg)


 ### WHAT IS ECN+ ??

          [RFC3168] specifies setting the ECN-Capable codepoint on TCP data packets, but not on TCP SYN and
	  SYN/ACK packets. 
           However, because of the high cost to the TCP transfer of having a SYN/ACK packet dropped, with 
	   the resulting retransmission timeout, the use of ECN for the SYN/ACK came into existance. 
           THIS IS ECN+ .

		   The use of ECN for SYN/ACK packets has the following potential
		   benefits:

		   1) Avoidance of a retransmission timeout;

		   2) Improvement in the throughput of short connections.
		
	      While the current ECN specification enables congested routers to mark TCP data packets during 
	      congestion, this is not the case with TCP control (TCP SYN and SYN ACK) packets.  This is simply 
	      because these packets are used initially to negotiate the use of ECN options between the two endpoints. 
	      Using ECN+ We can observe devastating effects that this can have on system performance, particularly
	      in AQM-enabled environments dominated by web-traffic.Then we explore possibilities of using ECN 
	      bits in the IP headersof TCP control packets.  We demonstrate that marking (instead of dropping) 
	      TCP SYN ACK packets, while leaving the treatment of the initial TCP SYN packet unchanged from 
	      current practice,can only improve performance without causing a threat for system security or stability.

### ECN+/TRY ONCE :

      * With ECN+, the TCP node sends a data packet immediately (with an initial congestion window of one segment).

      * With ECN+/TRY ONCE, receiver sends another syn/ack packet with not-ect on receving ece from sender and the sender goes into established state only after receiving this syn/ack packet with not-ect.
