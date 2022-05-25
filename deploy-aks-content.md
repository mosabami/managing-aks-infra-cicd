In this article, we'll cover how to utilize CI/CD and Infrastructure as Code (Iac) to deploy the [Azure Kubernetes Service (AKS) baseline cluster](https://docs.microsoft.com/en-us/azure/architecture/reference-architectures/containers/aks/secure-baseline-aks).

## Overview

A reference architecture for a standard, well-architected AKS cluster is available at GitHub in either [Terraform](https://github.com/Azure/aks-baseline-automation/tree/main/IaC/terraform) or [Bicep](https://github.com/Azure/aks-baseline-automation/tree/main/IaC/bicep) templates. These templates should be cloned and modified to meet the particular needs of your organization. GitHub Actions will then be utilized to automate the process of deploying these template to provision AKS and the associated Azure resources in the cloud. 

![Architecture Overview](https://github.com/Azure/aks-baseline-automation/raw/main/docs/.attachments/IaC.jpg)


## Benefits of using IaC and Automation for Deployments

There are many ways to deploy to Azure including the Azure Portal, CLI, API, and more. For this project we are choosing to leverage IaC and automation through CI/CD. This results in many benefits including: 

- **Declarative**: By defining you infrastructure and deployment process in code, it can be versioned and reviewed using the standard software development lifecycle. It also helps prevent any drift in your configuration. 

- **Consistency**: Following an IaC process ensures that the whole organization follows a standard, well-established process to deploy infrastructure that incorporates best practices and is hardened to meet your security needs. Any imporvements made to the central templates can easily be leveraged across the organization.  

- **Security**: Centrally managed templates can be hardened and approved by a cloud operations or security team. 

- **Self-service**: Teams can be empowered to deploy their own infrastructure by utilizing centrally managed templates.

- **Improved Productivity**: By utilizing standard templates, teams can quickly provision new environments without needing to worry about all the implementation details.


Additional information can be found under [repeatable infrastructure](
https://docs.microsoft.com/en-us/azure/architecture/framework/devops/automation-infrastructure) in the Architecture Center.


## Pre-requisites to Deployment

### Baseline Azure Environment

Generally AKS clusters do not exist in a standalone fashion and are instead tied into a broader networking topology such as [hub-spoke](https://docs.microsoft.com/en-us/azure/architecture/reference-architectures/hybrid-networking/hub-spoke?tabs=cli). Before starting you may need a new azure subscription, resource group, and virtual network. 

### Managing Credentials

The GitHub Actions Workflow will need permissions to you Azure environment.  This can be accomplished in at least 2 different ways:

- **GitHub Secrets**: Secrets are encrypted environment variables that you create in an organization, repository, or repository environment (see [documentation](https://docs.github.com/en/actions/security-guides/encrypted-secrets)). You would need to generate and store an [Azure Service Principal](https://docs.microsoft.com/en-us/azure/active-directory/develop/app-objects-and-service-principals#service-principal-object).

- **OpenID Connect (OIDC)**: You can leverage OIDC to give a GitHub Actions Workflow a managed identity that has access to your Azure resources (see [documentation](https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/configuring-openid-connect-in-azure)). 

## Deploying with GitHub Actions

The [AKS Baseline Automation](https://github.com/Azure/aks-baseline-automation) GitHub repository provides detailed instructions and sample workflows to deploy the AKS baseline to Azure.

### Using Bicep

[Sample Workflow](https://github.com/Azure/aks-baseline-automation/blob/main/.github/workflows/IaC-bicep-AKS.yml)

### Using Terraform

[Sample Workflow](https://github.com/Azure/aks-baseline-automation/blob/main/.github/workflows/IaC-terraform-AKS.yml)

## How to Manage your AKS Cluster

Provide details about the workflow to make changes to the IaC files and show the process to merge and deploy those changes. Examples include:

- Upgrading AKS version
- Adding/modifying node pools

Also should discuss monitoring for configuration drift

## How to Manage Multiple AKS Clusters

Discuss patterns for creating a centrally managed template using tf modules or bicep equivalent and [reusable workflows](https://docs.github.com/en/actions/using-workflows/reusing-workflows). Each team would then create their own repo and workflow to manage their infrastructure.  

## Deploying Applications to AKS

[CI/CD pipelines for container workloads](https://docs.microsoft.com/en-us/azure/architecture/example-scenario/apps/devops-with-aks)


