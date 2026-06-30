1) What is Terraform?
Terraform is an open-source Infrastructure as Code (IaC) tool by HashiCorp that allows users to define and provision infrastructure using a declarative configuration language.

2) What is Infrastructure as Code (IaC)?
IaC is the practice of managing and provisioning computing infrastructure through machine-readable configuration files, rather than physical hardware configuration or interactive configuration tools.

3) What are the key features of Terraform?
Key features include:
Execution plans
Resource graphs
Change automation
Support for multiple providers

4)  What is a Terraform provider?
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

52. What is a Terraform provider?
    
Terraform providers are like plugins that let Terraform talk to cloud services like Amazon Web Services or Azure. At the top of your configuration file, you say who the provider is. When you run terraform init, it gets downloaded.

53. What is the Terraform state file?

The state file for Terraform is very important. Terraform.tfstate is the name of the file. It keeps track of your infrastructure. Terraform doesn't know what resources are already there without it. When you work with a team, you should always keep this file somewhere, like Amazon S3 or Terraform Cloud.

54. What are Terraform modules?

Terraform modules are like packages that you can use again and again. You can make a module for something like a private cloud and then use it in a lot of different projects. These modules can take in data. Give output, like functions do in programming.

55. What's the difference between variables.tf and terraform.tfvars?

The variables.tf file is where you say what your variables are called and what kind they are. You set the values for those variables in the terraform.tfvars file. This keeps your configuration file clean and separate from the values that might change.

56. What is terraform taint?

Terraform taint is used to mark a resource as tainted, which tells Terraform that the resource should be destroyed and recreated during the next terraform apply, even if no configuration changes are detected.
It is useful when a resource becomes unhealthy or is manually modified outside Terraform, and you want Terraform to rebuild it.
 cross Q
 What is the difference between taint and destroy?
Answer:

terraform taint / -replace → Recreates only the specified resource.
terraform destroy → Deletes all resources managed by the Terraform configuration (unless specifically

57. How do you manage remote states?

Using a backend block. Common options: AWS S3 + DynamoDB (for state locking), Azure Blob Storage, Terraform Cloud. State locking prevents concurrent application conflicts in team environments.

58. What is terraform import?

It brings existing, manually created infrastructure under Terraform management. Essential for migrating legacy infrastructure to IaC without rebuilding from scratch.

59. count vs for_each — what's the difference?

Count creates resources by number. for_each iterates over a map or string set, keying each instance uniquely. With count, deleting a middle element triggers recreation of all subsequent resources. for_each handles deletions cleanly by key.

60. What are data sources?

Read-only lookups for existing infrastructure. Use them to reference an existing VPC, AMI, or security group without Terraform managing it.

61. How does Terraform handle resource dependencies?

Automatically, via a dependency graph built from resource references. Use depends on for explicit dependencies that aren't expressed through references.

62. Local vs remote backends?

Local stores state on your machine — fine for solo use. Remote backends (S3, Azure Blob, Terraform Cloud) enable team collaboration, state history, and locking. Always use remote in production.

63. What are Terraform workspaces?

They let you manage multiple environments (dev, staging, prod) from one configuration, each with an isolated state. For complex setups, separate directories per environment are often more maintainable.

63. How do you handle secrets in Terraform?

Mark variables as sensitive = true
Pass secrets via environment variables (TF_VAR_db_password)
Use AWS Secrets Manager, Azure Key Vault, or HashiCorp Vault
Always encrypt your remote state backend — state files can store values in plain text

64. What is the lifecycle block?

Controls resource lifecycle behaviour:

create_before_destroy — Zero-downtime replacements
prevent_destroy — Guards critical resources from accidental deletion
ignore_changes — Skips drift on specified attributes

64. How do you refactor a large Terraform codebase?

Extract repeated patterns into modules. Use moved blocks (Terraform 1.1+) to relocate resources in state without recreating them. Separate state by infrastructure layer — networking, compute, data — to reduce blast radius and speed up plans.

65. How do you integrate Terraform into CI/CD?

Standard pipeline: terraform fmt + terraform validate on PRs → plan on review → apply on merge to main. Tools like Atlantis or Terraform Cloud's VCS integration automate this workflow cleanly

66. What is Terraform drift?

When real infrastructure diverges from the recorded state, usually from manual console changes. The Terraform plan detects it. Terraform refresh updates the state to reflect reality. Long-term fix: enforce IaC-only changes through team policy.

67. How do you test Terraform code?

terraform validate — Syntax checks
TFLint — Provider-specific linting
Checkov / tfsec — Security static analysis
Terratest — Integration tests in Go
terraform test — Native testing framework (added in v1.6)






















Review provider documentation and resource dependencies.
Use terraform refresh to update state if drift is suspected.
