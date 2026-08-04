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

4) For the application entry point, we use an AWS Application Load Balancer to distribute traffic across healthy targets. Readiness probes prevent traffic from being sent to unhealthy Pods, while liveness probes allow Kubernetes to restart containers that are stuck.

5) For the database layer, we use AWS-managed services such as RDS with Multi-AZ for high availability, and appropriate backup and disaster-recovery mechanisms.

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

3) Yes, we can create a Pod without the kube-apiserver by placing a valid Pod manifest in the kubelet’s configured static Pod directory. The kubelet reads the manifest and starts the Pod directly.

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

9. What is RBAC (Role-Based Access Control) in Kubernetes?

1) RBAC in Kubernetes controls who can perform which actions on which Kubernetes resources. It follows the principle of least privilege.

2) The main RBAC objects are Role, ClusterRole, RoleBinding, and ClusterRoleBinding. A Role defines permissions within a specific namespace. A ClusterRole can define permissions at the cluster level or be reused across namespaces. RoleBinding connects a user, group, or ServiceAccount to a Role, while ClusterRoleBinding grants a ClusterRole at the cluster level.

3) Permissions are defined using verbs such as get, list, watch, create, update, patch, and delete against resources such as Pods, Deployments, or Secrets.

4) For example, if I want a developer to view Pods in the dev namespace but not delete them, I create a Role with get, list, and watch permissions and bind it to the developer’s identity.

In production, I avoid giving users cluster-admin unless absolutely necessary because it provides extremely broad permissions.”

======================================================================

10. How would you upgrade an Amazon EKS/Kubernetes cluster?

1) For an EKS upgrade, I follow a controlled and phased approach because the goal is to upgrade the control plane and worker nodes without causing application downtime.

2) First, I review the EKS version support matrix and Kubernetes release notes, identify deprecated APIs, and check application and add-on compatibility. I also test the upgrade in a non-production environment first.

3) Before production, I take backups of important Kubernetes resources and verify that critical workloads have multiple replicas, appropriate readiness probes, and PodDisruptionBudgets. I also verify the compatibility of CoreDNS, kube-proxy, VPC CNI, EBS CSI driver, ingress controller, and other critical add-ons.

4) Then I upgrade the EKS control plane. After that, I upgrade the managed add-ons and create or upgrade the worker node groups. For a safer production upgrade, I prefer a new node group with the new Kubernetes version, gradually move workloads to it using cordon and drain, and then remove the old node group.

5) During the upgrade I monitor Pod health, node health, application metrics, ALB target health, error rates, latency, and logs. After the upgrade, I perform smoke tests and verify that all workloads are healthy. If there is an application-level issue, I stop the rollout and use the appropriate rollback or remediation procedure.”

===================================================================

11. How would you distribute 10 Pods evenly across 10 Kubernetes nodes?

1) If I have 10 Pods and 10 nodes and my requirement is to distribute one Pod per node as evenly as possible, I would use Kubernetes topology spread constraints. This is preferable to manually assigning Pods to individual nodes because Kubernetes can maintain the distribution automatically as Pods and nodes change.

2) I would use topologySpreadConstraints with topologyKey: kubernetes.io/hostname and configure maxSkew: 1. I would also use whenUnsatisfiable: DoNotSchedule if it is a strict requirement that Pods must not be placed together on the same node.

3) Another option is Pod anti-affinity, using requiredDuringSchedulingIgnoredDuringExecution, but topology spread constraints are generally better when the requirement is specifically to distribute workloads evenly.

4) In production, I would also consider distributing Pods across Availability Zones using the zone topology label, because spreading across nodes alone doesn’t necessarily provide AZ-level high availability.”

===================================================================

12. My Kubernetes Pod is stuck in the Pending state. How would you troubleshoot it?

1) If a Pod is stuck in Pending, my first assumption is that the scheduler couldn’t find a suitable node, although Pending can also occur while waiting for resources such as a PVC. I first run kubectl describe pod and check the Events section because Kubernetes usually tells me why scheduling failed.

2) I check for insufficient CPU or memory, nodeSelector or node affinity rules, taints and tolerations, topology constraints, and whether the required nodes are Ready. I also check whether a PersistentVolumeClaim is still Pending.

3) Then I verify the cluster capacity using kubectl get nodes and kubectl describe nodes, and check resource requests and limits. If the Pod cannot be scheduled because the cluster has insufficient capacity, I determine whether Cluster Autoscaler or Karpenter can provision additional nodes.

4) If autoscaling doesn’t happen, I check whether the Pod’s scheduling requirements can actually be satisfied by the node group’s labels, instance types, taints, capacity limits, or cloud provider quotas.

I don’t immediately delete the Pod because the scheduler is already trying to schedule it. I first identify the scheduling constraint causing the Pending state.”

Cause

Example

