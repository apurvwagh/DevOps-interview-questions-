1. How do you rename Terraform resources without recreating them?

1) Terraform identifies resources using their resource address in the Terraform state. If I simply rename a resource in the Terraform code, Terraform may interpret it as the old resource being deleted and a new resource being created.

2) To rename it without recreating the actual infrastructure, I need to tell Terraform that the old state address should now correspond to the new address.

3) In modern Terraform, I prefer using a moved block because the change is declarative and becomes part of the Terraform configuration. Alternatively, I can use terraform state mv to move the state address manually.

4) For example, if I change aws_instance.web to aws_instance.application, I add a moved block mapping the old address to the new address. Terraform then updates the state address without destroying and recreating the EC2 instance.

5) Before doing this in production, I run terraform plan and verify that Terraform shows the resource as moved rather than destroyed and recreated.”

before

resource "aws_instance" "web" {
  # configuration
}

after

resource "aws_instance" "application" {
  # same configuration
}

moved {
  from = aws_instance.web
  to   = aws_instance.application
}

terraform plan
output 
# aws_instance.web has moved to aws_instance.application

====================================================================

2.Terraform created 10 resources, but terraform apply failed midway. What would you do?

1) First, I wouldn’t immediately run terraform destroy or manually recreate resources. Terraform is designed to handle partial failures.

2) If Terraform successfully created 10 resources and failed on the 11th, Terraform normally updates the state with the resources whose creation was successfully completed. I would first identify exactly which resource failed and why.

3) I would check the Terraform error message, AWS console/API events, CloudTrail where relevant, and Terraform logs if necessary. I would fix the underlying issue and then run terraform plan again.

4) Terraform compares the configuration with the state and actual infrastructure. It should recognize resources that were already successfully created and focus on whatever remains missing or needs correction.

5) If a resource was created in AWS but Terraform did not record it in state because of an unusual failure, I would verify the situation carefully and use terraform import if appropriate rather than creating a duplicate resource.

6) For production, I would also check state locking and make sure another engineer isn’t modifying the same state before rerunning the operation.”

Apply failed

    ↓
Read error

    ↓
Identify failed resource

    ↓
Check AWS/API logs

    ↓
Check Terraform state

    ↓
Fix root cause

    ↓
terraform plan

    ↓
terraform apply

====================================================================

3. What is Terraform Drift? How do you fix and prevent it?

1) Terraform drift occurs when the real infrastructure differs from what Terraform expects based on its configuration and state. A common example is when someone manually changes an AWS Security Group, EC2 instance configuration, or other Terraform-managed resource through the AWS console or CLI.

2) I detect drift by running terraform plan. Terraform refreshes the resource information and compares the actual infrastructure with the desired configuration.

3) If the manual change is not intended, I usually run terraform apply to bring the infrastructure back to the configuration defined in code. If the manual change is intentional, I update the Terraform code to reflect the desired configuration and then apply it.

4) To prevent drift, I follow Infrastructure-as-Code practices, restrict manual changes using IAM/RBAC, use CI/CD for Terraform changes, enable code reviews, and regularly run Terraform plans.

5) I also make sure engineers understand that the Terraform state is not the source of desired configuration—the Terraform code is. State is Terraform’s record of the resources it manages.”

Prevent drift

* Restrict console changes
* Use IAM least privilege
* Terraform through CI/CD
* Pull requests/code reviews
* Scheduled terraform plan
* Centralized state management
* Clear ownership of infrastructure

====================================================================

4. How does your team use Terraform in large environments?

1) In a large environment, I don’t allow every engineer to modify one huge Terraform configuration or state file. We structure Terraform into reusable modules and separate environments and state according to ownership and blast radius.

2) For example, we might have reusable modules for VPC, EKS, IAM, RDS, and ALB. Environment-specific configurations such as development, staging, and production consume those modules with different variables.

3) We use remote state, typically an S3 backend with state locking, and restrict access using IAM. Different teams or environments can have separate state files so that a change to one application doesn’t affect unrelated infrastructure.

4) Terraform changes go through Git-based pull requests. CI runs formatting, validation, security scanning, and terraform plan. After approval, the pipeline performs terraform apply, particularly for production.

5) We also use versioned modules, provider version constraints, tagging standards, naming conventions, and least-privilege IAM. The goal is repeatability, isolation, auditability, and minimizing the blast radius of changes.”

Terraform Repository
│
├── modules/

│   ├── vpc/

│   ├── eks/

│   ├── rds/

│   ├── iam/

│   └── alb/
│
└── environments/

    ├── dev/
    
    ├── staging/
    
    └── prod/


====================================================================

5. What is a Terraform Data Source (data block)?

1) A Terraform data source allows Terraform to read information about an existing resource without creating or managing that resource.

2) For example, if a VPC already exists, instead of creating another VPC, I can use a data source to retrieve its ID, CIDR, or other attributes and use those values in resources managed by Terraform.

3) Data sources are especially useful when infrastructure is shared between teams or managed by another Terraform stack. For example, a networking team may manage the VPC while the application team uses a data source to retrieve the VPC and subnet IDs.

The key difference is that a resource block manages infrastructure, while a data block reads existing information.”

====================================================================




















