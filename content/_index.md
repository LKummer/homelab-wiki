+++
title = "Homelab Framework"
[menu.main]
  name = "Documentation"
  weight = -10
+++

The Homelab Framework allows you to provision and manage a Kubernetes lab on Proxmox VE.
It is designed for quickly defining and provisioning Kubernetes clusters for all your lab needs.

An overview of how this documentation site is organized may help you find your way around:

* [Tutorial]({{< ref "tutorial/part-1-preparation" >}}) for deploying your own single node Kubernetes cluster with ArgoCD and observability tools.
* How-To Guides for common tasks.
* Reference documentation of all framework parts.

The framework consists of the following repositories:

* [Packer Alpine](https://github.com/LKummer/packer-alpine) template creating a cloud image for Proxmox.
* [Terraform Proxmox](https://github.com/LKummer/terraform-proxmox) module for cloning VMs from the cloud image.
* [Ansible Collection Homelab](https://github.com/LKummer/ansible-collection-homelab) includes inventory plugins and roles for configuring Kubernetes clusters. The roles configure K3s, ArgoCD, Prometheus Operator, Grafana, Loki, Promtail, and Cert Manager. The plugins populate Ansible inventory from Terraform state. 
* Documentation you are reading right now.