Insufficient CPU
Insufficient memory
Taint - Pod lacks toleration
Affinity - No matching node
NodeSelector -Label doesn’t exist
PVC -Volume cannot be provisioned
Topology - AZ/node constraint impossible
Autoscaler -Cannot provision matching node

===================================================================

13. A Pod is running, but the application is not working. How would you troubleshoot it?

1) A Pod being in Running state only tells me that the container process has started; it doesn’t mean the application is healthy or receiving traffic. I troubleshoot from the application layer toward the networking layer.

2) First, I check Pod status, readiness and liveness probe results, and Pod events using kubectl describe pod. Then I check application logs using kubectl logs and, if necessary, kubectl logs --previous to identify crashes or dependency failures.

3) Next, I verify that the application is actually listening on the expected port inside the container and test it locally using curl. I check the Kubernetes Service and its endpoints to ensure the Service is selecting the correct Pods.

4) If the Service has no endpoints, I verify the Service selector against the Pod labels and check readiness status. If endpoints exist but traffic still fails, I investigate NetworkPolicies, kube-proxy/CNI networking, Ingress or Load Balancer configuration, and security controls.

Finally, I check downstream dependencies such as databases, APIs, and external services because the Pod may be healthy from Kubernetes’ perspective while the application is unable to serve requests because a dependency is unavailable.”

===================================================================

14. A production Pod was OOMKilled at 2 AM. How would you investigate and resolve the issue?

1) If a production Pod is OOMKilled, I first confirm whether the container exceeded its Kubernetes memory limit or whether the node itself experienced memory pressure. I check the Pod status and previous container termination details using kubectl describe pod and kubectl get pod.

2) Then I review the application’s memory usage before the incident. I check metrics from Prometheus, Grafana, or CloudWatch to determine whether memory gradually increased, suddenly spiked, or increased because of traffic.

3) I also check the configured requests and limits. If the application consistently needs more memory than the configured limit, the limit may simply be too low. However, blindly increasing the limit isn’t always the correct solution because the application could have a memory leak.

4) I review application logs, heap usage, garbage collection metrics, request volume, and recent deployments. If the memory usage continuously increases over time, I investigate a possible memory leak. If the spike correlates with traffic, I evaluate HPA configuration and whether scaling should happen earlier.

5) For immediate recovery, I may increase the memory limit if justified and scale replicas if traffic is high. For permanent resolution, I fix the application memory issue, tune resource requests and limits, configure appropriate autoscaling, and add memory-based alerts.”

 Container OOM vs Node OOM

If the container exceeds its configured memory limit → Kubernetes/container runtime can terminate it.

If the node is under severe memory pressure → the kubelet may evict Pods based on eviction behavior and QoS.

===================================================================

15. A wrong Liveness/Readiness Probe configuration caused production downtime. How would you fix and prevent this?

1) First, I would stabilize production by correcting the probe configuration and ensuring healthy Pods are available. I would determine whether the issue was caused by an overly aggressive liveness probe, an incorrect readiness endpoint, wrong port, insufficient startup time, or an inappropriate timeout.

2) If the liveness probe is failing while the application is actually healthy, Kubernetes may continuously restart containers and create a restart loop. If the readiness probe is incorrect, healthy Pods may be removed from Service endpoints and users can receive 503 errors.

3) For applications with slow startup, I would use a startup probe so Kubernetes gives the application enough time to initialize before liveness checking begins. I would also make the readiness endpoint represent whether the application can actually serve traffic, while liveness should indicate whether the application is fundamentally stuck and needs a restart.

4) For prevention, I would test probes in staging under realistic startup and load conditions, use sensible initialDelaySeconds, timeoutSeconds, periodSeconds, and failure thresholds, and include probe validation in CI/CD. I would also monitor restart counts, readiness failures, and application error rates during deployments.”

==================================================================

16. One node was drained without a PodDisruptionBudget (PDB), causing downtime. How would you recover and prevent this?

1) First, I would restore application availability by checking which Pods were evicted during the drain and whether the workload has enough replicas running on other nodes. I would check the node and affected Pods, then verify Deployment and Service health.

2) If the application has insufficient replicas, I would scale the Deployment if required and make sure replacement Pods are scheduled on healthy nodes. If the cluster doesn’t have enough capacity, I would add nodes or allow Karpenter/Cluster Autoscaler to provision capacity.

3) For prevention, I would define a PodDisruptionBudget for critical applications. A PDB specifies how many Pods must remain available during voluntary disruptions such as node drain, node maintenance, or cluster upgrades.

4) I would also run multiple replicas and distribute them across nodes and Availability Zones using topology spread constraints or pod anti-affinity. During maintenance, I would use kubectl drain carefully and verify PDB behavior before proceeding.

5) One important point is that a PDB protects against voluntary disruptions; it doesn’t protect against every failure, such as a sudden node crash. That’s why PDB must be combined with replicas and multi-AZ distribution.”

==================================================================










