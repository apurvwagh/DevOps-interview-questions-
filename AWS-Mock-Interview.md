1. Explain VPC Peering. How does routing work?

1) VPC Peering is a private network connection between two VPCs that allows resources in those VPCs to communicate using private IP addresses. It can connect VPCs within the same AWS account or across different AWS accounts and regions, depending on the peering configuration.

2) VPC Peering itself does not automatically configure routing. After creating the peering connection, I need to update the route tables on both sides. For example, if VPC-A uses 10.1.0.0/16 and VPC-B uses 10.2.0.0/16, the route table in VPC-A needs a route for 10.2.0.0/16 pointing to the VPC peering connection, and VPC-B needs the reverse route for 10.1.0.0/16.

3) I also verify Security Groups and Network ACLs because routing alone doesn’t guarantee connectivity. The CIDR ranges must not overlap.

4) One important limitation is that VPC Peering is non-transitive. If VPC-A is peered with VPC-B and VPC-B is peered with VPC-C, VPC-A cannot automatically communicate with VPC-C through VPC-B. For many-to-many connectivity, I would consider AWS Transit Gateway.”

==================================================================
