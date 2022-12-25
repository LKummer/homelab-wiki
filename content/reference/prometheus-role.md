---
title: prometheus role
---

This Ansible role configures a Grafana, Prometheus, Loki and Promtail observability stack.

* It installs Prometheus Operator and creates a default Prometheus instance for the cluster.
* It installs Loki and Promtail, and configures them to scrape logs from all pods running in the cluster.
* It installs Grafana, and connects it to Loki and the default Prometheus instance.

It is designed to configure VMs cloned using [the `machine` Terraform module]({{< relref "machine-module" >}}), and configured [with k3s Ansible role]({{< relref "k3s-role" >}}).
It integrates with the ClusterIssuer deployed by default [by the cert_manager role]({{< relref "cert-manager-role" >}}).

The default Prometheus instance should be used for scraping metrics from apps running in the cluster.
Prometheus Operator PodMonitor and ServiceMonitor resources should be used to configure scraping of metrics.

## Variables

* `prometheus_kube_prometheus_stack_version` - Version of prometheus-community/kube-prometheus-stack Helm chart to deploy. Default is `39.9.0`.
* `prometheus_loki_stack_version` - Version of grafana/loki-stack Helm chart to deploy. Default is `2.8.0`.
* `prometheus_grafana_host` - Host for Grafana. **Required**.
* `prometheus_grafana_user` - Username of the Grafana admin. **Required**.
* `prometheus_grafana_password` - Password of the Grafana admin. **Required**.
* `prometheus_service_monitor_selector` - Additional labels for default Prometheus instance ServiceMonitor selector.
* `prometheus_pod_monitor_selector` - Additional labels for default Prometheus instance PodMonitor selector.

The Ingress for Grafana is annotated to use the `letsencrypt` ClusterIssuer configured by `cert_manager` role.
Make sure `prometheus_grafana_host` is in the zone (domain) the ClusterIssuer is configured for.

## Example Playbook

Given a production group in the Ansible inventory, this playbook installs a single node K3s cluster and configures the monitoring setup on each host:

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
```

It is recommended [to use Ansible Vault](https://docs.ansible.com/ansible/latest/cli/ansible-vault.html) to encrypt secrets stored in infrastructure repositories.

The default Prometheus instance will scrape PodMonitor and ServiceMonitor resources labelled `release: prometheus` by default.
You can set additional labels to scrape through role variables.

For example, the following PodMonitor will configure the default Prometheus instance to scrape pods for metrics:

```yaml
apiVersion: monitoring.coreos.com/v1
kind: PodMonitor
metadata:
  name: example-podmonitor
  labels:
    release: prometheus
spec:
  selector:
    matchLabels:
      app.kubernetes.io/name: example
  podMetricsEndpoints:
    - port: http
      path: /metrics
  namespaceSelector:
    matchNames:
      - example
```
