                                         eBPF XDP UDP Port & Magic Code Filter
A high-performance, low-latency Linux eBPF/XDP program designed to filter, validate, and secure incoming UDP traffic directly at the network driver layer before it hits the network stack.

🚀 Overview
This program attaches to a network interface using XDP (eXpress Data Path) to inspect incoming packets at line rate. It is engineered to protect specific services running on a designated port by enforcing a strict multi-layer validation check:

Protocol & Fragmentation Check: Filters out non-IP, non-UDP, and fragmented packets.

Port Guard: Intercepts traffic destined only for a protected port (9999).

Payload Verification: Validates a custom 4-byte magic code identifier (0x1337C0DE) at the beginning of the UDP payload.

Dynamic Allowlist: Performs a fast O(1) lookup in an eBPF hash map (allowed_src) against the source IPv4 address.
🛠️ How It Works (Packet Flow)
Incoming Packet 
  │
  ├──► [Ethernet Check] ──► Not IPv4? ──► XDP_PASS
  │
  ├──► [IP & UDP Check] ──► Fragment / Non-UDP? ──► XDP_DROP
  │
  ├──► [Port Check] ──────► Not Port 9999? ──► XDP_PASS
  │
  ├──► [Payload Check] ───► Invalid Magic Code (!= 0x1337C0DE)? ──► XDP_DROP
  │
  └──► [Map Lookup] ──────► Source IP Not in `allowed_src` Map? ──► XDP_DROP 
                           └─► Otherwise ──► XDP_PASS (Accepted)
                           📌 Technical Specifications
Target Port: 9999 (PROTECTED_PORT)

Magic Code ID: 0x1337C0DE (TARGET_CODE_ID)

Map Type: BPF_MAP_TYPE_HASH (stores allowed IPv4 source addresses in network byte order)

Max Entries: 1024 source IPs

License: GPL
