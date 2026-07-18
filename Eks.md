1)How do you deploy an application on Kubernetes (EKS)? Explain the complete architecture.

“In our production environment, the application is deployed on an Amazon EKS cluster running inside a highly available VPC across multiple Availability Zones. The VPC contains both public and private subnets. Public subnets host the Internet Gateway, NAT Gateway, and Application Load Balancer (ALB), while the EKS worker nodes and application pods run securely in private subnets so they are not directly exposed to the internet.

When a developer commits code to GitHub, it automatically triggers our CI pipeline in Jenkins  The pipeline checks out the source code, builds the application, runs unit tests, performs code quality analysis with SonarQube, executes security scans using tools like Trivy or Snyk, builds a Docker image, and pushes the image to Amazon ECR.

Once the image is available in ECR, the pipeline updates the Kubernetes manifest or Helm chart with the new image version in the Git repository. Since we follow GitOps, Argo CD continuously monitors the Git repository. It detects the updated manifest and synchronizes the desired state with the EKS cluster without anyone manually running kubectl commands.

The Kubernetes Scheduler then selects a suitable worker node based on CPU, memory, node affinity, taints and tolerations, and resource availability. The worker node pulls the Docker image securely from Amazon ECR using its IAM role or IRSA permissions. Kubernetes creates the new pods and performs Startup, Readiness, and Liveness probes to ensure the application is healthy before sending user traffic.

The application is exposed internally using a Kubernetes Service, and externally through an Ingress resource. The AWS Load Balancer Controller automatically provisions an Application Load Balancer (ALB), configures listeners and target groups, and registers only healthy pods. Route 53 maps our application domain, such as app.company.com, to the ALB, so when users access the application, their requests flow through Route 53, then the ALB, then the Ingress, then the Kubernetes Service, and finally to one of the healthy application pods.

To handle traffic spikes, we use Horizontal Pod Autoscaler (HPA) to automatically scale pods based on CPU or custom metrics, and Karpenter or Cluster Autoscaler provisions additional EC2 worker nodes when required. Throughout the deployment, we monitor infrastructure and application health using CloudWatch, Prometheus, Grafana, and centralized logging tools. This entire process is fully automated, secure, highly available, and minimizes manual intervention while providing reliable and repeatable deployments.”

During deployment, Kubernetes uses a Rolling Update strategy. It creates new pods first, waits until their Readiness probes pass, gradually shifts traffic to them, and only then terminates the old pods. This ensures zero downtime and a seamless user experience. If the deployment fails, Kubernetes or Argo CD can roll back to the previous stable version quickly.

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
     
     ▼                        ▼

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


Scenario 1 (Most Common Interview Question)

Interviewer

Your application suddenly receives 10x more traffic during a sale. How will Cluster Autoscaler behave?

Answer

“When traffic increases, the Horizontal Pod Autoscaler (HPA) first creates additional pods. If the existing worker nodes don’t have enough CPU or memory, these pods remain in a Pending state. Cluster Autoscaler detects the unschedulable pods and increases the desired capacity of the configured Auto Scaling Group. AWS launches new EC2 instances, the nodes join the cluster, and Kubernetes schedules the pending pods onto them. Once traffic decreases and nodes become underutilized, Cluster Autoscaler removes the extra nodes after its scale-down delay.”

Limitation: If the ASG only contains m5.large instances but the workload needs more memory or GPUs, Cluster Autoscaler cannot choose a different instance type—it is limited to the ASG configuration.

⸻

Scenario 2 (Karpenter)

Interviewer

The application suddenly requires a high-memory node. What happens with Karpenter?

Answer

“Karpenter watches for pending pods. It reads the pod’s resource requests, node selectors, taints, tolerations, and affinity rules. Instead of increasing an existing ASG, Karpenter directly launches the most appropriate EC2 instance type—for example, an r7g.xlarge if the workload is memory intensive or a c7g.large if it is CPU intensive. This avoids over-provisioning, reduces scheduling delays, and optimizes AWS costs.”

⸻

Scenario 3 (Real Production)

Interviewer

Your company runs 300 microservices on EKS. During business hours, traffic is normal, but at night only a few applications receive traffic. Which solution would you choose?

Answer

“I would choose Karpenter because workloads are dynamic. Karpenter provisions only the capacity needed by the current workloads and terminates unused nodes quickly. This provides better utilization, lower AWS costs, and faster scaling than managing multiple fixed Auto Scaling Groups.”

⸻

Scenario 4

Interviewer

Why do many organizations migrate from Cluster Autoscaler to Karpenter?

Answer

“Cluster Autoscaler requires maintaining multiple Auto Scaling Groups for different instance types and sizes, which increases operational complexity. Karpenter eliminates that requirement by selecting the optimal EC2 instance type automatically based on workload requirements. It also supports Spot and On-Demand instances intelligently, resulting in faster provisioning and better cost optimization.”

⸻

Real Production Example

Imagine an e-commerce platform during Black Friday.

* Normally:
    * 20 worker nodes
    * 100 pods
* Sale starts:
    * HPA scales to 500 pods.
    * Existing nodes become full.

With Cluster Autoscaler

* Detects pending pods.
* Scales ASG from 20 → 60 nodes.
* All new nodes are the same predefined instance type.
* Some workloads may waste resources if that instance type is oversized.

With Karpenter

* Detects pending pods.
* Launches a mix of instance types:
    * CPU-optimized for API services.
    * Memory-optimized for Redis.
    * Spot instances for batch jobs.
* Only the required capacity is provisioned, reducing both scaling time and cost.

⸻

Cross Questions

Q1. Does Karpenter replace HPA?

Answer: No. HPA scales pods, while Karpenter scales nodes. They work together.

⸻

