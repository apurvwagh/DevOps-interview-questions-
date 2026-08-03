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

========================================================================

3. When you create a Pod, how does the request flow in your Kubernetes cluster?

1) When I create a Pod using kubectl, the request first goes to the Kubernetes API server. The API server authenticates and authorizes the request and passes it through admission control. If everything is valid, the Pod object is stored in etcd.

2) Initially, the Pod doesn’t have a node assigned. The Kubernetes Scheduler watches for unscheduled Pods and evaluates available nodes based on resource requests, node affinity, taints and tolerations, topology constraints, and other scheduling rules. It selects an appropriate node and records the binding through the API server.

3) Once the Pod is assigned to a node, the kubelet on that node sees the Pod specification and asks the container runtime, such as containerd, to pull the image and start the container. The CNI plugin configures the Pod’s network interface and IP address. If volumes are required, the kubelet also handles volume mounting.

4) After the container starts, the kubelet executes the configured startup, readiness, and liveness probes. Once the Pod becomes Ready, Kubernetes Services can send traffic to it through the appropriate networking and service-routing mechanism.”

   kubectl → API Server → etcd → Scheduler → kubelet → containerd
   Client → Load Balancer → Service → Pod

======================================================================

4. How are High Availability (HA) and scaling managed in your project?

1) We achieve HA by eliminating single points of failure at both the infrastructure and application layers. At the AWS layer, we deploy resources across multiple Availability Zones. EKS worker nodes are distributed across multiple AZs, and the application runs with multiple Pod replicas.

2) At the Kubernetes layer, we use Deployments with multiple replicas, topology spread constraints or pod anti-affinity to distribute Pods across nodes and Availability Zones. We also use PodDisruptionBudgets to maintain the required number of available Pods during voluntary disruptions such as node maintenance.

3) For scaling, we use Horizontal Pod Autoscaler to increase or decrease Pod replicas based on metrics such as CPU, memory, or application-specific metrics. When the cluster doesn’t have enough capacity to schedule new Pods, Karpenter or Cluster Autoscaler provisions additional nodes.

4)For the application entry point, we use an AWS Application Load Balancer to distribute traffic across healthy targets. Readiness probes prevent traffic from being sent to unhealthy Pods, while liveness probes allow Kubernetes to restart containers that are stuck.

5)For the database layer, we use AWS-managed services such as RDS with Multi-AZ for high availability, and appropriate backup and disaster-recovery mechanisms.

So our HA strategy is multi-AZ infrastructure + multiple replicas + traffic distribution + health checks + PDB + autoscaling + database HA and monitoring.”

=====================================================================

5. What is an Admission Controller in Kubernetes?

1) An Admission Controller is a component in Kubernetes that intercepts API requests after authentication and authorization but before the object is persisted in etcd. It allows Kubernetes to validate or modify requests before they are accepted into the cluster.

2) There are two major types: validating admission controllers and mutating admission controllers. A mutating controller can modify the incoming object, while a validating controller checks whether the object meets specific policies and can reject it.

3) For example, when I deploy a Pod, a mutating admission controller could automatically inject a sidecar container or add required labels. A validating admission controller could reject the Pod if it violates security policies, such as running a privileged container or using an unapproved image registry.

4) In production environments, admission controllers are very useful for enforcing security and governance policies consistently across the cluster. Tools such as Kyverno or OPA Gatekeeper can be used to implement custom policies.

5) The important request flow is: the client sends the request to the API server, authentication and authorization happen, admission controllers validate or mutate the request, and only then is the accepted object persisted in etcd.”

One-line distinction to remember

Authentication = Who are you?
Authorization/RBAC = What are you allowed to do?
Admission Controller = Is this request allowed according to cluster policies, and should we modify it before storing it?

==================================================================

6. How does Kubernetes scheduling work?

1) Kubernetes scheduling is the process of selecting the most suitable node for a newly created Pod that doesn’t have a node assigned. The Kubernetes Scheduler watches the API server for unscheduled Pods.

2) First, the scheduler filters the available nodes based on the Pod’s requirements. It considers CPU and memory requests, nodeSelector, node affinity, taints and tolerations, topology constraints, and other scheduling rules. Nodes that don’t satisfy these requirements are eliminated.

3) After filtering, the scheduler scores the remaining nodes based on factors such as resource availability and topology preferences. The highest-scoring suitable node is selected. The scheduler then communicates the binding through the API server, and the kubelet on that node starts the Pod.

4) If no node satisfies the requirements, the Pod remains in Pending state. In that situation, I check kubectl describe pod and the scheduling events to identify the reason, such as insufficient CPU, memory, taints, affinity rules, or lack of available nodes.”

Scheduler selects the node; kubelet is responsible for running the Pod on that node.

======================================================================

7. What is a Static Pod? Can you create a Pod without the kube-apiserver?

1) A Static Pod is a Pod managed directly by the kubelet rather than by the Kubernetes API server, Scheduler, or Deployment controller. The kubelet watches a configured static Pod manifest directory and automatically creates the containers defined in those manifests on that particular node.

2) For example, in a kubeadm cluster, control-plane components such as the API server, scheduler, controller manager, and etcd are commonly deployed as static Pods. Their manifests are typically located under /etc/kubernetes/manifests/.

3)Yes, we can create a Pod without the kube-apiserver by placing a valid Pod manifest in the kubelet’s configured static Pod directory. The kubelet reads the manifest and starts the Pod directly.

4) However, the Pod can still appear in the API server when the API server is available. Kubernetes creates a mirror Pod object for visibility, but the kubelet remains responsible for the actual Pod lifecycle.

This is particularly important for troubleshooting control-plane components because if the API server is down, the kubelet can still manage those static Pods locally.”

/etc/kubernetes/manifests/
        │
        ├── kube-apiserver.yaml
        ├── kube-controller-manager.yaml
        ├── kube-scheduler.yaml
        └── etcd.yaml
        
======================================================================

8. Explain Readiness Probe and Liveness Probe.

1) Readiness and liveness probes both check application health, but they have different purposes.

2) A readiness probe determines whether a Pod is ready to receive traffic. If the readiness probe fails, Kubernetes removes the Pod from the Service’s ready endpoints, but it does not restart the container. This is useful when an application is temporarily unable to serve requests, such as during startup or when it loses a dependency.

3) A liveness probe determines whether the application is stuck or unhealthy and needs to be restarted. If the liveness probe continuously fails according to the configured thresholds, kubelet restarts the container.

4) For applications that take a long time to start, I also use a startup probe. It prevents liveness checks from restarting the application while it is still initializing.

In production, I make sure the probes test meaningful health conditions and don’t make them overly aggressive, because an incorrect liveness probe can create a restart loop and cause an outage.”
        
======================================================================





