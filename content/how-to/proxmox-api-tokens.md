---
title: Proxmox API Tokens
---

Proxmox API tokens are needed for authentication when using Packer and Terraform.
This guide tells you everything you need to create API tokens with the exact privileges for automation with Packer and Terraform.

## Creating Role For Automation

Before creating an API token and setting it's role we first need to create a role with exact permissions for Packer and Terraform.

This guide specifies two ways for creating a fitting Proxmox role.
It is your choice to create the role though shell or the Proxmox web interface.

### Through the Shell

Easiest way to create the role is with a shell command.
SSH into a Proxmox node or use the web interface to enter the following command:

```
pveum role add Provisioner -privs 'Datastore.AllocateSpace Datastore.AllocateTemplate Sys.Modify VM.Allocate VM.Audit VM.Clone VM.Config.CDROM VM.Config.CPU VM.Config.Cloudinit VM.Config.Disk VM.Config.HWType VM.Config.Memory VM.Config.Network VM.Config.Options VM.Console VM.Monitor VM.PowerMgmt'
```

You should now have a role with exact permissions for Packer and Terraform.

### Through the Web Interface

Login to the Proxmox web interface.

In "Server View",
select "Datacenter",
click "Permissions",
then select "Roles".
You should now see a list of roles.

Click "Create",
name it "Provisioner" and select the following privileges:

* `Datastore.AllocateSpace`.
* `Datastore.AllocateTemplate`.
* `Sys.Modify`.
* `VM.Allocate`.
* `VM.Audit`.
* `VM.Clone`.
* `VM.Config.CDROM`.
* `VM.Config.CPU`.
* `VM.Config.Cloudinit`.
* `VM.Config.Disk`.
* `VM.Config.HWType`.
* `VM.Config.Memory`.
* `VM.Config.Network`.
* `VM.Config.Options`.
* `VM.Console`.
* `VM.Monitor`.
* `VM.PowerMgmt`.

Click "Create".
You should now have a role with exact permissions for Packer and Terraform.

## Creating an API Token

Login to the Proxmox web interface and make sure to use "Server View".

First we must create an API token.
Inside "Server View", select "Datacenter", then select "Permissions", then click "API Tokens".
You should see the API tokens list.

Click "Add", select a user and token ID.
This is the place to set token expiration if needed.
Click "Add".

Now that we have an API token, we need to give it permissions as the Provisioner role described above.
Click "Permissions" (still under "Datacenter" in "Server View"),
then click "Add" and select "API Token Permission".
Select the path `/`, select the token you created in the previous steps,
and select the Provisioner role (described above).
Click "Add".

You should now have a Proxmox API token with privileges for Packer and Terraform.

## Managing Credentials When Working

Prefer to set credentials through environment variables.

It is recommended to use a `.env` files.
Simply store credentials inside and `.gitignore` it.
Note you need dotenv or direnv to load `.env` files.

The following `.env` example sets everything you need to work with Terraform and Packer:

```bash
export PROXMOX_URL='https://192.168.0.100:8006/api2/json'
export PROXMOX_USERNAME='user@pve!token'
export PROXMOX_TOKEN='xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx'
export PM_API_TOKEN_ID="$PROXMOX_USERNAME"
export PM_API_TOKEN_SECRET="$PROXMOX_TOKEN"
```
