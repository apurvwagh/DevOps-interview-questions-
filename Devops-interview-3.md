Devops mock interview Q

1. Your CI/CD pipeline takes 30 minutes. How would you reduce it to under 5 minutes?

“I wouldn’t immediately start optimizing the pipeline. My first step would be to analyze stage execution times to identify the bottleneck. Based on that, I would implement dependency caching for Maven or npm, enable Docker layer caching, parallelize independent stages such as unit tests, SonarQube, and Trivy scans, use shallow Git clones, optimize Docker images, and scale Jenkins agents to execute jobs concurrently. I would also separate fast CI checks from longer-running integration or performance tests where appropriate. These optimizations typically reduce pipeline execution time significantly while maintaining code quality and security.”

    2. Your cloud bill suddenly increased by 40% overnight. How would you investigate it?

My approach is to first identify which AWS service caused the increase using Cost Explorer. After identifying the service, I investigate the underlying usage metrics and recent infrastructure or application changes using CloudWatch, CloudTrail, deployment history, and service-specific dashboards. For example, if EC2 costs increased, I verify Auto Scaling, Karpenter, and EKS node activity. If NAT Gateway costs increased, I investigate outbound traffic and check whether VPC Endpoints could reduce those charges. If RDS costs increased, I review instance modifications, storage growth, and Read Replica usage. Once the root cause is identified, I implement corrective actions such as rightsizing, improving autoscaling policies, using Spot Instances, enabling VPC Endpoints, cleaning up unused resources, and configuring AWS Budgets and Cost Anomaly Detection to prevent similar issues in the future.”

    3. Your Kubernetes cluster is healthy, but requests intermittently return 503. How would you troubleshoot it?

When users receive intermittent 503 errors, I first determine whether the issue is at the load balancer, Kubernetes, or application layer. I start by checking pod readiness because a pod can be Running but not Ready. Then I verify Service endpoints, label selectors, Ingress configuration, and ALB target health. If those are healthy, I review application logs, resource utilization, and dependency health such as RDS or Redis. In one production scenario, we found that the Readiness Probe was configured too aggressively for a Spring Boot application, causing healthy pods to be removed from the Service before startup completed. Adjusting the startup and readiness probe configuration resolved the issue. My approach is always to trace the request end-to-end—from the ALB to the application—to identify the exact point of failure.”

User → Route 53 → ALB → Ingress → Service → Endpoints → Pods → Application → Database/Dependencies

4. How do you perform a zero-downtime Kubernetes cluster upgrade in production?

In production, an EKS upgrade is a carefully planned activity to ensure zero downtime. First, I verify that the control plane, worker nodes, and kubectl are within the supported Kubernetes version skew, review the release notes, check for deprecated APIs, and validate compatibility of Helm charts, EKS add-ons, CSI drivers, AWS Load Balancer Controller, and Cluster Autoscaler or Karpenter. I always test the upgrade in a staging environment and take backups of Kubernetes manifests and critical databases before starting.
 
I then upgrade the EKS control plane, followed by managed add-ons like CoreDNS, kube-proxy, and the VPC CNI plugin. Next, I create a new managed node group with the target Kubernetes version and migrate workloads by cordoning and draining the old nodes one at a time. Pod Disruption Budgets, multiple replicas, and readiness probes ensure zero downtime during pod rescheduling.
 
Throughout the upgrade, I monitor pod health, application logs, ALB target health, CloudWatch, Prometheus, and Grafana. After validating the applications with smoke tests and business transactions, I decommission the old node group. If any issues occur, I follow the rollback plan by moving workloads back to the previous node group or restoring from backups.”


5. Design a self-healing platform for critical production services.

A self-healing platform combines Kubernetes and AWS capabilities to automatically recover from failures. I would deploy workloads on Amazon EKS across multiple Availability Zones with managed node groups. Kubernetes health probes, ReplicaSets, and Deployments automatically restart or replace unhealthy pods. HPA scales pods based on demand, while Karpenter or Cluster Autoscaler adds worker nodes when required. An ALB routes traffic only to healthy pods, and Amazon RDS Multi-AZ provides automatic database failover. Prometheus, Grafana, CloudWatch, and Alertmanager provide monitoring and alerting, while Argo CD continuously reconciles the cluster with the desired state stored in Git. This architecture minimizes downtime, supports automatic recovery, and provides a resilient production platform.”

