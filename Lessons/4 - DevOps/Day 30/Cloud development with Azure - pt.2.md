# Cloud development with Azure pt.2
## Accounts
![[Cloud development with Azure - pt.1#Accounts]]

## Lesson
[Terraform](https://developer.hashicorp.com/terraform) is a powerful Infrastructure as Code (**IaaC**) tool developed by [HashiCorp](https://www.hashicorp.com) that enables declarative provisioning and management of Azure cloud resources through configuration files written in HashiCorp Configuration Language (HCL)

## What is Terraform for Azure
Terraform serves as an **automation layer** between developers and Azure infrastructure, allowing teams to *define*, *preview*, and *deploy* cloud resources using code rather than manual configuration through the Azure portal. This approach eliminates manual processes, ensures consistency across environments, and enables version-controlled infrastructure management.

## Core Workflow
The Terraform provisioning process follows a **four-step workflow** :
1. `{sh} terraform init` Initializes the working directory, downloads provider plugins, and configures backend state storage
2. `{sh} terraform plan` Creates an execution plan showing what changes will be made to infrastructure
3. `{sh} terraform apply` Executes the planned changes to provision or modify Azure resources
4. `{sh} terraform destroy` Removes all resources managed by the configuration when no longer needed

## Infrastructure Capabilities
Terraform can provision virtually any Azure service through **declarative configuration files**. Common resources include:
- **Resource groups** and **virtual networks**
- **Virtual machines** and **container instances**
- **Storage accounts** and **databases** (CosmosDB, SQL Database)
- **App Services** and **Function Apps**
- **Load balancers** and **application gateways**
- **Azure Kubernetes Service** clusters

## CI/CD Integration
Terraform integrates seamlessly with **Azure DevOps pipelines** to enable automated infrastructure deployment. Teams can create YAML-based pipelines that automatically run Terraform commands when code changes are pushed to repositories, ensuring infrastructure stays synchronized with application deployments.

## Key Benefits for Azure
Using TerraForm with Azure provides several advantages over manual provisioning:
- **Consistency** Infrastructure configurations are repeatable across development, staging, and production environments
- **Version Control** Infrastructure changes can be tracked, reviewed, and rolled back through Git workflows
- **Collaboration** Teams can share and review infrastructure code using standard development practices
- **Modularity** Reusable Terraform modules enable standardized infrastructure patterns across projects

https://github.com/boolean-uk/java-cloud-azure-day-4.git
![[Repository/Day 30/Theory/1 - Java Cloud Azure - 2/README]]