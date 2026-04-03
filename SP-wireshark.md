# Service Provider (SP) Technology - Wireshark Notes

=====================
## What is Service Provider Technology?
Network infrastructure operated by ISPs and carriers to deliver connectivity at scale.
Key protocols: MPLS, BGP, OSPF/IS-IS, QoS (DiffServ/DSCP), GRE, VLAN (802.1Q), Carrier Ethernet.

=====================
## MPLS (Multi-Protocol Label Switching)

### How MPLS Works
- Labels are inserted between L2 (Ethernet) and L3 (IP) header --> "Layer 2.5"
- Label tells router where to forward packet. No need to inspect IP header at every hop.
- Label Edge Router (LER): pushes/pops MPLS label at network edge
- Label Switch Router (LSR): forwards based on label only (fast switching)

### MPLS Header Structure
```
+-------+---+---+---+
| Label | Exp| S | TTL|
+-------+---+---+---+
  20 bit  3b  1b  8b
```
Label = 20 bit forwarding identifier
Exp   = 3 bit. used for QoS (Class of Service). also called TC (Traffic Class)
S     = Bottom of Stack bit. 1 = last label in stack
TTL   = 8 bit. same concept as IP TTL

### Wireshark Filter for MPLS
```
mpls                        // show all MPLS packets
mpls.label == 100           // filter by specific label value
mpls.exp == 5               // filter by EXP/TC value (QoS marking)
mpls.bottom_of_stack == 1   // last label in stack
mpls.ttl < 5                // low TTL (near edge of MPLS cloud)
```

### What to Look For
- Label value consistent along a path (same LSP = same label per hop)
- EXP bits matching expected QoS policy
- Label stack depth (VPN traffic usually has 2 labels: outer transport + inner VPN)

-------------------
## BGP (Border Gateway Protocol)

### Overview
- Protocol used between ISPs and between PE-CE in MPLS VPN
- eBGP = between different AS (external)
- iBGP = within same AS (internal, between SP routers)
- TCP port 179

### BGP Message Types
```
OPEN        = establish BGP session. includes AS number, BGP ID, hold timer
UPDATE      = advertise or withdraw routes
NOTIFICATION= error message. session will close after this
KEEPALIVE   = sent every 1/3 of hold timer to maintain session
ROUTE-REFRESH = request peer to resend routing table
```

### Wireshark Filter for BGP
```
bgp                             // all BGP traffic
bgp.type == 1                   // OPEN messages
bgp.type == 2                   // UPDATE messages
bgp.type == 3                   // NOTIFICATION messages (look for errors!)
bgp.type == 4                   // KEEPALIVE
tcp.port == 179                 // BGP TCP session
bgp.update.nlri_prefix          // prefixes being advertised
```

### Troubleshooting BGP with Wireshark
- Session keeps dropping --> look for NOTIFICATION messages. check error code + subcode
- Session not forming --> check OPEN message exchange. look for mismatch in AS or hold timer
- Route not appearing --> check UPDATE messages. look for the prefix in NLRI field
- KEEPALIVE missing --> session may timeout. hold timer negotiated in OPEN

### BGP NOTIFICATION Error Codes
```
1 = Message Header Error
2 = OPEN Message Error (AS mismatch, bad hold timer)
3 = UPDATE Message Error
4 = Hold Timer Expired
5 = Finite State Machine Error
6 = Cease (peer manually closed)
```

-------------------
## QoS - DiffServ / DSCP

### Overview
- DiffServ uses DSCP field in IP header (6 bits of ToS byte)
- Tells SP network how to treat packet (priority, drop preference)
- Exp/TC field in MPLS carries same info inside MPLS cloud

### DSCP Common Values
```
DSCP 46 (EF)    = Expedited Forwarding. voice/real-time. highest priority
DSCP 34 (AF41)  = Assured Forwarding class 4. video
DSCP 26 (AF31)  = Assured Forwarding class 3
DSCP 18 (AF21)  = Assured Forwarding class 2
DSCP 10 (AF11)  = Assured Forwarding class 1. bulk data
DSCP 0  (BE)    = Best Effort. default. lowest priority
```

### Wireshark Filter for QoS/DSCP
```
ip.dsfield.dscp == 46           // EF (voice traffic)
ip.dsfield.dscp == 34           // AF41 (video)
ip.dsfield.dscp == 0            // Best Effort
ip.dsfield.ecn == 3             // Congestion Experienced (ECN)
```

### What to Check
- Voice/video marked correctly as EF/AF41 at ingress
- DSCP preserved end-to-end (some networks remark or zero out DSCP)
- MPLS EXP bits matching IP DSCP (should be mapped consistently)

-------------------
## VLAN / 802.1Q

### Overview
- 802.1Q tag inserted into Ethernet frame between Src MAC and EtherType
- Used by SP for customer traffic separation
- QinQ (802.1ad) = double tagging. outer tag = SP VLAN, inner tag = customer VLAN

### 802.1Q Tag Structure
```
+------+------+--------+
| TPID | PCP  | VID    |
+------+------+--------+
 2byte  3 bit  12 bit
```
TPID = 0x8100 for 802.1Q. 0x88A8 for QinQ outer tag
PCP  = Priority Code Point. 3 bit CoS (Class of Service) 0-7
VID  = VLAN ID. 0-4094

