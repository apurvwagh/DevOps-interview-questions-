Q1. Jenkins pipeline needs a database password to deploy an application to EKS. Where would you store it and how would Jenkins retrieve it securely?

“I would not store the database password in the Jenkinsfile or Git repository.

If it’s a Jenkins-specific credential, I can store it in Jenkins Credentials Manager and reference it by credential ID.

However, if the organization uses HashiCorp Vault as the centralized secrets platform, I would prefer storing the database secret in Vault. Jenkins would authenticate to Vault using an approved authentication method, receive a short-lived Vault token with a least-privilege policy, and retrieve the secret only during the deployment stage.

The secret would be injected at runtime and never hardcoded or printed in the Jenkins console.

For an EKS application, I would also consider whether the application itself should retrieve the secret from Vault at runtime rather than passing the database password through Jenkins.”

Interviewer: “Why shouldn’t Jenkins pass the DB password to Kubernetes?”

“It can, but I would avoid unnecessarily exposing secrets through CI/CD. For workloads running on EKS, a stronger design is for the application to retrieve secrets at runtime using a workload identity and Vault integration, depending on the organization’s architecture.”

===================================================================

Q2. What is the difference between Jenkins Credentials Manager and HashiCorp Vault?

“Jenkins Credentials Manager is primarily designed to securely store and provide credentials that Jenkins jobs need, such as SSH keys, API tokens, username/password credentials and secret files.

HashiCorp Vault is a centralized enterprise secrets-management platform. It provides capabilities such as fine-grained policies, secret leasing, TTLs, dynamic credentials, rotation and audit logging.

So if I have a simple Jenkins-specific credential, Jenkins Credentials Manager may be sufficient.

If multiple systems such as Jenkins, Kubernetes applications and automation tools need centralized secret management, or if I need dynamic secrets and automated rotation, I would use Vault.

I also wouldn’t automatically put every credential into Vault. The choice depends on the organization’s security architecture and requirements.”

==================================================================

Q3. Jenkins needs to deploy infrastructure to AWS. How would you authenticate Jenkins without storing AWS access keys?

This is very important for your interviews, especially given your previous interview feedback around IAM/OIDC.

“I would avoid storing long-lived AWS access keys and secret keys in Jenkins.

I would use an identity-based authentication mechanism, preferably OIDC where supported, to establish trust between Jenkins and AWS IAM.

Jenkins authenticates using its identity, AWS STS validates the trust relationship, and STS issues temporary credentials associated with an IAM role.

That role would have only the permissions required for the pipeline, following least privilege.

The temporary credentials expire automatically, which is much safer than storing permanent access keys.”

Why not store AWS keys in Jenkins Credentials Manager?”

Answer:

“It’s possible, but long-lived access keys increase the blast radius if compromised. I prefer short-lived credentials through IAM roles and OIDC wherever the Jenkins environment and AWS integration support it.”

==================================================================

Q4. Explain Jenkins → Vault authentication and authorization.

This is probably the most important Vault question.

“First, Jenkins needs to authenticate to Vault using an authentication method configured by the organization. Depending on where Jenkins runs, this could be AppRole, Kubernetes authentication or another supported method.

Vault validates Jenkins’s identity and issues a Vault token.

That token is associated with a Vault policy. The policy determines exactly which secrets Jenkins can access.

Jenkins then uses that token to retrieve the required secret at runtime.

I would use short-lived tokens, least-privilege policies and Vault audit logging. I would also avoid putting the authentication secret itself directly into the Jenkinsfile.”

If interviewer asks:

“Authentication and authorization are the same?”

Say:

“No. Authentication establishes who Jenkins is; authorization determines what that identity is allowed to access.”

==================================================================

Q5. A developer accidentally prints a secret in Jenkins logs. What do you do?

This is a real production security scenario.

“First, I would treat the secret as compromised rather than assuming Jenkins masking makes it safe.

I would immediately identify which credential was exposed, revoke or rotate it, and determine whether the log was accessible to other users or systems.

If the credential was a Vault secret, I would revoke or rotate it according to the secret type and check Vault audit logs for suspicious access.

If it was an AWS credential, I would revoke the affected credentials and investigate CloudTrail for unauthorized activity.

Then I would remove or restrict access to the affected Jenkins build logs according to our incident procedure.

For prevention, I would review the pipeline to ensure secrets aren’t passed to commands that print them, use Jenkins credential masking appropriately, avoid shell debugging such as set -x, enforce least privilege, and add security checks/review guidelines around secret handling.”

==================================================================


Q6. Declarative vs Scripted Jenkins Pipeline


“Jenkins supports two pipeline syntaxes: Declarative and Scripted.

Declarative Pipeline uses a predefined, structured syntax. It is easier to read, maintain and validate, and is generally preferred for standard CI/CD pipelines.

Scripted Pipeline is Groovy-based and gives much more programming flexibility. I would use it when the pipeline requires complex custom logic that is difficult to express using Declarative syntax.

In production, I generally prefer Declarative Pipeline for the main pipeline structure and use script {} blocks when I need limited custom Groovy logic.”

pipeline {
    agent any

    environment {
        APP_NAME = 'payment-api'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'docker build -t payment-api:${BUILD_NUMBER} .'
            }
        }

        stage('Test') {
            steps {
                sh 'pytest'
            }
        }

        stage('Deploy') {
            steps {
                sh './deploy.sh'
            }
        }
    }

    post {
        success {
            echo 'Deployment successful'
        }

        failure {
            echo 'Pipeline failed'
        }
    }
}

Which one do you prefer?”

Don’t say:

“Declarative always.”

Say:

“I prefer Declarative Pipeline for standard production CI/CD because it provides a consistent structure and is easier for teams to maintain. If I have complex custom logic that isn’t cleanly represented declaratively, I can use Scripted Pipeline features or script {} blocks within a Declarative Pipeline.”

==================================================================

Q7. How do you handle credentials securely in Jenkins?

This is a high-priority question for your Mastercard interview because it connects Jenkins, AWS, Vault, IAM/OIDC and security.

“I never hardcode credentials in the Jenkinsfile, Git repository, Dockerfile, or shell scripts.

For Jenkins-specific credentials, I use Jenkins Credentials Manager and reference the credential by its ID. Jenkins injects the credential only during the required stage, and I make sure it isn’t printed in console logs.

For enterprise secret management, especially when multiple applications or environments need secrets, I would integrate Jenkins with HashiCorp Vault or the organization’s approved secrets manager. Jenkins authenticates to Vault using an appropriate identity mechanism and retrieves only the required secret at runtime using a least-privilege policy.

For AWS access, I prefer IAM roles with OIDC and short-lived STS credentials instead of storing long-lived AWS access keys in Jenkins.

I also apply RBAC, least privilege, credential rotation, short TTLs where possible, audit logging, environment separation, and secret masking.

If a secret is accidentally exposed, I treat it as compromised, immediately revoke or rotate it, investigate access logs, and fix the pipeline so it cannot happen again.”






