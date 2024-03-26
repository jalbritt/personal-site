+++
title = 'GitOps with ArgoCD'
date = 2024-03-20T12:20:11-05:00
draft = true
+++

With ArgoCD, we can make sure that the state of our kubernetes apps match their desired configurations as defined by code in GitHub. When changes happen in GitHub, ArgoCD will automatically deploy them to the cluster. Taking this apporoach guarantees that any manual changes to the cluster are overwritten by the configuration defined in GitHub. 

First, we need a kubernetes cluster to install ArgoCD. For this example, I created one using Terraform. You can find snippets for creating the cluster here, and documentation on ArgoCD at https://argo-cd.readthedocs.io/.

After installing ArgoCD, the next step is to give ArgoCD access to your code repository where application configurations are defined. 
