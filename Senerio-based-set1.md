=== Devops 15 senerio based Question

**Question 1
Scenario**
**A Kubernetes deployment appears healthy, and all pods are in the Running state. However, users receive HTTP 503 Service Unavailable errors when accessing the application.**

Issue

Users cannot access the application even though Kubernetes reports that all pods are running.

⸻

**Root Cause Analysis (RCA)**

Possible causes include:

Pods are Running but not Ready.

1) Readiness probe failures.
2) Service selector labels don’t match pod labels.
3) Service has no endpoints.
4) Ingress controller configuration issue.
5) Load Balancer health check failure.
6) DNS resolution issue.
7) NetworkPolicy blocking traffic.
8) Backend dependency (Database/Redis/API) unavailable.
9) Recent deployment introduced a configuration error.

**Resolution**

I troubleshoot the request path layer by layer:

**Client

  ↓
AWS ALB / NLB

  ↓
Ingress Controller

  ↓
Kubernetes Service

  ↓
Endpoints

  ↓
Pods

  ↓
Application

  ↓
Database / External APIs**

Identify the failing layer, restore service, validate application health, and monitor the environment before closing the incident.

⸻

**Prevention
**
1)Proper Readiness and Liveness probes
2)Deployment smoke tests
3) Canary or Blue-Green deployments
4) Continuous endpoint monitoring
5) Prometheus & Grafana alerts
6) Synthetic monitoring

Rollback automation

⸻

**How I Would Answer in the Interview**

“If users are receiving HTTP 503 errors while all Kubernetes pods are running, I wouldn’t assume the application is healthy because a Running pod doesn’t necessarily mean it’s Ready to serve traffic. My first priority is understanding the business impact by determining whether the issue affects all users, a specific region, or only certain APIs.

Then I’d troubleshoot the request path systematically instead of jumping directly into the pods. I start from the AWS Load Balancer to verify that the target group reports healthy targets. Next, I’d check the Ingress Controller to ensure routing rules are correct and there are no configuration errors. After that, I’d verify the Kubernetes Service selectors and confirm that Endpoints have been created correctly.

If the Service has no endpoints, I’d compare the Service selectors with the Pod labels. If endpoints exist, I’d inspect the Readiness Probe because a pod can be Running but still not Ready to receive traffic.

Once Kubernetes networking is validated, I’d investigate the application itself by reviewing container logs, application logs, resource utilization, and any dependency failures such as database connectivity, Redis availability, or third-party APIs. If a deployment was recently completed, I’d compare configuration changes and perform a rollback if necessary.

After identifying the root cause, I’d restore service, verify application functionality through health checks and user validation, continue monitoring for stability, document the RCA, and implement preventive improvements such as deployment validation, better monitoring, and automated rollback strategies.”

**kubectl get pods -o wide

kubectl describe pod

kubectl logs

kubectl get svc

kubectl get endpoints

kubectl describe ingress

kubectl get ingress

kubectl describe service

kubectl get events --sort-by=.metadata.creationTimestamp

kubectl exec -it -- sh

curl http://service-name

nslookup service-name**

Common Follow-up Questions

**Q1. Why can a pod be Running but still return 503?**

Because Running only indicates the container process is active. If the Readiness Probe fails, Kubernetes removes the pod from the Service endpoints, so traffic is not sent to it.

⸻

**Q2. What is the difference between Liveness Probe and Readiness Probe?**

Liveness Probe determines whether the container should be restarted.

Readiness Probe determines whether the pod is ready to receive traffic.

⸻

**Q3. How do you identify whether the problem is in the Ingress or the Service?**

I test each layer independently:

Verify Ingress routes.

Check Service endpoints.

Test connectivity directly to the Service using kubectl port-forward or an internal curl request.

If the Service works but the Ingress doesn’t, the issue is likely with the Ingress or Load Balancer.

⸻

**Q4. How would you prevent this issue in production?**

Configure proper readiness probes.

Use Blue-Green or Canary deployments.

Add automated smoke tests after deployment.

Monitor endpoint health and HTTP 5xx rates.

Implement automatic rollback if health checks fail

————————————————————————————————————————————

Question 2

Scenario

Terraform state contains resources that no longer exist in AWS, causing state drift. Infrastructure changes cannot be trusted because the Terraform state no longer reflects the actual cloud environment.

⸻

Issue

Terraform reports resources that are missing or attempts to recreate existing infrastructure because the state file and AWS infrastructure are out of sync.

⸻

Root Cause Analysis (RCA)

Possible causes include:

Manual changes in the AWS Console.

Resources deleted outside Terraform.

Failed terraform apply.

State file corruption.

Incorrect state manipulation.

Missing terraform import.

Multiple engineers modifying infrastructure simultaneously.

State locking issues.

⸻

Resolution

I first identify whether the drift is intentional or accidental, compare the Terraform state with the actual AWS infrastructure, safely recover the state, validate changes using terraform plan, and only then perform any infrastructure modifications.

