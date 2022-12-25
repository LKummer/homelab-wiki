---
title: k3s role
---

This Ansible role configures a single node K3s cluster on an Alpine Linux host.

It is designed to configure VMs cloned using [the `machine` Terraform module]({{< relref "machine-module" >}}).

After running, the kubeconfig is fetched to `secrets/k3s.<ip>.yaml` and can be used to authenticate with the cluster.

## Variables

* `k3s_version` - K3s version to use. Default is `v1.25.4+k3s1`.

It is recommended to specify `k3s_version` to avoid collection updates from upgrading your Kubernetes version.

It is recommended to use `k3s_version` [from the stable release channel](https://update.k3s.io/v1-release/channels).

## Example Playbook

Given a `production` group in the Ansible inventory, this playbook installs a single node K3s cluster on each host:

```yaml
---
- name: Configure Kubernetes cluster
  hosts: production
  roles:
    - role: lkummer.homelab.k3s
      vars:
        k3s_version: v1.24.3+k3s1
```

To access the cluster, set the `KUBECONFIG` environment variable to the `k3s.<ip>.yaml` file created in the `secrets` directory.
The following command sets `KUBECONFIG` to the latest config in the `secrets` directory.

```
export KUBECONFIG="$(ls -lt secrets/k3s* | head -n 1 | sed 's/.*\(secrets.*$\)/\1/')"
```

You can also put it in a `.env` file to automatically set your `KUBECONFIG`.
