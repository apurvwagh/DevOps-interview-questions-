1. Production Server CPU Suddenly Becomes 100%. What’s Your First Approach?

1) My first priority is to determine customer impact and identify what is consuming the CPU. 
2) I would not immediately restart the server because that could destroy useful evidence and may not fix the root cause.
3) I would check monitoring dashboards for CPU, load average, traffic, latency, error rate and recent changes. 
4) Then I would log into the server and use commands such as top, htop, ps, lsof and pidstat to identify the process or threads consuming CPU.
5) I would check recent deployments, there are any scheduled jobs, batch processes, or configuration changes boz these are main factors on high CPU load
6) If the application is responsible and customers are impacted then will roll back a recent deployment. If a non-critical process such as a backup or scheduled job is consuming CPU, I would investigate and safely stop or reschedule it.
7) After restoring service, I would perform RCA using logs and metrics
and implement preventive measures such as autoscaling, better alerts, application optimization, and capacity planning.”

==========================================================================================

2. What kind of DevOps work are you currently handling? Explain your day-to-day responsibilities.

1) Currently, I work as a Cloud DevOps Engineer, primarily around AWS, Kubernetes/EKS, Terraform and CI/CD. My responsibilities cover both infrastructure and application delivery.
2) On the infrastructure side, I work with AWS services such as VPC, EC2, ALB, IAM, EKS, RDS, S3 and Route 53, and I use Terraform for infrastructure provisioning and management.
3) On the Kubernetes side, I manage EKS workloads, deployments, Services, Ingress, HPA, readiness and liveness probes, resource requests and limits, node scheduling and production troubleshooting.
4) For CI/CD, I work with Jenkins and GitOps tools such as ArgoCD. The pipeline typically includes source checkout, build, testing, security scanning, Docker image creation, pushing the image to ECR and deploying to EKS.
5) I also work on monitoring and incident management using Prometheus, Grafana and CloudWatch. During production incidents, I troubleshoot the issue, identify the root cause, restore service and document the RCA and preventive actions.”

======================================================================================

3. How do you troubleshoot a Pod stuck in CrashLoopBackOff?

1) CrashLoopBackOff means the container is repeatedly starting and terminating. I first check the Pod status and events,
   then check the kubectl describe pod so in the event section would come to know the exact issue.
   also the current and previous container logs. The previous logs are especially important because the container may have already restarted.
2)  so i determine whether it is an application crash, missing configuration, Secret or environment variable, dependency failure, failed probe or OOMKilled condition.
4) I then compare the issue with recent deployments or configuration changes. If the new release is responsible and production is impacted, I roll back to the last known-good version and then investigate the RCA.”

========================================================================================

4. Pod is Running but application is not accessible. What do you check?
This is very important for your interview.

1) I would not assume that Running Pods mean the application is accessible. I would trace the request path from the user to the Pod and identify the first layer where traffic is failing.

2) First, I would check whether the Pods are Ready and whether the application is actually listening on the expected port. Then I would verify the Kubernetes Service and its EndpointSlices.

3) After that, I would check the Ingress and AWS ALB—listener rules, target groups, target health, security groups and health checks. I would also verify DNS and, if CloudFront is involved, caching and origin configuration.

user --> url--> route 53 --> LB (ALB) in alb check the target group --> LC ( check the congig corrected ) --> k8 service (check the service endpoint) --> pod --> application --> issue i found rediness prob is failing due to which application is not accessible

 =========================================================================================

5. Difference between Liveness and Readiness probes?
heath checks for application
“Readiness determines whether a Pod should receive traffic, while liveness determines whether the container is still healthy enough to continue running.”

==========================================================================================

6. Have you worked on Kubernetes production issues? Explain one.

Scenario 1: Pods Running but Users Getting 503 Errors

“One production issue we faced was that users were receiving intermittent HTTP 503 errors even though all Kubernetes nodes were healthy and the pods were in the Running state.
I started troubleshooting from the user request path by checking the ALB, Ingress, Service, Endpoints, and Pods.
I found that the pods were Running but not Ready because the readiness probe was failing. Since Kubernetes removes unready pods from the Service endpoints, the ALB had no healthy backend to route traffic.
We corrected the readiness probe configuration, restarted the deployment, and verified that the endpoints were healthy. To prevent recurrence, we improved health checks and configured startup probes for slow-starting applications.”

Scenario 2: Pods Stuck in Pending Interview Answer

“1) In another incident, newly created pods remained in the Pending state after deployment. I checked the pod events using kubectl describe pod and found an ‘Insufficient CPU’ scheduling error.

The cluster had reached its resource capacity. Since we were using Karpenter, I verified its logs and found that it couldn’t provision new nodes due to an IAM permission issue.

After correcting the IAM permissions, Karpenter launched new worker nodes, and Kubernetes scheduled the pending pods successfully. We later configured alerts for pending pods and node provisioning failures.”