Q2. Can HPA work without Cluster Autoscaler or Karpenter?

Answer: Yes, but only while enough node capacity exists. If all nodes are full, new pods remain in the Pending state.

⸻

Q3. Can Karpenter use Spot Instances?

Answer: Yes. It can provision Spot, On-Demand, or Reserved Capacity based on the policies you define, helping reduce infrastructure costs.

⸻

Q4. Can Cluster Autoscaler and Karpenter run together?

Answer: They can, but it’s generally not recommended to have both manage the same workloads because they may make conflicting scaling decisions. If used together, they should manage separate node groups or provisioning strategies.

⸻

Interview Closing Answer (2 minutes)

“Cluster Autoscaler and Karpenter both solve the node scaling problem, but Cluster Autoscaler scales predefined Auto Scaling Groups, whereas Karpenter provisions the most suitable EC2 instances dynamically based on workload requirements. In modern EKS environments with highly dynamic workloads, I prefer Karpenter because it provides faster provisioning, supports intelligent instance selection, improves resource utilization, and significantly reduces AWS costs. However, for existing environments already built around Auto Scaling Groups, Cluster Autoscaler remains a stable and proven solution.”

===========================================================

5) From Domain to Deployment: End-to-End Request Flow in Amazon EKS

“In our production environment, the application is hosted on Amazon EKS inside a highly available VPC  multiple Availability Zones. Users access the application using a domain name such as www.company.com. When a user enters this URL in the browser, the DNS request first reaches Amazon Route 53. Route 53 resolves the domain name and returns the DNS name of the Application Load Balancer (ALB).”

“The request is then sent to the internet through the Internet Gateway and reaches the ALB, which is deployed in the public subnets. The ALB terminates the SSL/TLS connection using certificates managed by AWS Certificate Manager (ACM) and performs health checks on backend targets. Based on the listener rules or path-based routing, the ALB forwards the request to the Kubernetes Ingress resource managed by the AWS Load Balancer Controller.”

“The Ingress maps the request to the appropriate Kubernetes Service. The Service provides a stable virtual IP and load balances traffic across all healthy pods belonging to the application. It continuously tracks healthy pod endpoints and automatically removes pods that fail their readiness probes, ensuring traffic is only sent to healthy application instances.”

“The selected pod runs on an EC2 worker node inside a private subnet. Since the worker nodes are not exposed to the internet, they communicate externally through the NAT Gateway whenever outbound internet access is required, such as pulling Docker images from Amazon ECR or downloading software updates. The application inside the pod processes the request and, if necessary, connects securely to backend services such as Amazon RDS, ElastiCache, or other microservices within the cluster.”

“The response generated by the application follows the same path in reverse—from the pod to the Kubernetes Service, then to the Ingress, back to the ALB, and finally through Route 53 to the user’s browser. Throughout this process, Kubernetes continuously performs Readiness and Liveness probes, while the ALB performs its own health checks. If a pod becomes unhealthy, Kubernetes removes it from the Service endpoints and the ALB automatically stops sending traffic to it.”

“On the deployment side, developers commit code to GitHub, which triggers a CI pipeline in Jenkins or GitHub Actions. The pipeline builds the application, runs automated tests, performs security scans, creates a Docker image, and pushes it to Amazon ECR. The updated Kubernetes manifests or Helm charts are committed to Git, where Argo CD detects the changes and synchronizes them with the EKS cluster. Kubernetes performs a rolling update by creating new pods, validating them through health probes, gradually shifting traffic, and removing the old pods only after the new version is confirmed healthy. If traffic increases, the Horizontal Pod Autoscaler scales the number of pods, while Karpenter or Cluster Autoscaler provisions additional EC2 worker nodes. This complete architecture provides automation, high availability, scalability, security, and zero-downtime deployments.”


6) Explain your CI/CD pipeline in your current project.

Interview Answer

“In my current project, we follow a GitOps-based CI/CD pipeline using GitHub, Jenkins, Docker, Amazon ECR, Argo CD, and Amazon EKS. 
“The process starts when a developer creates a feature branch, implements the code changes, and raises a Pull Request. Before the PR is merged, mandatory code reviews are performed and automated checks such as unit tests and SonarQube quality gates are executed. Once the PR is approved and merged into the main branch, GitHub triggers a Jenkins pipeline using webhooks.”

“Jenkins first checks out the latest source code from GitHub and builds the application using Maven. It then executes unit tests and generates the application artifact (JAR). Next, SonarQube performs static code analysis to detect bugs,duplicated code, and security vulnerabilities. 

“After the quality checks, Jenkins builds a Docker image using the application’s Dockerfile. The image is then scanned using Trivy to identify any critical operating system or vulnerabilities. If critical vulnerabilities are detected, the build is failed and the image is not pushed further.”

“Once the image passes all security checks, Jenkins authenticates with Amazon ECR using an IAM role and pushes the versioned Docker image to the ECR repository. The pipeline then updates the Helm chart or Kubernetes deployment manifest with the new image tag and commits that change to our GitOps repository.”

“Argo CD continuously monitors the GitOps repository. As soon as it detects the updated image tag, it compares the desired state stored in Git with the actual state of the EKS cluster and automatically synchronizes the changes. Kubernetes then performs a rolling update by creating new pods, waiting for their startup and readiness probes to succeed, gradually routing traffic to the new version, and finally terminating the old pods. This ensures zero downtime during deployment.”

“After deployment, we validate the rollout by checking pod status, rollout history, application logs, ALB target health, and Grafana dashboards. If any issue is detected, Argo CD or Kubernetes allows us to quickly roll back to the previous stable version. Throughout the pipeline, notifications about build status, deployment success, or failures are sent to Microsoft Teams or Slack, ensuring the team is informed in real time.”














