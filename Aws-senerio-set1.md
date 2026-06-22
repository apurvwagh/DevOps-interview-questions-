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


  =====================

2) You shared an AMI with another AWS account, but they still can’t launch an instance from it. What’s usually missed?

Ans: Sharing the AMI isn’t enough. You also need to share the associated EBS snapshot. Without that, the AMI looks valid but fails at launch.

Cross Question 1

Interviewer:

Why does AWS need the snapshot if the AMI is already shared?

Answer:

An AMI contains:

* OS configuration
* Launch settings
* Block device mappings
* References to EBS snapshots

The actual disk data resides in the EBS snapshots.

During launch, AWS creates EBS volumes from those snapshots. If snapshot permissions are missing, AWS cannot create the volume.

⸻

Cross Question 2

Interviewer:

How do you share an EBS snapshot?

Answer:

1. Open EC2 Console.
2. Go to Snapshots.
3. Select the snapshot.
4. Actions → Modify Permissions.
5. Add the target AWS Account ID.

Cross Question 3

Interviewer:

The AMI and snapshot are shared, but the launch still fails. What else could be wrong?

Answer:

The snapshot may be encrypted using a KMS key.

The target account must also have permission to use that KMS key.

This is a very common issue in enterprise environments.

⸻

Cross Question 4

Interviewer:

How do you share an encrypted AMI?

Answer:

Three things must be shared:

1. AMI
2. EBS Snapshot
3. KMS Key

The KMS key policy must allow the target account.

Without KMS permissions, instance launch fails because AWS cannot decrypt the snapshot.
Interviewer:

5: In a production environment, what is your preferred method—sharing or copying?

Answer:

For production, I prefer copying the AMI into the destination account.

Reasons:

* Independent ownership.
* No dependency on source account.
* Better disaster recovery.
* Easier lifecycle management.
* Avoids accidental deletion by source account.

⸻


“When an AMI is shared but instance launch fails, the most common issue is that the associated EBS snapshot wasn’t shared. If the snapshot is encrypted, the KMS key permissions must also be shared. I would verify AMI permissions, snapshot permissions, KMS access, and review EC2 launch error messages to identify the exact failure point.”

=============================

3: You restored an RDS snapshot for staging, but some queries behave differently than production. 

Ans: When you restore from a snapshot, RDS assigns the default parameter group by default. Custom parameter groups from production are not restored automatically. 
If not manually reassigned, staging may run with different settings, leading to changes in query behavior or performance.

Your Answer:

One possible reason is that the restored RDS instance is using a different parameter group.

When an RDS snapshot is restored, AWS may associate the instance with the default parameter group unless the original custom parameter group is manually attached.

Differences in parameters such as memory allocation, query cache settings, optimizer behavior, timeouts, or logging can cause query execution plans and performance to differ from production.

⸻

Cross Question 1

Interviewer:

What is a Parameter Group?

Answer:

A Parameter Group is a collection of database engine configuration settings.

It’s similar to:

* MySQL → my.cnf
* PostgreSQL → postgresql.conf

  Interviewer:

Q How would you ensure every restored environment matches production?

Answer:

I would automate provisioning using Terraform.

The RDS definition would explicitly include:

* Parameter Group
* Option Group
* Engine Version
* Storage Type
* Monitoring Configuration

This prevents manual mistakes during snapshot restores.

⸻

Real Production Scenario

Interviewer:

Have you seen this issue in production?

Answer:

Yes. A staging environment restored from production snapshots showed significantly slower queries.

Investigation revealed:

* Production used a custom parameter group.
* Staging was attached to the default parameter group.
* innodb_buffer_pool_size and connection settings differed.

After attaching the correct parameter group and rebooting the instance, query performance matched production.

⸻

“When an RDS snapshot is restored, I first verify that the same Parameter Group, Option Group, instance class, storage type, and engine version are being used. A common issue is that the restored database gets associated with a default Parameter Group instead of the production custom Parameter Group, which can change optimizer behavior and query performance.”

==================

4) You enabled IAM roles for service accounts in EKS, but your pod can’t access S3. The role looks fine. What’s the catch?

Ans: The pod must be using a service account with the right annotation linking to the IAM role. If the pod defaults to the default service account or the annotation is missing, the role doesn’t apply.

 The most common issue is that the pod is not using the Service Account that is annotated with the IAM role.

