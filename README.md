# Wireshark

Course on David Bombal 
https://courses.davidbombal.com/courses/1714194/lectures/38892909
Course on CBTNugget

Course on Youtube;


Personally the best course are in David Bombal's website taught by Christ

## Need to Personalise wireshark:
To analyse application
To analyse security
To troubleshoot network
To analyse TCP

### Preference
to custom wireshark
'edit > preference'


## Trick:

### time
WWW.XXXYYYZZZZ
w = second
x = millisec
y = microsec
z = nanosec

### statistic > endpoints

### statistic > conversation
summary off all pcap conversation captured 
port tuple
sort by highest bytes

### [information Wireshark]
square bracket is a information provided by wireshark


### Coloring rule
custom colour for packet

 ### edit pane 3 --> RFC
 'edit > preference > layout > pane3 > select packet diagram'
 


 ### after filter IP address
 'statistics > endpoints > tick limit to display filter'
 
 ### follow 1 set of conversation
 'packet > right click > conversation filter > TCP/ethernet '
 


--------------
###Add column
to view delay --> add "Delta" column to profile
Wireshark > view > Time Display Format > Seconds since previous displayed packet

-------------
###Add TTL as column

See TTL of every packet.
In layer 4. > right click > apply as column



======
## Filter
green --> worked
red --> filter got error
yellow --> filter might work but might not be the one I want to filter
#Display Filter
1 is true
0 is false

host:			ip.addr == 192.168.0.1 //doent matter dst or src
subnet:			ip.addr == 192.168.0.1/24 //network
source:			ip.src == 192.168.0.1
destination:	ip.dst == 192.168.0.1
port:			tcp.port==80 //will unclude http and tcp packet
protocol:		http
specific:		tcp.flags.syn==1



### to see active traffic use by the ip address
statistic > endpoints > limit to display filter

### shortcut for filter
filter section > most right > add new filter
 
### conversation filter
ping 192.168.1.1 -l 1600 //need to try


======
## Operator
eq		==
not		!
or		||
and		&&
gt		>
lt		greater sign
()		make the filter ack as single condition
contains (exact string)
matches ( regular expression) - does not matter uppercase or lowercase
in (range) - membership operator

EXAMPLE:
frame contains google				// find frame that have specific word google
http.host matches "\.(org|com|net)"	//match anything http that have .org or .com or .net
tcp.port in {80 443 8000..8004} 	//any packet that have port 80 or 443 8000 until 8004
http.request.method in {GET,POST}	//GET or POST request method
(ip.addr==10.0.2.15 && ip.addr==104.16.65.85) and frame contains udemy	//IP address for both IP(will ack as 1 condition) + must have udemy
(tcp.flags == 0x012) and ip.src == 10.0.2.15 // find SYN ACK flag + source IP 

=====================
## Before start capturing:
1. application the access
2. where the server
3. network path
4. how many affected
5. what the error

## Way to capture packet:
1. install wireshark in server
pros: easiest way. final option to have more direct view of what happen
cons: more workload on server

2. SPAN / Mirror
forward packet to one of available.
pros:
cons: overprovision. SPAN port cant handle too many port mirror to it.

3. TAP
seperated hardware for analysis packet. install between link that we want to capture.

### capture option
promiscuous - to let our packet not to capture unicast coming to the device
snaplen 	- if involve sensative data. will ignore the payload. can set to just capture packet header.

### long term capture --> using ring-buffer
intermittent Issue
capture option > output > save file fill > tick  create new file automatically > after: 500 mbps > use ring buffer with 100 files.

### ping with max mtu
ping 8.8.8.8 -s  1600 //ping with 1600bytes


======================
### capture using CMD
'dumpcap'
'dumpcap -D' //with interface option 
'dumpcap -i 1' //with interface option 1
'dumpcap -i 1 -w [filename]' // -w for savename

dumpcap -i 1 -w test.pcapng -b filesize :500000 -b files: 100 //500mbps -b for ring buffer
ctrl + c --> to end capture
=====================
maxmar database --> geolocation

=====================
## RFC
8 bit = 1 byte
=====================
## Anatomy of Packet
packet and protocol
OSI and TCP/IP

Data link: frame
+-----------------+------------+------+------+-----+
| Destination MAC | Source MAC | Type | Data | FCS |
+-----------------+------------+------+------+-----+
Destination: 6byte
Source: 6byte
EtherType field: 2byte. what kind of data are coming.
Data: payload
FCS: 4 byte. for checksum for every byte. --> not shown in wireshark. crc or fcs count


-------------------
Network layer: packet Format - IP header

+---------+--------+---------+----------------+---------+--------+----+-------+-----+----------+-----+------+---------+----------+
|Dest(MAC)|Src(MAC)|Type(MAC)| Version Length | DiffSrv | Length | ID | Flags | TTL | Checksum | Src | Dest |DATA(L2) | CRC(L2) |
+---------+--------+---------+----------------+---------+--------+----+-------+-----+----------+-----+------+---------+----------+

Version					= IP version (IPV4 or IPv6)
Header(Version) Length				= will dictact how many header needed to transfer packet
Diff Service			= for high priority traffice. priority can be set for voice etc
Length					= total length that are encapsulated. 1514 length max.
IDentification number	= will be unique from packet to packet.overwritten when pass NAT/PAT/proxy. 
						//to track packet end-to-end locally. both end have same ID 
Flags					= fragment, more fragment, not fragment
TTL						= 64 128 265. when 0 --> will sent icmp back to host. can know how far packet coming from
Protocol				= will tell us which protocol that it going to use for L4
Checksum				=  
Src						= will remain same.unless there is NAT/PAT
Dest					= will remain same.unless there is NAT/PAT



Flags					= 
Reserved  		: 
dont fragment
more fragment	: if flag == 1 //there will be next packet to continue carry data.
fragment offset	: tagging of fragmentation start. 1480 bit is max. 


-------------------
segment

====================
Unicast --> between 2 endpoint. local
Multicast -->  go everywhere, certain device will listen to it.
Broadcast --> mac: ff ff ff ff ff ff , will be send to same network 


=====================
#PACKET ANALYSIS
delta tally with TTL (hop count)

# DNS
A = IPV4
AAAA = IPV6

![image](https://user-images.githubusercontent.com/83261924/211143656-2cef07c5-ad0e-4a5a-9010-2c4e547da011.png)
Recursion Desired = additional forwader for dns server to ask if not listed in that particular server

![image](https://user-images.githubusercontent.com/83261924/211143813-4b649365-2561-4644-be24-2dfe0c65b337.png)
Response = 1 is true this is responce dns packet
Answer RRS = there total answer get
Queries = what we query
Answer = answer listed

