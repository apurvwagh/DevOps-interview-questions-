1. Explain VPC Peering. How does routing work?n

1) VPC Peering is a private network connection between two VPCs that allows resources in those VPCs to communicate using private IP addresses. It can connect VPCs within the same AWS account or across different AWS accounts and regions, depending on the peering configuration.

2) VPC Peering itself does not automatically configure routing. After creating the peering connection, I need to update the route tables on both sides. For example, if VPC-A uses 10.1.0.0/16 and VPC-B uses 10.2.0.0/16, the route table in VPC-A needs a route for 10.2.0.0/16 pointing to the VPC peering connection, and VPC-B needs the reverse route for 10.1.0.0/16.

3) I also verify Security Groups and Network ACLs because routing alone doesn’t guarantee connectivity. The CIDR ranges must not overlap.

4) One important limitation is that VPC Peering is non-transitive. If VPC-A is peered with VPC-B and VPC-B is peered with VPC-C, VPC-A cannot automatically communicate with VPC-C through VPC-B. For many-to-many connectivity, I would consider AWS Transit Gateway.”
==================================================================

2. Difference between ALB and NLB.

1) ALB and NLB are both AWS load balancers, but they operate at different layers and are designed for different use cases. ALB operates at Layer 7, the application layer, and understands HTTP and HTTPS. NLB operates at Layer 4 and primarily handles TCP, TLS, and UDP traffic.

2) I use ALB when I need application-level routing such as host-based routing, path-based routing, HTTP headers, or integration with web applications and Kubernetes Ingress. For example, /api can go to one target group and /payments to another.

3) I use NLB when I need very high performance, low latency, static IP addresses, or protocols that aren’t HTTP-based, such as TCP or UDP.

4) For a typical EKS microservices application using HTTP/HTTPS, I would generally use ALB. If I need TCP-based connectivity or a highly performant Layer-4 load balancer, I would consider NLB.”
   
==================================================================

 3. Difference between Load Balancer, Ingress, and Ingress Controller.

1) A Load Balancer is the actual traffic distribution component that receives client traffic and forwards it to backend targets. In AWS, examples are ALB and NLB.

2) Ingress is a Kubernetes API resource that defines how external HTTP or HTTPS traffic should be routed to Kubernetes Services. It contains rules such as host-based and path-based routing.

3) The Ingress Controller is the component that watches Kubernetes Ingress resources and actually implements those rules by configuring or operating the underlying traffic proxy/load balancer.

4) For example, in EKS, I can use the AWS Load Balancer Controller. I create an Ingress resource with rules such as /api going to the backend Service. The controller watches that resource and provisions/configures an AWS Application Load Balancer accordingly.

So, Ingress is the configuration, Ingress Controller is the implementation mechanism, and the Load Balancer is the traffic-handling infrastructure.”

Ingress
   ↓
"WHAT routing rules do I want?"
   ↓
Ingress Controller
   ↓
"HOW do I implement those rules?"
   ↓
AWS ALB
   ↓
"Actually handles the traffic"

==================================================================

4. How does a Kubernetes Pod access an S3 bucket?

1) For a Pod running on Amazon EKS to access S3 securely, I would use workload-level AWS identity rather than storing static AWS access keys inside the Pod. Depending on the EKS setup, I would use EKS Pod Identity or IAM Roles for Service Accounts, commonly called IRSA.

2) I create an IAM role with only the required S3 permissions, such as s3:GetObject on a specific bucket or prefix. Then I associate that role with the Kubernetes workload’s ServiceAccount. The Pod uses that ServiceAccount, and the AWS SDK obtains temporary credentials through the configured EKS identity mechanism.

3) For example, if an application only needs to download objects, I would grant s3:GetObject rather than s3:*. If the bucket is in the same account, the IAM role can be granted the required permissions through its identity policy and the bucket policy can also be used where appropriate.

4) I would also consider an S3 VPC endpoint so that S3 traffic can stay on AWS’s private network path instead of requiring a NAT Gateway for S3 access.”

                   EKS
                    │
             Kubernetes Pod
                    │
             ServiceAccount
                    │
                    ▼
          EKS Pod Identity / IRSA
                    │
                    ▼
                IAM Role
                    │
              S3 Permissions
                    │
                    ▼
               S3 Bucket

==================================================================

5. How do you securely allow an EC2 instance to access an S3 bucket?

