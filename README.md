
1) Explain your end-to-end CI/CD pipeline
Sample answer
“In my project, the CI/CD flow starts when a developer pushes code to Git. That triggers the pipeline through a webhook or GitHub Actions/Jenkins trigger. In the CI stage, we do source checkout, dependency installation, unit test execution, static code analysis, and security scanning. If those checks pass, we build the artifact or Docker image and push it to the artifact repository or container registry.
In the CD stage, we deploy first to lower environments like Dev or QA, run smoke tests, and then promote the same tested artifact to staging and production. For production, we usually keep approvals, health checks, and rollback steps in place. In Kubernetes-based deployments, we validate rollout status, pod health, readiness, logs, and monitoring after deployment.
In one of my real examples, I worked on pipelines where deployment changes also involved secrets/config handling and monitoring validation post-release. So for me, CI/CD is not just ‘build and deploy’; it includes code quality, secure secrets handling, observability checks, and rollback readiness.”

2) Deployment succeeded but app is not accessible. How do you troubleshoot?
Sample answer
“I troubleshoot this layer by layer instead of assuming it’s an application issue. First, I verify whether the pods are actually running and ready, because a successful deployment object does not always mean the application is healthy. I check pod status, readiness probe, liveness probe, restart count, logs, and recent events.
Next, I validate service routing — whether the Service is pointing to the correct labels, whether endpoints are populated, and whether ingress/load balancer configuration is correct. If the pods are healthy but traffic is not flowing, I check ingress rules, TLS, target groups, security groups, and network policies if applicable.
Then I test connectivity from inside the cluster and outside the cluster. If internal access works but external access fails, the issue is probably ingress, DNS, firewall, or load balancer related. If even internal access fails, then I check application startup, port mismatch, env vars, secrets, and downstream dependencies like DB or cache.
A real-world style example would be: deployment went green, but users couldn’t access the app. On validation, the pods were healthy, but the readiness probe path was incorrect after the new release. Because readiness never passed consistently, service endpoints were unstable, and traffic was not routed properly.”

3) How do you identify whether the issue is application, DB, or network?
Sample answer
“I usually isolate by evidence, not by guesswork. First, I check the user symptom: is it timeout, 5xx error, connection refused, high latency, or partial failure? That already gives direction. Then I correlate application logs, metrics, and dependency health.
If the application logs show exceptions, thread pool exhaustion, memory issues, or failed dependency calls, I suspect application logic or runtime behavior. If the app is healthy but DB calls are slow, connection pools are exhausted, or query latency is high, then I move toward database investigation. If requests are timing out, there is packet loss, or the app cannot even connect to the DB/service endpoint, then network becomes more likely.
I usually validate using three checks:

Application health — logs, CPU, memory, restarts, error rate
Database health — connection count, query latency, locks, saturation
Network health — DNS resolution, routing, security rules, latency, connectivity tests

For example, in a production-like scenario, if API latency suddenly increases while CPU is normal, I don’t blame the app immediately. I check database query timings, downstream service latency, and network path first.”

4) Pod is in CrashLoopBackOff — what steps will you take?
Sample answer
“My approach is structured. First, I describe the pod and inspect events to identify whether the failure is due to image, command, config, probes, resource pressure, or dependency failure. Then I check the current logs and previous logs because often the container crashes before the latest logs are enough.
I validate the startup command, entrypoint, environment variables, mounted secrets/config maps, and whether the application is trying to connect to something unavailable during initialization. I also check resource usage — for example, if the pod is getting OOMKilled, then the root cause is different from an app crash.
Then I compare this deployment with the last known working version. Many CrashLoopBackOff issues come from env var mismatch, secret changes, image tag mistakes, wrong port, or readiness/liveness misconfiguration.
A strong example answer is: ‘I had a situation where a pod entered CrashLoopBackOff after deployment. I checked describe output and previous logs, and found that the application failed during startup because a required secret was not mounted correctly. After validating the secret reference and redeploying, the application came up normally.’”

5) Application works locally but fails in Kubernetes — debugging approach
Sample answer
“When an app works locally but fails in Kubernetes, I focus on environment differences. Local success only proves the code works in one setup; Kubernetes introduces differences in networking, configs, secrets, DNS, resources, startup order, and probes.
I start with the container itself: is the same image running? Is the correct port exposed? Is ENTRYPOINT/CMD correct? Then I verify application config in Kubernetes — secrets, config maps, env vars, mounted files, service discovery endpoints, and external dependency URLs.
After that, I check platform-specific issues:

resource limits too low
readiness/liveness probes causing restarts
service account or permissions issues
DNS/service name issues
ingress/service port mismatch

For example, I would say: ‘The app worked locally, but in Kubernetes it failed because the environment variable for DB hostname was different and the service name in-cluster was not matching the local config. Once the config was corrected and validated through pod exec and connectivity checks, the issue was resolved.’”

6) How do you rollback a failed deployment safely in production?
Sample answer
“I treat rollback as a controlled operation, not just a command. First, I verify whether the issue is actually caused by the latest release. I check release timestamps, change records, monitoring, logs, and blast radius. If the new release is clearly causing user impact, I rollback to the last known stable version.
In Kubernetes, that could mean rolling back the Deployment/Helm release. But I also verify whether there were schema changes, config changes, feature flags, or dependency changes, because rolling back code alone may not fully restore service if the issue came from config or DB migration.
A safe rollback process for me includes:

