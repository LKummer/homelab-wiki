---
title: SSH Keys
---

SSH keys are used for SSH authentication to VMs.

Before cloning VMs with the `machine` Terraform module
([See reference documentation]({{< ref "reference/machine-module" >}}))
you need to create an SSH key to authenticate to the cloned VMs.

## Generate Key Pair

Use `ssh-keygen` to generate a new key pair:

```
ssh-keygen -t ed25519 -f <filename>
```

The default filename is `~/.ssh/id_ed25519`.

It is recommended to protect the key with a strong password.

## View Public Key

The public key is saved to `<filename>.pub`.

For example, to view the public key of the key with default filename:

```
cat ~/.ssh/id_ed25519.pub
```

## Use With Machine Module

The `machine` Terraform Module
([See reference documentation]({{< ref "reference/machine-module" >}}))
takes the `cloud_init_public_keys` variable.
This variable accepts a string of multiple SSH public keys and uses Cloud Init to make the cloned VM trust these keys.

Multiple public keys can be specified in multiple lines:

```
cloud_init_public_keys = EOF<<
ssh-ed25519 REDACTED lior@workstation
ssh-ed25519 REDACTED lior@laptop
EOF
```
