Q1. Jenkins pipeline needs a database password to deploy an application to EKS. Where would you store it and how would Jenkins retrieve it securely?

“I would not store the database password in the Jenkinsfile or Git repository.

If it’s a Jenkins-specific credential, I can store it in Jenkins Credentials Manager and reference it by credential ID.

However, if the organization uses HashiCorp Vault as the centralized secrets platform, I would prefer storing the database secret in Vault. Jenkins would authenticate to Vault using an approved authentication method, receive a short-lived Vault token with a least-privilege policy, and retrieve the secret only during the deployment stage.

The secret would be injected at runtime and never hardcoded or printed in the Jenkins console.

For an EKS application, I would also consider whether the application itself should retrieve the secret from Vault at runtime rather than passing the database password through Jenkins.”