confirm impact and root area
rollback deployment version
validate pods/readiness/endpoints
monitor error rate and latency
confirm user journey is restored

A practical example: if users started seeing errors immediately after a release, I would rollback the release version, validate that the older pods are healthy, confirm traffic is routed correctly, and then keep monitoring to ensure the issue is actually resolved.”

7) Monitoring alerts fired after deployment — how do you validate real issue vs false alert?
Sample answer
“I don’t assume every alert after deployment means the release is bad. I first correlate the deployment time with alert timing and then check whether the alert is based on real user impact or just temporary metric fluctuations.
For example, after deployment, some alerts may fire due to pod restarts, cold starts, cache warm-up, or expected scaling behavior. So I validate:

error rate trend
latency trend
traffic level
pod health
restart count
business/user impact

If the alert is sustained and correlated with symptoms like 5xx increase, failed requests, or readiness failures, then I treat it as real. If it clears quickly and user impact is absent, it may be alert noise, threshold sensitivity, or rollout-related transient behavior.
A good interview answer is: ‘I validate alerts using metrics, logs, and user-facing signals. If alerts persist and align with actual degradation, I escalate and investigate. If it’s short-lived and expected due to rollout behavior, I document it and tune alert rules if necessary.’”

8) How do Prometheus and Grafana work together?
Sample answer
“Prometheus is the system that collects and stores metrics. It scrapes exporters or application endpoints at intervals and stores that time-series data. Grafana sits on top as the visualization layer, which queries Prometheus and presents dashboards, graphs, and alert views in a readable way.
In practical terms, Prometheus answers questions like CPU usage, request latency, pod restarts, error rates, and memory utilization. Grafana helps teams visualize those metrics, correlate trends, and create dashboards for operations and leadership visibility.
In a real production support context, I’ve used this combination to validate service health, identify which pods or clusters are affected, and investigate whether issues are persistent or isolated. It’s especially powerful when paired with alerts, so the team gets notified and then uses Grafana to drill down into the impacted service or environment.”

9) GitHub Actions/Jenkins pipeline failed without code change — what will you check?
Sample answer
“If there is no code change, I investigate external and environmental dependencies first. Pipeline failures without code changes often come from expired credentials, runner/agent issues, tool version changes, plugin updates, package repository problems, artifact registry access issues, or infrastructure/network problems.
My approach is:

check exact failed stage
compare with last successful run
inspect logs for auth, DNS, timeout, version, or dependency issues
validate secrets/tokens/certificates
check runner health and recent platform changes

A realistic example would be a pipeline suddenly failing because a cloud credential expired or a package manager mirror was unavailable. In that case, the application code is fine — the failure is environmental. I always separate ‘code issue’ from ‘pipeline environment issue’ early to save debugging time.”

10) Terraform apply failed midway — how will you recover?
Sample answer
“If Terraform apply fails midway, my first concern is state consistency. I don’t immediately rerun apply blindly. First, I read the error carefully and identify whether the failure came from auth, quota, dependency ordering, provider issue, or resource conflict.
Then I check the Terraform state versus actual cloud resources. Sometimes some resources are created successfully but not fully reflected the way we expect. I run plan again after understanding the failure and validate whether Terraform wants to create, replace, or destroy anything unexpectedly.
If state drift exists, I reconcile it carefully — either by import, state correction, or fixing the underlying issue before rerunning. In team environments, remote backend and locking are important so only one apply is in progress.
A strong interview example is: ‘I had a partial apply scenario where some resources were already created, but the pipeline failed before completion. I validated the actual resources, checked state consistency, corrected the configuration issue, reran plan, and only then proceeded with a controlled apply.’”

11) Pod is running but not receiving traffic — what will you check?
Sample answer
“If the pod is running but not getting traffic, I validate the full request path. Running status alone is not enough. First, I check whether the pod is actually Ready. If readiness is failing, Kubernetes won’t send traffic even if the pod is in Running state.
Next, I check whether the Service selectors match the pod labels and whether endpoints are populated. Then I validate ingress rules, target ports, service ports, and any network policies. If external traffic is failing, I also check DNS, LB health checks, ingress controller logs, and TLS config.
A practical answer would be: ‘I start with readiness and service endpoints. Then I move outward—service, ingress, load balancer, DNS. In many cases the issue is label mismatch, wrong targetPort, or readiness not passing consistently.’”

12) Difference between readiness and liveness probe + real impact
Kubernetes provides health probes to determine the state of an application.

Readiness Probe checks whether the application is ready to receive traffic. If the readiness probe fails, Kubernetes removes the pod from the Service endpoints, so no new traffic is sent to it. However, the container continues running because it may recover.

Liveness Probe checks whether the application is still healthy. If the liveness probe fails repeatedly, Kubernetes assumes the application is unhealthy and restarts the container automatically.

The key difference is:

* Readiness = Traffic Control
* Liveness = Container Restart

In production, proper probe configuration is critical. For example, if a liveness probe is too aggressive during application startup, Kubernetes may continuously restart the container before it finishes initializing, resulting in a CrashLoopBackOff.