⸻

Prevention

Never make manual production changes.

Store state remotely (S3).

Enable DynamoDB state locking.

Enable S3 versioning.

Restrict IAM permissions.

Run regular drift detection in CI/CD.

Require code reviews before infrastructure changes.

⸻

How I Would Answer in the Interview

“If Terraform state shows resources that no longer exist in AWS, I wouldn’t immediately run terraform apply because that could recreate resources unexpectedly or impact production. First, I’d determine whether the missing resources were intentionally deleted or removed accidentally.

Then I’d compare the Terraform state with the actual AWS infrastructure using terraform state list, terraform plan, and the AWS Console or CLI. If the resources still exist but are missing from the state, I’d import them using terraform import. If the resources were intentionally deleted, I’d update the Terraform configuration or remove stale state entries only after validating the impact.

Before making any changes, I’d back up the current state using terraform state pull and ensure state locking is functioning correctly. Once the state is synchronized, I’d run another terraform plan to verify there are no unexpected changes. Only after confirming the plan is accurate would I proceed with an apply if required.

Finally, I’d document the RCA, identify why the drift occurred, and implement preventive controls such as Infrastructure as Code enforcement, drift detection in CI/CD, S3 versioning, and strict change management to minimize manual modifications.”

terraform state list

terraform state show

terraform plan

terraform state pull > backup.tfstate

terraform import

terraform state rm

terraform apply -refresh-only

terraform force-unlock <LOCK_ID>

Common Follow-up Questions

Q1. What causes Terraform state drift?

Manual infrastructure changes, failed deployments, deleted resources, imports not performed, or state corruption.

Q2. Can you edit the state file manually?

Only as a last resort and always with a verified backup. I prefer built-in Terraform state commands whenever possible.

Q3. How do you prevent drift?

Use remote state, state locking, code reviews, CI/CD pipelines, and prohibit manual production changes.

============================================================

Question 3

Scenario

A GitHub Actions CI/CD pipeline that normally completes in 10 minutes is suddenly taking 30–35 minutes, delaying production deployments.

Issue

Pipeline execution time has increased significantly without any obvious application changes.

⸻

Root Cause Analysis (RCA)

Possible causes include:

Cache misses (Maven, NPM, Gradle, Pip)

Docker layer cache invalidation

Self-hosted runner resource exhaustion

GitHub-hosted runner performance

Network latency

Slow dependency downloads

Additional security scans

Larger test suites

Container registry latency

Recent workflow configuration changes

⸻

Resolution

Review the workflow timeline, identify the slow stage, optimize caching, improve Docker builds, parallelize jobs, or scale runners.

Prevention

Dependency caching

Docker layer caching

Parallel job execution

Self-hosted runner monitoring

Pipeline performance dashboards

Regular workflow optimization

⸻

How I Would Answer in the Interview

“If a GitHub Actions pipeline suddenly starts taking three times longer than usual, I first determine whether the issue affects a single repository or multiple repositories because that helps identify whether it’s an application issue or an infrastructure issue. Then I review the workflow execution timeline in GitHub Actions to identify which stage has increased in duration, such as dependency installation, Docker image build, unit testing, security scanning, or deployment.

Next, I’d compare the current execution with previous successful runs to identify recent workflow changes. I’d verify whether dependency caches are being restored successfully and whether Docker layer caching is still effective. If self-hosted runners are being used, I’d check CPU, memory, disk utilization, and concurrent job execution to determine whether the runners are overloaded. I’d also investigate external dependencies such as package repositories, container registries, or vulnerability scanners that might be introducing latency.

Once I identify the bottleneck, I’d optimize the affected stage by restoring caching, parallelizing independent jobs, scaling runners if necessary, or optimizing Docker builds. After implementing the fix, I’d rerun the pipeline, compare execution times with historical data, and continue monitoring. Finally, I’d document the RCA and implement pipeline performance monitoring and alerts so similar regressions are detected early.”

Common Follow-up Questions

Q1. How do you identify which stage is slow?

Review the GitHub Actions workflow timeline and compare the duration of each job with previous successful runs.

⸻

Q2. How do you optimize Docker builds?

Multi-stage builds

Docker layer caching

Smaller base images

BuildKit

Cache dependencies

⸻

Q3. How do you improve CI/CD performance?

Parallel jobs

Dependency caching

Incremental testing

Self-hosted runners

Faster artifact storage

=============================================================

Question 4

Scenario

A microservice works normally under low traffic but starts failing intermittently when traffic increases during peak business hours.

⸻

Issue

Users experience intermittent 5xx errors and increased response times under heavy load.

Root Cause Analysis (RCA)

Possible causes include:

Database connection pool exhaustion

Thread pool exhaustion

Memory pressure

CPU bottlenecks

Network latency

External API failures

Kubernetes resource limits

Autoscaling delays

