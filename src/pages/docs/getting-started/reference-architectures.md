---
layout: src/layouts/Default.astro
pubDate: 2023-10-31
modDate: 2023-10-31
title: Reference architectures
description: Populate an Octopus space with example projects and supporting resources demonstrating reference architectures
navOrder: 55
---

Deployments are more than the sum of their parts, with well architected deployment processes empowering DevOps teams to release and maintain high quality software at a high velocity. The reference architecture steps provided by the [community step template library](/docs/projects/community-step-templates) allow DevOps teams to quickly populate an existing Octopus space with examples of well architected deployment projects, complete with all the supporting resources like environments, feeds, accounts, lifecycles etc.

## Common prerequisites

The reference architecture steps are typically run from a runbook. The runbook requires a small number of external resources to be defined:

1. A `Docker Container Registry` [feed](/docs/packaging-applications/package-repositories/guides/container-registries/docker-hub) called `Container Images` with the URL `https://index.docker.io`. This feed is used to access the [execution container for workers](/docs/projects/steps/execution-containers-for-workers) exposing a recent version of Terraform.
2. An [environment](/docs/infrastructure/environments) to execute runbooks in. This documentation assumes the environment is called `Admin`.
3. A [project](/docs/projects) to hold the runbooks. This documentation assumes the project is called `Reference Architecture`.

## EKS reference architecture

The [Octopus - EKS Reference Architecture](https://library.octopus.com/step-templates/87b2154a-5c8d-4c31-9680-575bb6df9789/actiontemplate-octopus-eks-reference-architecture) step populates an existing Octopus space with deployment projects demonstrating how DevOps teams can deploy applications to the AWS EKS platform.

### Supporting Videos

[Deploying to Kubernetes at scale with Octopus](https://www.youtube.com/watch?v=5q7s3vaGUN8)

### Configuring the step

Hosted Octopus users should use the `Hosted Ubuntu` worker pool and run the step with the `octopuslabs/terraform-workertools` container image accessed via the `Container Images` feed. On-premises Octopus users need to ensure the step is run on a worker with a recent version of Terraform installed, or can use the `octopuslabs/terraform-workertools` container image on a worker with Docker installed.

The step exposes a number of options, typically requesting credentials to the various platforms that are configured to support EKS deployments:

* `AWS Access Key` and `AWS Secret Key` require the [access keys](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_access-keys.html) of the user that will create the EKS cluster.
* `Docker Hub Username` and `Docker Hub Password` require the credentials of a [Docker Hub user](https://docs.docker.com/docker-id/) that is used to access sample Docker images from public DockerHub repositories. These credentials are also used by a sample GitHub Actions workflow that publishes Docker images.
* `GitHub Access Token` requires the [GitHub access token](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens) of a user that is used to create a new GitHub repository holding a sample application.
* `Octopus API Key` requires an [API key](https://octopus.com/docs/octopus-rest-api/how-to-create-an-api-key) to the Octopus instance where the reference architecture projects and supporting resources are created.
* `Octopus Space ID` requires the space ID where the reference architecture projects and supporting resources are created. Leave the default value to populate the same space as the runbook.
* `Octopus Server URL` requires the URL of the Octopus instance where the reference architecture projects and supporting resources are created. Leave the default value to populate the same instance as the runbook.

### Reference projects

The step creates a number of reference projects demonstrating how to deploy applications to an EKS cluster.

The `_ AWS EKS Infrastructure` project contains a runbook called `Create EKS Cluster`. This runbook creates a [Fargate](https://docs.aws.amazon.com/eks/latest/userguide/fargate.html) EKS cluster with the supplied name in the supplied region and then installs the NGINX ingress controller on it. The script then creates a new [Kubernetes target](/docs/infrastructure/deployment-targets/kubernetes-target) using [dynamic infrastructure](/docs/infrastructure/deployment-targets/dynamic-infrastructure). This cluster can be destroyed with the `Delete EKS Cluster` runbook.

The `EKS Octopub Audits`, `EKS Octopub Frontend`, `EKS Octopub Products` projects deploy the [Octopub](https://github.com/OctopusSolutionsEngineering/Octopub) sample application to the EKS cluster, performs a smoke test, and scans the [SBOM](https://www.cisa.gov/sbom) associated with each image using [Trivy](https://aquasecurity.github.io/trivy/). Each of these projects have a number of supporting runbooks to inspect Kubernetes resources. 

In addition, there are two runbooks called `Scale Pods to One` and `Scale Pods to Zero` that scale the number of pods associated with the deployment. These runbooks are expected to be triggered in the morning and afternoon to scale non-production environments up and down. Because the pods are run on Fargate nodes, scaling a deployment to zero removes the compute costs associated with them.

The `_ Deploy EKS Octopub Stack` project uses the [Deploy a release](/docs/projects/coordinating-multiple-projects/deploy-release-step) step to orchestrate the deployment of the individual microservices that make up the Octopub sample application. Orchestration projects provide a convenient way of promoting multiple related releases between environments in a predefined order, which may be required when applications are tightly bound or a well-defined set of release versions must be installed as a group. 

The `Docker Project Templates` project contains a runbook called `Create Template Github Node.js Project` that:

1. Creates a new GitHub repository
2. Adds [Github Actions secrets](https://docs.github.com/en/rest/actions/secrets) to allow [workflows](https://docs.github.com/en/actions/using-workflows/about-workflows) to interact with the Octopus server and the DockerHub repository
3. Populates the repo with a sample Node.js web application and GitHub Actions workflow to build the application, push it to DockerHub, and create a release in Octopus

This runbook is an example of platform engineering where DevOps teams can bootstrap sample applications with best practices such as versioning, security scanning, and CI/CD pipelines provided as part of a common base template.