Similarly, if the readiness probe is misconfigured, the application may be running correctly, but Kubernetes won’t send any traffic to it, causing users to experience 503 Service Unavailable errors.
⸻
Cross Question 1
Interviewer:
What happens if the readiness probe fails?

Answer

If the readiness probe fails, Kubernetes removes that pod from the Service endpoints.

The pod continues running, but no new traffic is routed to it.

Kubernetes keeps checking the readiness probe, and once it succeeds again, the pod is automatically added back to the Service endpoints without restarting the container.
⸻

Cross Question 2
Interviewer:

What happens if the liveness probe fails?

Answer

If the liveness probe fails continuously based on the configured failure threshold, Kubernetes restarts the container.

This is useful when the application is deadlocked or unresponsive and cannot recover on its own.
⸻

Cross Question 3
Interviewer:

Can a pod be Running but Not Ready?

Answer

Yes.

This is very common during application startup.

The container process is running, so the pod status is Running, but the readiness probe hasn’t passed yet.

Kubernetes waits until the readiness probe succeeds before routing traffic to the pod.

This is one of the favorite interviewer questions.
⸻

Cross Question 4
Interviewer:

Can a pod be Ready but later become Not Ready?

Answer

Yes.

If the application loses database connectivity, becomes overloaded, or a dependency becomes unavailable, the readiness probe may start failing.

Kubernetes immediately removes the pod from the Service endpoints but keeps the container running, allowing it to recover without a restart.
⸻

Cross Question 5
Interviewer:

Why not use only a liveness probe?

Answer

Because restarting the application isn’t always the correct action.

Sometimes the application is healthy but temporarily unable to serve requests—for example:

* Database unavailable
* Cache unavailable
* Application warming up
* High load

In these cases, removing the pod from traffic is preferable to restarting it.
⸻

Cross Question 6
Interviewer:

Why not use only a readiness probe?

Answer

A readiness probe only controls whether the pod receives traffic.

If the application becomes deadlocked or hangs indefinitely, the readiness probe alone won’t restart it.

That’s why a liveness probe is needed to recover from unrecoverable failures.
⸻

Cross Question 7 (Very Important)
Interviewer:

What is a Startup Probe?

Answer

A startup probe is designed for applications that take a long time to initialize.

While the startup probe is running, Kubernetes ignores both liveness and readiness probes.

Once the startup probe succeeds, Kubernetes starts evaluating the readiness and liveness probes.

This prevents applications from being restarted before they finish starting.
⸻

Cross Question 8
Interviewer:

Why do applications go into CrashLoopBackOff because of probes?

Answer

If the liveness probe starts checking too early, before the application has finished initializing, Kubernetes assumes the application is unhealthy and restarts it.

This repeats continuously, resulting in a CrashLoopBackOff.

We can prevent this by configuring:

* initialDelaySeconds
* failureThreshold
* timeoutSeconds
* Or by using a startup probe.

⸻
Cross Question 9
Interviewer:

Which probe affects the Service Endpoints?

Answer

Only the readiness probe.

When it fails, Kubernetes removes the pod from the Service endpoints, so it stops receiving traffic.
⸻

Cross Question 10 (Production Scenario)
Interviewer:

Users receive 503 after deployment. What will you check first?

Answer

My first step is to verify whether the new pods are Ready.

kubectl get pods
kubectl describe pod <pod-name>
kubectl get endpoints
kubectl describe svc
If the pods are Running but not Ready, the Service will have no healthy endpoints, which commonly results in 503 errors.
⸻
Production Example

In one EKS deployment, our application took about 60 seconds to start because it loaded configuration and established database connections.

The liveness probe was configured with an initial delay of only 10 seconds. Kubernetes assumed the application was unhealthy and restarted it repeatedly, leading to a CrashLoopBackOff.

We introduced a startup probe and increased the liveness probe’s initial delay. Once the application had enough time to initialize, it became Ready and the deployment completed successfully.

13) How do you debug high CPU / memory / OOMKilled issues in Kubernetes?

When I see high CPU, high memory usage, or an OOMKilled event, I first determine whether the issue is temporary or persistent. My goal is to identify the root cause before making configuration changes.

My troubleshooting approach is:

First, I verify the symptoms.

* Is the issue affecting one pod or multiple pods?
* Did it start after a deployment?
* Is it related to increased traffic or scheduled jobs?

Next, I check resource utilization.

* CPU and memory usage.
* Historical metrics.
* Requests and limits.
* Node resource utilization.

For an OOMKilled pod, I verify:

* Whether the container exceeded its memory limit.
* If the configured memory limit is too low.
* Whether there’s a memory leak or unbounded cache.
* Large payload processing or inefficient application behavior.

For high CPU, I investigate whether it’s expected or abnormal.
Common causes include:

* Traffic spikes.
* Inefficient algorithms.
* Retry storms.
* Infinite loops.
* Excessive logging.
* Dependency latency causing repeated retries.

I also correlate logs, metrics, deployment history, and traffic patterns. If the issue started immediately after a deployment, I compare the new release with the previous stable version to determine whether the application introduced inefficient processing.

I avoid simply increasing resource limits without understanding the root cause because that only masks the underlying problem.
⸻
Production Example (CPU Spike)