JVM Garbage Collection pauses

Load balancer configuration

How I Would Answer in the Interview

“If a microservice starts failing only during periods of high traffic, I first determine the business impact by identifying whether all users or only specific APIs are affected. Then I correlate application metrics, infrastructure metrics, logs, and distributed traces using tools such as Prometheus, Grafana, CloudWatch, and centralized logging platforms.

My next step is to verify CPU utilization, memory usage, thread counts, database connection pools, request latency, and downstream dependency health. I also review Kubernetes events and Horizontal Pod Autoscaler activity to confirm that the application is scaling correctly. If a recent deployment occurred, I’d compare application and infrastructure changes and perform a rollback if necessary.

Once the bottleneck is identified, whether it’s database contention, thread exhaustion, network latency, or resource limits, I’d implement the appropriate fix and validate it through load testing. After service stability is restored, I’d document the RCA and improve autoscaling policies, resource requests and limits, monitoring, and performance testing to reduce the likelihood of recurrence.”

kubectl top pod

kubectl top node

kubectl describe hpa

kubectl get events

kubectl logs

kubectl describe pod

netstat -an

top

free -m

Common Follow-up Questions

Q1. How do you determine whether it’s an application issue or an infrastructure issue?

By correlating application logs, infrastructure metrics, database performance, and distributed traces.

⸻

Q2. Why would an application fail only under load?

Because bottlenecks such as connection pools, thread pools, memory, CPU, or downstream services only become visible during high concurrency.

⸻

Q3. How do you prevent these issues?

Regular load testing, autoscaling validation, performance monitoring, and capacity planning.

==============================================================

Question 5

Scenario

A business-critical application must be deployed without causing any downtime for users.

kubectl rollout status deployment/app

kubectl rollout history deployment/app

kubectl rollout undo deployment/app

kubectl get pods

kubectl describe deployment app

kubectl get events

helm upgrade

helm rollback

Common Follow-up Questions

Q1. When would you choose Blue-Green instead of Rolling Update?

Blue-Green is ideal for high-risk releases because it enables instant rollback by switching traffic between environments.

⸻

Q2. What happens if readiness probes fail?

The pod remains Running but is removed from the Service endpoints, so Kubernetes does not send production traffic to it.

⸻

Q3. How do you deploy database schema changes without downtime?

I use the Expand → Migrate → Contract strategy:

Add backward-compatible schema changes.

Deploy the application.

Migrate data.

Remove deprecated schema only after all applications are updated.

Common Follow-up Questions

Q1. When would you choose Blue-Green instead of Rolling Update?

Blue-Green is ideal for high-risk releases because it enables instant rollback by switching traffic between environments.

⸻

Q2. What happens if readiness probes fail?

The pod remains Running but is removed from the Service endpoints, so Kubernetes does not send production traffic to it.

⸻

Q3. How do you deploy database schema changes without downtime?

I use the Expand → Migrate → Contract strategy:

Add backward-compatible schema changes.

Deploy the application.

Migrate data.

Remove deprecated schema only after all applications are updated.

=================================================

Question 6

Scenario

One of the Kubernetes worker nodes suddenly becomes NotReady in a production EKS cluster.

⸻

Issue

Pods running on that node may become unavailable, impacting application availability.

⸻

Root Cause Analysis (RCA)

Possible causes include:

Kubelet stopped running

EC2 instance failure

High CPU or memory utilization

Disk pressure

Network connectivity issues

Security Group or NACL changes

Container runtime failure

AWS infrastructure issue

Node filesystem full

⸻

Resolution

Identify the reason the node became NotReady, restore workloads by draining or replacing the node, and verify cluster stability.

⸻

Prevention

Multi-AZ node groups

Cluster Autoscaler

Managed Node Groups

Node monitoring

Auto replacement

Regular node health checks

⸻

How I Would Answer in the Interview

“If a Kubernetes node suddenly becomes NotReady, my first priority is understanding the business impact by identifying which workloads are running on that node and whether customer-facing services are affected. I immediately check the node status using kubectl get nodes and inspect the node events using kubectl describe node.

Next, I verify whether the issue is related to kubelet, container runtime, networking, disk pressure, or resource exhaustion. Since this is an EKS cluster, I’d also check the EC2 instance status, CloudWatch metrics, and Auto Scaling Group health. If the node is unhealthy but still accessible, I’d cordon and drain it to safely move workloads to healthy nodes. If the node is completely unavailable, I’d allow the Managed Node Group or Auto Scaling Group to replace it automatically.

Once workloads are rescheduled, I’d verify pod health, application availability, and cluster capacity before closing the incident. Finally, I’d perform a root cause analysis, document the findings, and improve monitoring, autoscaling policies, and node replacement automation to reduce future incidents.”

kubectl get nodes

kubectl describe node

kubectl get pods -o wide

kubectl drain --ignore-daemonsets

kubectl cordon

kubectl uncordon

