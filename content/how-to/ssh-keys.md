---
title: How to create an SSH key
---

This guide shows you how to generate a key pair for SSH authentication with VMs.

## Requirements

This guide assumes you are working on Linux.
Use latest Ubuntu LTS for best compatibility.

## Creating an SSH Key Pair

Use `ssh-keygen` to generate a new key pair:

```
ssh-keygen -t ed25519 -f <filename>
```

The default filename is `~/.ssh/id_ed25519`.

It is recommended to protect the key with a strong password.

The public key is saved to `<filename>.pub`.
For example, to view the public key of the key with default filename:

```
cat ~/.ssh/id_ed25519.pub
```

The public key can be used [with the `machine` Terraform module]({{< ref "reference/machine-module" >}}) to authenticate with cloned VMs.
The `cloud_init_public_keys` variable accepts a string of multiple SSH public keys in the following form:

```
cloud_init_public_keys = EOF<<
ssh-ed25519 REDACTED lior@workstation
ssh-ed25519 REDACTED lior@laptop
EOF
```

Make sure to set your public key in `cloud_init_public_keys`.
Once you apply the Terraform module you should be able to connect to the cloned VM with SSH.