1) I would use an IAM role attached to the EC2 instance through an instance profile rather than storing AWS access keys on the server. The IAM role would contain only the permissions required by the application.

2) For example, if the application only needs to download objects, I would grant s3:GetObject on the required bucket or prefix. If it needs to upload files, I would add s3:PutObject. I would avoid broad permissions such as s3:* or AdministratorAccess.

3) The application running on EC2 can obtain temporary credentials through the EC2 Instance Metadata Service, and the AWS SDK or AWS CLI can automatically use those credentials.

4) For additional security, I can use an S3 bucket policy to restrict access to the specific IAM role. I can also configure an S3 VPC endpoint, especially a Gateway Endpoint, so the EC2 instance can access S3 without requiring a NAT Gateway or Internet Gateway path.”

==================================================================

6. How does AWS Cross-Account Access work? How would an IAM user/role access an S3 bucket in another AWS account?

1) AWS cross-account access allows a principal from one AWS account to access resources owned by another AWS account. The standard approach is to establish trust between the accounts using IAM policies.

2) For S3, suppose Account A contains the user or application and Account B owns the S3 bucket. I would create an IAM role in Account B with the required S3 permissions and configure its trust policy to allow a principal from Account A to assume that role. The user or role in Account A must also have permission to call sts:AssumeRole on the role in Account B.

3) Once the role is assumed, AWS STS provides temporary credentials, and the application uses those credentials to access the S3 bucket.

Another option is direct cross-account S3 access using an S3 bucket policy, where the bucket owner explicitly grants access to a principal in Account A. For production, I generally prefer role assumption when I want centralized permissions and temporary credentials.”

==================================================================

7. How does a private EC2 instance access the internet? b

1) A private EC2 instance doesn’t have a direct route to an Internet Gateway because its subnet’s route table doesn’t contain a default route to the Internet Gateway. If the instance needs outbound internet access, I normally place a NAT Gateway in a public subnet.

2) The private subnet route table has 0.0.0.0/0 pointing to the NAT Gateway. The NAT Gateway is deployed in a public subnet whose route table has a default route to the Internet Gateway.

3) When the private EC2 instance sends an outbound request, the traffic goes from the private subnet to the NAT Gateway, then through the Internet Gateway to the internet. The return traffic comes back through the Internet Gateway and NAT Gateway to the private instance.

4) For production HA, I typically deploy NAT Gateways per Availability Zone and route each private subnet to the NAT Gateway in the same AZ, avoiding a single-AZ dependency.”

====================================================================

8. How can an EC2 instance access an S3 bucket without using an Internet Gateway or NAT Gateway?

1) I would use an S3 VPC Endpoint. For S3, the most common option is an S3 Gateway Endpoint. It allows resources in a VPC, including private EC2 instances, to access S3 without traversing an Internet Gateway or NAT Gateway.

2) I create the VPC endpoint and associate it with the route tables used by the private subnets. AWS then adds the appropriate S3 prefix-list route to those route tables. Traffic destined for S3 is routed through the endpoint instead of going to the NAT Gateway.

3) I would also use an IAM role on the EC2 instance with least-privilege S3 permissions and optionally restrict the S3 bucket policy to requests coming through the specific VPC endpoint.

4) This architecture improves security and can reduce NAT Gateway costs, especially when applications transfer significant amounts of data to and from S3.”

Private EC2
    │
    ▼
Private Route Table
    │
    ▼
S3 Gateway VPC Endpoint
    │
    ▼
Amazon S3

====================================================================

9. Your application on EC2 experiences traffic spikes only during business hours. How would you optimize cost and performance?

1) I would first analyze the traffic pattern using CloudWatch metrics such as CPU utilization, request count, latency, and network utilization. Since the traffic spike is predictable during business hours, I would combine scheduled scaling with dynamic Auto Scaling.

2) I would run an Auto Scaling Group behind an Application Load Balancer and configure minimum, desired, and maximum capacity. For predictable business-hour traffic, I can use scheduled scaling to increase the desired capacity shortly before peak hours and scale down after business hours.

3) I would also configure a target-tracking policy, such as maintaining average CPU utilization around a defined target, so the ASG can respond if traffic is higher or lower than expected.

4)For cost optimization, I would avoid keeping peak capacity running 24/7. For workloads that tolerate interruption, I could also use a mix of On-Demand and Spot Instances. I would right-size the EC2 instance types based on actual CPU and memory utilization.