kubectl get events

systemctl status kubelet

Common Follow-up Questions

Q1. What is the difference between Cordon and Drain?

Cordon

Marks the node as unschedulable.

Existing pods continue running.

Drain

Safely evicts workloads from the node before maintenance.

⸻

Q2. How does EKS recover automatically?

Using Managed Node Groups and Auto Scaling Groups, unhealthy nodes are terminated and replaced automatically.

⸻

Q3. What happens if the node hosting CoreDNS becomes unavailable?

CoreDNS pods are rescheduled to healthy nodes. If no healthy CoreDNS pods exist, service discovery inside the cluster is affected.

=========================================================

Question 7

Scenario

Prometheus reports very high application latency, but application logs contain no errors.

⸻

Issue

The application is slow even though logs show no obvious failures.

⸻

Root Cause Analysis (RCA)

Possible causes include:

Database latency

DNS resolution delays

External API latency

Network congestion

Thread pool exhaustion

Garbage Collection pauses

Storage latency

Load balancer delays

Service mesh latency

⸻

Resolution

Use metrics, traces, and infrastructure monitoring to isolate the slow component.

⸻

Prevention

Distributed tracing

End-to-end observability

Database monitoring

Synthetic monitoring

Performance testing

⸻

How I Would Answer in the Interview

“If Prometheus shows increasing latency but application logs contain no errors, I wouldn’t assume the application is healthy because latency issues often occur outside the application itself. I first determine whether the latency affects all APIs or only specific endpoints. Then I’d correlate metrics from Prometheus, dashboards from Grafana, application logs, and distributed traces from tools like Jaeger or OpenTelemetry.

I’d investigate CPU, memory, request duration, database response time, DNS lookup latency, network throughput, and external service dependencies. If infrastructure metrics look normal, I’d focus on downstream services such as databases or third-party APIs because they often introduce latency without generating application errors.

Once the slow component is identified, I’d implement the appropriate fix, whether it’s database optimization, connection pool tuning, DNS improvements, or scaling. After confirming latency returns to normal, I’d document the RCA and improve observability by adding tracing, dashboards, and proactive alerts.”

kubectl top pods

kubectl logs

kubectl describe pod

kubectl exec

curl

ping

traceroute

nslookup

dig

Common Follow-up Questions

Q1. Why can latency increase without errors?

Because the application may be waiting for external services rather than failing.

⸻

Q2. Why are logs alone insufficient?

Logs show events, while metrics reveal trends and traces show where requests spend time.

⸻

Q3. Which observability pillars do you use?

Metrics

Logs

Traces

===============================================================

Question 8

Scenario

Your company has over 100 microservices, and each team manages its own deployments.

⸻

Issue

Managing deployments consistently while preventing configuration drift.

⸻

Root Cause Analysis (RCA)

Challenges include:

Different deployment methods

Duplicate pipelines

Manual deployments

Configuration drift

Security inconsistencies

Slow releases

⸻

Resolution

Build a standardized GitOps-based CI/CD platform using reusable templates.

⸻

Prevention

GitOps

Reusable pipelines

Infrastructure as Code

Policy as Code

Environment promotion

⸻

How I Would Answer in the Interview

“For an organization managing more than 100 microservices, my objective is to standardize deployments while allowing teams to remain independent. Instead of maintaining separate pipelines for every service, I’d build reusable CI/CD templates that handle building, testing, security scanning, container image creation, and deployment.

I’d adopt GitOps using ArgoCD or Flux so Git becomes the single source of truth. Terraform would provision infrastructure, Helm would manage Kubernetes deployments, and environment promotion would move changes from Development to QA, UAT, and Production through controlled approvals.

I would also integrate security scanning, policy enforcement, automated rollback, and deployment monitoring into every pipeline. This approach improves consistency, reduces manual effort, and allows hundreds of services to be deployed reliably with minimal operational overhead.”

helm upgrade

kubectl apply

terraform apply

argocd app sync

kubectl rollout status

Common Follow-up Questions

Q1. Why GitOps?

Git provides version control, auditability, rollback, and automated synchronization.

⸻

Q2. How do you avoid duplicate pipelines?

Reusable GitHub Actions or Jenkins shared libraries.

⸻

Q3. How do you deploy 100 services safely?

Standardized templates, Helm charts, ArgoCD, automated testing, and progressive deployment.

===============================================================

Question 9

Scenario

The application works perfectly in Development but fails after deployment to Production.

⸻

Issue

Environment-specific failure.

⸻

Root Cause Analysis (RCA)

Possible causes include:

Missing Secrets

Incorrect ConfigMaps

IAM permissions

Resource limits

DNS differences

Security Groups

NetworkPolicy

Different container images

⸻

Resolution

Compare environments systematically and identify configuration differences.

⸻

Prevention

Environment parity

Infrastructure as Code

Automated validation

Configuration management

⸻

How I Would Answer in the Interview

