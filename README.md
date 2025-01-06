# CONGESTION-CONTROLLED-PIPELINED-RDT-PROTOCOL-OVER-UDP 
# CODE AVAILABLE UPON REQUEST
All packets exchanged between the sender and the receiver have the following structure:
integer type; // 0: ACK, 1: Data, 2: EOT, 3: SYN
integer seqnum; // Modulo 32
integer length; // Length of the String variable ‘data’
String data; // String with Max Length 500
2
Each integer field is a 4-byte unsigned integer in network byte order. The type field indicates the type
of the packet. It is set to 0 if it is an ACK, 1 if it is a data packet, 2 if it is an end-of-transmission (EOT)
packet, and 3 if it is the synchronization (SYN) packet (see the definition and use of EOT and SYN packets
below). For data packets, seqnum is the modulo 32 sequence number of the packet. The seqnum of the
SYN packet and the first data packet should be zero. For ACK packets, seqnum is the sequence number of
the packet being acknowledged. The length field specifies the number of characters carried in the data
field. It will be in the range of 0 to 500. The data string is be exactly length bytes long. For EOT,
SYN and ACK packets, length is be set to zero. A reference implementation of the packet format is
provided to you as a Python 3 file named “packet.py”. 

# Sender
* host address of the network emulator,
* UDP port number used by the emulator to receive data from the sender,
* UDP port number used by the sender to receive ACKs from the emulator,
* timeout interval in units of millisecond, and
* name of the file to be transferred

Upon execution, the sender program goes through 3 stages: (i) connection establishment, (ii) data
transmission, and (iii) connection termination. In the connection establishment stage, the sender sends
a SYN packet to the receiver (through the network emulator) every 3 seconds, until it receives a SYN
packet from the receiver. The SYN packets have a seqnum of 0.
In the data transmission stage, the sender program reads data from the specified file and sends it using
the congestion controlled RDT protocol to the receiver via the network emulator. The initial window size
is to be set to N=1 packet, and the seqnum for the initial data packet is set to 0. After all content of
the file has been transmitted successfully to the receiver (and all corresponding ACKs have been
received), the sender transitions to the connection termination stage.

In the connection termination stage, the sender sends an EOT packet to the receiver every 3 seconds,
until it receives an EOT packet from the receiver. The sender can close its connection and exit only after
it has received ACKs for all data packets it has sent and received an EOT packet from the receiver. For
simplicity, you can assume that the EOT packet from the receiver never gets lost in the network.
To ensure reliable transmission and congestion control in the data transmission stage, your program
should implement the congestion controlled pipelined RDT protocol as follows:
3
If the sender has a packet to send, it first checks to see if the window is full, i.e., whether there are N
outstanding, unacknowledged packets. If the window is not full, the packet is sent, the appropriate
variables are updated, and a timer is started if not done before. The sender will use only a single timer
that will be set for the oldest transmitted-but-not-yet-acknowledged packet. If the window is full, the
sender will try sending the packet later.
When the sender receives an acknowledgement packet with seqnum n, the ACK will be considered a
cumulative acknowledgement, indicating that all packets with a sequence number up to and including n
have been correctly received at the receiver. If a timeout occurs, the sender sets N=1 and retransmits
the packet that caused the timer to timeout (i.e., only that one packet and not all the non-ACKed
packets). If a packet is retransmitted, the timer is reset.
If a new ACK (and not a duplicate ACK) is received, but there are still additional transmitted-but-yet-tobe-
acknowledged packets, the timer is restarted. If there are no outstanding packets, the timer is
stopped. Also, if a new ACK is received, N is incremented by 1 up to a maximum of 10 (N cannot exceed
10). The first data packet is transmitted with seqnum=0, the second packet is transmitted with seqnum=1,
and so on. After the packet with seqnum=31 is transmitted, the next packet is transmitted with
seqnum=0.



# Reciever:
* hostname for the network emulator,
* UDP port number used by the link emulator to receive ACKs from the receiver,
* UDP port number used by the receiver to receive data from the emulator, and
* name of the file into which the received data is written.
  
When receiving packets sent from the sender via the network emulator, it will execute the following:
• If it is a SYN packet, send a SYN packet back, otherwise:
o Check the sequence number of the packet.
o If the sequence number is the one that it is expecting:
▪ If the packet is an EOT packet, send an EOT packet back and terminate the
program. Otherwise, write the data from the packet to the output file. Then check
if the packet with the next sequence number is in the buffer.
▪ If the packet exists, remove the packet from the buffer, write the data of the
packet to the output file, then repeat the previous step.
▪ If the packet does not exist, send an ACK packet back to the sender with the
seqnum equal to the seqnum of the last packet written to disk and set the expected
seqnum to the seqnum of the missing packet.
o Otherwise, if the sequence number is not the one that it is expecting:
▪ If the sequence number is within the next 10 sequence numbers, store the
received packet in a buffer if the packet is not already stored.
▪ In all other cases (e.g., duplicate/old packet), discard the received packet.
▪ For both the cases above, send an ACK packet for the most recently received inorder
packet.
Once the receiver has received all data packets and an EOT from the sender, it sends an EOT packet
and then exit.



