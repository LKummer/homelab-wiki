# Homelab Infrastructure Framework Wiki

This repository documents my homelab infrastructure framework.
[See the Pages site for the documentation itself.](https://lkummer.github.io/homelab-wiki/)

The homelab infrastructure framework is built to dynamically deploy Kubernetes clusters in a homelab environment.
It helps configuring said clusters with everything needed to deploy cloud native applications, namely observability and CD.

The framework consists of the following repositories:

* [Packer Alpine](https://github.com/LKummer/packer-alpine) - template for creating an Alpine Linux Proxmox template. It includes Cloud Init, Python, and everything required to clone it and configure it without human intervention. It also includes tests with Terratest and a CI pipeline.
* [Terraform Proxmox](https://github.com/LKummer/terraform-proxmox) - Terraform module for easily cloning the Proxmox template. Includes Terratest test and CI pipeline.
* [Ansible Collection Homelab](https://github.com/LKummer/ansible-collection-homelab) - Ansible plugins for populating inventory, and roles for configuring K3s, an observability stack and CD.

## Development Guide

This development guide refers to the wiki itself, not the framework.
For more details on the framework, see the wiki and repositories linked above.

### Installing Dependencies

Hugo is managed through NPM in this repository.
Use NPM to install all dependencies:

```
npm ci
```

### Development Server

Run the `dev` script through NPM to bring up a development server:

```
npm run dev
```

You should now have a hot-reload development server running on `localhost:1313`.

### Deployment

Public deployment to GitHub Pages is managed with GitHub Actions.

Private deployment to GitLab Pages is managed with GitLab CI.

Simply commit changes and push to have your changes deployed.
