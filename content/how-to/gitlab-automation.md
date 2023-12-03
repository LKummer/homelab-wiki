---
title: How to provision infrastructure with GitLab CI
---

This tutorial walks you through cloning
[the example repository](https://github.com/LKummer/homelab-tutorial-infrastructure)
and configuring credentials for it's CI pipeline.

If you are looking to set up automation for an existing repository,
[check out the pipeline configuration as an example.](https://github.com/LKummer/homelab-tutorial-infrastructure/blob/main/.gitlab-ci.yml)

## Requirements

- GitLab CI runner with access to Proxmox API
- Proxmox token [(see how-to for creation instructions)]({{< relref "proxmox-api-tokens#creating-an-api-token" >}})
- [Infrastructure repository cloned on your machine](https://github.com/LKummer/homelab-tutorial-infrastructure)

## Preparation

Create a new GitLab project.
Do not push any commits to it yet.
**Make sure it is private**
CI artifacts may contain secrets.

If you want to provision infrastructure locally, [prepare your local environment]({{< relref "gitlab-managed-state#preparing-local-environment" >}}).

## Creating SSH key for provisioning

Create an SSH key for use in CI jobs:

```
ssh-keygen -t ed25519 -f ci_key -C tutorial -P ''
```

## Populating GitLab CI variables

In the GitLab project you created in the previous step, go to "Settings", "CI/CD".
Expand the "Variables" section.

Create the following environment variables:

- `PM_API_TOKEN_ID` ID of the form `user@pve!token`
- `PM_API_TOKEN_SECRET` secret of the ID stored in `PM_API_TOKEN_ID`
- `SSH_PRIVATE_KEY_FILE` (file) content of `ci_key`, make sure it ends with an empty line
- `SSH_PUBLIC_KEY` content of `ci_key.pub`
- `ANSIBLE_VAULT_PASSWORD_FILE` (file, protected) Ansible Vault password

Leave `ANSIBLE_VAULT_PASSWORD_FILE` unset if you do not need Ansible Vault.

Delete the keys we created earlier:

```
rm ci_key ci_key.pub
```

## Deployment

Now that we have configured the project we can push the example repository:

```
git remote set-url origin git@gitlab.example.com:devops/example.git
git push
```

Once the `build-terraform` job is done you can view the Terraform plan, and
manually trigger a deployment.