6. Explain the most challenging production incident you've handled and the architectural improvements you made afterward.

One of the most challenging production incidents I handled involved intermittent 503 errors from an application running on Amazon EKS during peak traffic. The Kubernetes cluster itself was healthy, so I investigated the application and database layers. Application logs showed HikariCP connection timeout errors, and RDS metrics confirmed that the connection pool was exhausted due to heavy traffic and slow database queries. We restored the service by tuning the HikariCP pool after validating database capacity and optimizing slow queries. Following the incident, we implemented RDS Proxy for better connection management, introduced Read Replicas for read scalability, improved monitoring with Prometheus, Grafana, and CloudWatch, and tuned autoscaling policies. These architectural improvements reduced future incidents, improved application resilience, and significantly reduced our mean time to recovery.”

7)Explain any kubernetes troubleshooting scenarios

Scenario 1: Pods Running but Users Getting 503 Errors

Interview Answer

“One production issue we faced was that users were receiving intermittent HTTP 503 errors even though all Kubernetes nodes were healthy and the pods were in the Running state. I started troubleshooting from the user request path by checking the ALB, Ingress, Service, Endpoints, and Pods. I found that the pods were Running but not Ready because the readiness probe was failing. Since Kubernetes removes unready pods from the Service endpoints, the ALB had no healthy backend to route traffic. We corrected the readiness probe configuration, restarted the deployment, and verified that the endpoints were healthy. To prevent recurrence, we improved health checks and configured startup probes for slow-starting applications.”

 ⸻ 
 
Scenario 2: Pods Stuck in Pending 
Interview Answer
“In another incident, newly created pods remained in the Pending state after deployment. I checked the pod events using kubectl describe pod and found an ‘Insufficient CPU’ scheduling error. The cluster had reached its resource capacity. Since we were using Karpenter, I verified its logs and found that it couldn’t provision new nodes due to an IAM permission issue. After correcting the IAM permissions, Karpenter launched new worker nodes, and Kubernetes scheduled the pending pods successfully. We later configured alerts for pending pods and node provisioning failures.”

 ⸻
 
 Scenario 3: CrashLoopBackOff
Interview Answer
“We also encountered an application repeatedly entering the CrashLoopBackOff state. I reviewed the pod logs and events and discovered that the application was failing during startup because it couldn’t connect to the database. The database credentials stored in a Kubernetes Secret had been updated, but the application deployment hadn’t been restarted. After updating the Secret and performing a rolling restart of the deployment, the application started successfully. To avoid similar incidents, we integrated secret rotation with our deployment pipeline and improved startup validation.”

 ⸻ 
 
Scenario 4: ImagePullBackOff
Interview Answer
“A deployment failed because the pods were in the ImagePullBackOff state. I checked the pod events and found an authentication error while pulling the image from Amazon ECR. The node IAM role lacked the required ECR permissions after a recent policy change. After restoring the correct IAM permissions, the nodes were able to pull the image successfully, and the deployment completed.”

 ⸻
 
 Scenario 5: High CPU Usage
Interview Answer
“During peak business hours, application latency increased significantly. Prometheus showed CPU utilization above 90% across the pods. The Horizontal Pod Autoscaler was configured to scale only after CPU exceeded 90%, causing delayed scaling. We lowered the HPA threshold to 70%, increased the replica count, and optimized a CPU-intensive API. This improved response times and allowed the application to scale proactively.”

 ⸻ 
 
Scenario 6: Node NotReady
Interview Answer
“One worker node entered the NotReady state due to a kubelet failure. Kubernetes automatically stopped scheduling new pods on that node and rescheduled the affected workloads onto healthy nodes. We investigated the kubelet logs, restored the node, and Auto Scaling replaced the unhealthy EC2 instance. Because the application had multiple replicas across Availability Zones, users did not experience downtime.”

Overall answers 

Whenever I troubleshoot Kubernetes issues, I follow a structured approach instead of making assumptions. I first identify whether the problem is related to the infrastructure, Kubernetes platform, or the application. I check the node health, pod status, events, logs, services, endpoints, ingress configuration, and resource utilization. In one production incident, users were receiving intermittent 503 errors even though the pods were Running. I found that the readiness probe was failing, so Kubernetes removed the pods from the Service endpoints. After correcting the readiness probe configuration and validating the application, the issue was resolved. In another incident, pods were stuck in Pending because the cluster lacked resources and Karpenter couldn’t provision new nodes due to an IAM permission issue. After fixing the permissions, new nodes were created automatically and the pods were scheduled successfully. My troubleshooting approach is always systematic, using Kubernetes events, logs, metrics, and application dependencies to identify the root cause before implementing a fix.”

User Report → ALB → Ingress → Service → Endpoints → Pods → Container Logs → Node → Application → Database/External Dependencies → Metrics → Root Cause → Resolution → RCA & Preventive Actions

8. What is your deployment strategy

In my current project, we primarily use the Rolling Update deployment strategy for application deployments. Once the CI pipeline builds and pushes the Docker image to Amazon ECR, Argo CD synchronizes the updated manifests with the EKS cluster. Kubernetes then gradually replaces the old pods with new ones using Rolling Update. Readiness probes ensure that only healthy pods receive traffic, while multiple replicas and Pod Disruption Budgets ensure zero downtime. For high-risk production releases that require fast rollback, we would consider Blue-Green or Canary deployments, but our day-to-day application deployments use Rolling Updates.”


9. How do you manage infrastructure cost optimization without impacting performance?

My approach to infrastructure cost optimization is based on monitoring and data rather than assumptions. I first analyze resource utilization using CloudWatch, Prometheus, Grafana, and AWS Cost Explorer. Then I optimize EC2 and Kubernetes resources through right-sizing, HPA, and Karpenter.I optimize databases with query tuning and Read Replicas, reduce networking costs using VPC Endpoints, and manage storage with lifecycle policies and cleanup of unused resources. Finally, I use AWS Budgets, Cost Anomaly Detection, and resource tagging to continuously monitor spending. Every optimization is validated against application performance metrics to ensure we reduce costs without impacting user experience or system reliability.”

Cost Optimization Areas

AWS Service	Optimization
EC2	Right-sizing, Reserved Instances/Savings Plans, Spot Instances
EKS	HPA, Karpenter/Cluster Autoscaler, correct requests & limits
RDS	Read Replicas, query tuning, right-sizing
S3	Lifecycle policies, Intelligent-Tiering, Glacier
EBS	Delete unused volumes & snapshots
NAT Gateway	Use VPC Endpoints for ECR and S3
Load Balancer	Remove unused ALBs/NLBs
Monitoring	Budgets, Cost Explorer, Cost Anomaly Detection


10. A production deployment failed. What would be your first troubleshooting steps?

“When a production deployment fails, I follow a structured troubleshooting process. I first determine whether the issue occurred during CI or CD by reviewing Jenkins pipeline logs and Argo CD synchronization status. If the deployment reached Kubernetes, I check the rollout status, pod health, events, logs, readiness probes, services, ingress, and ALB target health. If Kubernetes is healthy, I investigate application logs and backend dependencies such as RDS, Redis, or Kafka. Throughout the process, I use CloudWatch, Prometheus, and Grafana to correlate infrastructure and application metrics. If the deployment is impacting users and cannot be resolved quickly, I immediately roll back to the previous stable version using Argo CD or Kubernetes rollout history. Once the service is restored, I perform an RCA and implement preventive improvements to avoid recurrence.”

