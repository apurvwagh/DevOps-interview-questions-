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
