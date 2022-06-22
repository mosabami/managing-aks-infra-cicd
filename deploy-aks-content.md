In this article, we'll cover how to utilize CI/CD and Infrastructure as Code (IaC) to deploy an Azure Kubernetes Service cluster in an automated and repeatable fashion.  

A reference architecture for a standard, well-architected AKS cluster is available at GitHub in either [Terraform](https://github.com/Azure/aks-baseline-automation/tree/main/IaC/terraform) or [Bicep](https://github.com/Azure/aks-baseline-automation/tree/main/IaC/bicep) formats. These templates should be cloned and modified to meet the particular needs of your organization. GitHub Actions will then be utilized to automate the process of deploying these template to provision AKS and the associated Azure resources in the cloud.ss 

![Architecture Overview](https://github.com/Azure/aks-baseline-automation/raw/main/docs/.attachments/IaC.jpg)


## Benefits of using IaC and Automation for Deployments

There are many ways to deploy to Azure including the Azure Portal, CLI, API, and more. For this project we are choosing to leverage IaC and automation through CI/CD. This results in many benefits including: 

- **Declarative**: By defining you infrastructure and deployment process in code, it can be versioned and reviewed using the standard software development lifecycle. It also helps prevent any drift in your configuration. 

- **Consistency**: Following an IaC process ensures that the whole organization follows a standard, well-established process to deploy infrastructure that incorporates best practices and is hardened to meet your security needs. Any improvements made to the central templates can easily be leveraged across the organization.  

- **Security**: Centrally managed templates can be hardened and approved by a cloud operations or security team. 

- **Self-service**: Teams can be empowered to deploy their own infrastructure by utilizing centrally managed templates.

- **Improved Productivity**: By utilizing standard templates, teams can quickly provision new environments without needing to worry about all the implementation details.


Additional information can be found under [repeatable infrastructure](
https://docs.microsoft.com/en-us/azure/architecture/framework/devops/automation-infrastructure) in the Architecture Center.


## Pre-requisites to Deployment

### Baseline Azure Environment

The reference templates are able to be deployed in a standalone fashion using the broader [Cloud Adoption Framework Terraform modules](https://github.com/aztfmod/terraform-azurerm-caf) (for Terraform) or [Common Azure Resource Modules Library](https://github.com/azure/resourcemodules) (for Bicep). If you are deployin as part of an existing [hub-spoke](https://docs.microsoft.com/en-us/azure/architecture/reference-architectures/hybrid-networking/hub-spoke?tabs=cli) topology the variables files will need to be update to reference your existing infrastructure. 

To setup your environment to run the scripts please see [IaC Prequisities](https://github.com/Azure/aks-baseline-automation/blob/main/docs/IaC-prerequisites.md).

### Managing Credentials

The GitHub Actions Workflow will need permissions to you Azure environment.  This can be accomplished in 2 different ways:

- **GitHub Secrets**: Secrets are encrypted environment variables that you create in an organization, repository, or repository environment (see [documentation](https://docs.github.com/en/actions/security-guides/encrypted-secrets)). You would need to generate and store an [Azure Service Principal](https://docs.microsoft.com/en-us/azure/active-directory/develop/app-objects-and-service-principals#service-principal-object).

- **OpenID Connect (OIDC)**: You can leverage OIDC to give a GitHub Actions Workflow a managed identity that has access to your Azure resources (see [documentation](https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/configuring-openid-connect-in-azure)). 

## Deploying with GitHub Actions

The [AKS Baseline Automation](https://github.com/Azure/aks-baseline-automation) GitHub repository provides detailed instructions and sample workflows to deploy the AKS baseline to Azure.

### Using Bicep

`TODO` update with new links once merged
[Sample Workflow](https://github.com/Azure/aks-baseline-automation/blob/main/.github/workflows/IaC-bicep-AKS.yml)

### Using Terraform

`TODO` update with new links once merged
[Sample Workflow](https://github.com/Azure/aks-baseline-automation/blob/main/.github/workflows/IaC-terraform-AKS.yml)

## Managing Your AKS Cluster

After the initial provisioning of your AKS cluster, there will arise situations where you need to make modifications to the clusters. Common examples include:
- Upgrading the Kubernetes version of the cluster
- Add or modify a node pool
- Change an auto scaling profile
Utilizing the IaC model, these changes to the environment should be performed by updating the Terraform or Bicep code rather than using the Azure Portal or CLI directly. This way the changes can be properly tracked and approved using the standard pull request process. 

### Architecture:
`TODO`: Insert Visio diagram here>
  
### Dataflow:
  
1.	Create a new branch and check in the needed IaC code modifications.
2.	Create a Pull Request (PR) in GitHub once you are ready to merge your changes into your environment.
3.	A GitHub Actions workflow will trigger to ensure your code is well formatted. In addition, a Terraform Plan or Bicep what-if analysis should run to generate a preview of the changes that will happen in your Azure environment. 
4.	Once appropriately reviewed, the PR can be merged into your main branch.
5.	Another GitHub Actions workflow will trigger from the main branch and execute the changes using your IaC provider. 
6.	A regularly scheduled GitHub Action workflow should also run to look for any configuration drift in your environment and create a new issue if changes are detected.


## Next Steps
- For deploying container workloads using CI/CD, see [CI/CD pipelines for container workloads](https://docs.microsoft.com/en-us/azure/architecture/example-scenario/apps/devops-with-aks)
- To learn about using GitOps to synchronize Kubernetes objects to a git repository, see [GitOps for Azure Kubernetes Service](https://docs.microsoft.com/en-us/azure/architecture/example-scenario/gitops-aks/gitops-blueprint-aks)