“If an application works in Development but fails in Production, I wouldn’t immediately assume there’s a code defect because environment differences are a common cause. First, I’d compare deployment manifests, ConfigMaps, Secrets, resource limits, IAM permissions, networking, DNS configuration, and container images between the two environments.

I’d verify application logs, Kubernetes events, readiness probes, and connectivity to external services such as databases or APIs. If a recent deployment introduced the issue, I’d compare image versions and configuration changes before deciding whether a rollback is necessary.

Once the configuration difference is identified, I’d implement the fix, validate production health, and continue monitoring. Finally, I’d document the RCA and improve environment standardization using Infrastructure as Code and automated configuration validation.”

kubectl describe pod

kubectl logs

kubectl get configmap

kubectl get secret

kubectl exec

kubectl diff

helm diff

Common Follow-up Questions

Q1. Why does software work in Dev but fail in Prod?

Configuration differences are usually responsible.

⸻

Q2. How do you compare environments?

Using Git, Helm values, Terraform state, Kubernetes manifests, IAM policies, and Secrets.

⸻

Q3. How do you prevent this?

Environment parity, Infrastructure as Code, automated validation, and GitOps.

===============================================================

Question 10

Scenario

Your organization manages hundreds of application secrets across AWS, Kubernetes, Jenkins, and GitHub Actions.

⸻

Issue

Securely managing secrets without exposing credentials.

⸻

Root Cause Analysis (RCA)

Common issues:

Hardcoded passwords

Long-lived credentials

Weak RBAC

Shared accounts

Poor secret rotation

Secrets stored in Git

⸻

Resolution

Implement centralized secrets management with least privilege and automatic rotation.

⸻

Prevention

AWS Secrets Manager

HashiCorp Vault

IAM Roles

OIDC authentication

Secret rotation

RBAC

Audit logging

⸻

How I Would Answer in the Interview

“Secrets management is one of the most critical aspects of securing cloud infrastructure. I never store secrets in source code or container images. Instead, I use centralized solutions such as AWS Secrets Manager or HashiCorp Vault integrated with Kubernetes, Jenkins, and GitHub Actions.

For AWS workloads, I prefer IAM Roles and OIDC authentication instead of long-lived access keys. Applications retrieve secrets dynamically at runtime, eliminating the need for hardcoded credentials. Access is controlled using the principle of least privilege, and all secret access is logged for auditing.

I also implement automatic secret rotation, regular permission reviews, and secret scanning within CI/CD pipelines to detect accidental credential exposure. Finally, I continuously monitor secret usage and document security improvements as part of the organization’s governance process.”

kubectl get secret

kubectl describe secret

aws secretsmanager get-secret-value

vault kv get

kubectl exec

Common Follow-up Questions

Q1. Why shouldn’t secrets be stored in Git?

Because Git history is permanent, making exposed credentials difficult to remove completely.

⸻

Q2. Why use OIDC instead of AWS Access Keys?

OIDC provides short-lived credentials, reducing the risk of credential leakage.

⸻

Q3. Which secrets manager do you recommend?

For AWS-native environments, AWS Secrets Manager is ideal. In multi-cloud or hybrid environments, HashiCorp Vault is often the better choice.

===============================================================

Question 11

Scenario

Your application requires a database schema change in production, but downtime is not acceptable because the application serves thousands of active users.

⸻

Issue

Deploy database schema changes without breaking existing application versions or causing downtime.

⸻

Root Cause Analysis (RCA)

Common causes of deployment failures include:

Breaking schema changes

Dropping columns before application migration

Long-running table locks

Incompatible application versions

Failed migration scripts

Rollback difficulties

⸻

Resolution

Implement the Expand → Migrate → Contract strategy to ensure backward compatibility throughout the deployment.

⸻

Prevention

Version-controlled migrations (Flyway/Liquibase)

Blue-Green or Canary deployments

Backward-compatible schema changes

Database backups before migration

Migration testing in lower environments

⸻

How I Would Answer in the Interview

“Whenever a production database schema change is required, my first priority is ensuring there is no impact on running applications. I never perform breaking schema changes during a live deployment because different application versions may run simultaneously during rolling updates.

I follow the Expand → Migrate → Contract strategy. First, I expand the schema by adding new columns or tables without removing existing ones. Then I deploy the new application version that supports both the old and new schema. After validating production traffic and migrating existing data, I remove deprecated columns or objects in a later deployment. Before starting the migration, I verify database backups, review rollback procedures, and test the migration in lower environments. During deployment, I continuously monitor database performance, application error rates, and replication health. If unexpected issues occur, I stop the migration and roll back the application while preserving database integrity. After successful completion, I document the RCA and improve migration automation to minimize future deployment risks.”

kubectl rollout status deployment/app

flyway migrate

liquibase update

kubectl logs

helm rollback

Common Follow-up Questions

Q1. Why use Expand → Migrate → Contract?