Scenario 3: CrashLoopBackOff Interview Answer “1) We also encountered an application repeatedly entering the CrashLoopBackOff state. I reviewed the pod logs and events and discovered that the application was failing during startup because it couldn’t connect to the database. 2) The database credentials stored in a Kubernetes Secret had been updated, but the application deployment hadn’t been restarted. 3) After updating the Secret and performing a rolling restart of the deployment, the application started successfully. To avoid similar incidents, we integrated secret rotation with our deployment pipeline and improved startup validation.”

Scenario 4: ImagePullBackOff Interview Answer

“A deployment failed because the pods were in the ImagePullBackOff state. I checked the pod events and found an authentication error while pulling the image from Amazon ECR.
The node IAM role lacked the required ECR permissions after a recent policy change. After restoring the correct IAM permissions, the nodes were able to pull the image successfully, and the deployment completed.

======================================================================================

7. What kind of Bash/Shell scripting have you done?

“I’ve mainly used Bash for operational automation and troubleshooting. I’ve written scripts for log cleanup, service health checks, disk and CPU monitoring, deployment validation, AWS CLI automation, Kubernetes troubleshooting and CI/CD helper tasks.”
1) Log Cleanup & Archival Script: Use Case: Prevent disk space issues by archiving old/large logs.
2) Kubernetes Pod Health Check Script: Daily health validation of Kubernetes workloads.
3) 3. Automated Backup Script: Database/filesystem backup automation via cron.
4) 
=====================================================================================

8. Linux server has high CPU or disk usage. How do you troubleshoot?
CPU
commands: top , htop, isof, df -kh
“First I confirm the CPU utilization and identify which process is consuming it. Then I determine whether it is application traffic, a runaway process, CPU-intensive job or system process. I correlate it with application logs and monitoring metrics and check whether the issue started after a deployment or traffic spike.”

====================================================================================

9. Explain one CI/CD pipeline you’ve worked on.

1) In my current project, we follow a GitOps-based CI/CD pipeline using GitHub, Jenkins, Docker, AWS ECR, Argo CD, and EKS.
   
2) Developers create feature branches and raise Pull Requests. After code review, Jenkins is triggered automatically when the code is merged into the main branch.

3) Jenkins builds the application using Maven, runs unit tests, performs SonarQube code quality checks, and creates a Docker image. The image is then scanned for vulnerabilities using Trivy. If all checks pass, the image is pushed to Amazon ECR.

4) Next, Jenkins updates the image version in the GitOps repository. Argo CD detects this change and automatically deploys it to the EKS cluster.

5) Kubernetes performs a rolling deployment to ensure zero downtime. After deployment, we verify pod health, logs, and Grafana dashboards. If any issue is found, we can quickly roll back to the previous stable version.

5) The entire pipeline is automated, secure, and provides continuous feedback through Teams/Slack notifications.

One-line version (for quick interviews):

"We use GitHub → Jenkins → SonarQube → Trivy → Docker → ECR → Argo CD → EKS. Jenkins builds, tests, scans, and pushes the image, while Argo CD automatically deploys it to Kubernetes using GitOps principles with zero-downtime deployments."

====================================================================================

10. Exposure to Selenium / Playwright / Cypress?

“I have more hands-on experience on the DevOps and CI/CD side than on UI automation. I understand Selenium, Playwright and Cypress as browser automation and end-to-end testing frameworks used to validate application behavior through a real or automated browser.
From a DevOps perspective, my role would be to integrate these tests into the CI/CD pipeline, execute them in a suitable test environment, collect reports and artifacts, and use the results as a deployment quality gate.”

===================================================================================

11; How do you perform a zero-downtime Kubernetes cluster upgrade in production?
In production, an EKS upgrade is a carefully planned activity to ensure zero downtime.

First, I verify that the control plane, worker nodes, and kubectl are within the supported Kubernetes version.
review the release notes, check for deprecated APIs, and validate compatibility of Helm charts,
EKS add-ons, CSI drivers, AWS Load Balancer Controller, and Cluster Autoscaler or Karpenter.
I always test the upgrade in a staging environment and take backups of Kubernetes manifests and critical databases before starting. 5)I then upgrade the EKS control plane, followed by managed add-ons like CoreDNS, kube-proxy, and the VPC CNI plugin.
Next, I create a new managed node group with the target Kubernetes version and migrate workloads by cordoning and draining the old nodes one at a time.
Pod Disruption Budgets, multiple replicas, and readiness probes ensure zero downtime during pod rescheduling.  
Throughout the upgrade, I monitor pod health, application logs, ALB target health, CloudWatch, Prometheus, and Grafana.
After validating the applications with smoke tests and business transactions, I decommission the old node group. If any issues occur, I follow the rollback plan by moving workloads back to the previous node group or restoring from backups.”

=====================================================================================
































