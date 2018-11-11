# ECN-try-once-in-ns3
This project is to implement ECN+/TryOnce in ns-3.

### We uploaded following files and folders in our repository

## modified src files 
This folder contains complete modification for tcp-socketbase.h(headers) and tcp-socketbase.cc files.

## Results
This folder contains the pcap results of the executed network topology test.cc.

## References
This folder contains all the papers referenced.

## ns-3 concept.md
This file consists of the complete concept of ecn and ecn+ and diffrences between them.

## output.png
This image consists of the screenshot of the pcap result file after execution of the test.cc file.

## socket-base-changes.md
This file consists of the  changes we made for the implementation of ecn+ try once in ns-3 at every phase.

## test.cc
This file is the topology used for running the implemented changes and verify the changes implemented. We used a dumbbell topology with 5 senders, 2 routers and 5 senders each starting at time=0.00 seconds and stopping at 10.00 seconds. Tcp Bulksend application is installed on each of the sender nodes.<br>

# How to build

Replace the tcp-socket-base.cc and .h files in your current ns3/src/internet/model folder.<br>
Download the test.cc file into your scratch folder.<br>
build ns-3<br>
Run the file using the following command:<br>
```./waf --run test; ```<br>
Results are stored in the results/gfc-dumbbell folder named with the time of execution.<br>
Open the pcap files in the folder to view the results.<br>
go through the sender and receiver flags that are there in the pcap




