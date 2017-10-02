# NANOG 71 Hachathon
## Presented by Facebook

### Team Flow Symmetry 
* Adam Mills
* Malachi Middlebrook
* Chris Woodfield
* Vasily Averin

### Abstract
Egress peer engineering controller to enforce symmetric routing on a distributed Edge using inbound flow data
Description:
This would be in regard to a single AS design network with multiple and/or geographically diverse eBGP peers or different Edge Nodes. Similar to SR (segment/spring routing) egress peer engineering (epe) without the label/id stack requirements. The idea as the Internet is built on hot potato routing that any peer would send you traffic on its lowest latency or best path inbound but due to your own AS equal routing preference with multiple peers and/or distributed edge your egress traffic goes out of a different edge node or site then it came in causing asymmetric routing within your local AS.

Potential Solution:
Ingredients Require:
- IGP (OSFP/ISIS)
- iBGP
- Out of path (or in path) RR with data plane (IGP) knowledge for NHT
- Flow data - Netflow/sFlow/IPFIX of Edge nodes/interfaces

Use the default ifindex tracking in most flow analysis tools to track the interface the packet (source IP of a client) came in on and use that information via a RR to inject the source /32 or /128 (or /24 and /64 depending on many entries your hardware ASICs FIB can handle) into the FIB to use that interface as the next-hop for that particular source prefix. The next-hop of the edge interface is injected within your IGP using passive for all edge facing interfaces ensure next-hop reachability in iBGP

### Solution
1. Create a Netflow/Sflow collector using Kibana and Elasticsearch
https://gitlab.com/thart/flowanalyzer
2. Use GOBGP to create a iBGP daemon to insert /32 and /128 routes to nexthop interfaces
https://github.com/nemith/nanog71/tree/master/pkg/bgpctrl
3. (WIP) Create the glue that pulls the Netflow records out of elasticsearch and inject them into the iBGP fabric.

### Unresolved issues
1. Team Flow Symmetry did not address the potential of spoofing traffic.
* Potential solution in this space could be authentication API against the desired endpoints. Basically the IPv4/v6 address would be queried against a database of known user IPs.
2. Routes should be validated
* There is the potential to insert multiple next hops for the same address. The "glue" should validate that a route for the desired nexthop is not already present, then it should make sure that another route does exist for the IPv4/v6 address.
3. Should the route age out?
* The question was posed of how long should the routes exist? It is assumed that as long as an Eyeball is attempting to access the service the route should exist, however should the route age out over time.
