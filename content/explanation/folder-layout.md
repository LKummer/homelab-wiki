---
title: Recommended Folder Layout
---

This page describes the recommended folder layout for multi-environment infrastructure repositories.
We will be looking at a repository with `development`, `staging` and `production` environments:

![](../folder-structure.svg)

## Terraform Modules

Each subfolder of the `terraform` folder corresponds to an environment, and contains a Terraform module provisioning VMs for that environment.

The subfolders should follow [the Terraform module structure](https://developer.hashicorp.com/terraform/language/modules/develop/structure).

Separate Terraform modules are used for environments to separate the Terraform state of each environment, avoiding state locking conflicts.

### Managing Credentials

It is recommended to manage Proxmox credentials for Terraform in a `.env` file.
[See guide on configuring Proxmox credentials for more information]({{< ref "how-to/proxmox-api-tokens" >}}).

**Never commit `.env` files to version control.**
The point of using environment variables is that they are easy to keep out of version control.

## Ansible Playbook

The playbook layout is inspired by [Ansible's recommended layout](https://docs.ansible.com/ansible/2.8/user_guide/playbooks_best_practices.html#directory-layout).
Only differences are that `main.yml` contains the playbook itself, and an `inventory` folder is used for inventory configs.

### Inventory

Inventory configs use
[`terraform_local`]({{< ref "reference/terraform-local-plugin" >}})
or
[`terraform_http`]({{< ref "reference/terraform-http-plugin" >}})
inventory plugins to construct inventory for each environment based on Terraform state.

It is recommended to split hosts to groups based on environments.
Then we can use `group_vars` to configure the different environments.

### Managing Secrets

If possible, it is recommended to use a secret management service such as Hashicorp Vault.

Use [Ansible Vault](https://docs.ansible.com/ansible/latest/cli/ansible-vault.html) if it is impractical to use a secret management service.
Keep repositories using Ansible Vault private. If an old vault password leaks secrets may be extracted from Git history.

**Never commit plain text secrets to version control.**
