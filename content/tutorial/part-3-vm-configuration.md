---
title: Part 3 - Configuring the VM with Ansible
---

In this part you will use Ansible to configure the VM you cloned earlier.
By the end you will have K3s running on the VM with ArgoCD, Cert Manager and an observability stack all ready to use.
Make sure to have environment variables [configured as described in part 1]({{< relref "part-1-preparation#configuring-environment-variables" >}}).

## Domains for Grafana and ArgoCD

For the last part of the tutorial you are going to need domains for ArgoCD and Grafana UIs.
It is easiest to use subdomains, for example: `grafana.example.com` and `argo.example.com`.

If you want TLS to work, you will need to actually own the domains used.
You can still continue the tutorial without a domain or with an unregistered domain, but TLS certificates will not be issued.

There are instructions to access Grafana and ArgoCD with `kubectl port-forward` later on if you do not want to mess with DNS or edit your `hosts` file.

## Downloading Ansible Requirements

Change to the `playbook` folder:

```
cd playbook
```

Install Ansible Collections from the requirements file:

```
ansible-galaxy collection install --requirements requirements.yaml
```

**Install collection dependencies every time before running Ansible.**
Collections are not scoped per project, and mixing collection versions between projects may be destructive.

## Editing Grafana and ArgoCD Hosts

Open up `playbook/main.yaml` and change `observability_grafana_host` and `argo_host` to the domains you wish to use.
The example uses `grafana.example.com` and `argo.example.com` for Grafana and ArgoCD respectively:

```yaml
# Inside playbook/main.yaml
    - role: lkummer.homelab.observability
      vars:
        observability_grafana_host: grafana.example.com
        # ...
    - role: lkummer.homelab.argo
      vars:
        argo_host: argo.example.com
```

## Editing Cloudflare Credentials For Certificate Issuing

If you are not using a domain you own, skip this section.

Still inside `playbook/main.yaml`: Change `cert_manager_cloudflare_email` to your Cloudflare email, `cert_manager_cloudflare_token` to a Cloudflare API token with `Zone - DNS - Edit` and `Zone - DNS - Read` permissions for the domain you are using, and `cert_manager_cloudflare_zone` to the domain you are using.

```yaml
# Inside playbook/main.yaml
    - role: lkummer.homelab.cert_manager
      vars:
        cert_manager_cloudflare_email: you@example.com
        # Use Ansible Vault for actual secrets!
        cert_manager_cloudflare_token: REDACTED
        cert_manager_cloudflare_zone: example.com
```

**Do not commit secrets to Git. Use Ansible Vault or another secret storage solution.**

## Configuring the VM

Still in the `playbook` folder,
run the playbook with the following command to configure the VM to run K3s, and deploy Prometheus Operator, Grafana, Loki, Promtail and ArgoCD on it.

```
ansible-playbook --inventory=inventory main.yaml
```

## Accessing the Kubernetes Cluster

[The `lkummer.homelab.k3s` role]({{< ref "reference/k3s-role" >}}) in the playbook you just ran created a `playbook/secrets` folder with a kubeconfig you can use to access the Kubernetes cluster.

You can manually configure `kubectl`, or set the `KUBECONFIG` environment variable by running the following command in the `playbook` folder:

```
export KUBECONFIG="$(ls -lt secrets/k3s* | head -n 1 | sed 's/.*\(secrets.*$\)/\1/')"
```

## Accessing Grafana

If you configured DNS or your `hosts` file, you can use the domain you configured to access Grafana.

If you have not configured a domain or are having trouble with it, use `kubectl port-forward` and access Grafana on `localhost:3000` with this command:

```
kubectl port-forward --namespace prometheus service/prometheus-grafana 3000:http-web
```

If you have left the example settings in `playbook/main.yaml` you can connect to the `admin` user with the password `admin`.

## Accessing ArgoCD

If you configured DNS or your `hosts` file, you can use the domain you configured to access ArgoCD.

If you have not configured a domain or are having trouble with it, use `kubectl port-forward` and access ArgoCD on `localhost:3001` with this command:

```
kubectl port-forward --namespace argo-cd service/argocd-server 3001:http
```

The password for the ArgoCD `admin` user is stored in a Kubernetes secret. Use the following command to view it:

```
kubectl get --namespace argo-cd secrets/argocd-initial-admin-secret --output 'jsonpath={.data.password}' | base64 -d
```