After a deployment on EKS, CPU utilization increased from around 40% to over 95% across all application pods.

I checked the deployment timeline and confirmed the spike began immediately after the release. Using Grafana, I noticed that requests to an external API had increased significantly.

The new application version introduced aggressive retry logic without exponential backoff. When the downstream service became slow, every pod repeatedly retried failed requests, driving CPU usage much higher.

We rolled back the deployment to stabilize production, then updated the retry mechanism with exponential backoff and circuit breaking. CPU usage returned to normal after the fix.
⸻
Production Example (OOMKilled)

In another incident, several pods entered OOMKilled status.

I confirmed this using kubectl describe pod, which showed the termination reason as OOMKilled.

Grafana indicated that memory usage steadily increased over time rather than spiking suddenly. After reviewing the application, the development team identified a memory leak caused by objects remaining in memory longer than expected.

As a short-term mitigation, we increased the memory limit to restore service stability. The long-term fix was correcting the memory leak in the application code.
Would you immediately increase the memory limit?”

A strong answer is:

No. Increasing the limit may temporarily reduce incidents, but it’s only a mitigation. First, I determine whether the problem is caused by an application memory leak, traffic growth, incorrect resource configuration, or inefficient processing. Once the root cause is identified, I apply the appropriate long-term fix.

14) What happens when a node becomes NotReady?

When a Kubernetes node becomes NotReady, it means the control plane is no longer receiving regular heartbeats from the node, usually because the kubelet has stopped reporting its status or there’s a network or infrastructure issue.

Once the node is marked NotReady, Kubernetes immediately stops scheduling new pods onto that node.

Existing pods may continue running for a short period if the node is still functioning, but if the node remains unavailable beyond the configured timeout, the Node Controller marks those pods for eviction. If replicas are available and there is sufficient cluster capacity, Kubernetes reschedules the affected pods onto healthy nodes.

During troubleshooting, I follow a structured approach:

* Check the node conditions using kubectl describe node.
* Verify whether the kubelet service is running.
* Check node events and Kubernetes events.
* Verify network connectivity between the node and the control plane.
* Check CPU, memory, and disk pressure.
* Verify container runtime health (Docker or containerd).
* If it’s a cloud-managed cluster, verify the EC2 or VM instance health.

The business impact depends on the application architecture. If workloads have multiple replicas distributed across different nodes and Availability Zones, users may not notice any interruption. However, if critical applications are running as a single replica on the failed node, service disruption is likely until the workload is rescheduled.

In production, I reduce this risk by designing for high availability using multiple replicas, Pod Anti-Affinity, Pod Disruption Budgets, Cluster Autoscaler, and node groups spread across multiple Availability Zones.
⸻
Production Example (AWS EKS)

In one production EKS cluster, an EC2 instance became unreachable due to an underlying infrastructure issue.

Kubernetes marked the node as NotReady after it stopped receiving kubelet heartbeats.

I confirmed the issue by checking the node conditions and events. The node was no longer communicating with the control plane, and after the eviction timeout, Kubernetes rescheduled the affected pods onto healthy worker nodes.

Since our deployments had multiple replicas across three Availability Zones, users experienced little to no downtime. The Cluster Autoscaler later launched a replacement worker node to restore cluster capacity.
⸻
What Happens Internally?
A senior engineer should know this sequence:

1. Worker node stops sending heartbeats.
2. Control plane detects missed heartbeats.
3. Node status changes to NotReady.
4. Kubernetes stops scheduling new pods on that node.
5. Existing pods remain temporarily if the node is still partially functional.
6. If the node doesn’t recover within the eviction timeout, pods are evicted.
7. ReplicaSets/Deployments create replacement pods on healthy nodes.
8. Cluster Autoscaler may provision a new node if required.
⸻
Common Reasons a Node Becomes NotReady

* Kubelet stopped or crashed
* Network partition between node and control plane
* EC2 or VM failure
* High CPU utilization
* Memory exhaustion
* Disk pressure
* Disk full
* Container runtime failure (Docker/containerd)
* Kernel panic
* Node reboot
* Cloud infrastructure issue
* Certificate expiration
* Security group or firewall changes

15) Your service returns 502/503 after deployment — root cause possibilities?
HTTP 502 and 503 errors after a deployment usually indicate that the Load Balancer or Ingress cannot successfully communicate with the backend application. My first step is to determine whether the issue is at the application layer, Kubernetes layer, or infrastructure layer.

I follow a structured troubleshooting process.

First, I verify whether the new pods are healthy.

* Are all pods in the Running state?
* Are the readiness and liveness probes passing?
* Are there any CrashLoopBackOff or restart events?

Next, I check the Kubernetes Service and Endpoints.

* Has the Service discovered the new pods?
* Are the pod labels correctly matching the Service selector?
* Is the targetPort correctly configured?

Then, I investigate the Ingress or Load Balancer.

* Review Ingress Controller or ALB/NLB logs.
* Verify routing rules and backend health.
* Confirm health checks are passing.

If the infrastructure looks healthy, I move to the application layer.

* Review application startup logs.
* Check whether the application is listening on the expected port.
* Verify environment variables and ConfigMaps.
* Ensure secrets are mounted correctly.