5) I would monitor application latency and error rates while scaling because the goal isn’t just reducing cost; it is maintaining the required performance and availability.”

====================================================================

10. Your EC2 instance crashed unexpectedly. How would you identify the issue and restore service?

1) If an EC2 instance crashes unexpectedly, my first priority is to determine whether the failure is at the AWS infrastructure level, operating-system level, or application level, while restoring service as quickly as possible.

2) First, I check the EC2 console and CloudWatch for instance status, system status checks, CPU, memory if available through the CloudWatch agent, disk, network metrics, and recent alarms. I check whether it was a system status check failure or an instance status check failure.

3) If the instance is still reachable, I connect through SSH or SSM and check system logs, kernel messages, disk space, memory, and running services. I use commands such as journalctl, dmesg, df -h, free -m, and systemctl status.

4) If I cannot connect, I check security groups, networking, SSM availability, and boot/system logs. If the instance has an underlying host or system problem, I may stop and start the instance, depending on the instance type and recovery procedure.

5) For production, I don’t want the application dependent on a single EC2 instance. If it’s behind an Auto Scaling Group and the instance is unhealthy, the ASG should terminate and replace it automatically. I then perform RCA using CloudWatch logs, application logs, OS logs, and AWS events.”

====================================================================

11. An EC2 application cannot access the internet to download files/images. How would you troubleshoot it?

1) I would troubleshoot this from the EC2 instance outward, starting with DNS and routing, then checking NAT or Internet Gateway connectivity, Security Groups, NACLs, and finally the external destination.

2) First, I check whether the instance can resolve DNS using nslookup or dig. If DNS fails, I check the VPC DNS settings and /etc/resolv.conf.

3) Next, I determine whether the instance is in a public or private subnet. For a private instance, I verify that the route table has 0.0.0.0/0 pointing to a NAT Gateway. For a public instance, I verify the route to the Internet Gateway and that the instance has a public IPv4 address or appropriate public connectivity.

4)Then I check Security Groups and NACLs. The instance needs outbound access, and NACLs must allow the return traffic through ephemeral ports. I also verify that the NAT Gateway is available and located in a public subnet with a route to the Internet Gateway.

Finally, I test connectivity step by step using curl, ping where appropriate, traceroute, and DNS tools. I also check whether the destination itself is blocking the request.”

===================================================================

12. How do you connect an on-premises data center to AWS?

1) There are two primary options for connecting an on-premises data center to AWS: Site-to-Site VPN and AWS Direct Connect. The choice depends on requirements around bandwidth, latency, reliability, and cost.

2) For a quick and cost-effective connection, I can establish an IPsec Site-to-Site VPN between the on-premises firewall/router and an AWS Virtual Private Gateway or Transit Gateway. Traffic is encrypted over the internet.

3) For predictable network performance, higher bandwidth, and more consistent latency, I would use AWS Direct Connect. Direct Connect provides a dedicated network connection between the data center and AWS. For production, I would normally design redundant connections, potentially across different locations.

4)For larger environments with multiple VPCs, I would typically terminate connectivity into a Transit Gateway and use routing to connect the on-premises network to the required VPCs. I would also configure appropriate route tables, security groups, NACLs, and DNS resolution between environments.

For critical workloads, I can use Direct Connect as the primary path and VPN as a backup path.”

===================================================================

13. Two EC2 instances in different VPCs cannot communicate. How would you fix it?

1) I would troubleshoot this layer by layer. First, I verify that the two VPC CIDR ranges do not overlap. Then I establish whether the VPCs are actually connected using VPC Peering, Transit Gateway, or another supported connectivity mechanism.

2) If VPC Peering is used, I verify that the peering connection is active and that both VPC route tables contain routes to the remote VPC CIDR through the peering connection. If Transit Gateway is used, I verify attachment state, Transit Gateway route tables, and propagation or static routes.

3)Next, I check Security Groups on both EC2 instances. The destination instance must allow the required inbound port from the source VPC CIDR or, where supported, the appropriate security-group reference. I also verify NACLs and ensure return traffic is allowed.

4) Finally, I check the EC2 operating-system firewall and application listener. I test connectivity using the private IP and the actual application port rather than just ping, because ICMP may be blocked even when TCP connectivity works.”

===================================================================


















