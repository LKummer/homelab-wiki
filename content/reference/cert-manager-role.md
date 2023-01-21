---
title: cert_manager role
---

This Ansible role installs Cert Manager operator and configures a ClusterIssuer that issues Let's Encrypt certificates.

It is designed to configure VMs cloned using [the `machine` Terraform module]({{< relref "machine-module" >}}), and configured [with k3s Ansible role]({{< relref "k3s-role" >}}).

The ClusterIssuer configured by this role issues certificates by solving ACME DNS01 challenge automatically through Cloudflare DNS API.
This configuration only works for domains using Cloudflare DNS.

## Variables

* `cert_manager_chart_version` - Cert Manager operator Helm chart version to use. Default is `v1.10.1`.
* `cert_manager_production_version` - Use Let's Encrypt production environment when `true`, staging environment when `false`. Default is `false`.
* `cert_manager_clusterissuer_name` - Name of ClusterIssuer resource to create. Default is `letsencrypt`.
* `cert_manager_cloudflare_email` - Email for Cloudflare authentication and Let's Encrypt account. **Required**.
* `cert_manager_cloudflare_token` - Token for Cloudflare authentication. **Required**.
* `cert_manager_cloudflare_zone` - DNS zone (domain) to issue certificates for. **Required**.

For `cert_manager_cloudflare_token` permissions [see Cert Manager documentation](https://cert-manager.io/docs/configuration/acme/dns01/cloudflare/#api-tokens).
Make sure to generate a token with access for only the specific zone (domain) used.

## Example Playbook

Given a `production` group in the Ansible inventory, this playbook installs a single node K3s cluster and Cert Manager on said cluster.

Head to Cloudflare dashboard and create an API token [with the permissions specified by Cert Manager documentation](https://cert-manager.io/docs/configuration/acme/dns01/cloudflare/#api-tokens).

```yaml
---
- name: Configure Kubernetes cluster
  hosts: production
  roles:
    - role: lkummer.homelab.k3s
    - role: lkummer.homelab.cert_manager
      vars:
        cert_manager_cloudflare_email: you@example.com
        cert_manager_cloudflare_token: REDACTED
        cert_manager_cloudflare_zone: example.com
```

It is recommended [to use Ansible Vault](https://docs.ansible.com/ansible/latest/cli/ansible-vault.html) to encrypt secrets stored in infrastructure repositories.

Cert Manager is configured to create a ClusterIssuer called `letsencrypt`, which will issue certificates for `example.com`.

You can now use the `letsencrypt` ClusterIssuer, for example to issue a certificate for an Ingress resource:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt
spec:
  rules:
    - http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: example
                port:
                  name: http
      host: example.com
  tls:
    - hosts:
        - example.com
      secretName: example-tls
```
