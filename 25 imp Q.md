Q1. You have 20 microservices running in an AKS cluster. Will you create one LoadBalancer Service for each application?

“No, I would generally not create a separate Azure LoadBalancer Service for every microservice. That would create unnecessary load balancers, increase cost, and make traffic management harder to operate.

For 20 microservices, I would normally use an Ingress-based architecture. I would expose the application through a single entry point, such as an Azure Application Gateway or an NGINX Ingress Controller, and use host-based or path-based routing to send requests to the appropriate Kubernetes Services.

Each microservice would typically have an internal Kubernetes Service, usually ClusterIP, and the Ingress would route external traffic to those Services.

For example, api.company.com/orders can route to the Orders Service, while api.company.com/payments routes to the Payments Service.

I would use a LoadBalancer Service directly when a service genuinely needs its own external endpoint—for example, a public TCP service, a dedicated network endpoint, or a requirement that cannot be handled by the Ingress layer.”

==============≠=======================================================

Q2. Your EKS cluster is private, but internet users should still be able to access your application. How would you design the traffic flow?

🎯 2-Minute Interview Answer

“I would keep the EKS worker nodes and Pods in private subnets, but expose the application through a public Application Load Balancer. I would use the AWS Load Balancer Controller with an Ingress to provision and manage the ALB.

The internet user first resolves the application domain through Route 53. Route 53 points to the public ALB, which is deployed across multiple public subnets/AZs. The ALB then forwards traffic to the application Pods running in the private EKS subnets.

I would configure security groups so that the ALB accepts internet traffic on HTTPS, while the application security group allows traffic only from the ALB security group. The Pods themselves would not have public IPs or direct internet exposure.

For outbound internet access from private Pods, I would use a NAT Gateway where required. For AWS services such as S3 or ECR, I would use VPC endpoints where appropriate to reduce NAT dependency and keep traffic private.”

==============≠=======================================================

Q3. Two DevOps Engineers Accidentally Ran terraform apply at the Same Time. How Would You Prevent State Corruption?

🎯 2-Minute Interview Answer

“I would prevent this by using a remote Terraform backend with state locking. In AWS, I would typically store the Terraform state in an S3 backend and enable state locking using the supported locking mechanism.

When Engineer A runs terraform apply, Terraform acquires the state lock. If Engineer B tries to run terraform apply against the same state at the same time, Terraform will detect that the state is locked and prevent the second operation from proceeding.

I would also enforce CI/CD-based Terraform execution rather than allowing engineers to run production applies directly from their laptops. The pipeline can provide approvals, concurrency control, and consistent credentials.

For team environments, I would separate state by environment and component—for example, dev, staging, and production—so unrelated changes don’t compete for the same state file.”

=============≠=======================================================



