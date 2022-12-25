---
title: argo role
---

This role configures Argo CD.

This role configures Prometheus Operator ServiceMonitor resources to scrape metrics from Argo CD.
Make sure to **install the `prometheus` role first**.

## Variables

* `argo_host` - Host for Argo CD UI. **Required**.
* `argo_chart_version` - Argo CD chart version to deploy. Default is `5.16.2`.
* `argo_cert_manager_enabled` - Create Cert Manager Certificate resources for UI Ingress. Default is `true`.
* `argo_cert_manager_issuer_kind` - Kind of Cert Manager Issuer to use for UI Ingress certificate. Default is `ClusterIssuer`.
* `argo_cert_manager_issuer_name` - Issuer name to use for UI Ingress certificate. Default is `letsencrypt`.

The password for the `admin` account is automatically generated and stored in `argocd-initial-admin-secret` Secret in the `argo-cd` Namespace.
See instructions below for decoding it to access the UI.

## Example Playbook

Given a production group in the Ansible inventory, this playbook installs a single node K3s cluster and configures Argo CD and monitoring on each host:

```yaml
---
- name: Configure Kubernetes cluster
  hosts: production
  roles:
    - role: lkummer.homelab.k3s
      vars:
        k3s_version: v1.24.3+k3s1
    - role: lkummer.homelab.prometheus
      vars:
        prometheus_grafana_host: grafana.example.com
        prometheus_grafana_user: admin
        prometheus_grafana_password: REDACTED
    - role: lkummer.homelab.argo
      vars:
        argo_host: argo.example.com
        # Remove if you also installed Cert Manager.
        argo_cert_manager_enabled: false
```

Set the `KUBECONFIG` environment variable to point to the K3s config in the `secrets` directory.
[See instructions in k3s role example]({{< relref "k3s-role.md#example-playbook" >}}).

The password for the `admin` account is automatically generated and stored in `argocd-initial-admin-secret` Secret in the `argo-cd` Namespace.
Use the following command to decode it:

```
kubectl get --namespace argo-cd secrets/argocd-initial-admin-secret --output 'jsonpath={.data.password}' | base64 -d
```
