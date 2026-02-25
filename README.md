# High-Level Design
Router starts by parsing the neighbor arguments from the CLI, and for every neighbor, creates a UDP socket and binds it to localhost:0 so the OS picks a source port, then stores the neighbor’s IP, UDP port, and relationship (customer/peer/provider) in dictionaries, since they need to be looked up. When the router starts, it sends a handshake message to each neighbor and everything runs inside one main event loop using select(). Whenever a socket has data ready, packet is read and the JSON is decoded and sent to the handler for its type.

There are two structures, a route candidate cache, for all known routes, and a forwarding table, for the best route for each prefix.

For UPDATE messages, I insert or replace the candidate from that neighbor, recompute the best route using the BGP tibreaking rules, and then export the update to the right neighbors and routes from customers are forwarded to all, but but routes from peers/providers only get forwarded to customers.

For WITHDRAW messages, I remove the matching candidate routes and recompute the best one if needed, with the same relationships rules as UPDATE messages.

For forwarding data, I convert IPs to integers and use bitmasking to test if the destination falls under a prefix then apply longest prefix first, and if multiple tie, I use the BGP ordering rules localpref, selfOrigin, ASPath length, origin, and lowest next-hop IP. After choosing a route the data can be forwarded only if at least one side is a customer.

For DUMP messages, I aggregate routes before responding. Aggregation groups routes share every attribute (same next-hop, localpref, origin, ASPath, etc.) and tries to merge siblings into a shorter prefix if they're side by side. Also, withdrawals force disaggregation since aggregation is rebuuilt every time.

# Testing

I tested everything using the provided ./run script and the configs directory, and while developing, printed out a lot of debug information, mostly the chosen best routes, so I could match my router’s behavior to the expected outputs in each config.

For the harder tests I added temporary prints of the route candidate sets so I could see exactly what was being kept or discarded then removed most of the debugging messages after making sure the behavior was expecte.d

# Challenges

Test 3-1 was the absolute WORST and gave me the most trouble out of all the configs, my router kept passing the early parts of the test but then broke on every single withdraw, and I kind of just got stuck rereading the expected output and failure output, because whatever I changed to fix the withdraw would break anoter test. Eventually I figured out I was focusing too much on the withdraw method rather than the helpers, and the biggest issue was how I was replacing older routes from the same neighbor and prefix, so after fixing how I filtered candidates before inserting new ones, the rest of the test finally worked. This test also made me rewrite my withdraw handling so that I recompute the best route more cleanly.

CITATION: Unsure how the citation stuff works, but I used this video to help with understanding the aggregate part, NO CODE WAS COPIED DIRECTLY FROM THE VIDEO: https://www.youtube.com/watch?v=sS0kimy4rj0


