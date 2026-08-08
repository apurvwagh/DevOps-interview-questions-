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
























