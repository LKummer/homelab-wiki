---
title: Part 3 - Configuring the VM with Ansible
---

In this part you will configure the VM you cloned with Ansible.
By the end you will have K3s running on the VM with ArgoCD, Grafana, Prometheus Operator, Loki and Promtail all ready.
Make sure to have environment variables [configured as described in part 1]({{< relref "part-1-preparation#configuring-environment-variables" >}}).

## Domains for Grafana and ArgoCD

For the last part of the tutorial you are going to need domains for ArgoCD and Grafana UIs.
It is easiest to use subdomains, for example: `grafana.example.com` and `argo.example.com`.

There are instructions to access Grafana and ArgoCD with `kubectl port-forward` later on if you do not want to mess with DNS or edit your `hosts` file.

## Editing Grafana and ArgoCD Hosts

Open up `playbook/main.yaml` and change `prometheus_grafana_host` and `argo_host` to the domains you wish to use.
The example uses `grafana.example.com` and `argo.example.com` for Grafana and ArgoCD respectively:

```yaml
# Inside playbook/main.yaml
    - role: lkummer.homelab.prometheus
      vars:
        prometheus_grafana_host: grafana.example.com
        # ...
    - role: lkummer.homelab.argo
      vars:
        argo_host: argo.example.com
```

## Configuring the VM

Change to the `playbook` folder:

```
cd playbook
```

Run the playbook with the following command to configure the VM to run K3s, and deploy Prometheus Operator, Grafana, Loki, Promtail and ArgoCD on it.

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