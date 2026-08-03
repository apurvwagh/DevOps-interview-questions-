1. Explain your CI/CD flow in your current project.

1) In my current project, we follow a Git-based CI/CD and GitOps approach. Developers push code to Git, which triggers the CI pipeline. The pipeline performs code quality checks, security scanning, unit tests, builds the application, creates a Docker image, and pushes it to Amazon ECR.

2) For deployment, we use Kubernetes on Amazon EKS. After the image is available in ECR, the deployment configuration is updated through our GitOps repository. Argo CD detects the change and synchronizes the desired state with the EKS cluster.

3) Before production, we typically deploy to lower environments such as development and staging, perform automated tests and validation, and then promote the release to production based on the required approval process.

4) During deployment, Kubernetes performs a rolling update using readiness probes, liveness probes, replica configuration, and PodDisruptionBudgets to minimize downtime. After deployment, we monitor application and infrastructure metrics using Prometheus, Grafana, CloudWatch, and application logs. If there is an issue, we can roll back to the previous application version through GitOps or Kubernetes rollout mechanisms.

So overall, my flow is: Developer → Git → CI pipeline → Build/Test/Security Scan → Docker Image → ECR → GitOps Repository → Argo CD → EKS → Monitoring and Alerting.”
