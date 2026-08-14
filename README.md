# 🪐 Space Probe Infosec Hardening Lab

Welcome to the security architecture portfolio for a deep-space research probe. 

## Project Overview
This repository serves as a hands-on cybersecurity home lab built on **CentOS Linux**. Conceptually, this system simulates the onboard computer of an interstellar probe orbiting a black hole. 

## Project Objective
The goal of this initiative is to architect, configure, and validate an enterprise-grade Linux security baseline from scratch using a conceptual deep-space business logic. This project shifts away from abstract theoretical learning by enforcing hands-on **Blue Team** (systems hardening, identity management, network containment) and **Red Team** (compliance auditing, access-violation testing) methodologies guided by the **NIST Cybersecurity Framework (CSF) v2.0** [NIST].

---
## Lab Architecture
The infrastructure establishes a hybrid management perimeter consisting of two primary operational nodes:
*   **Node 1: `Black-Meridian` (CentOS Stream 9)** — The target virtual machine deployment representing the probe's onboard computer. This node is the direct focus of the OS hardening and security policies.
*   **Node 2: `Mission Control` (macOS Host Terminal)** — The dedicated physical hardware workstation representing Earth ground control. Communication is established remotely over an isolated virtual network bridge via cryptographic SSH channels.

---
## Lab Documentation Directory

To maintain an architectural timeline, the project is structured into sequential engineering modules. Click on any module to review the explicit threat models, implementation commands, and validation logs:

| Registry Index | Module Documentation File | Security Domain Focus (NIST CSF Alignment) | Status |
| :---: | :--- | :--- | :---: |
| **0** | [📄 Module 0: Infrastructure Deployment](./Module-0-System-Specs.md) | Base Image Sanitization & Environmental Verification | ✅ Complete |
| **1** | [📄 Module 1: Identity & Access Management](./Module-1-IAM.md) | Role-Based Access Control (RBAC) & Collaborative SGID Permissions | ✅ Complete |
| **2** | [📄 Module 2: System Hardening & Sudo Isolation](./Module-2-Hardening.md) | Secure Remote Management (SSH), Root Exclusion, and Sudo Controls | ✅ Complete |
| **3** | [📄 Module 3: Network Filtration & Perimeter Defense](./Module-3-Network.md) | Default-Deny Cloaking, Firewalld Drop Containment, and ICMP Auditing [NIST] | ✅ Complete |