Finally, I investigate downstream dependencies.

* Database connectivity.
* Redis or cache availability.
* External APIs.
* DNS resolution.
* Certificate or authentication issues.

If everything appears healthy, I compare metrics, logs, traces, and deployment history to identify what changed during the release.

In production, 502/503 errors are most commonly caused by readiness probe failures, missing service endpoints, port mismatches, application startup delays, or unhealthy downstream dependencies.
⸻
Production Example (AWS EKS)

During one deployment on Amazon EKS, users immediately started receiving 503 Service Unavailable errors.

I first checked the Ingress Controller and confirmed that requests were reaching Kubernetes. Then I verified the Service endpoints and found that no pods had been registered as healthy backends.

Further investigation showed that the readiness probe was configured with a very short initial delay. Since the application required about 45 seconds to initialize, Kubernetes marked the pods as unready, so the Service had no healthy endpoints.

We updated the readiness probe by increasing the initial delay and failure threshold. Once the pods became Ready, Kubernetes added them to the Service endpoints, and the 503 errors were resolved.
⸻
Common Root Causes (Senior-Level Knowledge)

Mentioning these demonstrates practical production experience:

Kubernetes Issues

* Readiness probe failures
* Liveness probe failures
* CrashLoopBackOff
* Incorrect Service selector labels
* No Service endpoints
* Wrong targetPort
* Wrong containerPort
* Failed rolling update
* Pod scheduling failures
* Insufficient CPU or memory

Networking Issues

* Ingress misconfiguration
* Load Balancer health check failures
* DNS resolution issues
* Network Policies blocking traffic
* Service mesh routing issues (e.g., Istio)

Application Issues

* Application startup failure
* Port binding mismatch
* Configuration errors
* Missing environment variables
* Secret or ConfigMap issues

Dependency Issues

* Database unavailable
* Redis unavailable
* External API timeouts
* Certificate expiration
* Authentication failures

16) How do you troubleshoot a P1 production incident step-by-step?
Sample answer
“My approach is: stabilize first, then deep-dive. In a P1, the first goal is reducing user impact. I start by confirming scope: full outage or partial, one service or multiple, one region or all, and whether it started after a release or infra event.
Then I create a focused troubleshooting path:

validate user impact
identify affected service and blast radius
check recent changes (deployment, config, infra)
review metrics, logs, traces, alerts
attempt mitigation — rollback, scale-up, failover, restart, traffic shift
communicate status clearly
document root cause and preventive action after stabilization

In interviews, I say: ‘During a P1, I avoid random debugging. I prioritize restoring service fast, preserving evidence, and communicating clearly while different engineers validate app, infra, and dependency layers.’”

17) High latency but pods are healthy — what could be wrong?
Sample answer
“Healthy pods only tell me the app is running and passing probes; they don’t guarantee performance. High latency with healthy pods can come from slow DB queries, connection pool saturation, downstream service slowness, network congestion, DNS issues, retries, GC pauses, or load imbalance.
My next step is to break latency by layer:

app processing time
database query time
downstream API latency
network path
ingress/load balancer timing

A real-world style answer: ‘If pods are healthy but latency increases, I check request trace path, dependency response times, and whether a new release changed query behavior or increased synchronous calls.’”

18) Rollback didn’t fix the issue — what next?
If a rollback doesn’t resolve the issue, I don’t assume the application is still the problem. A rollback only restores the previous application version—it doesn’t revert changes to the runtime environment, infrastructure, or external dependencies.

My next step is to investigate everything that changed around the deployment.

I typically check:

* Configuration changes – Were ConfigMaps, environment variables, or application properties modified?
* Secrets and credentials – Have any API keys, certificates, or tokens expired or changed?
* Database changes – Were schema migrations executed? Are they backward compatible?
* Infrastructure changes – Any Kubernetes, network, load balancer, DNS, or Terraform changes?
* Dependencies – Are databases, caches, message queues, or external APIs healthy?
* Traffic patterns – Has there been a sudden traffic spike, autoscaling event, or DDoS-like behavior?
* Caching or queues – Is there stale cache, queue backlog, or delayed message processing?

I also compare system metrics, logs, traces, deployment history, and infrastructure events around the time the issue started. The objective is to identify what changed besides the application code.

In my experience, rollback is often the first mitigation to reduce user impact, but it’s not always the final solution because many production issues are caused by configuration, infrastructure, or dependency changes rather than the application itself.
⸻
Production Example (AWS + Kubernetes)

During one deployment, users started receiving HTTP 500 errors. We immediately rolled back the application, but the errors continued.

Since the rollback didn’t help, we investigated other recent changes. We found that a database schema migration had already been applied during the deployment. The previous application version expected the old schema, making the rollback incompatible with the updated database.

We restored compatibility by updating the schema and ensuring future migrations were backward compatible. We also changed our deployment strategy so database migrations were validated before application rollout.
⸻
If the Interviewer Asks:

“What would you check first after rollback fails?”

You can answer:

1. Application configuration
2. Secrets and certificates
3. Database health and migrations
4. Infrastructure changes
5. Load Balancer and Ingress
6. Kubernetes pod health
7. Cloud resources (CPU, Memory, Disk, Network)
8. External dependencies
9. Recent deployments or infrastructure changes
10. Monitoring dashboards and traces
⸻
Common Reasons Rollback Doesn’t Work