With IRSA, permissions are not attached directly to the pod. The pod must use a Kubernetes Service Account that contains the annotation:
eks.amazonaws.com/role-arn: arn:aws:iam::<account-id>:role/s3-access-role

If the pod is using the default Service Account or the annotation is missing, the pod won’t assume the IAM role and S3 access will fail.

Interviewer:

How do you verify which Service Account the pod is using?
kubectl get pod <pod-name> -o yaml
serviceAccountName: my-service-account

Interviewer:

What is the OIDC Provider and why is it needed?

Answer:

IRSA uses OIDC (OpenID Connect) to authenticate Kubernetes Service Accounts with AWS IAM.

The pod receives a projected token.

AWS STS validates the token against the EKS OIDC Provider and issues temporary credentials.

Without an OIDC Provider, IRSA cannot function.

The IAM role may be correct, but with IRSA the pod must use a Service Account that is annotated with the role ARN. I would verify the Service Account, annotation, IAM trust policy, OIDC provider, and confirm from inside the pod using aws sts get-caller-identity that the role is actually being assumed.”

============================

5) ALB is marking your targets as unhealthy, but hitting the app directly works fine.

Ans: ALB health checks are strict. If your app returns a 301 or a login page without a clean 200 OK, it’ll fail the check even if the app seems fine in the browser.

The first thing I would check is the ALB health check path and response code. ALB health checks are strict and typically expect a 200 OK response. If the application returns a 301/302 redirect, login page, or any other non-success response, ALB marks the target unhealthy even though users can access the application. I would validate the health endpoint, port, security groups, and CloudWatch target health metrics.”

Interviewer:

The endpoint returns 200 from EC2 itself, but ALB still marks it unhealthy. What next?

Answer:

I would verify:

1. Security Group between ALB and EC2
2. NACL rules
3. Correct health check port
4. Correct health check path
5. Host header requirements
6. TLS/HTTPS mismatch
7. Application binding

==========================

6: You pushed a new image to ECR and updated your ECS task definition, but it still runs the old version.

Ans: If you're using mutable tags like latest, ECS often pulls from cache. Unless you force a new digest or use a unique tag per version, you’ll keep running stale containers.
Interviewer:

Have you faced this in production?

Answer:

Yes.

A team was using:
They pushed a new image expecting ECS to update automatically.

However:

* No new task definition revision was created.
* Existing tasks continued using the previously resolved image digest.

  and automatically generated a new task definition revision during deployment.

The issue never occurred again.

⸻

“The most common cause is using mutable tags such as latest. ECS tasks may continue running the image digest already associated with the task definition. I would verify the task definition revision, image digest, and running tasks. In production, I always use immutable image tags and create a new task definition revision for every deployment, followed by a service update or force-new-deployment.”

Interviewer: “How do you roll back if deployment v1.0.26 fails?”

Answer:

“Since I use immutable tags and task definition revisions, I simply update the ECS service to the previous task definition revision (for example, revision 45 instead of 46). ECS redeploys the known-good image immediately, making rollback fast and reliable.”

==============================

7: In EKS, your stateful pod using an EBS volume is stuck in Pending. Why doesn't it reschedule?

Ans: EBS volumes are limited to a single Availability Zone. If EKS places the pod on a node in a different AZ, the volume cannot attach. Make sure your node group includes nodes in the same AZ as the volume.
The most common reason is an Availability Zone mismatch.

Amazon EBS volumes are AZ-specific and can only be attached to EC2 instances within the same Availability Zone.

If the Persistent Volume was created in:
ap-south-1a
and Kubernetes schedules the pod on a node in:
ap-south-1b
the volume cannot attach and the pod remains Pending.

I would check the PV, PVC, node AZ, and scheduler events.
Interviewer:

What Kubernetes object is responsible for this scheduling logic?

Answer:

Volume Node Affinity.

The scheduler checks:

* Pod requirements
* Node labels
* Volume location

The pod can only run on nodes that can access the volume.

Interviewer:

What if I need storage accessible from multiple AZs?

Answer:

Use:

Amazon EFS

Benefits:

* Multi-AZ
* Shared filesystem
* Multiple pods can mount simultaneously
  
Access Mode:ReadWriteMany (RWX)
EBS is block storage.

EFS is shared file storage.

Interviewer:

A node in the volume’s AZ failed. What happens?

Answer:

Kubernetes attempts to reschedule the pod.

However, if no worker nodes exist in that AZ:
Volume cannot attach
Pod remains Pending

