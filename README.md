# SUSE Rancher Prime Guide

Welcome to the `SUSE Rancher Prime` Suite Guide GitHub Repo — your go-to resource for understanding, deploying, configuring, managing, and troubleshooting the `SUSE Rancher Prime` Suite portfolio. Whether you’re preparing a demo, running a proof-of-concept, testing features, or simply learning the stack, this repo is designed to walk you through everything, step by step. Inside, you’ll find high-level explanations and hands-on instructions covering all components of the `SUSE Rancher Prime` Suite. While this guide is primarily built for `SUSE` Solution Architects, it’s also a practical resource for Infrastructure, Cloud-Native, and DevOps teams managing the `SUSE Rancher Prime` solution in real-world environments. Each section is built to be clear, practical, and easy to follow — written with a friendly, conversational tone to make your experience as smooth as possible.

If you’re looking for a single source of truth to master `SUSE Rancher Prime` Suite stack, you’re in the right place.

---

<p align="center">
    <img src="Images/Repo-PIC.png">
</p>

---

> ⚠️ Disclaimer:
> 
> This is not an official `SUSE` document. While it is based on practical experience and best practices, it is strongly recommended to refer to the official `SUSE Rancher` documentation for the most accurate and up-to-date guidance: https://documentation.suse.com

---

## About This Repo

This repository is built to be your easy, go-to guide when deploying, configuring, managing, or troubleshooting the `SUSE Rancher Prime` Suite stack. It’s designed to simplify your experience — whether you’re setting up a quick lab, running a customer demo, building a PoC, or working in a real-world environment.

Here, you’ll find documented step-by-step guides and supporting scripts (where available) that walk you through each stage of working with the `SUSE Rancher Prime` Suite stack. The goal is to save you time, reduce complexity, and give you confidence while working with these solutions.

**What’s Covered:**

This repo covers the entire `SUSE Rancher Prime` Suite, which includes:
- **SUSE Rancher**: Centralized management for Kubernetes clusters — regardless of where they run (on-prem, cloud, or edge). Rancher simplifies cluster operations, policy enforcement, and multi-tenant environments.
- **SUSE Kubernetes (RKE & K3s)**: Two lightweight and production-grade Kubernetes distributions. RKE2 is fully CNCF-compliant and secure by default, while K3s is ideal for edge and small-footprint use cases.
- **SUSE Storage (formerly Longhorn)**: A cloud-native distributed block storage solution for Kubernetes. It offers snapshots, backups, replication, and easy recovery — making it great for persistent workloads.
- **SUSE Security (formerly NeuVector)**: A zero-trust, end-to-end container security platform. It provides runtime protection, vulnerability scanning, network segmentation, and compliance for Kubernetes environments.
- **SUSE Observability (formerly StackState)**: Full-stack observability built for modern applications — enabling intelligent correlation, topology mapping, and faster root cause analysis across your infrastructure.
- **SUSE Application Collection**: Pre-packaged Helm charts and curated apps to simplify Kubernetes adoption. These tools help users accelerate time-to-value when deploying workloads on SUSE Rancher.
- **SUSE Private Registry**: A secure, self-hosted container image registry that supports air-gapped deployments and local image management for tighter control and compliance.
- **SUSE Rancher Fleet (GitOps)**: GitOps-based continuous delivery and lifecycle management for Kubernetes at scale. Fleet enables version-controlled, repeatable deployments across clusters.
- **SUSE Virtualization (formerly Harvester)**: A modern hyperconverged infrastructure (HCI) solution that combines VMs and containers on a single platform — fully integrated with Rancher.

> **Note:** This repository is a work in progress. New content will be added and existing material refined regularly, so check back often for updates.

---

## Resources Available In This Repo

This repo will provide high-level explanation, step-by-step-guides, and scripts for the installation and configuration of the `SUSE Rancher` DevSecOps solution portfolio. Thus the resources available in this repo are as follows:

- Install - _provides solution installation guides and scripts_
  - [RKE2](/1-Install/RKE2/)
  - [K3S](/1-Install/K3S/)
  - [Rancher](/1-Install/Rancher/)
  - [Longhorn](/1-Install/Longhorn/)
  - [NeuVector](/1-Install/NeuVector/)
- Configure _provides solution configuration guides and scripts_
  - [RKE2](/2-Configure/RKE2/)
  - [K3S](/2-Configure/K3S/)
  - [Rancher](/2-Configure/Rancher/)
  - [Longhorn](/2-Configure/Longhorn/)
  - [NeuVector](/2-Configure/NeuVector/)
- Troubleshoot _provides solution installation and configuration troubleshooting guides and scripts_
  - [RKE2](/3-Troubleshoot/RKE2/)
  - [K3S](/3-Troubleshoot/K3S/)
  - [Rancher](/3-Troubleshoot/Rancher/)
  - [Longhorn](/3-Troubleshoot/Longhorn/)
  - [NeuVector](/3-Troubleshoot/NeuVector/)

---

## Official References:

- [SUSE Official Documentation](https://documentation.suse.com)

---

**Enjoy** :blush:
