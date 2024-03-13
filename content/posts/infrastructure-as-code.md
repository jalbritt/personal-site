+++
title = 'Managing Infrastructure as Code with Terraform'
date = 2024-03-13T16:26:09-05:00
draft = false
+++

Having infrastructure as code (IaC) means we can better manage software infrastructure components throughout their lifecycles. Terraform provides a way to responsibly manage infrastucture state at scale and has achieved mass adoption in industry as an open-source project. You can tell terraform what exactly you would like to create, and terraform gives you a plan showing the configuration changes that will happen upon approval. Here's an example of some terraform code that could be used to create a storage bucket in Google Cloud Platform.

```
provider "google" {
  credentials = "google_credentials.json"
  project     = "example-project"
}

resource "google_storage_bucket" "example" {
  name          = "example-storage-bucket-1234"
  location      = "US"
  force_destroy = true

  public_access_prevention = "enforced"
}
```

If we make a file `bucket.tf` with the above code and place it in the same directory as our keyfile, then we can run `terraform init` and then `terraform plan` to get the following output.

```
Terraform used the selected providers to generate the following execution plan.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # google_storage_bucket.example will be created
  + resource "google_storage_bucket" "example" {
      + effective_labels            = (known after apply)
      + force_destroy               = true
      + id                          = (known after apply)
      + location                    = "US"
      + name                        = "example-storage-bucket-1234"
      + project                     = (known after apply)
      + public_access_prevention    = "enforced"
      + rpo                         = (known after apply)
      + self_link                   = (known after apply)
      + storage_class               = "STANDARD"
      + terraform_labels            = (known after apply)
      + uniform_bucket_level_access = (known after apply)
      + url                         = (known after apply)
    }

Plan: 1 to add, 0 to change, 0 to destroy.
```

Running `terraform apply` at this point would apply the changes in the plan and create a single private storage bucket in Google Cloud. Running the `terraform destroy` command from the same directory will delete the bucket. 

In an organization having multiple developers and lots more infrastructure in terraform, things are organized roughly as follows. Terraform code will be stored in a git repository like github where developers can publish new code, re-use existing code, and review code changes. Any proposed change automatically runs the `terraform plan` command and attaches the output to the proposal--making it easy for reviewers to see what will be added, changed, or destroyed according to the plan.

Here is an example using GitHub Actions. The folder structure looks something like this:

```
.
├── .github/
│   └── workflows/
│       ├── tfplan.yaml
│       └── tfapply.yaml
└── terraform/
    ├── bucket.tf
    ├── variables.tf
    ├── backend.tf
    ├── provider.tf
    └── terraform.tfvars
```

Workflow #1: Generate the plan and post it as a comment to the pull request. Our credentials are now stored in GitHub environment variables for the repository.

```
name: Terraform Plan

on:
  pull_request:
    branches:
      - main

jobs:
  terraform:
    runs-on: ubuntu-latest
    env:
      GOOGLE_CREDENTIALS: ${{ secrets.GOOGLE_CREDENTIALS }}
    steps:
      - name: Checkout code // Get the latest changes
        uses: actions/checkout@v2

      - name: Install Terraform
        uses: hashicorp/setup-terraform@v1
        with:
          terraform_version: 1.0.0

      - name: Terraform Init
        run: |
          cd terraform          
          terraform init -reconfigure

      - name: Terraform Plan
        id: plan
        run: |
          cd terraform
          terraform plan -no-color -out=tfstate // Generate plan
          terraform show tfstate // Output plan

      - name: Post Plan Output to PR
        if: ${{ github.event_name == 'pull_request' }}
        uses: actions/github-script@v6
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          script: |
            const plan_output = `${{ steps.plan.outputs.stdout }}`
            .replace(/\u001b\[.*?m/g, ''); // Remove ANSI escape codes
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: '```' + plan_output + '```'
            });
```

Workflow #2: Regenerate a plan (in case other PRs have merged in the meantime) and execute.

```
name: Terraform Apply

on:
  pull_request:
    types:
      - closed // PR is closed
    branches:
      - main

jobs:
  terraform:
    runs-on: ubuntu-latest
    if: github.event.pull_request.merged == true // PR is merged
    env:
      GOOGLE_CREDENTIALS: ${{ secrets.GOOGLE_CREDENTIALS }}
    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Install Terraform
        uses: hashicorp/setup-terraform@v1
        with:
          terraform_version: 1.0.0

      - name: Terraform Init
        run: |
          cd terraform       
          terraform init

      - name: Terraform Plan
        id: plan
        run: |
          cd terraform
          terraform plan

      - name: Terraform Apply
        run: terraform apply -auto-approve
```

Running these plans without configuring proper permissions and integrations in GitHub will result in an error like "resource not available for integration." To set up the app, go to https://github.com/settings/apps and create a new app with the repository link as the Homepage URL and read/write permissions on issues, discussions, and pull requests.

