---
title: observability role
---

This Ansible role configures a full observability stack, including: Prometheus Operator, Grafana Agent, Grafana, Prometheus, Alertmanager, Loki, Promtail, Tempo, and OpenTelemetry Collector.

It is designed to configure VMs cloned using [the `machine` Terraform module]({{< relref "machine-module" >}}), and configured [with k3s Ansible role]({{< relref "k3s-role" >}}).
It integrates with the ClusterIssuer deployed by default [by the cert_manager role]({{< relref "cert-manager-role" >}}) for issuing TLS certificates.

The default Prometheus instance should be used for scraping metrics from apps running in the cluster.
Prometheus Operator PodMonitor and ServiceMonitor resources should be used to configure scraping of metrics.

This role uses Cert Manager for generating certificates for the OpenTelemetry Operator.
Make sure to **install the `cert_manager` role first**.

## Variables

* `observability_kube_prometheus_stack_chart_version` - Version of prometheus-community/kube-prometheus-stack Helm chart to deploy. Default is `45.8.1`.
* `observability_loki_chart_version` - Version of grafana/loki Helm chart to deploy. Default is `4.8.0`.
* `observability_promtail_chart_version` - Version of grafana/promtail Helm chart to deploy. Default is `6.8.1`.
* `observability_tempo_chart_version` - Version of grafana/tempo Helm chart to deploy. Default is `1.0.2`.
* `observability_opentelemetry_operator_chart_version` - Version of OpenTelemetry Operator Helm chart to deploy. Default is `0.23.0`.
* `observability_grafana_host` - Host for Grafana. **Required**.
* `observability_grafana_user` - Username of the Grafana admin. **Required**.
* `observability_grafana_password` - Password of the Grafana admin. **Required**.
* `observability_service_monitor_selector` - Additional labels for default Prometheus instance ServiceMonitor selector. Default is `{}`.
* `observability_pod_monitor_selector` - Additional labels for default Prometheus instance PodMonitor selector. Default is `{}`.

The Ingress for Grafana is annotated to use the `letsencrypt` ClusterIssuer configured by `cert_manager` role.
Make sure `observability_grafana_host` is in the zone (domain) the ClusterIssuer is configured for.

## Example Playbook

Given a production group in the Ansible inventory, this playbook installs a single node K3s cluster and configures the monitoring setup on each host:

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
    - role: lkummer.homelab.prometheus
      vars:
        observability_grafana_host: grafana.example.com
        observability_grafana_user: admin
        observability_grafana_password: REDACTED
```

It is recommended [to use Ansible Vault](https://docs.ansible.com/ansible/latest/cli/ansible-vault.html) to encrypt secrets stored in infrastructure repositories.

The default Prometheus instance will scrape PodMonitor and ServiceMonitor resources labelled `release: prometheus`.
You may set additional labels to scrape with the `observability_service_monitor_selector` and `observability_pod_monitor_selector` role variables.

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
