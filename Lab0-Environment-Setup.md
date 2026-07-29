## Course Information
---
**Course:** IKB42603 Cloud Computing Security Essentials
**Lab:** Lab 0 - Environment Setup
**Name:** MUHAMMAD AMEER BIN IDRIS 
**Date:** 30 July 2026

# Lab 0: Environment Setup Report

This report documents the step-by-step completion of the Lab 0 Environment Setup, providing evidence via screenshots for each required tool and configuration.

## 1. Install Docker
- **Objective:** Install and verify Docker, which is required to run containers, LocalStack, and kind.
- **Evidence of Installation:**
  ![Install Docker](1.%20install%20docker.png)

## 2. Install AWS CLI v2
- **Objective:** Install and verify AWS CLI v2 to send commands to the local AWS simulator (LocalStack).
- **Installation Evidence:**
  ![Install AWS CLI](2.%20install%20AWS%20CLI%20v2.png)
- **Verification Evidence:**
  ![Verify AWS CLI](3.%20verify%20AWS%20CLI%20v2.png)

## 3. Install kind & kubectl
- **Objective:** Install `kind` (to run a local Kubernetes cluster inside Docker) and `kubectl` (to control the cluster).
- **Installation Evidence:**
  ![Install kind and kubectl](4.%20install%20kind%20and%20kubectl.png)
- **Verification Evidence:**
  ![Verify kind and kubectl](5.%20verify%20kind%20&%20kubectl.png)

## 4. Helper Tools
- **Objective:** Verify and install helper tools (OpenSSL, oathtool) which are used for encryption, keys, certificates, and generating MFA/TOTP codes.
- **Installation and Verification Evidence:**
  ![Install and verify helper tools](6.%20install%20and%20verify%20helper%20tools.png)

## 5. Start & Stop the Lab Environment

### LocalStack (The Local AWS Simulator)
- **Objective:** Start the LocalStack instance locally and verify its health.
- **Evidence of Start and Checks:**
  ![LocalStack check](8.%20LocalStack%20checked.png)
- **Evidence of Healthy Status and Port 4566 Mapped:**
  ![LocalStack healthy](9.%20localstack%20healthy%20status%20and%20port%204566%20mapped.png)

### Kubernetes Cluster (kind)
- **Objective:** Initialize a Kubernetes cluster using kind (named ccse) and verify it is running successfully.
- **Evidence of Cluster Creation:**
  ![Create cluster](10.%20create%20cluster%20.png)
- **Evidence of Verification:**
  ![Cluster Verification](11.%20cluster%20was%20verified.png)

## 6. One-Time AWS CLI Configuration
- **Objective:** Pre-configure the AWS CLI with dummy credentials to smoothly interface with LocalStack.
- **Configuration Verification Evidence:**
  ![Configure AWS CLI](12.%20configure%20AWS%20CLI%20for%20LocalStack.png)

---
**Summary Checklist (All criteria met successfully):**
- [x] `docker --version` prints a version and hello-world runs.
- [x] `aws --version` prints correctly.
- [x] `kind --version` and `kubectl version --client` both run successfully.
- [x] LocalStack starts and health endpoints return successfully.
- [x] `aws sts get-caller-identity` successfully returns dummy caller identity using the LocalStack endpoint.
- [x] Kubernetes cluster `ccse` created and node is accessible.
