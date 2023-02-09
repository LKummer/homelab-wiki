---
title: terraform_http plugin
---

This Ansible inventory plugin templates inventory from Terraform state stored in HTTP Terraform state backend.

It is used for populating Ansible inventory with VMs created with [the `machine` Terraform module]({{< relref "machine-module" >}}) when using HTTP Terraform state backend with GitLab-managed Terraform state.

There is a sister [`terraform_local` Ansible inventory plugin]({{< relref "terraform-local-plugin" >}}) doing the same thing for local Terraform state.

## Options

* `hosts` - List of hosts to add to the Ansible inventory. **Required**.

Each `hosts` entry contains the following options.
All values accept template strings, which are rendered with Terraform outputs as context.

* `ansible_host` - IP of the host. **Required**.
* `ansible_port` - Port to use for SSH connection. **Required**.
* `ansible_group` - Group to add the host to. **Required**.
* `vars` - Dictionary of additional host vars to add.

## Terraform Configuration

This inventory plugin works with Terraform HTTP state backend using GitLab-managed Terraform state.
Make sure to configure Terraform to use the HTTP state backend:

``` hcl
# Inside versions.tf
terraform {
  # ...
  backend "http" {}
}
```

[See GitLab's guide for instructions on configuring the Terraform HTTP state backend.](https://docs.gitlab.com/ee/user/infrastructure/iac/terraform_state.html#migrate-to-a-gitlab-managed-terraform-state)
It is recommended to use the `.env` example below for configuring your local machine to use GitLab-managed Terraform state.

## GitLab Authentication


This plugin uses `TF_HTTP_ADDRESS`, `TF_HTTP_USERNAME` and `TF_HTTP_PASSWORD` environment variables [in the same way Terraform does](https://developer.hashicorp.com/terraform/language/settings/backends/http#configuration-variables).

The GitLab token for `TF_HTTP_PASSWORD` requires full API access, just `read_repository` and `write_repository` are not enough.

If you configure your Terraform state backend through environment variables, this plugin works without any configuration.

### During Development

It is recommended to use a `.env` file to configure both Terraform and `terraform_http` Ansible inventory plugin.
For example, if our GitLab instance is `git.example.com` and the project ID is 57, we may use the following `.env` file:

```bash
# Inside .env
# Authentication details for GitLab managed Terraform state.
export GITLAB_USER='Lior'
export GITLAB_TOKEN='glpat-xxxxxxxxxxxxxxxxxxxx'

# Immitating CI environment variables.
export CI_PROJECT_ID='57'
export CI_API_V4_URL='https://git.example.com/api/v4'
export CI_ENVIRONMENT_NAME='dev'

# Terraform state configuration.
export TF_HTTP_ADDRESS="${CI_API_V4_URL}/projects/${CI_PROJECT_ID}/terraform/state/${CI_ENVIRONMENT_NAME}"
export TF_HTTP_LOCK_ADDRESS="${CI_API_V4_URL}/projects/${CI_PROJECT_ID}/terraform/state/${CI_ENVIRONMENT_NAME}/lock"
export TF_HTTP_LOCK_METHOD='POST'
export TF_HTTP_UNLOCK_ADDRESS="${CI_API_V4_URL}/projects/${CI_PROJECT_ID}/terraform/state/${CI_ENVIRONMENT_NAME}/lock"
export TF_HTTP_UNLOCK_METHOD='DELETE'
export TF_HTTP_USERNAME="${GITLAB_USER}"
export TF_HTTP_PASSWORD="${GITLAB_TOKEN}"
```

### In GitLab CI

The `TF_*` environment variables can be reused when configuring GitLab CI jobs provisioning VMs for a seamless experience:

```yaml
# Inside .gitlab-ci.yml
variables:
  TF_HTTP_ADDRESS: ${CI_API_V4_URL}/projects/${CI_PROJECT_ID}/terraform/state/${CI_ENVIRONMENT_NAME}
  TF_HTTP_LOCK_ADDRESS: ${CI_API_V4_URL}/projects/${CI_PROJECT_ID}/terraform/state/${CI_ENVIRONMENT_NAME}/lock
  TF_HTTP_LOCK_METHOD: POST
  TF_HTTP_UNLOCK_ADDRESS: ${CI_API_V4_URL}/projects/${CI_PROJECT_ID}/terraform/state/${CI_ENVIRONMENT_NAME}/lock
  TF_HTTP_UNLOCK_METHOD: DELETE
  TF_HTTP_USERNAME: gitlab-ci-token
  TF_HTTP_PASSWORD: ${CI_JOB_TOKEN}
```

With these environment variables set, you can both provision VMs with Terraform and configure them with Ansible.

## Example

Assuming you have environment variables configured as described above, and GitLab Terraform state contains the `machine_ssh_ip` and `machine_ssh_user` outputs.
This inventory config reads the GitLab Terraform state and adds a host to the `production` group:

```yaml
---
plugin: lkummer.homelab.terraform_http
hosts:
  - ansible_host: '{{ machine_ssh_ip }}'
    ansible_group: production
    ansible_port: '2222'
    vars:
      ansible_user: '{{ machine_ssh_user }}'
```

Note it is designed to integrate [with outputs from `machine` Terraform modules]({{< relref "machine-module#outputs" >}}).
