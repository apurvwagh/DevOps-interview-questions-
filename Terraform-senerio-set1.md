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


What is a Terraform provider?
A provider is a plugin that allows Terraform to interact with APIs of external services, such as AWS, Azure, or Google Cloud.


What is the purpose of the terraform init command?
It initializes a Terraform working directory, downloading necessary provider plugins and setting up the backend.


What does the terraform plan command do?
It creates an execution plan, showing what actions Terraform will take to reach the desired state.


How does terraform apply differ from terraform plan?
terraform apply executes the actions proposed in the plan, making changes to real resources.


What is a Terraform state file?
It’s a file that keeps track of the infrastructure resources managed by Terraform, mapping real-world resources to your configuration.


Why is state management important in Terraform?
It ensures that Terraform has up-to-date information about resources, enabling accurate planning and application of changes.


What is the use of terraform destroy?
It deletes all resources defined in your Terraform configuration.
