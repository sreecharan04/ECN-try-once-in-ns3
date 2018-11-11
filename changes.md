
OUR APPROACH:

1.  We are using the dumbbell topology with 5 sender nodes,2 routers and 5 receiver nodes. We have enabled the p-cap to capture the packets on Wireshark and check the marked bits.

2. We must implement the changes in the tcp-socket-base files to support the congestion control for control packets.

3. Since the tos field in the Ip header is being set for each packet in the Addsockettags() method, we changed the function by adding a Boolean variable which is marked true only if it is a SYN/ACK packet. The function sets the ECT bit whenever the data packets are sent in an ECN enabled connection and also if the packet is SYN/ACK packet from an ECN enabled responder. We altered the condition in SendEmptypacket() method where the AddSockettag() method is called and set the state of the responder to ECN_IDLE before sending the SYN/Ack Packet. Also, the Boolean variable is set to true if the packet is SYN/Ack packet. We don’t change the other TCP flags being set as the other flags are same in already implemented ClassicEcn and Ecn++.


4. Now, since we were unable to create congestion explicitly for the SYN/Ack packets, we manually set the CE bit in the SYN/ACK packet to show the changes between the normal implementation and ECN+/Try Once implementation. We added a function SetCE() to set the CE bit in any packet desired.
5. On receiving the SYN/ACK packet with ect, instead of establishing a connection, we remain in syn sent state and send an ack packet with ECE marked as an indication for congestion.
6. If we receive the Ack packet with ECE, we reduce the initial window size of the responder to 1 segment. If the responder sends data, it will start its congestion window from one segment. If the responder receives data, then it simply reduces its congestion window and keeps receiving data. We now send a SYN/ACK packet with not-ect as an indication for corrected window size and received notification. Upon receiving this packet, the initiator moves to connection established state and sends an ack packet along with pending data.
