1: Your EC2 has a public IP and the port is open in the security group, but it's unreachable. Why?

Ans: Check the subnet’s NACL. If inbound or outbound rules are blocking traffic, the security group won’t help. NACLs silently drop traffic with no message.

I would troubleshoot layer by layer:

1. Check Security Group
    * Verify inbound rule allows the required port (e.g., 80, 443, 22).
    * Verify source CIDR is correct (e.g., 0.0.0.0/0 or specific IP range).
2. Check Network ACL (NACL)
    * NACLs are stateless, so both inbound and outbound rules must allow traffic.
    * A deny rule or missing ephemeral port range can block communication.
    * Traffic is dropped silently without logs.
3. Verify Route Table
    * Ensure the subnet has a route to the Internet Gateway (IGW):
      0.0.0.0/0 → Internet Gateway

          * Without this route, the instance cannot communicate with the internet.
4. Check Internet Gateway
    * Confirm the VPC has an Internet Gateway attached.
5. Verify the Application is Running
    * Check if the service is actually listening
      sudo netstat -tulpn
      Confirm the application is bound to 0.0.0.0 and not only 127.0.0.1.
      
“Even if the EC2 has a public IP and the Security Group allows the port, I would verify the NACL rules, route table, Internet Gateway attachment, OS firewall, and whether the application is actually listening on the expected port and bound to 0.0.0.0. In many cases, the issue is either a NACL block, missing route, or the application not listening externally.”

Cross Question 1

Interviewer:
Security Group allows traffic. Why should I check NACL?

Answer:
Security Groups are stateful, but NACLs are stateless. Even if inbound traffic is allowed by the Security Group, the NACL can still block inbound or outbound traffic. Since NACLs evaluate rules in order, a deny rule can silently drop packets.

⸻

Cross Question 2

Interviewer:
What do you mean by stateful and stateless?

Answer:

Security Group (Stateful)

If inbound traffic is allowed, the return traffic is automatically allowed.

Example:

* Allow inbound TCP 80.
* User accesses website.
* Response traffic automatically returns.

NACL (Stateless)

You must explicitly allow both directions.

Example:

* Allow inbound TCP 80.
* Also allow outbound ephemeral ports (1024-65535).
* Otherwise communication fails.

⸻

Cross Question 3

Interviewer:
How will you identify whether traffic is blocked by NACL?

Answer:
I would:

* Review NACL inbound and outbound rules.
* Enable VPC Flow Logs.
* Check whether traffic is marked as REJECT.
* Verify ephemeral ports are allowed.

⸻

Cross Question 4

Interviewer:
The NACL is fine. What’s next?

Answer:
I would verify:

* Route table has:
0.0.0.0/0 → Internet Gateway
    Internet Gateway is attached to the VPC.
* The subnet is public. 
  

2) 
