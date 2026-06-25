What is Terraform?
Terraform is an open-source Infrastructure as Code (IaC) tool by HashiCorp that allows users to define and provision infrastructure using a declarative configuration language.
What is Infrastructure as Code (IaC)?
IaC is the practice of managing and provisioning computing infrastructure through machine-readable configuration files, rather than physical hardware configuration or interactive configuration tools.
What are the key features of Terraform?
Key features include:
Execution plans
Resource graphs
Change automation
Support for multiple providers
4. What is a Terraform provider?
A provider is a plugin that allows Terraform to interact with APIs of external services, such as AWS, Azure, or Google Cloud.

5. What is the purpose of the terraform init command?
It initializes a Terraform working directory, downloading necessary provider plugins and setting up the backend.

6. What does the terraform plan command do?
It creates an execution plan, showing what actions Terraform will take to reach the desired state.

7. How does terraform apply differ from terraform plan?
terraform apply executes the actions proposed in the plan, making changes to real resources.

8. What is a Terraform state file?
It’s a file that keeps track of the infrastructure resources managed by Terraform, mapping real-world resources to your configuration.

9. Why is state management important in Terraform?
It ensures that Terraform has up-to-date information about resources, enabling accurate planning and application of changes.

10. What is the use of terraform destroy?
It deletes all resources defined in your Terraform configuration.

11. What are Terraform modules?
Modules are containers for multiple resources that are used together. They enable reusability and organization of code.

12. How do you define variables in Terraform?
Using the variable block, e.g.,

variable "instance_type" {
  default = "t2.micro"
}
13. What is the purpose of output variables?
They allow you to extract and display information from your Terraform configuration, often used to output resource attributes.

14. What is the terraform validate command used for?
It checks whether the configuration is syntactically valid and internally consistent.

15. What is the difference between terraform taint and terraform untaint?

terraform taint marks a resource for destruction and recreation on the next apply.
terraform untaint removes the taint from a resource.
16. What is a backend in Terraform?
A backend defines where Terraform’s state is stored, such as locally or remotely (e.g., in AWS S3, Azure Blob Storage).

17. How do you manage remote state in Terraform?
By configuring a backend in your Terraform configuration to store the state file in a remote location.

18. What are workspaces in Terraform?
Workspaces allow for multiple state files within a single configuration, enabling environments like dev, staging, and prod.

19. How do you handle sensitive data in Terraform?
By using environment variables, the sensitive attribute, or external secrets management tools like HashiCorp Vault.

20. What is the purpose of the terraform import command?
It brings existing infrastructure into Terraform management by importing resources into the state file.

21. Explain the use of count and for_each in Terraform.

count allows you to specify the number of resource instances.
for_each lets you iterate over a map or set of strings to create multiple resources.
22. What is a data source in Terraform?
Data sources allow Terraform to fetch data from external sources to be used in your configuration.

23. How do you manage dependencies between resources?
Terraform automatically manages dependencies by analyzing resource references; explicit dependencies can be set using depends_on.

24. What is the purpose of the terraform fmt command?
It formats Terraform configuration files to a canonical format and style.

25. How can you lock the Terraform state file?
By using backends that support state locking, such as AWS S3 with DynamoDB for locks.

26. What is the function of terraform refresh?
It updates the state file with the real infrastructure, detecting any changes made outside of Terraform.

27. How do you upgrade provider versions in Terraform?
By modifying the provider version in the configuration and running terraform init -upgrade.

28. What is the purpose of the .terraform.lock.hcl file?
It records the provider versions used in your configuration, ensuring consistent installs across machines.

29. How do you use conditional expressions in Terraform?
Using the ternary operator, e.g., condition ? true_val : false_val.

30. What is the difference between local-exec and remote-exec provisioners?

local-exec runs commands on the machine where Terraform is executed.
remote-exec runs commands on the resource being provisioned.
31. What is Terraform Cloud?
Terraform Cloud is a SaaS platform by HashiCorp for managing Terraform runs, state, and collaboration.

32. How does Terraform handle resource drift?
By using terraform plan or terraform refresh to detect changes in real infrastructure that differ from the state file.

33. What is Sentinel in Terraform?
Sentinel is a policy-as-code framework by HashiCorp for enforcing governance policies in Terraform configurations.

34. How do you manage multiple providers in a single Terraform configuration?
By defining multiple provider blocks with aliases and specifying the alias in resource definitions.

35. What are dynamic blocks in Terraform?
Dynamic blocks allow you to construct repeatable nested blocks within resources using expressions.

36. How do you handle module versioning in Terraform?
By specifying the version in the module source, e.g., version = ">= 1.0.0".

37. What is the purpose of the terraform graph command?
It generates a visual representation of the dependency graph of resources.

38. How can you prevent resource deletion in Terraform?
By setting the prevent_destroy lifecycle meta-argument to true.

39. What is the use of terraform workspace?
It manages multiple state files within a single configuration, useful for managing different environments.

40. How do you handle secrets in Terraform?
By using external secrets management systems like HashiCorp Vault or AWS Secrets Manager, and avoiding hardcoding secrets in code.

41. What is the purpose of the lifecycle block in a resource?
It allows you to customize how Terraform handles resource creation, update, and deletion.

42. How do you perform a rolling update in Terraform?
By using features like create_before_destroy and depends_on to control the order of resource updates.

43. What is the difference between null_resource and regular resources?
null_resource allows you to define provisioners without managing any real infrastructure.

44. How do you manage resource naming conflicts?
By using unique identifiers and naming conventions, and leveraging variables to parameterize names.

45. What are some best practices for Terraform module development?

Keep modules small and focused.
Use input and output variables effectively.
Document module usage.
Version control your modules.
46. How do you test Terraform configurations?
By using tools like terraform validate, terraform plan, and third-party tools like Terratest for automated testing.

47. What is the difference between terraform apply -auto-approve and regular terraform apply?
terraform apply -auto-approve skips the interactive approval step and applies the changes directly.
Regular terraform apply prompts the user to review and confirm the execution plan before applying changes.

48. How do you reuse code across multiple Terraform projects?
By creating and using Terraform modules. Modules can be stored locally or in remote repositories (e.g., GitHub or Terraform Registry) and can be reused across multiple configurations.

49. What is the terraform state command used for?
The terraform state command is used for advanced state file operations, such as listing resources, moving, removing, or modifying items in the state manually.

50. What is a resource target in Terraform and how is it used?
Resource targeting allows you to apply or destroy specific resources using the -target flag, e.g.,
terraform apply -target=aws_instance.example.
This is useful for partial applies but should be used with caution to avoid unintended side effects.

51. How do you troubleshoot Terraform provisioning errors?

Use terraform plan to check changes before applying.
Check logs and error messages during terraform apply.
Enable detailed logs with TF_LOG=DEBUG.
Review provider documentation and resource dependencies.
Use terraform refresh to update state if drift is suspected.
