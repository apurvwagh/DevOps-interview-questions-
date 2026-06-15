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
Sample answer
“Readiness probe tells Kubernetes whether the container is ready to receive traffic. Liveness probe tells Kubernetes whether the container is still alive and should keep running. The key difference is traffic routing versus restart behavior.
If readiness fails, the pod is removed from service endpoints but the container is not necessarily restarted. If liveness fails, Kubernetes restarts the container. So misconfiguring these probes can directly cause downtime or unnecessary restarts.
A practical example: if a liveness probe is too aggressive during startup, the app keeps restarting before it fully initializes. If readiness probe points to the wrong endpoint, the app may run but receive no traffic. So probe tuning is operationally critical.”

13) How do you debug high CPU / memory / OOMKilled issues in Kubernetes?
Sample answer
“I first confirm whether the issue is transient or sustained. Then I inspect pod resource usage, container limits/requests, and the exact termination reason. For OOMKilled, I confirm whether memory limits are too low or the application is leaking memory.
My debugging path is:

check current and historical resource metrics
verify limits/requests
inspect logs near crash time
correlate with traffic spikes/new release/background jobs
compare with previous stable release

For high CPU, I check whether it’s expected from traffic growth or caused by inefficient processing, retry storms, or hot loops. For memory, I look for unbounded caches, leaks, or oversized payload handling.
A strong example is: ‘After deployment, CPU spiked across pods. I correlated that with a traffic increase and a code path change that caused excessive retries to a dependency. After rollback and fix, CPU stabilized.’”

14) What happens when a node becomes NotReady?
Sample answer
“When a node becomes NotReady, Kubernetes marks it as unhealthy because the control plane is no longer receiving expected heartbeats or status. Existing workloads may continue briefly depending on what exactly failed, but scheduling of new pods to that node stops, and affected pods may later be rescheduled elsewhere if the cluster can recover capacity.
Operationally, I check:

node conditions and events
kubelet status
network connectivity between node and control plane
disk pressure / memory pressure / container runtime health
cloud instance health if managed cluster

The impact depends on whether workloads have replicas on other nodes. In production, the goal is to protect availability by ensuring replicas, autoscaling, and proper pod distribution are already designed into the platform.”

15) Your service returns 502/503 after deployment — root cause possibilities?
Sample answer
“502/503 usually indicate a gateway, upstream, or availability issue rather than just a code exception. After deployment, common causes include readiness failures, no healthy backend endpoints, ingress misconfiguration, target port mismatch, upstream app crash, timeout to dependency, or sudden overload.
So I check:

ingress/controller logs
service endpoints
pod readiness
app startup errors
resource pressure
dependency failures

A practical answer is: ‘If 502/503 appears right after release, I first verify whether the new pods are healthy and registered as service endpoints. If not, it is often readiness or port configuration. If endpoints are healthy, I investigate upstream dependency timeouts or ingress routing issues.’”

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
Sample answer
“If rollback didn’t fix the issue, that tells me the problem may not be fully in the application version. I investigate config changes, database changes, feature flags, expired credentials, infrastructure drift, or downstream dependency issues. Sometimes rollback restores code but not the runtime condition.
So I ask:

Was there a config/secret change?
Was there a DB migration?
Did traffic pattern change?
Is dependency/downstream unhealthy?
Is there stale cache or queue backlog?

This answer shows maturity because not every production issue is solved by reverting code. Sometimes rollback is only the first mitigation, not the final fix.”

19) How do you debug intermittent failures (not reproducible)?
Sample answer
“For intermittent failures, I rely less on local reproduction and more on correlation. I look for patterns in time, load, instance, region, dependency, deployment history, autoscaling events, and network conditions.
I compare successful vs failed requests and check whether failures align with:

scale events
specific nodes/pods
one AZ/region
dependency timeouts
certificate/token expiry windows
traffic bursts

My answer in interview would be: ‘For intermittent issues, I collect evidence around failure windows and compare distributed signals—metrics, logs, traces, events—because these issues often come from timing, scaling, or dependency instability rather than deterministic code bugs.’”

20) Logs show no errors but users complain — how do you approach?
Sample answer
“I never conclude ‘no issue’ just because logs are clean. Logs are only one signal. If users are impacted, I trust user experience metrics and request-path validation first. I check error rate, latency, frontend or edge symptoms, network path, ingress behavior, downstream dependencies, and regional differences.
Sometimes logs are quiet because:

log level is too low
timeout happens upstream
request never reaches the service
the issue is performance, not exception
dependency failure is handled poorly but not logged

A strong answer is: ‘If logs show no errors, I shift to metrics, traces, request flow validation, and dependency health to find silent failures or performance degradation.’”

21) What is Terraform state and why remote state is used?
Sample answer
“Terraform state is the mapping between Terraform configuration and the real infrastructure it manages. It allows Terraform to know what already exists, what changed, and what needs to be created, updated, or destroyed.
Remote state is used for team collaboration, security, and consistency. In enterprise setups, local state is risky because it causes drift, duplicate operations, and lack of visibility. Remote state with locking prevents concurrent changes and makes CI/CD-driven infrastructure changes safer.
In practice, remote backend also helps with auditability and operational control, especially when multiple engineers or pipelines manage the same environments.”

22) How do you detect and fix infrastructure drift?
Sample answer
“Infrastructure drift happens when actual infrastructure no longer matches the desired IaC configuration — usually due to manual changes, partial failures, or unmanaged updates. I detect it by running Terraform plan regularly, reviewing cloud changes, and using operational controls to reduce out-of-band modifications.
To fix drift, I first understand whether the manual change was intentional or accidental. Then I either bring the config in line with reality or restore the infrastructure back to the desired state. I avoid blind applies on critical infrastructure because drift correction can trigger destructive changes.
A mature answer is: ‘I treat drift as both a technical and process issue. I detect it with IaC checks and reduce it by enforcing infrastructure changes through pipelines rather than manual console changes.’”

23) Difference between count vs for_each
Sample answer
“Both are used to create multiple resources, but they behave differently. count is index-based and is useful when resources are nearly identical. for_each is key-based and better when resources are defined by unique names or maps.
In production IaC, I prefer for_each when identity matters because it is more stable. With count, if the list order changes, resource addressing may shift and cause unintended replacement. With for_each, the key remains stable, so lifecycle is more predictable.
So in interview, I’d say: ‘I use count for simple repeated resources and for_each when I want better control, readability, and safer changes for uniquely named resources.’”

24) How do you achieve zero-downtime deployments?
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
