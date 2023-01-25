---
title: Example Lab Architecture
---

This page describes an example multi-tenant lab architecture that can be achieved using the framework.

Our example lab will supply each tenant with everything they need to run and develop cloud native applications.

This example is designed to run on a single Proxmox VE server, running commodity hardware on a *typical* home network.

We will start by describing the Kubernetes clusters we wish to run.
Then we will see what VMs we need in Proxmox to run the clusters.
Finally we will go over how you may structure infrastructure code for such a lab.

## Designing Our Kubernetes Clusters

Each tenant gets their own K3s cluster, labelled "Tenant K3s Cluster" in the diagram below.
The cluster includes everything needed to run applications, namely an observability stack, CD solution and TLS certificate automation.

Each tenant also gets access to central services running on the "Main K3s Cluster".
The services include GitLab as Git hosting, CI and container registry, Vault as a secret management service, and MinIO as a storage service.

![](../architecture-clusters.svg)

We can run as many tenant clusters as we have tenants.

We would want separate development, staging and production environments for the main cluster and each tenant cluster.

## Infrastructure For the Clusters

We will run single node clusters.
This means we will be needing one VM per cluster, or 6 VMs in total.

Why single node clusters?
This lab is designed to run on a single server and wont gain much from high availability clusters.
This lab also focuses on development and quick iterations,
cloning fewer VMs makes cluster creation faster.

We are looking at managing the following VMs:

![](../architecture-vms.svg)

## Structuring Infrastructure as Code

It is recommended to split each cluster into a repository: Main cluster, and tenant cluster repositories.

Each cluster gets it's own repository because maintenance is usually performed on one cluster at a time.
Having all clusters in a single repo prevents clusters from using different Ansible role versions, and requires a lot of Terraform `--target` and Ansible `--limit` usage.
Since all shared code is in the framework, we barely get any code duplication when using multiple repositories.

Each cluster repository should follow [the recommended folder layout]({{< relref "folder-layout" >}}).
