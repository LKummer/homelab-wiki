---
title: How to use GitLab managed Terraform state
---

This guide walks you through configuring GitLab managed Terraform state.

## Requirements

You are going to need the following credentials:

- Proxmox token [(see how-to for instructions)]({{< relref "proxmox-api-tokens" >}})
- GitLab token with full API access

In addition, you will need a GitLab repository to host your Terraform state and infrastructure code.

## Preparing local environment

In this step we will configure our local environment to use GitLab managed Terraform state.

Since we need to define many environment variables, we will use a `.env` file.
Create a `.env` file with this template:

```bash
# Inside .env
# Credentials for GitLab managed Terraform state:
export GITLAB_USER=Example
# Full api access is required, just read_repository and write_repository are not enough.
export GITLAB_TOKEN='glpat-xxxxxxxxxxxxxxxxxxxx'

# Immitating CI environment variables.
export CI_PROJECT_ID='80'
export CI_API_V4_URL=https://gitlab.example.com/api/v4
export TF_STATE_NAME=dev

# Terraform state configuration.
export TF_HTTP_ADDRESS="${CI_API_V4_URL}/projects/${CI_PROJECT_ID}/terraform/state/${TF_STATE_NAME}"
export TF_HTTP_LOCK_ADDRESS="${CI_API_V4_URL}/projects/${CI_PROJECT_ID}/terraform/state/${TF_STATE_NAME}/lock"
export TF_HTTP_LOCK_METHOD=POST
export TF_HTTP_UNLOCK_ADDRESS="${CI_API_V4_URL}/projects/${CI_PROJECT_ID}/terraform/state/${TF_STATE_NAME}/lock"
export TF_HTTP_UNLOCK_METHOD=DELETE
export TF_HTTP_USERNAME="${GITLAB_USER}"
export TF_HTTP_PASSWORD="${GITLAB_TOKEN}"

export ANSIBLE_HOST_KEY_CHECKING=0
```

Edit the following environment variables:

- `GITLAB_USER` GitLab username
- `GITLAB_TOKEN` token with full API access for the user in `GITLAB_USER`
- `CI_PROJECT_ID` ID of the GitLab repository you are using
- `CI_API_V4_URL` url of GitLab REST API

Source the `.env` file:

```bash
source .env
```

## Ansible inventory configuration

Use `lkummer.homelab.terraform_http` inventory plugin:

```yml
# Inside playbook/inventory/example.yml
---
plugin: lkummer.homelab.terraform_http
hosts:
  - ansible_host: "{{ ssh_ip }}"
    ansible_group: example
    vars:
      ansible_user: "{{ ssh_user }}"
```

Hosts are templated with Terraform outputs as context.

## Terraform configuration

Configure Terraform to use `http` state backend in `main.tf` or `versions.tf`.

```hcl
# Inside terraform/versions.tf
terraform {
  backend "http" {}
}
```

## Migrating Terraform state

Initialize the Terraform module after modifying the configurations:

```
terraform init
```

Terraform will prompt you asking if you want to migrate state to the new backend.
Answer yes.

You should now be ready to apply Terraform and run Ansible with your newly
configured state backend.
