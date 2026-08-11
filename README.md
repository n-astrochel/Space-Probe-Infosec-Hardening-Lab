# Space Probe Infosec Hardening Lab

Welcome to the security architecture portfolio for a deep-space research probe. 

## Project Overview
This repository serves as a hands-on cybersecurity home lab built on **CentOS Linux**. Conceptually, this system simulates the onboard computer of an interstellar probe orbiting a black hole. 

The goal of this project is to progressively implement core cybersecurity principles (aligned with the Google Cybersecurity Certificate) on an enterprise-grade Linux infrastructure, balancing **Blue Team** (defense & architecture) and **Red Team** (security auditing) methodologies.

---

## Lab Architecture
The infrastructure consists of two isolated virtual environments:
*   **Node 1: `Black-Meridian` (CentOS)** — The main onboard computer of the probe. This is the target for security hardening.
*   **Node 2: `Earth-Mission-Control` (CentOS/Kali)** — The ground station used for monitoring, log analysis, and authorized security testing.

---

### Infrastructure Baseline Specifications
*   **Hypervisor:** VMware Fusion (Apple Silicon Architecture)
*   **Guest OS:** CentOS Stream 9 (64-bit ARM)
*   **Installation Profile:** Minimal Install (CLI Only, No GUI for maximum security baseline)
*   **Virtual Hardware Resources:** 2 vCPUs, 2 GB RAM, 20 GB HDD
*   **Network Configuration:** NAT (Network Address Translation)
