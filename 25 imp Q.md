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

Q4. Production Server CPU Suddenly Becomes 100%. What’s Your First Approach?

🎯 2-Minute Interview Answer

“My first priority is to determine customer impact and identify what is consuming the CPU. I would not immediately restart the server because that could destroy useful evidence and may not fix the root cause.

I would check monitoring dashboards for CPU, load average, traffic, latency, error rate and recent changes. Then I would log into the server and use tools such as top, htop, ps, and pidstat to identify the process or threads consuming CPU.

I would correlate the CPU spike with recent deployments, traffic increases, scheduled jobs, batch processes, or configuration changes.

If the application is responsible and customers are impacted, I might scale out, remove the affected instance from the load balancer, or roll back a recent deployment. If a non-critical process such as a backup or scheduled job is consuming CPU, I would investigate and safely stop or reschedule it.

After restoring service, I would perform RCA using logs and metrics and implement preventive measures such as autoscaling, better alerts, application optimization, and capacity planning.”

=============≠=======================================================

Q5. Deployment Completed Successfully, but Production Still Shows the Old Version

🎯 2-Minute Interview Answer

“I would start by verifying whether the new version is actually running in production. A successful CI/CD deployment doesn’t necessarily mean users are receiving the new version.

First, I would check the Kubernetes Deployment, ReplicaSets and running Pods and verify the actual image tag or, preferably, image digest/commit SHA. I would also confirm that the deployment went to the correct production cluster, namespace and environment.

If the new Pods are running the correct version, I would trace the traffic path—Service, Ingress, ALB target group and DNS—to make sure traffic is reaching the new Pods.

If traffic is reaching the new Pods but users still see the old version, I would investigate caching such as CloudFront, browser cache or application-level caching.

Finally, I would check GitOps or CI/CD history to make sure another process didn’t revert the deployment. Once I identify the mismatch, I would fix the relevant layer and verify the version from an external request.”

=============≠=======================================================

Q6. Your Pod is in Pending state for the last 30 minutes. How will you troubleshoot it?

🎯 2-Minute Interview Answer

“A Pending Pod means Kubernetes has not successfully scheduled it onto a node, or the Pod is waiting for some required condition before it can start. My first step is to check the Pod events because they usually tell me exactly why scheduling failed.

I would run kubectl describe pod and look at the Events section. Then I would check node availability, CPU and memory capacity, resource requests, node selectors, affinity/anti-affinity, taints and tolerations, topology constraints, PVC availability and any scheduling restrictions.

If the issue is insufficient capacity, I would check whether Cluster Autoscaler or Karpenter can provision additional nodes. If the Pod has an invalid node selector, affinity rule, taint/toleration or PVC/AZ constraint, I would correct the configuration.

After fixing the root cause, I would verify that the Pod gets scheduled and becomes Ready. I would also add appropriate monitoring and capacity planning to prevent recurrence.”

Pod Pending
    ↓
kubectl describe pod
    ↓
Check Events
    ↓
FailedScheduling?
    │
    ├── Insufficient CPU/Memory
    │       ↓
    │   Check nodes / Autoscaler
    │
    ├── Taint/Toleration
    │       ↓
    │   Fix scheduling rules
    │
    ├── NodeSelector/Affinity
    │       ↓
    │   Check matching nodes
    │
    ├── PVC Pending
    │       ↓
    │   Check StorageClass/AZ
    │
    └── Other constraints

=============≠=======================================================


Q7. One of your applications suddenly starts showing CrashLoopBackOff. Where do you start debugging?

🎯 2-Minute Interview Answer

“CrashLoopBackOff means the container is repeatedly starting and then terminating, and Kubernetes is applying an increasing restart backoff. I first check the Pod status, container exit code and previous container logs because the current container may have already restarted.

I would use kubectl logs --previous and kubectl describe pod to identify whether the application is crashing because of a configuration issue, missing Secret, bad environment variable, dependency failure, application bug, failed startup probe, or OOMKilled condition.

Then I would compare the failure with recent deployments or configuration changes. If the new release introduced the issue and production is impacted, I would roll back to the last known-good version.

After recovery, I would identify the root cause and improve startup validation, probes, resource limits, configuration validation and deployment health checks.”

=============≠=======================================================

Q8. Your application cannot connect to the database after deployment. How will you troubleshoot it?

🎯 2-Minute Interview Answer

“Because the issue started after deployment, I would first compare the new version and configuration with the last known-good version. I would check the application logs to determine whether the failure is DNS, network timeout, authentication, TLS, or connection-limit related.

Then I would verify the database hostname, port, database name and credentials being injected into the new Pods. I would test DNS resolution and TCP connectivity from inside the Pod.

For AWS, I would check the EKS-to-RDS network path, including security groups, route tables, NACLs and NetworkPolicies. Then I would check the database itself for availability, connection limits, CPU, storage and active connections.