To avoid this:

* Maintain worker nodes in all required AZs.
* Use Cluster Autoscaler/Karpenter.
* Use EFS if true multi-AZ storage is required.

  ============================
  
8) You have been assigned to design a VPC architecture for a 2-tier application. The application needs to be highly available and scalable. 
How would you design the VPC architecture?

For a highly available and scalable 3-tier application, I would design the VPC across at least 2 or 3 Availability Zones.

Tier 1 – Web Layer (Public Subnets)

* Create Public Subnets in each AZ.
* Deploy an Application Load Balancer (ALB).
* Internet Gateway attached to the VPC.
* Route Table:0.0.0.0/0 → Internet Gateway

Purpose:

* Accept internet traffic.
* Distribute requests across application servers.

⸻

Tier 2 – Application Layer (Private Subnets)

* Create Private App Subnets in each AZ.
* Deploy EC2 instances or EKS pods.
* Use Auto Scaling Groups.
* No public IPs assigned.

Purpose:

* Process business logic.
* Scale horizontally based on demand.

⸻

Tier 3 – Database Layer (Private DB Subnets)

* Create dedicated Database Subnets.
* Deploy RDS Multi-AZ.
* No internet access.

Purpose:

* Securely store application data.
* Automatic failover during AZ outages.

⸻

Internet Access for Private Resources

Deploy NAT Gateways:

AZ-A → NAT Gateway A
AZ-B → NAT Gateway B
Private subnet route table:
0.0.0.0/0 → NAT Gateway

This allows:

* OS patching
* Package downloads
* Docker image pulls

without exposing servers publicly.

                           Internet
                               |
                         Route 53 (DNS)
                               |
                        +--------------+
                        |     ALB      |
                        | (Public Sub) |
                        +--------------+
                          /          \
                         /            \
                        /              \
               +-------------+   +-------------+
               | Public AZ-A |   | Public AZ-B |
               +-------------+   +-------------+
                      |                 |
               +-------------+   +-------------+
               | App Server  |   | App Server  |
               | EC2/EKS Pod |   | EC2/EKS Pod |
               | Private Sub |   | Private Sub |
               +-------------+   +-------------+
                      |                 |
                      +-------+ +-------+
                              | |
                    +----------------------+
                    |   RDS Multi-AZ       |
                    | Primary + Standby    |
                    | Private DB Subnets   |
                    +----------------------+

                      |                 |
               +-------------+   +-------------+
               | NAT GW AZ-A |   | NAT GW AZ-B |
               +-------------+   +-------------+
                      |                 |
               +-------------+   +-------------+
               | Internet GW |---| Attached to |
               +-------------+   |     VPC     |
                                 +-------------+
VPC (10.0.0.0/16)

├── Public Subnet AZ-A (10.0.1.0/24)
│   ├── ALB
│   └── NAT Gateway-A
│
├── Public Subnet AZ-B (10.0.2.0/24)
│   ├── ALB
│   └── NAT Gateway-B
│
├── Private App Subnet AZ-A (10.0.11.0/24)
│   └── EC2 / EKS Nodes
│
├── Private App Subnet AZ-B (10.0.12.0/24)
│   └── EC2 / EKS Nodes
│
├── Private DB Subnet AZ-A (10.0.21.0/24)
│   └── RDS Primary
│
└── Private DB Subnet AZ-B (10.0.22.0/24)
    └── RDS Standby
    
traffic flow

User
  ↓
Route53
  ↓
ALB (Public Subnets)
  ↓
Application Servers (Private Subnets)
  ↓
RDS Multi-AZ (Private DB Subnets)

Security Group

ALB SG
 └── Allow 80/443 from Internet

APP SG
 └── Allow 80/443 from ALB SG only

DB SG
 └── Allow 3306 from APP SG only

“I would design a VPC across at least two Availability Zones with separate public, application, and database subnets. Public subnets would host an ALB connected to an Internet Gateway. Application servers would run in private subnets behind Auto Scaling Groups, while the database tier would use RDS Multi-AZ in dedicated private database subnets. NAT Gateways would provide outbound internet access for private resources. Security Groups would restrict communication between tiers, and CloudWatch, WAF, Route 53, and CloudTrail would be integrated for monitoring, security, and operational visibility.”

============================================

9) Your organization has a VPC with multiple subnets. You want to restrict outbound internet access for resources in one subnet, but allow outbound internet access for resources in another subnet. How would you achieve this?

