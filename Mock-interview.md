1. Explain your CI/CD flow in your current project.

1) In my current project, we follow a Git-based CI/CD and GitOps approach. Developers push code to Git, which triggers the CI pipeline. The pipeline performs code quality checks, security scanning, unit tests, builds the application, creates a Docker image, and pushes it to Amazon ECR.

2) For deployment, we use Kubernetes on Amazon EKS. After the image is available in ECR, the deployment configuration is updated through our GitOps repository. Argo CD detects the change and synchronizes the desired state with the EKS cluster.

3) Before production, we typically deploy to lower environments such as development and staging, perform automated tests and validation, and then promote the release to production based on the required approval process.

4) During deployment, Kubernetes performs a rolling update using readiness probes, liveness probes, replica configuration, and PodDisruptionBudgets to minimize downtime. After deployment, we monitor application and infrastructure metrics using Prometheus, Grafana, CloudWatch, and application logs. If there is an issue, we can roll back to the previous application version through GitOps or Kubernetes rollout mechanisms.

So overall, my flow is: Developer → Git → CI pipeline → Build/Test/Security Scan → Docker Image → ECR → GitOps Repository → Argo CD → EKS → Monitoring and Alerting.”

========================================================================

2. Explain your current project architecture and application traffic flow.

1) Our application follows a highly available, containerized microservices architecture running on AWS. We use a multi-AZ VPC with public and private subnets. The EKS worker nodes run in private subnets, while internet-facing components such as the Application Load Balancer are deployed in public subnets.

2) When a user accesses the application, DNS resolves the application domain through Route 53 to the Application Load Balancer. The ALB terminates HTTPS and uses listener rules to route the request to the appropriate Kubernetes service. The Kubernetes Service then distributes traffic to healthy application Pods.

3) For example, frontend traffic may go to the frontend Pods, while API requests are routed to backend microservices. The backend communicates with databases and other internal services through private networking. Databases such as RDS are kept in private subnets and are not directly accessible from the internet.

4) For high availability, we distribute EKS nodes and Pods across multiple Availability Zones and use multiple replicas, PodDisruptionBudgets, readiness probes, and autoscaling. We use HPA for Pod-level scaling and Karpenter or Cluster Autoscaler for node-level scaling.

For observability, we use Prometheus and Grafana for metrics, CloudWatch for AWS infrastructure monitoring, and centralized logging for application troubleshooting.”
