1)How do you deploy an application on Kubernetes (EKS)? Explain the complete architecture.

“In our production environment, the application is deployed on an Amazon EKS cluster running inside a highly available VPC spread across multiple Availability Zones. The VPC contains both public and private subnets. Public subnets host the Internet Gateway, NAT Gateway, and Application Load Balancer (ALB), while the EKS worker nodes and application pods run securely in private subnets so they are not directly exposed to the internet.

When a developer commits code to GitHub, it automatically triggers our CI pipeline in Jenkins or GitHub Actions. The pipeline checks out the source code, builds the application, runs unit tests, performs code quality analysis with SonarQube, executes security scans using tools like Trivy or Snyk, builds a Docker image, and pushes the image to Amazon ECR.

Once the image is available in ECR, the pipeline updates the Kubernetes manifest or Helm chart with the new image version in the Git repository. Since we follow GitOps, Argo CD continuously monitors the Git repository. It detects the updated manifest and synchronizes the desired state with the EKS cluster without anyone manually running kubectl commands.

The Kubernetes Scheduler then selects a suitable worker node based on CPU, memory, node affinity, taints and tolerations, and resource availability. The worker node pulls the Docker image securely from Amazon ECR using its IAM role or IRSA permissions. Kubernetes creates the new pods and performs Startup, Readiness, and Liveness probes to ensure the application is healthy before sending user traffic.

The application is exposed internally using a Kubernetes Service, and externally through an Ingress resource. The AWS Load Balancer Controller automatically provisions an Application Load Balancer (ALB), configures listeners and target groups, and registers only healthy pods. Route 53 maps our application domain, such as app.company.com, to the ALB, so when users access the application, their requests flow through Route 53, then the ALB, then the Ingress, then the Kubernetes Service, and finally to one of the healthy application pods.

During deployment, Kubernetes uses a Rolling Update strategy. It creates new pods first, waits until their Readiness probes pass, gradually shifts traffic to them, and only then terminates the old pods. This ensures zero downtime and a seamless user experience. If the deployment fails, Kubernetes or Argo CD can roll back to the previous stable version quickly.

To handle traffic spikes, we use Horizontal Pod Autoscaler (HPA) to automatically scale pods based on CPU or custom metrics, and Karpenter or Cluster Autoscaler provisions additional EC2 worker nodes when required. Throughout the deployment, we monitor infrastructure and application health using CloudWatch, Prometheus, Grafana, and centralized logging tools. This entire process is fully automated, secure, highly available, and minimizes manual intervention while providing reliable and repeatable deployments.”

Interview Cross Question

Interviewer: Why do we need Route 53?

Answer:

“Route 53 provides DNS resolution. Instead of users accessing a changing ALB DNS name, they use a stable domain like app.company.com, which Route 53 maps to the ALB.”

⸻

Interview Cross Question

Interviewer: Why do we need Amazon ECR?

Answer:

“ECR is the container image registry. The CI pipeline pushes versioned Docker images there, and Kubernetes pulls the required image when creating new pods.”

⸻

Interview Cross Question

Interviewer: Why do we need Argo CD if Jenkins already deployed the image?

Answer:

“Jenkins is responsible for Continuous Integration—building, testing, scanning, and publishing the image. Argo CD handles Continuous Deployment using GitOps. It continuously compares the cluster state with Git and automatically synchronizes changes, making deployments declarative, auditable, and easy to roll back.”

Developer
     │
     ▼
 GitHub Repository
     │
     ▼
Jenkins / GitHub Actions
     │
     ├── Build Application
     ├── Run Unit Tests
     ├── SonarQube Scan
     ├── Trivy Security Scan
     ├── Build Docker Image
     ▼
Amazon ECR
     │
     ▼
Update Helm Chart / Kubernetes Manifest
     │
     ▼
Argo CD (GitOps)
     │
     ▼
Amazon EKS Cluster
     │
     ▼
Kubernetes Scheduler
     │
     ▼
EC2 Worker Nodes (Private Subnets)
     │
     ▼
Application Pods
     │
     ▼
Kubernetes Service
     │
     ▼
Ingress
     │
     ▼
AWS Load Balancer Controller
     │
     ▼
Application Load Balancer (Public Subnet)
     │
     ▼
Route 53
     │
     ▼
Users


……………………………………………………………………………………………………………………………………………

2) How would you architect a 3-tier application in AWS?

Interview Answer

“I would design the application using a highly available and secure architecture across at least two Availability Zones within a VPC. The VPC would contain public and private subnets to separate internet-facing components from backend services. The presentation tier (Web), application tier, and database tier would each be isolated for security and scalability.”

“In the public subnets, I would deploy an Internet Gateway, NAT Gateways, and an Application Load Balancer (ALB). Route 53 would provide DNS resolution and route user traffic to the ALB. If HTTPS is required, I would use AWS Certificate Manager (ACM) to manage SSL/TLS certificates.”

“The web and application layers would run on Amazon EC2 instances in Auto Scaling Groups or on Amazon EKS if the application is containerized. These instances would be deployed in private subnets across multiple Availability Zones. The ALB distributes incoming traffic to healthy application instances using health checks, ensuring high availability and fault tolerance.”

“For the database tier, I would use Amazon RDS Multi-AZ deployed in private subnets. The database would not be publicly accessible and would only accept connections from the application tier through tightly controlled Security Groups. For better performance, I could add RDS Read Replicas for read-heavy workloads.”

“Private instances requiring internet access for updates or pulling container images would use a NAT Gateway rather than public IP addresses. Security Groups would restrict communication between tiers—for example, the ALB can communicate with the application tier, and only the application tier can communicate with the database. Network ACLs provide an additional layer of subnet-level security.”

