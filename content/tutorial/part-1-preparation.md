---
title: Part 1 - Preparation
---

By the end of the tutorial you will have infrastructure code deploying a single node Kubernetes cluster with ArgoCD, Grafana, Prometheus Operator, Loki and Promtail.

You will be guided through the following stages:

1. Preparing the example repository and all required credentials.
1. Cloning a VM to be used as our Kubernetes node.
1. Configuring the VM to run K3s and deploy ArgoCD, Grafana, Prometheus, Loki and Promtail.

## Assumptions

This tutorial assumes you:

1. Have access to a Proxmox VE node. This tutorial requires 2 cores, 4GB of RAM, and 10GB of storage available on the Proxmox VE node.
1. Are working on a Linux system and have Python 3.10, Packer 1.9, and Terraform 1.6 or compatible versions installed. Use Debian Bookworm for best compatibility.
1. Have an SSH key to use for SSH authentication. [See how to create an SSH key guide]({{< ref "how-to/ssh-keys" >}}) if not.
1. Have a user and token for Proxmox VE. [See how to configure Proxmox credentials guide]({{< ref "how-to/proxmox-api-tokens" >}}) if not.
1. Have an Alpine template named `alpine-3.18.4-1` built [from Packer Alpine tag `3.18.4-1`](https://github.com/LKummer/packer-alpine/releases/tag/3.18.4-1) in your Proxmox VE node. [See how to build the Proxmox template]({{< ref "how-to/packer-template" >}}) if not.

You will have trouble completing this tutorial if any assumptions are broken.

## Cloning the Tutorial Repository

Clone [the tutorial infrastructure repository](https://github.com/LKummer/homelab-tutorial-infrastructure).

We will be using local Terraform state in this tutorial for ease of configuration.
[Configuring remote Terraform state is recommended.]({{< ref "how-to/gitlab-managed-state" >}})

Run the following commands to use local Terraform state:

```
sed 's/terraform_http/terraform_local/' -i playbook/inventory/tutorial.yaml
echo tfstate_path: ../terraform/terraform.tfstate >> playbook/inventory/tutorial.yaml
sed '/backend "http"/d' -i terraform/main.tf
```

## Configuring Environment Variables

Create a `.env` file like the following example.
Replace `PM_API_TOKEN_ID` and `PM_API_TOKEN_SECRET` values with your Proxmox username and token respectively.
Make sure your Proxmox username contains your username, realm and token ID.

```bash
# Inside .env
export PM_API_TOKEN_ID='user@pve!token'
export PM_API_TOKEN_SECRET='xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx'
export ANSIBLE_HOST_KEY_CHECKING=0
```

Source the `.env` file:

```bash
source .env
```

## Next Steps

We are now ready to start cloning VMs.
[Head over to the next part, where we will be using Terraform to clone VMs.]({{< relref "part-2-vm-clone" >}})
