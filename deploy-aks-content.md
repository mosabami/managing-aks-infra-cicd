In this article, we'll cover how to utilize CI/CD and Infrastructure as Code (IaC) to deploy an Azure Kubernetes Service cluster in an automated and repeatable fashion.  

A reference architecture for a standard, well-architected AKS cluster is available at GitHub in either [Terraform](https://github.com/Azure/aks-baseline-automation/tree/main/IaC/terraform) or [Bicep](https://github.com/Azure/aks-baseline-automation/tree/main/IaC/bicep) formats. These templates should be cloned and modified to meet the particular needs of your organization. GitHub Actions will then be utilized to automate the process of deploying this template to provision AKS and the associated Azure resources in the cloud.

![Architecture Overview](https://github.com/Azure/aks-baseline-automation/raw/main/docs/.attachments/IaC.jpg)


## Benefits of using IaC and Automation for Deployments

There are many ways to deploy to Azure including the Azure Portal, CLI, API, and more. For this project we are choosing to leverage IaC and automation through CI/CD. This results in many benefits including: 

- **Declarative**: By defining your infrastructure and deployment process in code, it can be versioned and reviewed using the standard software development lifecycle. It also helps prevent any drift in your configuration. 

- **Consistency**: Following an IaC process ensures that the whole organization follows a standard, well-established process to deploy infrastructure that incorporates best practices and is hardened to meet your security needs. Any improvements made to the central templates can easily be leveraged across the organization.  

- **Security**: Centrally managed templates can be hardened and approved by a cloud operations or security team. 

- **Self-Service**: Teams can be empowered to deploy their own infrastructure by utilizing centrally managed templates.

- **Improved Productivity**: By utilizing standard templates, teams can quickly provision new environments without needing to worry about all the implementation details.


Additional information can be found under [repeatable infrastructure](
https://docs.microsoft.com/en-us/azure/architecture/framework/devops/automation-infrastructure) in the Architecture Center.


## Pre-requisites to Deployment

### Baseline Azure Environment

The reference template is able to be deployed in a standalone fashion using the broader [Cloud Adoption Framework Terraform modules](https://github.com/aztfmod/terraform-azurerm-caf) (for Terraform) or [Common Azure Resource Modules Library](https://github.com/azure/resourcemodules) (for Bicep). If you are deploying as part of an existing [hub-spoke](https://docs.microsoft.com/en-us/azure/architecture/reference-architectures/hybrid-networking/hub-spoke?tabs=cli) topology the variable files will need to be updated to reference your existing infrastructure. 

To setup your environment to run the scripts please see [IaC Prequisities](https://github.com/Azure/aks-baseline-automation/blob/main/docs/IaC-prerequisites.md).

### Managing Credentials

The GitHub Actions Workflow will need permissions to your Azure environment. This can be accomplished in 2 different ways:

- **GitHub Secrets**: Secrets are encrypted environment variables that you create in an organization, repository, or environment (see [documentation](https://docs.github.com/en/actions/security-guides/encrypted-secrets)). You would need to generate and store an [Azure Service Principal](https://docs.microsoft.com/en-us/azure/active-directory/develop/app-objects-and-service-principals#service-principal-object).

- **OpenID Connect (OIDC)**: You can leverage OIDC to give a GitHub Actions Workflow a managed identity that has access to your Azure resources (see [documentation](https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/configuring-openid-connect-in-azure)). 

## Deploying with GitHub Actions

The [AKS Baseline Automation](https://github.com/Azure/aks-baseline-automation) GitHub repository provides detailed instructions and sample workflows to deploy the AKS baseline to Azure.

### Using Bicep

`TODO`: These are links to my personal repo and should be updated if/when we move it the aks-baseline-automation repo

There are two main workflows included in the reference architecture:
1.  [**Bicep Validate**](https://github.com/tjcorr/bicep-pipeline-demo/blob/main/.github/workflows/bicep-validate.yml)

    This workflow is designed to be run on every commit and is composed of a set of unit tests on the infrastructure code. It runs [bicep build](https://docs.microsoft.com/en-us/cli/azure/bicep?view=azure-cli-latest#az-bicep-build) to compile the bicep to an ARM template. This ensure there are no formatting errors. Next it performs a [validate](https://docs.microsoft.com/en-us/cli/azure/deployment/sub?view=azure-cli-latest#az-deployment-sub-validate) to ensure the template is able to be deployed.

2.  [**Bicep Whatif / Deploy**](https://github.com/tjcorr/bicep-pipeline-demo/blob/main/.github/workflows/bicep-deploy.yml)

    This workflow runs on every pull request and on each commit to the main branch. The whatif stage of the workflow is used to understand the impact of the IaC changes on the Azure environment by running [whatif](https://docs.microsoft.com/en-us/cli/azure/deployment/sub?view=azure-cli-latest#az-deployment-sub-what-if). This report is then attached to the PR for easy review. The deploy stage runs after the whatif analysis when the workflow is triggered by a push to the main branch. This stage will [deploy](https://docs.microsoft.com/en-us/cli/azure/deployment/sub?view=azure-cli-latest#az-deployment-sub-create) the template to Azure after a manual review has signed off.

`TODO`: remove
[Sample Workflow](https://github.com/Azure/aks-baseline-automation/blob/main/.github/workflows/IaC-bicep-AKS.yml)

### Using Terraform
`TODO`: These are links to my personal repo and should be updated if/when we move it the aks-baseline-automation repo

There are three main workflows included in the reference architecture:
1.	[**Terraform Validate**](https://github.com/tjcorr/tf-pipeline-demo/blob/main/.github/workflows/tf-validate.yml)

    This workflow is designed to be run on every commit and is composed of a set of unit tests on the infrastructure code. It runs [terraform fmt]( https://www.terraform.io/cli/commands/fmt) to ensure the code is properly linted and follows terraform best practices. Next it performs [terraform validate](https://www.terraform.io/cli/commands/validate) to check that the code is syntactically correct and internally consistent.

2.	[**Terraform Plan / Apply**](https://github.com/tjcorr/tf-pipeline-demo/blob/main/.github/workflows/tf-plan-apply.yml)

    This workflow runs on every pull request and on each commit to the main branch. The plan stage of the workflow is used to understand the impact of the IaC changes on the Azure environment by running [terraform plan](https://www.terraform.io/cli/commands/plan). This report is then attached to the PR for easy review. The apply stage runs after the plan when the workflow is triggered by a push to the main branch. This stage will take the plan document and [apply](https://www.terraform.io/cli/commands/apply) the changes after a manual review has signed off if there are any pending changes to the environment.

3.	[**Terraform Drift Detection**](https://github.com/tjcorr/tf-pipeline-demo/blob/main/.github/workflows/tf-drift.yml)

    This workflow runs on a periodic basis to scan your environment for any configuration drift (i.e. changes made outside of terraform). If any drift is detected a GitHub Issue is raised to alert the maintainers of the project.

`TODO`: remove
Old link: [Sample Workflow](https://github.com/Azure/aks-baseline-automation/blob/main/.github/workflows/IaC-terraform-AKS.yml)

## Managing Your AKS Cluster

After the initial provisioning of your AKS cluster, there will arise situations where you need to make modifications to the clusters. Common examples include:
- Upgrading the Kubernetes version of the cluster
- Add or modify a node pool
- Change an auto scaling profile
Utilizing the IaC model, these changes to the environment should be performed by updating the Terraform or Bicep code rather than using the Azure Portal or CLI directly. This way the changes can be properly tracked and approved using the standard pull request process. 

### Architecture:
`TODO`: Rework diagram into Visio

<img width="1391" alt="image" src="https://user-images.githubusercontent.com/1248896/175121262-a7733f34-f4ad-42ed-983c-c41eccf289dc.png">
  
### Dataflow:
  
1.	Create a new branch and check in the needed IaC code modifications.
2.	Create a Pull Request (PR) in GitHub once you are ready to merge your changes into your environment.
3.	A GitHub Actions workflow will trigger to ensure your code is well formatted. In addition, a Terraform Plan or Bicep whatif analysis should run to generate a preview of the changes that will happen in your Azure environment. 
4.	Once appropriately reviewed, the PR can be merged into your main branch.
5.	Another GitHub Actions workflow will trigger from the main branch and execute the changes using your IaC provider. 
6.	A regularly scheduled GitHub Action workflow should also run to look for any configuration drift in your environment and create a new issue if changes are detected.


## Next Steps
- For deploying container workloads using CI/CD, see [CI/CD pipelines for container workloads](https://docs.microsoft.com/en-us/azure/architecture/example-scenario/apps/devops-with-aks)
- To learn about using GitOps to synchronize Kubernetes objects to a git repository, see [GitOps for Azure Kubernetes Service](https://docs.microsoft.com/en-us/azure/architecture/example-scenario/gitops-aks/gitops-blueprint-aks)