“For storage, I would use Amazon S3 for static content and backups, Amazon ECR for container images, and IAM roles to provide secure access to AWS services without storing credentials. Secrets such as database passwords would be stored in AWS Secrets Manager. Monitoring would be implemented using CloudWatch, Prometheus, and Grafana, with logs centralized in CloudWatch Logs or OpenSearch. This architecture provides high availability, scalability, security, fault tolerance, and supports zero-downtime deployments.”

                Users
                  │
                  ▼
             Route 53 (DNS)
                  │
                  ▼
          Application Load Balancer
                  │
     ┌────────────┴────────────┐
     ▼                         ▼
 Availability Zone A     Availability Zone B
 ┌─────────────────┐     ┌─────────────────┐
 │  Public Subnet  │     │  Public Subnet  │
 │  ALB + NAT GW   │     │  ALB + NAT GW   │
 └─────────────────┘     └─────────────────┘
           │                       │
           ▼                       ▼
 ┌─────────────────┐     ┌─────────────────┐
 │ Private Subnet  │     │ Private Subnet  │
 │ App EC2 / EKS   │     │ App EC2 / EKS   │
 │ Auto Scaling    │     │ Auto Scaling    │
 └─────────────────┘     └─────────────────┘
           │                       │
           └───────────┬───────────┘
                       ▼
           ┌──────────────────────┐
           │   RDS Multi-AZ        │
           │  Private Subnets      │
           └──────────────────────┘

 Common Cross Questions

Q1. Why keep EC2/EKS in private subnets?

Answer: To prevent direct internet access. All inbound traffic comes through the ALB, improving security. Outbound internet access is provided via the NAT Gateway.

⸻

Q2. Why use an ALB instead of exposing EC2 directly?

Answer: The ALB distributes traffic across multiple instances, performs health checks, supports SSL termination, path-based routing, and enables zero-downtime deployments.

⸻

Q3. Why use Auto Scaling?

Answer: It automatically adds or removes instances based on demand, ensuring performance during traffic spikes while reducing costs during low usage.

⸻

Q4. Why use Multi-AZ for RDS?

Answer: Multi-AZ provides high availability. If the primary database or its Availability Zone fails, AWS automatically fails over to the standby instance with minimal downtime.

⸻

Q5. Why use a NAT Gateway?

Answer: Instances in private subnets need outbound internet access for tasks such as OS updates or pulling images from ECR. The NAT Gateway allows outbound connectivity without exposing those instances to inbound internet traffic.

⸻

What interviewers expect

As a Senior DevOps Engineer, don’t just list AWS services. Explain:

* Why you selected each service.
* Where it is placed in the architecture.
* How traffic flows through the system.
* How you achieve high availability, security, scalability, and fault tolerance.
* 
* ========≠======================================
 
3) Kubernetes Cluster Autoscaler vs Karpenter 🚀

One of the most common questions in Kubernetes interviews and production environments is:

“Why use Karpenter when Kubernetes already has Cluster Autoscaler?”

Both solve the same problem: scaling Kubernetes worker node, but they do it in very different ways.

🔹 Cluster Autoscaler (CA)

Cluster Autoscaler works with Auto Scaling Groups (ASGs).

How it works

1. A pod becomes Pending because there aren’t enough resources.
2. Kubernetes marks the pod as unschedulable.
3. Cluster Autoscaler checks the configured ASGs.
4. It increases the size of an existing ASG.
5. AWS launches a new EC2 instance.
6. The node joins the cluster, and Kubernetes schedules the pending pod.

Pros

✅ Mature and widely adopted
✅ Easy to configure if you are already using Managed Node Groups
✅ Stable for predictable workloads

Limitations

* Limited to predefined Auto Scaling Groups.
* Instance types are restricted by the ASG configuration.
* Scale-up can be slower because it depends on ASG operations.
* Can lead to over-provisioning.

⸻

🔹 Karpenter

Instead of relying on Auto Scaling Groups, it communicates directly with the AWS EC2 API.

How it works

1. A pod becomes Pending.
2. Karpenter watches for unschedulable pods.
3. It evaluates the pod’s CPU, memory, labels, taints, tolerations, and affinity requirements.
4. It matches those requirements against the configured NodePool.
5. Using the associated EC2NodeClass, it selects the best EC2 instance type.
6. AWS launches the instance.
7. The node joins the cluster.
8. Kubernetes schedules the pod automatically.

Because Karpenter isn’t constrained by predefined Auto Scaling Groups, it can choose the most suitable instance type for the workload.

⸻

🔹 Why many teams are adopting Karpenter

Compared to Cluster Autoscaler, Karpenter offers:

✅ Faster node provisioning
✅ Dynamic selection of EC2 instance types
✅ Native support for Spot and On-Demand capacity
✅ Built-in consolidation to remove underutilized nodes

_____

🔹 Real-world use case

In one of our lower Kubernetes environments example sbox, we combined Karpenter with KubeDownscaler.

* KubeDownscaler automatically scaled non-production workloads to zero replicas outside business hours or in weekend.
* Once those pods terminated, Karpenter detected idle and underutilized nodes.
* Using consolidation, Karpenter removed unnecessary ec2 instances node.

This approach significantly improved cluster utilization and helped reduce AWS infrastructure costs while ensuring production workloads remained unaffected.

⸻

📌 Key takeaway

* Cluster Autoscaler is a great choice for stable, predictable workloads built around Auto Scaling Groups.
* Karpenter is ideal for modern, cloud-native Kubernetes platforms where speed, flexibility, and cost optimization are priorities.