These are worth mentioning because they reflect real production experience:

* Database migration already executed
* Configuration or ConfigMap changes
* Expired certificates or secrets
* Infrastructure drift
* DNS changes
* Load Balancer misconfiguration
* Kubernetes networking issues
* Cache corruption
* Queue backlog
* External API outage
* Cloud resource exhaustion
* Incorrect feature flag settings


19) How do you debug intermittent failures (not reproducible)?
Sample answer

Intermittent failures are usually the most challenging because they cannot be reproduced consistently. In such cases, I focus on collecting evidence rather than trying to reproduce the issue immediately.

My first step is to identify patterns around the failure:

* Does it happen only during peak traffic?
* Is it affecting a specific node, pod, Availability Zone, or region?
* Did it start after a deployment, autoscaling event, or infrastructure change?
* Is a downstream dependency such as the database or an external API involved?

Next, I correlate multiple observability signals instead of relying only on logs:

* Metrics – CPU, memory, latency, error rates, network usage.
* Logs – Application, Kubernetes, and infrastructure logs.
* Distributed traces – To identify where requests are slowing down or failing.
* Events – Kubernetes events, autoscaling activities, deployments, node restarts, and cloud infrastructure events.

I compare successful requests with failed requests to identify differences. Many intermittent issues are caused by timing, scaling, network instability, resource contention, or dependency failures rather than application code.

If needed, I temporarily increase logging, enable debug logs for the affected component, or add additional monitoring to capture more details during the next occurrence.

My goal is not just to resolve the issue but to identify the root cause and implement preventive measures to avoid recurrence.
⸻
Production Example (AWS + Kubernetes)

In one production environment, users occasionally received HTTP 502 errors, but only a few times a day. The issue couldn’t be reproduced in lower environments.

I analyzed the failure timestamps and compared them with Kubernetes events. I found that the failures consistently occurred during pod scale-up events triggered by the Horizontal Pod Autoscaler.

The new pods were receiving traffic before they were fully initialized because the readiness probe wasn’t configured correctly.

As a result, the Load Balancer routed requests to pods that weren’t ready, causing intermittent failures.

We corrected the readiness probe configuration, adjusted the startup probe, and configured the Load Balancer to send traffic only to healthy pods. After these changes, the intermittent failures were eliminated.
⸻
Common Causes of Intermittent Failures

Mentioning these shows production experience:

* Autoscaling events
* Pod restarts
* Incorrect readiness/liveness probes
* Network latency or packet loss
* DNS resolution issues
* Database connection pool exhaustion
* External API timeouts
* Load Balancer health check failures
* Certificate or token expiration
* Memory leaks
* CPU throttling
* Race conditions
* Temporary cloud service degradation

20) Logs show no errors but users complain — how do you approach?

If users are reporting an issue but the application logs show no errors, I don’t assume the system is healthy. Logs are only one part of observability. My first priority is to validate the user impact and trace the entire request path.

I follow a structured troubleshooting approach:

First, I verify the symptoms.

* Is the issue affecting all users or only specific regions?
* Is it intermittent or continuous?
* Did it start after a deployment or infrastructure change?

Next, I check metrics because performance problems don’t always generate application errors.

* Request latency (P95/P99)
* Error rates (4xx/5xx)
* CPU and memory utilization
* Pod restarts
* Network throughput
* Database latency

Then I use distributed tracing to follow a request across services. This helps identify where the request is slowing down or failing, even if every service logs “success.”

I also validate each layer of the request path:

* DNS resolution
* Load Balancer or Ingress
* Kubernetes Services
* Pods and container health
* Database
* External APIs
* Network connectivity

Sometimes the application logs are clean because:

* The log level is too restrictive.
* The request never reaches the application due to a load balancer or ingress issue.
* A timeout occurs upstream.
* A dependency is slow but returns a response instead of an error.
* The issue is latency rather than an exception.

Finally, I correlate logs, metrics, traces, deployment history, and infrastructure changes to identify the root cause instead of relying on logs alone.
⸻

Production Example (AWS + Kubernetes)

In one production incident, users reported that the application was taking nearly 20 seconds to load, but the application logs showed no errors.

I first checked our dashboards and noticed that application error rates were normal, but the P99 latency had increased significantly.

Then I checked the Kubernetes Ingress and found that one backend service was timing out because the database response time had increased due to high CPU utilization on the RDS instance.

Since the database was still responding, the application never logged an exception—it was simply waiting longer for the query to complete.

We temporarily increased database capacity, optimized the slow queries, and latency returned to normal.

This incident reinforced that metrics and traces often reveal problems that logs alone cannot.
⸻

If the Interviewer Asks:

“What tools would you use?”

You can answer:

* Logs: Splunk / ELK / CloudWatch Logs
* Metrics: Prometheus, Grafana, CloudWatch
* Tracing: OpenTelemetry, Jaeger, Zipkin
* Kubernetes: kubectl logs, kubectl describe, kubectl top, kubectl get events
* AWS: CloudWatch Metrics, ALB Target Health, VPC Flow Logs, RDS Performance Insights
* Networking: curl, ping, traceroute, nslookup, dig


