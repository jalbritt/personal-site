+++
title = 'GitOps with ArgoCD'
date = 2024-03-20T12:20:11-05:00
draft = true
+++

ArgoCD is one tool which makes it possible to achieve GitOps--a delivery framework which uses Git as the single source of truth for what gets deployed. 

ArgoCD is an open-source continuous delivery platform which makes GitOps achievable for kubernetes clusters. 

ArgoCD will make sure that the state of our kubernetes apps match their desired configurations as defined in Git. When changes happen in GitHub, ArgoCD will automatically deploy them to the cluster. Taking this apporoach guarantees that any manual changes to the cluster are overwritten by the configuration defined in GitHub. 

First, we need a kubernetes cluster to install ArgoCD. For this example, I created one using Terraform. You can find snippets for creating the cluster here, and documentation on ArgoCD at https://argo-cd.readthedocs.io/.

After installing ArgoCD, the next step is to give ArgoCD access to your code repository where application configurations are defined. 

Argo syncs a single repository.
- Any change which happens outside of Git will be overwritten by ArgoCD.

Argo 'bootstraps' multiple apps as part of the 'app of apps' pattern.
- 
# with the app of apps pattern, we can define multiple applications in a single repository. It goes something like this. First, we define an application which is responsible for deploying all other applications. This application is called the 'app of apps'. The 'app of apps' application is responsible for deploying all other applications. The next step is to define the applications which the 'app of apps' application will deploy. Each application is defined in a separate directory. Each application directory contains a kustomization.yaml file which defines the resources which should be deployed. The workflow is as follows. When a change is made to the 'app of apps' application, ArgoCD will deploy all other applications. When a change is made to an application, ArgoCD will deploy that application.
# 