It ensures backward compatibility during rolling deployments.

⸻

Q2. What if migration fails halfway?

Stop the deployment, restore from backup if required, or execute rollback scripts while maintaining data consistency.

⸻

Q3. How do you avoid locking large tables?

Perform online schema changes, batch migrations, or use database-specific online DDL features.

============================================================

Question 12

Scenario

AWS monthly cloud cost suddenly increases by 40% without any major product release.

⸻

Issue

Unexpected cloud spending impacts the company’s budget.

⸻

Root Cause Analysis (RCA)

Possible causes include:

Autoscaling events

Unused EC2 instances

Overprovisioned EKS nodes

Idle Load Balancers

EBS volume growth

Snapshot accumulation

NAT Gateway traffic

S3 storage growth

Cross-region data transfer

Memory leaks causing scaling

⸻

Resolution

Analyze AWS Cost Explorer, CloudWatch metrics, Trusted Advisor, and infrastructure utilization to identify unnecessary spending.

⸻

Prevention

AWS Budgets

Cost Anomaly Detection

Rightsizing

Autoscaling optimization

Scheduled shutdowns

Reserved Instances

Savings Plans

⸻

How I Would Answer in the Interview

“If AWS costs suddenly increased by 40%, I wouldn’t immediately start deleting resources because that could affect production. First, I’d identify which AWS services contributed to the increase using Cost Explorer and Cost and Usage Reports. Then I’d compare current usage with previous billing periods to identify changes in EC2, EKS, RDS, S3, NAT Gateway, or data transfer costs.

After identifying the affected service, I’d correlate billing data with CloudWatch metrics and recent infrastructure changes. For example, I’d verify whether autoscaling created additional EC2 instances, whether storage usage increased unexpectedly, or whether idle resources were left running after testing. Once I identify the root cause, I’d safely remove unused resources, right-size workloads, optimize autoscaling policies, or purchase Savings Plans where appropriate.

Finally, I’d document the RCA and implement AWS Budgets, Cost Anomaly Detection, resource tagging, and regular cost reviews to prevent unexpected spending in the future.”

Common Follow-up Questions

Q1. Which AWS services most commonly increase costs?

EC2, EKS, RDS, NAT Gateway, S3, and data transfer.

⸻

Q2. How do you reduce Kubernetes costs?

Cluster Autoscaler, Karpenter, Spot Instances, right-sized requests and limits, and removing idle workloads.

⸻

Q3. What AWS tools help optimize costs?

AWS Cost Explorer, Trusted Advisor, Compute Optimizer, AWS Budgets, and Cost Anomaly Detection.

========================≠==================================

Question 13

Scenario

An application’s memory usage continuously increases until Kubernetes eventually kills the pod with an OOMKilled event.

⸻

Issue

Application becomes unstable because memory consumption keeps growing.

⸻

Root Cause Analysis (RCA)

Possible causes include:

Memory leak

Cache leak

Thread leak

Unreleased file handles

Infinite object creation

Large in-memory datasets

Inefficient Garbage Collection

⸻

Resolution

Analyze heap dumps, memory metrics, garbage collection behavior, and application profiling to identify the leak.

⸻

Prevention

Memory monitoring

Heap analysis

Load testing

Code reviews

Resource limits

Performance profiling

⸻

How I Would Answer in the Interview

“If an application’s memory usage continuously increases until the pod is OOMKilled, I first determine whether the issue affects one instance or all application replicas. Then I’d review Kubernetes events, application logs, memory metrics, and recent deployments. I’d verify whether memory growth is expected due to traffic or indicates a leak.

If a memory leak is suspected, I’d capture heap dumps, analyze garbage collection behavior, and profile the application to identify unreleased objects or caches. I also review resource requests and limits to ensure Kubernetes isn’t terminating the pod due to incorrect configuration. If the issue began after a recent deployment, I’d compare application changes and roll back if necessary.

After resolving the leak, I’d validate memory stability under load testing, continue monitoring, and document the RCA. Finally, I’d improve observability by implementing memory dashboards, alerting, and regular performance testing.”

Common Follow-up Questions

Q1. What is OOMKilled?

The Linux kernel terminates the process because it exceeded its memory limit.

⸻

Q2. How do you identify a memory leak?

Heap dumps, profiling tools, GC logs, and long-term memory trend analysis.

⸻

Q3. Why doesn’t restarting the pod solve the problem?

It restores service temporarily but doesn’t eliminate the underlying application defect.

==============================================================

Question 14

Scenario

An entire Kubernetes production cluster becomes unavailable because of a regional outage.

⸻

Issue

Critical applications are offline and require disaster recovery.

⸻

Root Cause Analysis (RCA)

Possible causes include:

Region outage

Cluster corruption

etcd failure

Storage failure

Network failure

Human error

⸻

Resolution

Recover infrastructure using Infrastructure as Code, restore workloads from GitOps repositories, and recover persistent data from backups.