I would use different route tables for different subnets.

Subnet A (Internet Access Allowed)

Associate the subnet with a route table containing:
Destination      Target
10.0.0.0/16      Local
0.0.0.0/0        NAT Gateway (Private Subnet) or
0.0.0.0/0 → Internet Gateway

if it is a public subnet.

This allows outbound internet access.

⸻

Subnet B (Internet Access Restricted)

Associate it with a separate route table:
Destination      Target
10.0.0.0/16      Local
no
0.0.0.0/0
route exists.

As a result, resources can communicate within the VPC but cannot reach the internet.

                    Internet
                        |
                 Internet Gateway
                        |
        ---------------------------------
        |                               |
   Public Subnet                  Public Subnet
        |                               |
        |                          NAT Gateway
        |                               |
   App Subnet A                  App Subnet B
 (Internet Allowed)          (Internet Restricted)

Route Table A                Route Table B
0.0.0.0/0 → NAT             No 0.0.0.0/0 Route

Cross Question 1

Interviewer:

Why not use Security Groups instead?

Answer:

Security Groups are primarily used to control traffic at the instance level.

Although Security Groups can restrict outbound traffic, the recommended network-level control is through route tables.

If no route exists:0.0.0.0/0
traffic never leaves the subnet.

This is more secure and easier to manage.

⸻

Cross Question 2

Interviewer:

Can NACLs be used?

Answer:

Yes.

A NACL can explicitly deny outbound traffic.

Example:Rule 100
DENY
Destination: 0.0.0.0/0
However, using route tables is cleaner because traffic never reaches the internet path.

NACLs are generally an additional layer of control.

Interviewer:

What if the subnet needs access only to AWS services like S3 but not the internet?

Answer:

I would use a VPC Endpoint.

For S3:

Amazon S3

Traffic flow:

Private Subnet
      ↓
VPC Endpoint
      ↓
S3
No NAT Gateway or Internet Gateway required.
Interviewer:

What is the difference between NAT Gateway and Internet Gateway?

Answer:

Internet Gateway
Inbound: Yes
Outbound: Yes Used by public subnets.

NAT gateway
Inbound: No
Outbound: Yes
Used by private subnets.

Private instances can access the internet, but the internet cannot initiate connections back.

Interviewer:

You have 3 subnets:

* Web
* Application
* Database

Which should have internet access?

Answer:

Web Tier

* Public subnet
* Internet access required

Application Tier

* Private subnet
* Outbound via NAT Gateway only

Database Tier

* Private subnet
* No internet access

Typical flow:
Internet
   ↓
ALB
   ↓
Web/App Tier
   ↓
Database Tier
I would associate each subnet with a different route table. For the subnet that requires internet access, I would configure a default route (0.0.0.0/0) to an Internet Gateway or NAT Gateway. For the subnet that should not access the internet, I would remove the default route and allow only local VPC routes. This provides subnet-level control over outbound internet connectivity while maintaining internal VPC communication.”

10) You have a VPC with a public subnet and a private subnet. Instances in the private subnet need to access the internet for software updates. How would you allow internet access for instances in the private subnet?

Ans: To allow internet access for instances in the private subnet, we can use a NAT Gateway or a NAT instance. 
   We would place the NAT Gateway/instance in the public subnet and configure the private subnet route table to send outbound traffic to the NAT Gateway/instance.
   This way, instances in the private subnet can access the internet through the NAT Gateway/instance.

I would deploy a NAT Gateway in the public subnet and associate it with an Elastic IP.

The public subnet route table would have: 0.0.0.0/0 → Internet Gateway
The private subnet route table would have: 0.0.0.0/0 → NAT Gateway
This allows instances in the private subnet to initiate outbound internet connections for software updates while remaining inaccessible from the internet.

                   Internet
                       |
                Internet Gateway
                       |
              Public Subnet (AZ-A)
                       |
                +-------------+
                | NAT Gateway |
                +-------------+
                       |
------------------------------------------------
|                                              |
|                                              |
Private Subnet (AZ-A)                   Private Subnet (AZ-B)
EC2/App Servers                         EC2/App Servers
       |                                       |
       +---------------Outbound----------------+

Cross Question 1

Interviewer:

Why can’t I attach an Internet Gateway directly to a private subnet?

Answer:

Internet Gateways are attached to the VPC, not to individual subnets.

Even if a private subnet has a route to the Internet Gateway, instances still need:

* Public IP or Elastic IP
* Appropriate routing

Private subnet instances typically have no public IPs, so they cannot directly use the Internet Gateway.

Interviewer:

Why is a NAT Gateway placed in a public subnet?

Answer:

Because the NAT Gateway itself needs internet connectivity.

The public subnet route table contains:

0.0.0.0/0 → Internet Gateway

The NAT Gateway uses the Internet Gateway to communicate with external services on behalf of private instances.

⸻

Cross Question 4

Interviewer:

What is the disadvantage of using a NAT Instance?

Answer:

NAT Instance requires:

* Patch management
* OS maintenance
* Scaling management
* High availability setup

NAT Gateway is a managed AWS service and is preferred in production.

⸻

Cross Question 5

Interviewer:

How would you make the NAT architecture highly available?

Answer:

Deploy one NAT Gateway per Availability Zone.

AZ-A → NAT Gateway A
AZ-B → NAT Gateway B
route tables

Private AZ-A → NAT Gateway A
Private AZ-B → NAT Gateway B

This avoids cross-AZ dependency and improves resilience.

Cross Question 6

Interviewer:

Can instances in the private subnet receive inbound traffic from the internet?

Answer:

No.

The NAT Gateway only allows:
Private Instance → Internet

it doesnt allow

Internet → Private Instance This is why NAT is considered more secure.

Interviewer:

How can you avoid NAT Gateway costs for S3 access?

Answer:

Use an S3 Gateway VPC Endpoint.

Benefits:

* Traffic stays within AWS network.
* No NAT charges.
* Improved security.

Use Amazon S3 through a VPC Endpoint instead of routing through the internet.

⸻

Cross Question 9

Interviewer:

What AWS service can help troubleshoot connectivity issues?

Answer:

I would use:

* VPC Flow Logs
* Reachability Analyzer
* CloudWatch Metrics

These help identify routing, security group, or NACL issues.

⸻

Cross Question 10 (Senior Level)

Interviewer:

Instances in the private subnet suddenly cannot access the internet. How would you troubleshoot?

Answer:

I would verify:

1. NAT Gateway status
2. Elastic IP attached to NAT Gateway
3. Route tables
4. Security Groups
5. NACLs
6. Internet Gateway attachment
7. VPC Flow Logs
8. DNS resolution

A common production issue is a route table accidentally modified from: 
0.0.0.0/0 → NAT Gateway incorect target

I would deploy a NAT Gateway in a public subnet and associate an Elastic IP with it. The private subnet route table would send all outbound traffic (0.0.0.0/0) to the NAT Gateway, while the NAT Gateway routes traffic through the Internet Gateway. This allows private instances to download updates and access external services without exposing them directly to the internet.”

10) Your application needs to access AWS services, such as S3 securely within your VPC. How would you achieve this?

Ans: To securely access AWS services within the VPC, we can use VPC endpoints. VPC endpoints allow instances in the VPC to communicate with AWS services privately, without requiring internet gateways or NAT gateways. 
  We can create VPC endpoints for specific AWS services, such as S3 and DynamoDB, and associate them with the VPC. 
  This enables secure and efficient communication between the instances in the VPC and the AWS services.

  Interviewer:

What are the types of VPC Endpoints?

Answer:

Gateway Endpoint

Supported services:

* Amazon S3
* Amazon DynamoDB

Works through route tables.

⸻

Interface Endpoint (PrivateLink)

Supported services:

* AWS Secrets Manager
* Amazon CloudWatch
* Amazon ECR
* AWS Systems Manager

Creates ENIs (Elastic Network Interfaces) in your subnet.

Interviewer:

You want a fully private VPC with no NAT Gateway. What endpoints would you create?

Answer:

Typically:

* S3 Gateway Endpoint
* DynamoDB Gateway Endpoint
* ECR API Interface Endpoint
* ECR DKR Interface Endpoint
* CloudWatch Interface Endpoint
* Systems Manager Interface Endpoint
* Secrets Manager Interface Endpoint
* STS Interface Endpoint

This enables:

* Image pulls
* Logging
* Monitoring
* Secret retrieval
* Remote administration

without any internet connectivity.

⸻

Real Production Scenario

Interviewer:

Have you implemented this in production?

Answer:

Yes.

In a banking environment, EKS worker nodes were deployed in completely private subnets.

We created:

* S3 Gateway Endpoint
* ECR Interface Endpoints
* CloudWatch Endpoint
* Systems Manager Endpoint

As a result:

* No Internet Gateway dependency
* No NAT Gateway charges for AWS service traffic
* Compliance requirements satisfied
* All AWS service communication remained private

⸻


“I would use VPC Endpoints to access AWS services privately from within the VPC. For S3 and DynamoDB, I would create Gateway Endpoints, and for services like Secrets Manager, ECR, CloudWatch, and Systems Manager, I would use Interface Endpoints (PrivateLink). This keeps traffic on the AWS network, improves security, reduces NAT Gateway dependency, and lowers costs.”


11) 9: What is the difference between NACL and Security groups ? Explain with a use case ?

Ans: For example, I want to design a security architecture, I would use a combination of NACLs and security groups. 
  NACL: At the subnet level, I would configure NACLs to enforce inbound and outbound traffic restrictions based on source and destination IP addresses, ports, and protocols. 
  NACLs are stateless and can provide an additional layer of defense by filtering traffic at the subnet boundary.
  SG: At the instance level, I would leverage security groups to control inbound and outbound traffic. Security groups are stateful and operate at the instance level. 
  By carefully defining security group rules, I can allow or deny specific traffic to and from the instances based on the application's security requirements.
  By combining NACLs and security groups, I can achieve granular security controls at both the network and instance level, providing defense-in-depth for the sensitive application.

12) What is the difference between IAM users, groups, roles and policies ?

An IAM User represents a person or application and has long-term credentials. An IAM Group is a collection of users used for centralized permission management. An IAM Role provides temporary credentials and is commonly assumed by AWS services such as EC2, EKS, Lambda, and ECS. An IAM Policy is a JSON document that defines what actions are allowed or denied on specific AWS resources. In production, I prefer IAM Roles over IAM Users wherever possible because they eliminate the need to manage long-term credentials.”

Ans: 1) IAM User: An IAM user is an identity within AWS that represents an individual or application needing access to AWS resources. 
        IAM users have permanent long-term credentials, such as a username and password, or access keys (Access Key ID and Secret Access Key). 
        IAM users can be assigned directly to IAM policies or added to IAM groups for easier management of permissions.
     2) IAM Role: An IAM role is similar to an IAM user but is not associated with a specific individual. 
        Instead, it is assumed by entities such as IAM users, applications, or services to obtain temporary security credentials. 
        IAM roles are useful when you want to grant permissions to entities that are external to your AWS account or when you want to delegate access to AWS resources across accounts. 
        IAM roles have policies attached to them that define the permissions granted when the role is assumed.
     3) IAM Group: An IAM group is a collection of IAM users. By organizing IAM users into groups, you can manage permissions collectively.
        IAM groups make it easier to assign permissions to multiple users simultaneously. Users within an IAM group inherit the permissions assigned to that group.
        For example, you can create a "Developers" group and assign appropriate policies to grant permissions required for developers across your organization.
     4) IAM Policy: An IAM policy is a document that defines permissions and access controls in AWS. IAM policies can be attached to IAM users, 
        IAM roles, and IAM groups to define what actions can be performed on which AWS resources. 
        IAM policies use JSON (JavaScript Object Notation) syntax to specify the permissions and can be created and managed independently of the users, roles, or groups.
        IAM policies consist of statements that include the actions allowed or denied, the resources on which the actions can be performed, and any additional conditions.


13) You have a private subnet in your VPC that contains a number of instances that should not have direct internet access.
    However, you still need to be able to securely access these instances for administrative purposes. How would you set up a bastion host to facilitate this access?

Ans: To securely access the instances in the private subnet, you can set up a bastion host (also known as a jump host or jump box). 
     The bastion host acts as a secure entry point to your private subnet. Here's how you can set up a bastion host:
      1) Create a new EC2 instance in a public subnet, which will serve as the bastion host. Ensure that this instance has a public IP address or is associated with an Elastic IP address for persistent access.
      2) Configure the security group for the bastion host to allow inbound SSH (or RDP for Windows) traffic from your IP address or a restricted range of trusted IP addresses. 
         This limits access to the bastion host to authorized administrators only.
      3) Place the instances in the private subnet and configure their security groups to allow inbound SSH (or RDP) traffic from the bastion host security group.
      SSH (or RDP) into the bastion host using your private key or password. From the bastion host, you can then SSH (or RDP) into the instances in the private subnet using their private IP addresses.





































