21) What is Terraform state and why remote state is used?
Terraform state is a file that keeps track of the infrastructure Terraform manages. It acts as a mapping between the Terraform configuration and the actual resources running in the cloud.

Terraform uses the state file to understand what resources already exist, compare the current infrastructure with the desired configuration, and determine what needs to be created, modified, or deleted during a terraform plan or terraform apply.

In a small personal project, the state can be stored locally. However, in an enterprise environment, we use a remote backend because multiple engineers and CI/CD pipelines work on the same infrastructure.

A remote state provides:

* Centralized storage for the state file.
* State locking to prevent multiple users from applying changes at the same time.
* Versioning and backup in case the state needs to be restored.
* Better security through encryption and controlled access.
* Consistency across team members and automation pipelines.

In AWS, a common setup is storing the Terraform state in an S3 bucket with versioning enabled and using a DynamoDB table for state locking. This prevents concurrent modifications and reduces the risk of state corruption.

Overall, I consider the Terraform state file a critical asset because losing or corrupting it can make Terraform lose track of the infrastructure it manages.

⸻

Real Production Example

In one of my projects, our Terraform state was stored in an S3 bucket with versioning enabled, and we used a DynamoDB table for state locking.

Whenever a deployment pipeline ran, Terraform first acquired the lock. If another engineer or pipeline had already started an apply operation, the second execution waited or failed with a lock error instead of corrupting the state.

This approach ensured that only one infrastructure change happened at a time and protected the environment from conflicting updates.

⸻

Follow-up Questions the Interviewer May Ask

1. What happens if you lose the Terraform state file?

Answer:

Terraform loses the mapping between the configuration and the existing infrastructure. It may try to recreate resources that already exist or fail to manage them correctly. That’s why we always store state remotely with versioning and backups.

⸻

2. Why not store the state in Git?

Answer:

The state file can contain sensitive information such as resource IDs, IP addresses, and sometimes secrets. It also changes frequently, which leads to merge conflicts. Git doesn’t provide state locking, so it’s not suitable for Terraform state management.

⸻

3. What is state locking?

Answer:

State locking prevents multiple users or pipelines from modifying the same Terraform state simultaneously. Without locking, concurrent terraform apply operations could corrupt the state or create inconsistent infrastructure.

⸻

4. Can multiple teams use the same state file?

Answer:

It’s technically possible, but not recommended. Each environment or application should have its own state file. Splitting state into smaller, logical units improves isolation, reduces risk, and speeds up Terraform operations.

23) How do you detect and fix infrastructure drift?

Infrastructure drift occurs when the actual infrastructure differs from what’s defined in Infrastructure as Code (Terraform). This usually happens because of manual console changes, emergency fixes, failed deployments, or changes made outside the CI/CD pipeline.

To detect drift, I primarily use terraform plan, which compares the Terraform state with the actual infrastructure. In production, I also schedule regular plan executions through CI/CD and review AWS CloudTrail logs to identify manual console changes. Some organizations also use AWS Config to detect configuration drift.

Before fixing drift, I first determine whether the change was intentional or accidental.

* If the change was approved, I update the Terraform code and commit it to Git so Terraform becomes the source of truth.
* If it was an unauthorized or accidental change, I use Terraform to restore the infrastructure to the desired state.

I never run terraform apply blindly in production. I carefully review the execution plan because drift correction can sometimes recreate or destroy critical resources, leading to downtime.

Finally, I try to prevent drift by restricting manual console access, enforcing all infrastructure changes through CI/CD pipelines, enabling code reviews, and maintaining Terraform as the single source of truth.
⸻

If the interviewer asks, “Have you handled this in production?”

You can answer:

Yes. We had a case where someone manually changed the EC2 Security Group in the AWS console to temporarily allow external access. During our scheduled Terraform plan, we detected that the Security Group rules no longer matched the Terraform configuration.

We first confirmed with the application owner that the change was temporary. Since it was no longer required, we removed the manual rule through Terraform instead of editing it directly in the console. This restored the infrastructure to the desired state while keeping Terraform as the source of truth.

⸻

Interview Tip

A strong answer always follows this structure:

1. What is infrastructure drift?
2. How do you detect it?
    * terraform plan
    * AWS Config
    * CloudTrail
    * CI/CD drift checks
3. How do you fix it?
    * Understand the reason for the drift.
    * Update Terraform if the change is valid.
    * Revert the infrastructure if it’s unauthorized.
4. How do you prevent it?
    * No manual console changes
    * CI/CD-only deployments
    * Code reviews
    * Least-privilege IAM
    * Terraform as the single source of truth
“

====================================================================================================================================================================================================================================================

Question 1

Scenario

A Kubernetes deployment appears healthy, and all pods are in the Running state. However, users receive HTTP 503 Service Unavailable errors when accessing the application.

Issue

Users cannot access the application even though Kubernetes reports that all pods are running.

⸻

Root Cause Analysis (RCA)

Possible causes include:

Pods are Running but not Ready.

Readiness probe failures.

Service selector labels don’t match pod labels.

Service has no endpoints.

Ingress controller configuration issue.

Load Balancer health check failure.

DNS resolution issue.

