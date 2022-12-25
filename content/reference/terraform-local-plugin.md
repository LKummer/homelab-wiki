---
title: terraform_local plugin
---

This Ansible inventory plugin templates Ansible inventory from local Terraform state files.

It is used for populating Ansible inventory with VMs created with [the `machine` Terraform module]({{< relref "machine-module" >}}) when using local Terraform state.

There is a sister [`terraform_http` Ansible inventory plugin]({{< relref "terraform-http-plugin" >}}) doing the same thing for Terraform state stored in GitLab HTTP state backend.

## Options

* `tfstate_path` - Path to `tfstate` file, relative to Playbook root directory. **Required**.
* `hosts` - List of hosts to add to the Ansible inventory. **Required**.

Each `hosts` entry contains the following options.
All values accept template strings, which are rendered with Terraform outputs as context.

* `ansible_host` - IP of the host. **Required**.
* `ansible_port` - Port to use for SSH connection. **Required**.
* `ansible_group` - Group to add the host to. **Required**.
* `vars` - Dictionary of additional host vars to add.

## Example

Assuming you have a Terraform module located in `../terraform` relative to the Ansible Playbook root,
with `machine_ssh_ip`, and `machine_ssh_user` outputs.
This inventory config reads the Terraform state file and adds a host to the `production` group:

```yaml
# Inside inventory/production.yaml
---
plugin: lkummer.homelab.terraform_local
tfstate_path: ../terraform/terraform.tfstate
hosts:
  - ansible_host: '{{ machine_ssh_ip }}'
    ansible_group: production
    ansible_port: '2222'
    vars:
      ansible_user: '{{ machine_ssh_user }}'
```

Note it is designed to integrate [with outputs from `machine` Terraform modules]({{< relref "machine-module#outputs" >}}).
