---
title: How to build the Proxmox template
---

This guide shows you how to build a Proxmox template from the [Packer Alpine cloud image repository](https://github.com/LKummer/packer-alpine).

You should build the cloud image manually only when bootstrapping.
If you have access to a GitLab instance, use GitLab CI instead.

## Requirements

This guide assumes you are working on Linux. Use latest Ubuntu LTS for best compatibility.

* Proxmox VE node.
* Packer `v1.8.7` or similar installed.
* [Packer Alpine cloud image repository](https://github.com/LKummer/packer-alpine) which we will be working inside.
* Proxmox API Token, [see Proxmox API Tokens how-to guide for instructions]({{< relref "proxmox-api-tokens" >}}).

Inside the Packer Alpine repository,
set the `PROXMOX_URL`, `PROXMOX_USERNAME` and `PROXMOX_TOKEN` environment variables.
It is recommended to use a `.env` file to manage credentials during development:

```bash
# Inside .env
export PROXMOX_URL='https://192.168.0.100:8006/api2/json'
export PROXMOX_USERNAME='user@pve!token'
export PROXMOX_TOKEN='xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx'
```

The `.env` file can be used with `source .env`.

## Building the Image

Open a shell inside the Packer Alpine repository.
Make sure to have environment variables set with credentials as described above.

We will build from a tag to get a stable build.
Find the latest tag in the Packer Alpine repository.
You may use this command:

```
git tag | tail -n 1
```

When writing this guide, the tag is `3.17.0-1`.
Replace it with the latest tag when reading.

```
git checkout 3.17.0-1
```

Install the required Packer plugins:

```
packer init alpine.pkr.hcl
```

Copy the `secrets.example.pkr.hcl` file to `secrets.pkr.hcl`.
Set `proxmox_node` to the Proxmox node you want to create the template on.
Set `ssh_password` to a random string, it does not matter as it is locked before cloning.

Build the Packer image with the following command.
Make sure to edit the `template_name_suffix` variable with the tag you are building.

```
packer build --var-file secrets.pkr.hcl --var template_name_suffix=-3.17.0-1-manual alpine.pkr.hcl
```

For this guide I have also added `-manual` to the template name to differentiate the image being built from images built by the CI pipeline.

You should now have a Proxmox template called `alpine-3.17.0-1-manual`.
It is ready to be cloned [with the `machine` Terraform module]({{< ref "reference/machine-module" >}}).

## Troubleshooting

If your build did not succeed, try following these steps:

* Set `PACKER_LOG=1` for debug logs.
* Make sure your `PROXMOX_URL` has the `/api2/json` path.
* Make sure your `PROXMOX_USERNAME` is of the form `user@realm!token`.
* Make sure your `PROXMOX_TOKEN` has the privileges [specified in the Proxmox API Tokens how-to guide]({{< relref "proxmox-api-tokens#creating-role-for-automation" >}}).
* Check out the `build-packer` job in `.gitlab-ci.yml` and see if you are doing anything differently.