⸻

Prevention

Multi-region architecture

Velero backups

GitOps

Regular disaster recovery drills

Cross-region database replication

⸻

How I Would Answer in the Interview

“If an entire Kubernetes cluster becomes unavailable, my first priority is restoring business-critical services rather than investigating the root cause immediately. I first activate the disaster recovery plan and assess whether the failure is limited to the cluster or affects the entire AWS region.

If a secondary region is available, I’d restore infrastructure using Terraform and deploy applications using GitOps tools such as ArgoCD. Persistent data would be recovered from database replicas or backup solutions like Velero. Throughout the recovery process, I’d validate application health, networking, DNS, and user access before declaring service restored.

After recovery, I’d conduct a detailed RCA, document lessons learned, and schedule disaster recovery drills to ensure the recovery objectives continue to meet business requirements.”

Common Follow-up Questions

Q1. What are RTO and RPO?

RTO (Recovery Time Objective): Maximum acceptable recovery time.

RPO (Recovery Point Objective): Maximum acceptable data loss.

⸻

Q2. Why use GitOps for disaster recovery?

Git contains the desired cluster state, allowing rapid and consistent recovery.

⸻

Q3. Which Kubernetes backup tool do you prefer?

Velero for cluster resources and persistent volume backups.

=============================================================

Question 15

Scenario

You receive a critical alert at 2:00 AM stating that the production application is completely unavailable.

⸻

Issue

High-severity production outage affecting customers.

⸻

Root Cause Analysis (RCA)

Potential causes include:

Failed deployment

Infrastructure failure

Database outage

DNS issues

Certificate expiration

Kubernetes failure

Cloud service outage

External dependency failure

⸻

Resolution

Restore customer service first, then investigate the root cause and implement preventive measures.

⸻

Prevention

Comprehensive monitoring

Runbooks

Automated rollback

Disaster recovery testing

On-call procedures

Incident reviews

⸻

How I Would Answer in the Interview

“If I receive a critical production alert at 2 AM, my first priority is restoring customer service as quickly and safely as possible. I begin by assessing the business impact, identifying which services are affected, and checking dashboards, alerts, and recent deployment history. I avoid making assumptions and instead follow the production runbook.

I then trace the request flow from the Load Balancer through Kubernetes, application services, databases, and external dependencies to isolate the failing component. If the issue is related to a recent deployment, I perform an immediate rollback while continuing the investigation. If it’s an infrastructure failure, I activate the appropriate disaster recovery or failover procedures.

Once service is restored, I continue monitoring until the system is stable before declaring the incident resolved. Afterward, I conduct a blameless post-incident review, document the root cause, identify contributing factors, and implement preventive improvements such as enhanced monitoring, automation, deployment validation, or architectural changes. My objective is not only to restore service quickly but also to reduce the likelihood of the same incident happening again.”

kubectl get pods

kubectl get events

kubectl logs

kubectl rollout undo

terraform plan

aws cloudwatch describe-alarms

systemctl status

journalctl -xe

Common Follow-up Questions

Q1. What’s your first priority during a P1 incident?

Restore customer-facing service safely while maintaining clear communication with stakeholders.

⸻

Q2. Do you investigate first or restore first?

Restore service first if a safe rollback or failover is available, then perform a thorough investigation.

⸻

Q3. What is a blameless postmortem?

A structured review that focuses on improving systems and processes rather than assigning individual blame.

Difference between count vs for_each Sample answer “Both are used to create multiple resources, but they behave differently. count is index-based and is useful when resources are nearly identical. for_each is key-based and better when resources are defined by unique names or maps. In production IaC, I prefer for_each when identity matters because it is more stable. With count, if the list order changes, resource addressing may shift and cause unintended replacement. With for_each, the key remains stable, so lifecycle is more predictable. So in interview, I’d say: ‘I use count for simple repeated resources and for_each when I want better control, readability, and safer changes for uniquely named resources.’”

How do you achieve zero-downtime deployments? Sample answer “Zero-downtime deployment means users should continue to get service while the new version is rolled out. In Kubernetes, that typically means having multiple replicas, proper readiness probes, rolling update strategy, and enough capacity so the old pods are only terminated after the new pods are actually ready. Depending on business criticality, I may also use blue-green or canary strategies. The core idea is:

never drain all healthy capacity at once validate readiness before traffic shift keep rollback fast monitor during rollout

My interview answer would be: ‘I design zero-downtime using readiness-driven rolling deployments, controlled traffic switching, and post-deploy validation. For critical releases, I prefer canary or blue-green depending on risk and infrastructure design.’”