### Wireshark Filter for VLAN
```
vlan                            // all VLAN tagged frames
vlan.id == 100                  // specific VLAN
vlan.priority == 5              // CoS value
vlan.id == 100 && vlan.id == 200  // QinQ: outer=100 inner=200
eth.type == 0x8100              // 802.1Q
eth.type == 0x88a8              // QinQ (outer tag)
```

-------------------
## GRE Tunnel

### Overview
- Generic Routing Encapsulation. wraps any L3 protocol inside IP
- Used by SP for pseudowire, L3VPN transport, or overlay networks
- Protocol number: 47

### GRE Header
```
+-------+---+--------+----------+------+
| Flags | Ver| Protocol| Checksum| Key |
+-------+---+--------+----------+------+
```

### Wireshark Filter for GRE
```
gre                             // all GRE traffic
ip.proto == 47                  // GRE by IP protocol number
gre.proto == 0x0800             // GRE carrying IPv4
gre.proto == 0x86DD             // GRE carrying IPv6
gre.key                         // GRE tunnel key present
```

-------------------
## OSPF (Open Shortest Path First)

### Overview
- IGP used within SP network to carry loopbacks and infrastructure routes
- Used as underlay for MPLS LDP/RSVP-TE
- IP protocol 89. uses multicast 224.0.0.5 (all OSPF routers) and 224.0.0.6 (DR/BDR)

### OSPF Packet Types
```
1 = HELLO         // neighbor discovery. sent every hello-interval
2 = DBD           // Database Description. list of LSAs
3 = LSR           // Link State Request
4 = LSU           // Link State Update. carries actual LSA
5 = LSAck         // Acknowledgement
```

### Wireshark Filter for OSPF
```
ospf                            // all OSPF packets
ospf.msg == 1                   // HELLO packets
ospf.msg == 4                   // LSU (topology updates)
ip.dst == 224.0.0.5             // OSPF multicast
ospf.hello.router_id            // router ID in HELLO
```

### Troubleshooting OSPF with Wireshark
- Neighbor not forming --> look at HELLO. check area ID, hello/dead timer, auth mismatch
- Topology not converging --> look at LSU frequency and LSAck. confirm flooding is working
- Flapping --> watch HELLO gap. if dead timer expires = interface issue

-------------------
## LDP (Label Distribution Protocol)

### Overview
- Used to distribute MPLS labels in SP network
- Runs over TCP/UDP port 646
- Works alongside OSPF/IS-IS as signaling protocol for MPLS forwarding

### Wireshark Filter for LDP
```
ldp                             // all LDP messages
tcp.port == 646                 // LDP TCP session
udp.port == 646                 // LDP UDP discovery
ldp.msg.type == 0x0100          // Hello (discovery)
ldp.msg.type == 0x0200          // Initialization
ldp.msg.type == 0x0300          // Label Mapping
ldp.msg.type == 0x0400          // Label Request
ldp.msg.type == 0x0500          // Label Withdraw
```

-------------------
## Common SP Troubleshooting Workflow

### Step 1: Identify the traffic type
```
mpls        // Is it MPLS encapsulated?
vlan        // Is there VLAN tagging?
bgp         // Is it control plane BGP?
ospf        // Is it OSPF?
```

### Step 2: Check packet markings
```
ip.dsfield.dscp             // QoS marking correct?
mpls.exp                    // EXP/TC matches DSCP?
vlan.priority               // CoS on VLAN correct?
```

### Step 3: Check reachability / session health
```
bgp.type == 3               // BGP NOTIFICATION = session error
ospf.msg == 1               // OSPF HELLO frequency ok?
ldp.msg.type == 0x0500      // LDP label withdraw = route removed
```

### Step 4: Trace path / TTL analysis
```
mpls.ttl < 5                // packet near end of MPLS path
ip.ttl == 1                 // ICMP TTL exceeded coming back
icmp.type == 11             // TTL exceeded from intermediate hop
```

### Step 5: Look at timing
- Use Delta column (View > Time Display Format > Seconds since previous displayed packet)
- BGP hold timer = 3x keepalive interval. if KEEPALIVE gap exceeds this --> session drops
- OSPF dead timer = 4x hello interval. missing HELLO = neighbor drops

=====================
## Useful SP Display Filters Cheatsheet

```
mpls                            // Any MPLS
bgp                             // Any BGP
ospf                            // Any OSPF
ldp                             // Any LDP
vlan                            // Any 802.1Q
gre                             // Any GRE tunnel

bgp.type == 3                   // BGP errors (NOTIFICATION)
ospf.msg == 1                   // OSPF HELLO
mpls.label == 0                 // IPv4 Explicit Null label
mpls.exp >= 4                   // High priority MPLS traffic
ip.dsfield.dscp == 46           // Voice (EF)
ip.dsfield.dscp == 0            // Best effort

(mpls && mpls.ttl == 1)         // Packet at penultimate hop
(bgp.type == 2 && bgp.update.nlri_prefix)  // BGP route advertisements
(ospf && ospf.msg == 4)         // OSPF topology updates
(vlan.id >= 100 && vlan.id <= 200)  // VLAN range filter
```

=====================
## Protocol Number Reference (SP Context)
```
Protocol 47  = GRE
Protocol 89  = OSPF
Protocol 103 = PIM (multicast)
Protocol 112 = VRRP
TCP 179      = BGP
TCP/UDP 646  = LDP
```
