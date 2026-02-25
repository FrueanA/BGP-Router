This project implements a simplified Border Gateway Protocol (BGP) router capable of maintaining routing state, processing neighbor announcements and forwarding data. The router handles update, withdraw, data, dump, and table messages using JSON over UDP sockets.

## Features

- **UDP-based multi-socket router** using `select()` for event-driven I/O  
- **BGP Update + Withdraw handling** with propagation rules based on peer/provider/customer relationships  
- **Forwarding table construction** including:  
  - Longest prefix match  
  - Local preference  
  - AS-path comparison  
  - Origin type ranking  
  - Consistent deterministic tiebreaking  
- **Data message forwarding** with legality checks (customer vs peer/provider relationships)  
- **Error handling:** returns `no route` messages when forwarding is not possible  
- **Route aggregation & disaggregation** to maintain a compressed routing table  
- **Supports simulator "dump" → "table" responses** with serialized route state  

## High-Level Approach

I implemented the router in an event-driven architecture, using `select()` to listen across all neighbor sockets. Each incoming JSON message is parsed and dispatched to handlers for update, withdraw, data, or dump events.

Forwarding table state is stored as structured route objects containing prefix, mask, AS-path, attributes, and next-hop information. After each update or withdraw, the table is rebuilt or aggregated as needed.

For forwarding data packets, the router performs longest-prefix matching followed by tie-breaking rules. Relationship rules (customer, peer, provider) determine whether a packet may legally be forwarded through a given neighbor.

## Testing

I validated the router using the provided simulator and all 16 configuration files under `configs/`:

```bash
./run configs/<config-file>```

Tests were evaluated across all levels (1–6), covering:
- Basic update forwarding
- Tie-breaking and preference rules
- Withdraw propagation
- Illegal forwarding behavior
- Longest-prefix match correctness
- Aggregation and disaggregation
All test cases pass without crashes or undefined behavior.

## How to Run
```./4700router <asn> <port-ip.type> <port-ip.type> ...```

Example:
```./4700router 7 7833-1.2.3.2-cust 2374-192.168.0.2-peer 1293-67.32.9.2-prov```

The router then:
- Binds local UDP sockets
- Sends handshake messages
- Processes all subsequent traffic from the simulator

## Repository Contents
- 4700router — main executable/script
- Makefile
- README.md
- Source code implementing router logic