Difference between Blue-Green, Canary, Rolling deployment Sample answer “Rolling deployment gradually replaces old instances with new ones. It is simple and efficient, but if the issue is subtle, some users may still hit the bad version during rollout. Blue-Green deployment keeps two full environments: current and new. Traffic switches only when the new environment is validated. It gives safer rollback but requires more infrastructure. Canary deployment releases the new version to a small percentage of users first. It is best when I want controlled exposure and metric-based validation before full rollout. In interview, I’d answer: ‘Rolling is efficient for normal releases, blue-green is strong for safer cutover, and canary is best when I want gradual risk-controlled validation in production.’”
19)How To Secure Your Kubernetes Cluster in Production Kubernetes gives you enormous flexibility in how you run workloads. That flexibility is also what makes it easy to misconfigure. A cluster with default settings, no network policies, and containers running as root is not a production ready cluster, it is a set of open doors. Security in Kubernetes is not a single setting you toggle on. It is a layered set of controls that together limit what can happen if something goes wrong. Today we cover the most important security controls in Kubernetes. Authentication, Authorization, and RBAC Every request to the Kubernetes API server goes through two checks before anything happens. Authentication answers the question of who is making the request - is this a human user, a service account, or a pod? Authorization answers what that identity is allowed to do. Kubernetes uses Role Based Access Control as its authorization mechanism, and understanding how RBAC works is the foundation of everything else.

https://ci3.googleusercontent.com/meips/ADKq_NYf8Cf5adOHrq6PZ0fl36qPkw0ngrIjnz5J_v-U3OZAVj4n9LDaZZi5IYtZHubcAIYiVe1JIGKgOdQKtKMF5rV_rOLbDJO6PSWMHPAq285jGkkT-Q-mZ_OMrrEX3896HfT_z4iGfOutcKZrsFhqhf2B51fUNfSkhV8XK5UI92OYQgFuaj7RltHPPeGMyLjY0xPELJygZoxbsI69Mc2rNryxjQH7WyH35rSBnosbEZKYld5TV4_d6sATwhand_xoZQ=s0-d-e1-ft#https://media.beehiiv.com/cdn-cgi/image/fit=scale-down,format=auto,onerror=redirect,quality=80/uploads/asset/file/59fa4c57-775c-4e29-8a2d-79a18a207a99/1.png?t=1781943812

RBAC has two core object types that work together. Roles define what actions can be performed on which resources. A Role might say: get, list, and watch pods in a specific namespace. A ClusterRole says the same but applies cluster wide rather than being scoped to a single namespace. RoleBindings and ClusterRoleBindings then assign those roles to specific identities, a user, a group, or a service account. The separation is important: the Role describes capabilities, the Binding describes who has them. The most common RBAC mistake is over permissioning. A service account that only needs to read ConfigMaps in one namespace should not have cluster admin. A developer who needs to view pod logs should not have the ability to delete namespaces. The principle of least privilege applies here exactly as it does everywhere else in security. Every role you create should grant only the permissions the identity actually needs and nothing more. Audit your RBAC rules regularly, permission creep in Kubernetes is real and it accumulates silently. Network Policies: Controlling pod-to-pod traffic By default, every pod in a Kubernetes cluster can reach every other pod, regardless of namespace. There are no network level restrictions. In a small cluster with trusted workloads this is manageable. In a multi tenant cluster or any environment handling sensitive data, it is a significant risk. If one pod is compromised, it can freely connect to any other pod in the cluster.

https://ci3.googleusercontent.com/meips/ADKq_NY1f9YVUvWoxqjGInmi1Rpk-K8EAdAiJCP__x-SMhXAceE8PWVU4KjJhug9vhYuCIFtsU7-Y5UBTzohwTSKlkTg6mnywDz5MpOJdXNJ54QEClPenqRLaZpHGYMUvUwHp8qphW5mtp_udCGzymlGGlktV8N1xUXfYLgWZRCmBCbl8sKu9QL8N0wQuBT8Qatcj46fOrbgHVzwAOznxXl1CmCVh-L_pO0F1apQiMKhC5hvXdBYtcdv42A3BFjApwJgjg=s0-d-e1-ft#https://media.beehiiv.com/cdn-cgi/image/fit=scale-down,format=auto,onerror=redirect,quality=80/uploads/asset/file/be3fe4fd-c60c-42d8-b29c-c5cd070494e3/2.png?t=1781943820

Network Policies are Kubernetes objects that define rules for which pods can communicate with which other pods and on which ports. They work at the IP and port level inside the cluster and are enforced by the network plugin, not Kubernetes itself. This means you need a CNI plugin that supports NetworkPolicy enforcement, Canal, Calico, Cilium, and Weave all do. Flannel alone does not. The recommended approach is to start with a default deny policy in every namespace. An empty podSelector with policyType: Ingress blocks all inbound traffic to all pods in the namespace. You then add specific allow policies for the traffic that actually needs to flow. A policy allowing the web pod to reach the db pod on port 3748 is explicit, auditable, and scoped precisely to what is needed. Everything else stays blocked. This deny-by-default posture means that a new pod added to the namespace has no network access until someone explicitly creates a policy for it, which is the correct default behaviour in a production environment.
