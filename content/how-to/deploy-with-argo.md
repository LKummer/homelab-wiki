---
title: How to deploy Helm charts with Argo CD
---

This guide walks you through configuring a Helm deployment inside an
infrastructure repository using Argo CD.

## Requirements

- Your project structure is [similar to the tutorial project.](https://github.com/LKummer/homelab-tutorial-infrastructure#using-local-terraform-state)
- Your cluster [has Argo CD installed]({{< ref "reference/argo-role" >}}).

## Creating a Role

Edit `playbook/main.yaml` to add the `dns` role:

```yaml
# Inside playbook/main.yaml
---
- name: Configure Kubernetes cluster
  hosts: tutorial
  roles:
    # ...
    - role: dns
```

Create a folder for tasks:

```
mkdir --parents roles/dns/tasks
```

## Preparing Namespace and AppProject

Add the following tasks to `playbook/roles/dns/tasks/main.yaml`.
Tailor the namespace and project name to your needs.

```yaml
# Inside playbook/roles/dns/tasks/main.yaml
---
- name: Namespace is present
  become: true
  kubernetes.core.k8s:
    kubeconfig: /etc/rancher/k3s/k3s.yaml
    api_version: v1
    kind: namespace
    name: dns

- name: ArgoCD AppProject is present
  become: true
  kubernetes.core.k8s:
    kubeconfig: /etc/rancher/k3s/k3s.yaml
    namespace: argo-cd
    definition:
      apiVersion: argoproj.io/v1alpha1
      kind: AppProject
      metadata:
        name: dns
      spec:
        clusterResourceWhitelist:
          - group: "*"
            kind: "*"
        destinations:
          - namespace: dns
            server: "*"
        sourceRepos:
          - "*"
```

## Creating an Application

Adjust the following task to deploy your Helm chart:

```yaml
# Inside playbook/roles/dns/tasks/main.yaml
- name: ArgoCD application is present
  become: true
  kubernetes.core.k8s:
    kubeconfig: /etc/rancher/k3s/k3s.yaml
    namespace: argo-cd
    definition:
      apiVersion: argoproj.io/v1alpha1
      kind: Application
      metadata:
        name: coredns
      spec:
        project: dns
        foo:
        source:
          chart: coredns
          repoURL: https://git.example.com/api/v4/projects/106/packages/helm/stable
          targetRevision: 1.x
          helm:
            releaseName: coredns
            values: >
              podMonitor:
                enabled: true
        destination:
          server: https://kubernetes.default.svc
          namespace: dns
        syncPolicy:
          automated: {}
```

Run the playbook.
You should be able to open Argo CD and see your project.