NetworkPolicy blocking traffic.

Backend dependency (Database/Redis/API) unavailable.

Recent deployment introduced a configuration error.

Resolution

I troubleshoot the request path layer by layer:

Client

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

Database / External APIs

Identify the failing layer, restore service, validate application health, and monitor the environment before closing the incident.

⸻

Prevention

Proper Readiness and Liveness probes

Deployment smoke tests

Canary or Blue-Green deployments

Continuous endpoint monitoring

Prometheus & Grafana alerts

Synthetic monitoring

Rollback automation

⸻

How I Would Answer in the Interview

“If users are receiving HTTP 503 errors while all Kubernetes pods are running, I wouldn’t assume the application is healthy because a Running pod doesn’t necessarily mean it’s Ready to serve traffic. My first priority is understanding the business impact by determining whether the issue affects all users, a specific region, or only certain APIs.

Then I’d troubleshoot the request path systematically instead of jumping directly into the pods. I start from the AWS Load Balancer to verify that the target group reports healthy targets. Next, I’d check the Ingress Controller to ensure routing rules are correct and there are no configuration errors. After that, I’d verify the Kubernetes Service selectors and confirm that Endpoints have been created correctly.

If the Service has no endpoints, I’d compare the Service selectors with the Pod labels. If endpoints exist, I’d inspect the Readiness Probe because a pod can be Running but still not Ready to receive traffic.

Once Kubernetes networking is validated, I’d investigate the application itself by reviewing container logs, application logs, resource utilization, and any dependency failures such as database connectivity, Redis availability, or third-party APIs. If a deployment was recently completed, I’d compare configuration changes and perform a rollback if necessary.

After identifying the root cause, I’d restore service, verify application functionality through health checks and user validation, continue monitoring for stability, document the RCA, and implement preventive improvements such as deployment validation, better monitoring, and automated rollback strategies.”

kubectl get pods -o wide

kubectl describe pod <pod-name>

kubectl logs <pod-name>

kubectl get svc

kubectl get endpoints

kubectl describe ingress

kubectl get ingress

kubectl describe service <service>

kubectl get events --sort-by=.metadata.creationTimestamp

kubectl exec -it <pod-name> -- sh

curl http://service-name

nslookup service-name

Common Follow-up Questions

Q1. Why can a pod be Running but still return 503?

Because Running only indicates the container process is active. If the Readiness Probe fails, Kubernetes removes the pod from the Service endpoints, so traffic is not sent to it.

⸻

Q2. What is the difference between Liveness Probe and Readiness Probe?

Liveness Probe determines whether the container should be restarted.

Readiness Probe determines whether the pod is ready to receive traffic.

⸻

Q3. How do you identify whether the problem is in the Ingress or the Service?

I test each layer independently:

Verify Ingress routes.

Check Service endpoints.

Test connectivity directly to the Service using kubectl port-forward or an internal curl request.

If the Service works but the Ingress doesn’t, the issue is likely with the Ingress or Load Balancer.

⸻


Q4. How would you prevent this issue in production?

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

terraform state show <resource>

terraform plan

terraform state pull > backup.tfstate

terraform import <resource> <resource-id>

terraform state rm <resource>

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

kubectl describe node <node-name>

kubectl get pods -o wide

kubectl drain <node-name> --ignore-daemonsets

kubectl cordon <node-name>

kubectl uncordon <node-name>

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
 

24) Difference between count vs for_each
Sample answer
“Both are used to create multiple resources, but they behave differently. count is index-based and is useful when resources are nearly identical. for_each is key-based and better when resources are defined by unique names or maps.
In production IaC, I prefer for_each when identity matters because it is more stable. With count, if the list order changes, resource addressing may shift and cause unintended replacement. With for_each, the key remains stable, so lifecycle is more predictable.
So in interview, I’d say: ‘I use count for simple repeated resources and for_each when I want better control, readability, and safer changes for uniquely named resources.’”

25) How do you achieve zero-downtime deployments?
Sample answer
“Zero-downtime deployment means users should continue to get service while the new version is rolled out. In Kubernetes, that typically means having multiple replicas, proper readiness probes, rolling update strategy, and enough capacity so the old pods are only terminated after the new pods are actually ready.
Depending on business criticality, I may also use blue-green or canary strategies. The core idea is:

never drain all healthy capacity at once
validate readiness before traffic shift
keep rollback fast
monitor during rollout

My interview answer would be: ‘I design zero-downtime using readiness-driven rolling deployments, controlled traffic switching, and post-deploy validation. For critical releases, I prefer canary or blue-green depending on risk and infrastructure design.’”

25) Difference between Blue-Green, Canary, Rolling deployment
Sample answer
“Rolling deployment gradually replaces old instances with new ones. It is simple and efficient, but if the issue is subtle, some users may still hit the bad version during rollout.
Blue-Green deployment keeps two full environments: current and new. Traffic switches only when the new environment is validated. It gives safer rollback but requires more infrastructure.
Canary deployment releases the new version to a small percentage of users first. It is best when I want controlled exposure and metric-based validation before full rollout.
In interview, I’d answer: ‘Rolling is efficient for normal releases, blue-green is strong for safer cutover, and canary is best when I want gradual risk-controlled validation in production.’”
