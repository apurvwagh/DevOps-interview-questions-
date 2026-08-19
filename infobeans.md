1)  Production Server CPU Suddenly Becomes 100%. What’s Your First Approach?

1) My first priority is to determine customer impact and identify what is consuming the CPU. 
2) I would not immediately restart the server because that could destroy useful evidence and may not fix the root cause.
3)I would check monitoring dashboards for CPU, load average, traffic, latency, error rate and recent changes. 
4) Then I would log into the server and use tools such as top, htop, ps, and pidstat to identify the process or threads consuming CPU.
5) I would correlate the CPU spike with recent deployments, traffic increases, scheduled jobs, batch processes, or configuration changes.
6) If the application is responsible and customers are impacted, I might scale out, remove the affected instance from the load balancer,
 or roll back a recent deployment. If a non-critical process such as a backup or scheduled job is consuming CPU, I would investigate and safely stop or reschedule it.
7) After restoring service, I would perform RCA using logs and metrics
and implement preventive measures such as autoscaling, better alerts, application optimization, and capacity planning.”

===================================================================================================================

2) What kind of DevOps work are you currently handling? Explain your day-to-day responsibilities.

1) Currently, I work as a Cloud DevOps Engineer, primarily around AWS, Kubernetes/EKS, Terraform and CI/CD. My responsibilities cover both infrastructure and application delivery.
2) On the infrastructure side, I work with AWS services such as VPC, EC2, ALB, IAM, EKS, RDS, S3 and Route 53, and I use Terraform for infrastructure provisioning and management.
3) On the Kubernetes side, I manage EKS workloads, deployments, Services, Ingress, HPA, readiness and liveness probes, resource requests and limits, node scheduling and production troubleshooting.
4) For CI/CD, I work with Jenkins and GitOps tools such as ArgoCD. The pipeline typically includes source checkout, build, testing, security scanning, Docker image creation, pushing the image to ECR and deploying to EKS.
5) I also work on monitoring and incident management using Prometheus, Grafana and CloudWatch. During production incidents, I troubleshoot the issue, identify the root cause, restore service and document the RCA and preventive actions.”

======================================================================================

3) How do you troubleshoot a Pod stuck in CrashLoopBackOff?

1) CrashLoopBackOff means the container is repeatedly starting and terminating. I first check the Pod status and events,
   then look at the current and previous container logs. The previous logs are especially important because the container may have already restarted.
2)  I check the exit code and last state to determine whether it is an application crash, missing configuration, Secret or environment variable, dependency failure, failed probe or OOMKilled condition.
3) I then compare the issue with recent deployments or configuration changes. If the new release is responsible and production is impacted, I roll back to the last known-good version and then investigate the RCA.”

========================================================================================

3. Pod is Running but application is not accessible. What do you check?
This is very important for your interview.

1) I would not assume that Running Pods mean the application is accessible. I would trace the request path from the user to the Pod and identify the first layer where traffic is failing.

2) First, I would check whether the Pods are Ready and whether the application is actually listening on the expected port. Then I would verify the Kubernetes Service and its EndpointSlices.

3) After that, I would check the Ingress and AWS ALB—listener rules, target groups, target health, security groups and health checks. I would also verify DNS and, if CloudFront is involved, caching and origin configuration.

4) I would test the application from inside the cluster as well as externally. This helps me determine whether the issue is inside Kubernetes or somewhere in the external traffic path.”

============================================================================================

4. Difference between Liveness and Readiness probes?
🎯 Interview Answer
“Readiness determines whether a Pod should receive traffic, while liveness determines whether the container is still healthy enough to continue running.”


5. Have you worked on Kubernetes production issues? Explain one.
This is where you should give a realistic production story.
7)Explain any kubernetes troubleshooting scenarios

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


7. What kind of Bash/Shell scripting have you done?
🎯 Interview Answer
“I’ve mainly used Bash for operational automation and troubleshooting. I’ve written scripts for log cleanup, service health checks, disk and CPU monitoring, deployment validation, AWS CLI automation, Kubernetes troubleshooting and CI/CD helper tasks.”

8. Linux server has high CPU or disk usage. How do you troubleshoot?
CPU
🎯 Interview Answer
“First I confirm the CPU utilization and identify which process is consuming it. Then I determine whether it is application traffic, a runaway process, CPU-intensive job or system process. I correlate it with application logs and monitoring metrics and check whether the issue started after a deployment or traffic spike.”

9. Explain one CI/CD pipeline you’ve worked on.
This is another very important question.
🎯 2-Minute Interview Answer
“One of my CI/CD pipelines uses Jenkins for continuous integration and ArgoCD for GitOps-based deployment to EKS.
The developer pushes code to Git. Jenkins checks out the code, runs unit tests and static/security checks, builds the application and Docker image, and pushes the image to Amazon ECR.
The deployment manifest is then updated with the new immutable image tag or digest. ArgoCD detects the Git change and synchronizes the Kubernetes Deployment to EKS.
After deployment, Kubernetes performs readiness and liveness checks, and we monitor the rollout using Prometheus, Grafana and CloudWatch. If the deployment is unhealthy, we stop or roll back to the previous known-good version.”

10. Exposure to Selenium / Playwright / Cypress?
This is the one question where you should not overclaim if you haven’t actually used these tools.
Safe professional answer
“I have more hands-on experience on the DevOps and CI/CD side than on UI automation. I understand Selenium, Playwright and Cypress as browser automation and end-to-end testing frameworks used to validate application behavior through a real or automated browser.
From a DevOps perspective, my role would be to integrate these tests into the CI/CD pipeline, execute them in a suitable test environment, collect reports and artifacts, and use the results as a deployment quality gate.”

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