Cross Questions
Q1. What if the deployment is stuck?
Answer:
“I would check kubectl rollout status, pod events, and describe the deployment to determine whether the rollout is blocked by readiness probe failures, insufficient resources, image pull errors, or scheduling issues.”
⸻ Q2. What if the new pods are running but users still cannot access the application?
Answer:
“I would verify Service endpoints, Ingress configuration, ALB target group health, and application logs. Running does not necessarily mean the pods are Ready to receive traffic.”
⸻
Q3. When would you perform a rollback?
Answer:
“If the deployment is causing customer impact and the issue cannot be resolved quickly, I would roll back immediately to restore service. After stabilization, I would investigate the failed release in detail.

 11) A Kubernetes Pod is stuck in Pending state. What would you check?

When a Pod is stuck in the Pending state, I first describe the Pod to review the scheduler events because they usually indicate the exact reason for the failure. Then I verify worker node health and available resources. If capacity is insufficient, I check whether Karpenter or Cluster Autoscaler is provisioning new nodes. If resources are available, I investigate node selectors, affinity rules, taints and tolerations, Persistent Volume Claims, and namespace resource quotas. In one production incident, Pods remained Pending because Karpenter couldn’t provision new nodes due to an IAM permission issue. After correcting the IAM configuration, new nodes were created automatically and the Pods transitioned to the Running state. My approach is always to identify the scheduler’s reason first and then resolve the underlying cause rather than making assumptions.”

Cross Questions
Q1. Can a Pod remain Pending even if nodes are healthy?
Answer:
“Yes. Healthy nodes alone don’t guarantee scheduling. The Pod can still remain Pending due to node affinity, taints, PVC issues, namespace quotas, or insufficient allocatable resources.” ⸻ 
Q2. What is the first command you run?
Answer:
”kubectl describe pod <pod-name> because the Events section usually explains why the scheduler couldn’t place the Pod.”
⸻
 Q3. What if there is enough CPU but the Pod is still Pending?
Answer:
“I would investigate node affinity, taints and tolerations, Persistent Volume binding, resource quotas, and scheduler events.”

    12. Users report the application is slow. How would you identify the bottleneck?

When users report that an application is slow, I don’t assume the problem is Kubernetes. I follow an end-to-end approach by checking the ALB metrics, Kubernetes pod health, CPU, memory, and HPA status, followed by application logs and dependency health. If Kubernetes is healthy, I investigate the application, HikariCP connection pool, and Amazon RDS performance using CloudWatch and Performance Insights. In one production incident, we identified that the application slowdown was caused by an exhausted HikariCP connection pool due to slow SQL queries. We resolved the issue by increasing the connection pool after validating database capacity, optimizing SQL queries, and improving monitoring with Grafana and CloudWatch. My focus is always to identify the actual bottleneck before implementing any fix.”

Application is very slow during peak hours.”

Investigation:

* ALB → Healthy ✅
* Pods → Healthy ✅
* CPU → 45% ✅
* Memory → 60% ✅
* Application Logs → HikariCP timeout ❌
* RDS → Active connections at maximum ❌
* Slow queries detected ❌

Root Cause

* Database connection pool exhausted.
* Slow SQL queries holding connections.

Resolution

* Increased HikariCP pool size from 20 to 60 after verifying RDS capacity.
* Optimized slow queries and indexes.
* Added CloudWatch and Grafana alerts for connection pool utilization.

Result:

* API latency reduced significantly.
* No further timeouts during peak traffic.

13. An application cannot connect to the database. What would you check first?

When an application cannot connect to the database, I first review the application logs to understand the exact error. Then I verify the database configuration stored in Kubernetes Secrets or ConfigMaps, including the endpoint, port, username, and password. If the configuration is correct, I check DNS resolution and network connectivity, including Security Groups, NACLs, VPC routing, and Kubernetes Network Policies. If the network is healthy, I investigate the database by reviewing its availability, CloudWatch metrics, active connections, CPU, and slow queries. In one production incident, we found that the application couldn’t obtain database connections because the HikariCP connection pool was exhausted during peak traffic. We increased the pool size after confirming database capacity, optimized slow queries, and improved monitoring to prevent the issue from recurring. My troubleshooting approach is always systematic, moving from the application layer to the infrastructure until the root cause is identified.”

14) A recent deployment caused production issues. How would you perform a rollback?

If a recent deployment causes production issues, my first priority is to restore service as quickly as possible while minimizing customer impact. Before making any changes,

