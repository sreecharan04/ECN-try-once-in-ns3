
OUR APPROACH:

1.  We are using the dumbbell topology with 5 sender nodes,2 routers and 5 receiver nodes in the topology. We have enabled the p-cap to capture the packets on Wireshark and check the marked bits.

2. We must implement the changes in the tcp-socket-base files to support the congestion control for control packets.

3. Since the tos field in the Ip header is being set for each packet in the Addsockettags() method, we changed the function by adding a Boolean variable which is marked true only if it is a SYN/ACK packet. The function sets the ECT bit whenever the data packets are sent in an ECN enabled connection and also if the packet is SYN/ACK packet from an ECN enabled responder. We altered the condition in SendEmptypacket() method where the AddSockettag() method is called and set the state of the responder to ECN_IDLE before sending the SYN/Ack Packet. Also, the Boolean variable is set to true if the packet is SYN/Ack packet. We don’t change the other TCP flags being set as the other flags are same in already implemented ClassicEcn and Ecn++.
```
TcpSocketBase::AddSocketTags (const Ptr<Packet> &p,bool withEct) const //withEct boolean variable added here
{
if (GetIpTos ())
{
SocketIpTosTag ipTosTag;

if (m_tcb->m_ecnState != TcpSocketState::ECN_DISABLED && (CheckEcnEct0 (GetIpTos ()) ||
withEct))
{
ipTosTag.SetTos (MarkEcnEct0 (GetIpTos ()));
}
p->AddPacketTag (ipTosTag);
}
else
{
// std::cout<<m_tcb->m_ecnState
if (m_tcb->m_ecnState != TcpSocketState::ECN_DISABLED && (p->GetSize () > 0 || withEct))
{
// Set ECT(0) if ECN is enabled and ipTos is 0
SocketIpTosTag ipTosTag;
ipTosTag.SetTos (MarkEcnEct0 (GetIpTos ()));
p->AddPacketTag (ipTosTag);
}
```
```

if (hasSyn & hasAck & hasEce & markect)
{
//std::cout<<"1\n";
m_tcb->m_ecnState = TcpSocketState::ECN_IDLE;
//std::cout<<m_tcb->m_ecnState;
AddSocketTags (p,true);
SetCE(p);
//std::cout<<"Marking CE bit\n";
// m_tcb->m_ecnState == TcpSocketState::ECN_CE_RCVD;
// m_congestionControl->CwndEvent (m_tcb, TcpSocketState::CA_EVENT_ECN_IS_CE);
//std::cout<< m_tcb->m_ecnState<<"-receiver ecn-state\n" ;
}
else
{
// std::cout<<"hi";
AddSocketTags (p,false);
}
```
4. Now, since we were unable to create congestion explicitly for the SYN/Ack packets, we manually set the CE bit in the SYN/ACK packet to show the changes between the normal implementation and ECN+/Try Once implementation. We added a function SetCE() to set the CE bit in any packet desired. The function has also been declared in tcp-socket-base.h
```
void
TcpSocketBase::SetCE(Ptr<Packet> p)
{
  SocketIpTosTag ipTosTag;
  ipTosTag.SetTos (MarkEcnCe (0));
  p->ReplacePacketTag (ipTosTag);
}


```

5. On receiving the SYN/ACK packet with ect, instead of establishing a connection, we remain in syn sent state and send an ack packet with ECE marked as an indication for congestion.

```

     else if(tcpflags & (TcpHeader::SYN | TcpHeader::ACK)
         && m_tcb->m_nextTxSequence + SequenceNumber32 (1) == tcpHeader.GetAckNumber () && (m_ecnMode == EcnMode_t::ClassicEcn && m_tcb->m_ecnState == TcpSocketState::ECN_CE_RCVD))
        {

          if (m_ecnMode == EcnMode_t::ClassicEcn && (tcpflags & (TcpHeader::ECE)))
           {
             NS_LOG_INFO ("Received ECN SYN-ACK packet.");
             NS_LOG_DEBUG (TcpSocketState::EcnStateName[m_tcb->m_ecnState] << " -> ECN_IDLE");
           //  std::cout<<"Setting sender's ecn state to idle\n";
             m_tcb->m_ecnState = TcpSocketState::ECN_IDLE;
           }
          else
            {
             m_tcb->m_ecnState = TcpSocketState::ECN_DISABLED;
            }
        // std::cout<<"Receiving SYN/ACK with CE mark\n";
          m_retxEvent.Cancel ();
          m_rxBuffer->SetNextRxSequence (tcpHeader.GetSequenceNumber () + SequenceNumber32 (1));
        // m_tcb->m_highTxMark = ++m_tcb->m_nextTxSequence;
        // m_txBuffer->SetHeadSequence (m_tcb->m_nextTxSequence);
          SendEmptyPacket ((TcpHeader::ACK | TcpHeader::ECE),false);
        // std::cout<<"sending ack with ece\n";
          m_tcb->m_ecnState = TcpSocketState::ECN_IDLE;
         } 

```
6. If we receive the Ack packet with ECE, we reduce the initial window size of the responder to 1 segment. If the responder sends data, it will start its congestion window from one segment. If the responder receives data, then it simply reduces its congestion window and keeps receiving data. We now send a SYN/ACK packet with not-ect as an indication for corrected window size and received notification. Upon receiving this packet, the initiator moves to connection established state and sends an ack packet along with pending data.
```
if (m_ecnMode == EcnMode_t::ClassicEcn && (tcpHeader.GetFlags () & TcpHeader::ECE))
{
m_tcb->m_cWnd = 1;
m_tcb->m_cWndInfl = m_tcb->m_cWnd;
m_retxEvent.Cancel();
m_tcb->m_highTxMark = ++m_tcb->m_nextTxSequence;
m_txBuffer->SetHeadSequence (m_tcb->m_nextTxSequence);
//std::cout<<"A segment size is:\t">>m_tcb->m_segmentSize>>"\n";
// m_rxBuffer->SetNextRxSequence (tcpHeader.GetSequenceNumber () + SequenceNumber32 (1));
SendEmptyPacket ((TcpHeader::SYN | TcpHeader::ACK | TcpHeader::ECE),false);
}

```