If the deployment increased the number of Pods, I would specifically check database connection-pool usage because scaling application Pods can exhaust the database’s maximum connections. If the new deployment is causing customer impact, I would roll back to the last known-good version and then perform RCA.”

=============≠=======================================================

Q9. Your EKS cluster has both system and user node groups. Why not deploy everything on a single node group?

🎯 2-Minute Interview Answer

“I prefer separating system and application workloads because they have different responsibilities and operational requirements.

The system node group is dedicated to critical Kubernetes components such as CoreDNS, kube-proxy, AWS VPC CNI components, and other platform-level workloads. User node groups run business applications.

This separation prevents application workloads from consuming all the resources needed by critical cluster services. It also allows independent scaling, maintenance, upgrades and instance-type selection.

For example, if a business application suddenly consumes significant CPU or memory, it shouldn’t starve CoreDNS or other system components. Similarly, I can upgrade or scale application node groups without unnecessarily disturbing the system capacity.

I would use taints and tolerations to keep normal application Pods away from system nodes, and labels/node affinity to control placement.”

=============≠=======================================================

Q10. A deployment failed in production halfway through. Some Pods are running the new version while others are still on the old version. What will you do?

🎯 2-Minute Interview Answer

“First, I would assess customer impact and check the rollout status rather than immediately making another deployment. I would check the Deployment, ReplicaSets and Pods to determine why the rollout stopped.

I would verify whether the new Pods are healthy by checking readiness probes, application logs, events, image issues, resource constraints and scheduling problems.

If the new version is unhealthy and production is impacted, I would stop the rollout and roll back to the last known-good version. If the new Pods are healthy and the issue is capacity or scheduling related, I would fix that blocker and allow the rollout to continue.

After recovery, I would perform RCA and improve readiness/startup probes, PDBs, resource requests, deployment health checks and progressive rollout mechanisms.”

Impact → Rollout → New Pods → Root Cause → Rollback/Fix → Verify → RCA

=============≠=======================================================

Q11. Your application is running, but users are getting 502 Bad Gateway. How will you troubleshoot it?

🎯 2-Minute Interview Answer

“I would first identify which layer is generating the 502—CloudFront, ALB, API Gateway, or the application. Then I would trace the request from the client to the backend.

For an AWS ALB in front of EKS, I would check ALB target health, listener rules, target groups, security groups and whether the Service is forwarding traffic to healthy Pods. On Kubernetes, I would verify the Service endpoints, Pod readiness and application listening port.

I would also check application logs for connection resets, crashes or upstream failures and verify that the application is listening on the expected interface and port. Finally, I would check recent deployments or configuration changes.”

=============≠=======================================================

Q12. Your application should access S3 privately. You don’t want traffic going over the public internet. How would you design it?

🎯 2-Minute Interview Answer

“For an application running on private EC2 instances or EKS Pods, I would use an Amazon S3 VPC Gateway Endpoint. This allows the workload to access S3 through the AWS private network without requiring an Internet Gateway or NAT Gateway.

I would associate the endpoint with the appropriate private route tables and restrict the S3 bucket using a bucket policy so that access is allowed only through the specific VPC endpoint where appropriate. I would also use IAM roles with least-privilege S3 permissions.”

For private S3 access from a VPC, I would prefer an S3 Gateway Endpoint rather than sending the traffic through NAT. It keeps the traffic on the AWS network and can also reduce NAT Gateway cost.”

=============≠=======================================================

Q13. Someone manually changed an AWS resource from the AWS Console. How will Terraform detect it?

🎯 2-Minute Interview Answer

“This is Terraform drift. Terraform detects it when I run terraform plan or terraform apply, because Terraform refreshes the state against the actual AWS infrastructure and compares the actual configuration with the desired configuration defined in code.

For example, if Terraform manages an EC2 instance and someone manually changes its instance type from t3.medium to t3.large, Terraform will detect the difference during planning and may propose changing it back to the value defined in Terraform.

I would investigate the plan before applying it, because not every manual change should automatically be reverted. I would either update the Terraform code if the manual change was intentional or apply the Terraform configuration to bring the resource back to the desired state.”

=============≠=======================================================

Q14. Your production server suddenly runs out of disk space. What’s your debugging approach?

🎯 2-Minute Interview Answer

“First I would confirm which filesystem is full and determine what is consuming the disk. I would use df -h to identify the full filesystem and du to find the largest directories and files. Then I would check logs, temporary files, application-generated files, deleted-but-open files and Docker/container logs.

I would immediately mitigate the customer impact by safely removing or rotating unnecessary files or expanding the EBS volume if appropriate. I would not blindly delete files from production.

After recovery, I would identify the root cause—for example, uncontrolled application logs, a failed log rotation, temporary files, core dumps or container images—and implement log rotation, retention policies, monitoring and disk-space alerts.”