If a recent deployment causes production issues, I first confirm that the deployment is the root cause by reviewing deployment history, logs, metrics, and user reports. I immediately roll back to the previous stable version instead of troubleshooting in production. In our environment, we use Argo CD with GitOps, so I revert to the previous version Kubernetes performs a rolling rollback, ensuring only healthy pods receive traffic through readiness probes and multiple replicas. After the rollback, I validate application health, ALB target status, business transactions, and monitoring dashboards. Once production is stable, I perform an RCA, fix the issue in a lower environment, and redeploy only after proper validation. This approach minimizes downtime while ensuring a safe and reliable recovery.”

A new release was deployed.

After deployment:

* Users received 503 errors
* API latency increased
* Readiness probes failed

Action Taken:

* Confirmed the issue started immediately after deployment.
* Rolled back to the previous Git commit using Argo CD.
* Kubernetes replaced the new pods with the previous stable ReplicaSet.
* Verified ALB target health, Prometheus dashboards, and application logs.
* Service was restored within a few minutes.

Root Cause:

Incorrect application configuration introduced in the new release.
⸻
Cross Questions

Q1. Why roll back instead of fixing the issue?

Answer:

“If production is impacted, restoring service is the highest priority. Fixing the issue can take time, so rolling back minimizes downtime and customer impact. The fix can then be developed and tested safely before redeployment.”
⸻

Q2. Can Argo CD perform rollbacks?

Answer:

“Yes. Since Argo CD follows GitOps, I can roll back by reverting to a previous Git commit or Helm chart version and syncing the application.”

⸻
Q3. What if the rollback also fails?

Answer:

“I would verify the previous version, investigate dependency changes such as database schema or configuration updates, and if necessary use the disaster recovery plan or restore from backups. Database migrations should always be backward compatible to support safe rollbacks.”


15. Kubernetes Pods keep restarting. What could be the possible reasons?

“When Pods keep restarting, I first check the restart count, pod events, and previous container logs to identify the reason. Common causes include application crashes, failed liveness or startup probes, OOMKilled events due to insufficient memory, configuration issues, database connectivity problems, or failures in external dependencies. I also verify whether the issue started after a recent deployment. In one production incident, the application kept restarting because it couldn’t obtain database connections due to an exhausted HikariCP connection pool. After validating RDS capacity, we increased the pool size, optimized slow queries, and the application stabilized. My approach is to use logs, events, and metrics to identify the root cause before making any changes.”

Common reason 
Cause	What to Check
Application crash	Application logs
Liveness probe failure	Probe configuration
Startup probe missing/incorrect	Slow application startup
OOMKilled	Memory limits, kubectl describe pod
CPU starvation	CPU requests/limits
Configuration issue	ConfigMap, Secret
Database connection failure	RDS, HikariCP
External API failure	Application logs
Node issues	kubectl describe node
Bad deployment	Roll back to previous version

16. An EC2 instance is running but SSH access is failing. What would you check?

When an EC2 instance is running but SSH access fails, I follow a structured troubleshooting approach. I first verify the instance state, IP address, and whether it is in a public or private subnet. Then I check the Security Group, Network ACLs, and route tables to ensure SSH traffic is allowed. After that, I confirm I’m using the correct key pair and username. If the AWS networking is correct, I investigate the operating system by checking the SSH service, disk space, firewall rules, and system logs using Session Manager or the EC2 serial console if necessary. In one production incident, SSH failed because the Network ACL blocked the return traffic on ephemeral ports. After correcting the NACL rules, SSH connectivity was restored. My approach is always to troubleshoot layer by layer until the root cause is identified.”

Problem	Check
Wrong Security Group	Port 22 inbound
Missing Public IP	Public IP or Elastic IP
Private Subnet	Bastion Host, VPN, Session Manager
Wrong Username	ec2-user, ubuntu, etc.
Wrong Key Pair	Correct .pem file
SSH Service Stopped	systemctl status sshd
Disk Full	df -h
High CPU	CloudWatch metrics
Firewall	iptables / firewalld
NACL or Route Issue	VPC networking
