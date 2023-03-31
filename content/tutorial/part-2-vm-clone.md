---
title: Part 2 - Cloning a VM with Terraform
---

In this part you will clone a VM with Terraform.
By the end of it you will have a VM you can SSH into.
Make sure to have environment variables [configured as described in part 1]({{< relref "part-1-preparation#configuring-environment-variables" >}}).

## Edit Variables

Open `terraform/terraform.tfvars`.

1. Set `proxmox_api_url` to your Proxmox VE API endpoint, with `/api2/json` suffix.
1. Set `proxmox_target_node` to the Proxmox node you want to create the machine on.
1. Set `cloud_init_public_keys` to your SSH public key. [See related how-to guide for more information]({{< ref "how-to/ssh-keys" >}})

Once done your file should look like this:

```hcl
# Inside terraform/terraform.tfvars
proxmox_api_url        = "https://192.168.0.105:8006/api2/json"
proxmox_target_node    = "bfte"
cloud_init_public_keys = <<EOF
ssh-ed25519 REDACTED
EOF
```

## Applying the Configuration

Change to the `terraform` folder:

```
cd terraform
```

Initialize Terraform:

```
terraform init
```

Apply the Terraform module to clone the `alpine-3.17.0-1` template to a new VM:

```
terraform apply
```

Ensure the plan creates a machine, and type `yes` to verify the Terraform plan and apply the resources.

When Terraform is done cloning the VM, it will print the `ssh_ip` and `ssh_user` you can use to connect to the VM:

```
Apply complete! Resources: 3 added, 0 changed, 0 destroyed.

Outputs:

ssh_ip = "192.168.0.173"
ssh_user = "robust-marmoset"
```

## Connecting to the VM

Using the `ssh_ip` and `ssh_user` from the previous step, we can connect to the VM:

```
ssh robust-marmoset@192.168.0.173 -p 2222
```

The template is configured to use port 2222 for SSH to leave port 22 open for Git servers and other applications.

## Next Steps

Now that we have a VM, we are ready to configure it.
[Head over to the next part, where we will be configuring the VM with Ansible.]({{< relref "part-3-vm-configuration" >}})