=============≠=======================================================

Q15. A developer says, “The pipeline is failing only in production. Dev and QA work perfectly.” How will you troubleshoot it?

🎯 2-Minute Interview Answer

“I would first compare the production pipeline and environment with Dev and QA rather than assuming the application code is the problem. I would identify the exact pipeline stage that fails and compare credentials, IAM permissions, secrets, environment variables, AWS account, region, network connectivity and deployment configuration.

Because production usually has stricter permissions and controls, I would specifically investigate IAM authorization, security policies, private networking, approval gates, artifact access and production-only secrets.

I would also verify that the same artifact/image is being promoted from QA to production rather than rebuilding different artifacts for each environment.

Once I identify the difference, I would fix the production-specific configuration or permission issue, rerun the pipeline safely, and then add automated validation so the same problem is detected before production.”

=============≠=======================================================

Q16. Deployment is successful, all Pods are Running, but users still can’t access the application. Where will you start debugging?

🎯 2-Minute Interview Answer

“I would not assume that Running Pods mean the application is accessible. I would trace the request path from the user to the Pod and identify the first layer where traffic is failing.

First, I would check whether the Pods are Ready and whether the application is actually listening on the expected port. Then I would verify the Kubernetes Service and its EndpointSlices.

After that, I would check the Ingress and AWS ALB—listener rules, target groups, target health, security groups and health checks. I would also verify DNS and, if CloudFront is involved, caching and origin configuration.

I would test the application from inside the cluster as well as externally. This helps me determine whether the issue is inside Kubernetes or somewhere in the external traffic path.”

=============≠=======================================================

Q17. Your EKS cluster needs internet access to pull Docker images, but worker nodes shouldn’t have Public IPs. How would you design it?

🎯 2-Minute Interview Answer

“I would keep the EKS worker nodes in private subnets without public IPs. For pulling images from Amazon ECR, I would preferably use VPC endpoints for ECR and S3, because ECR image layers are stored in S3. This allows image pulls without traversing the public internet.

If the nodes need general outbound internet access—for example, to download packages or access external APIs—I would route their traffic through a NAT Gateway in a public subnet. The private subnet’s route table would point the default route to the NAT Gateway.

The NAT Gateway has a public IP and communicates through the Internet Gateway, while the worker nodes remain private.

Private nodes don’t need public IPs. I use NAT Gateway for required general outbound internet access and VPC endpoints for AWS services such as ECR and S3. This gives private workers controlled outbound connectivity while keeping them unreachable directly from the internet.”

=============≠=======================================================

Q18. Your Terraform deployment accidentally deleted an AWS resource. How would you prevent this in production?

🎯 2-Minute Interview Answer

“First, I would prevent Terraform from being able to accidentally destroy critical resources by using lifecycle protection such as prevent_destroy = true where appropriate. I would also enforce a production workflow where engineers cannot directly run terraform apply; changes go through pull requests, Terraform plan review and an approval gate.

I would carefully review any plan containing destroy or replace, especially for stateful resources such as RDS, S3 or production networking.

I would also use remote state with locking, least-privilege IAM, separate production state, and CI/CD controls. Finally, I would maintain backups and recovery mechanisms because Terraform safeguards don’t replace data protection.”

=============≠=======================================================

Q19. Production application suddenly fails with Too many open files. How will you troubleshoot it?

🎯 2-Minute Interview Answer

”Too many open files normally indicates that the process has reached its file descriptor limit. I would first check the application’s current file descriptor usage and limits, then determine what is consuming the descriptors—files, sockets, connections or leaked resources.

I would check ulimit, /proc/<pid>/fd, lsof, application metrics and logs. I would specifically investigate whether there is a file descriptor leak, too many concurrent network connections, improperly closed files, or an unexpectedly high traffic level.

For immediate mitigation, I might restart or scale the affected application if appropriate, but I would not consider that the permanent fix. I would identify and fix the leak or tune the limits only after understanding the cause.”

=============≠=======================================================

Q20. Pipeline completed successfully, but one microservice wasn’t updated while all others were. How will you troubleshoot it?

🎯 2-Minute Interview Answer

“I would first determine whether the problem is in the build, artifact, deployment or traffic layer. Since the other microservices updated successfully, I would compare the failing service’s pipeline stages and configuration with the successful services.

I would verify that the service actually built a new image, that the expected image was pushed to ECR, and that the deployment manifest references the correct immutable image tag or digest. Then I would check the Kubernetes Deployment, ReplicaSet and Pods to verify which version is actually running.

If we’re using ArgoCD or another GitOps tool, I would check whether the manifest change was committed, detected and synchronized. Finally, I would verify that the Service/Ingress is sending traffic to the new Pods.

I would also check whether the pipeline had a conditional step, incorrect service path, wrong namespace or deployment target that caused this particular microservice to be skipped.”

=============≠=======================================================































































