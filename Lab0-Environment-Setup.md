## Course Information
---
**Course:** IKB42603 Cloud Computing Security Essentials

**Lab:** Lab 0 - Environment Setup

**Name:** MUHAMMAD AMEER BIN IDRIS

**Date:** 30 July 2026

# Lab 0: Environment Setup Report

## 1. Docker

### Objective
To install, configure, and verify the Docker engine on the host machine to enable containerization capabilities, which serves as the foundational layer for running LocalStack and the Kubernetes cluster.

### Conceptual Background
Docker is a platform that uses OS-level virtualization to deliver software in packages called containers. In Cloud Computing Security, containerization is a critical paradigm. It ensures that applications run predictably across different environments, enforces isolation boundaries between services, and is central to modern microservices architectures. Understanding Docker is essential for securing container images, managing runtime security, and mitigating risks such as container breakout or privilege escalation.

### Execution Details
The installation involves downloading the Docker binaries or using a package manager to install and configure the Docker daemon (`dockerd`). The verification command (e.g., executing `docker --version` or running a test container) communicates with the Docker daemon via a local API socket. This confirms that the daemon is actively running, network interfaces are correctly bound, and the engine is successfully authorized to pull and execute container images from a registry.

### Evidence
Install Docker:

<img width="801" height="132" alt="1  install docker" src="https://github.com/user-attachments/assets/65c8123a-6ae5-4f3a-a721-2814fe1bd2a5" />


---

## 2. AWS CLI v2

### Objective
To install and verify the official Command Line Interface (v2) for Amazon Web Services, ensuring the ability to programmatically interact with cloud infrastructure and mocked LocalStack services.

### Conceptual Background
The AWS CLI is a unified tool used to manage AWS cloud services directly from the terminal. From a Cloud Computing Security perspective, mastering the CLI is vital. Security configurations, Identity and Access Management (IAM) audits, policy enforcements, and incident responses in cloud environments are largely driven by automated scripts executing CLI commands. It enables precise, automatable control over cloud resources and eliminates the reliance on the web console.

### Execution Details
The execution process installs the pre-compiled AWS CLI v2 binary, placing the primary `aws` executable in the system's PATH. The verification command `aws --version` queries the executable to display the installed CLI version, the underlying Python context, and OS architecture specifics. Behind the scenes, this validates that cryptographic dependencies are correctly linked and the tool is primed to construct, sign, and reliably transmit secure REST API requests to AWS endpoints.

### Evidence
Install AWS CLI: 

<img width="822" height="172" alt="2  install AWS CLI v2" src="https://github.com/user-attachments/assets/bf0f341a-a029-4ba3-ab54-55de499c8936" />


Verify AWS CLI:

<img width="821" height="87" alt="3  verify AWS CLI v2" src="https://github.com/user-attachments/assets/ce011cb5-473f-4377-885a-11e65e90a9ba" />

---

## 3. kind & kubectl

### Objective
To install `kind` (Kubernetes IN Docker) alongside `kubectl` (the Kubernetes command-line tool) in order to securely deploy, manage, and interact with a local, lightweight Kubernetes cluster.

### Conceptual Background
`kind` allows the deployment of complete Kubernetes instances by treating Docker containers as cluster nodes, providing a fast local cluster simulation. `kubectl` is the universal interface for managing Kubernetes resources. In cloud security, Kubernetes is the de-facto standard for container orchestration. Security professionals must understand how to secure cluster configurations, implement Role-Based Access Control (RBAC), enforce network policies, and audit API server interactions. Using `kind` establishes an isolated, safe sandbox for practicing advanced Kubernetes security assessments.

### Execution Details
Installing `kind` downloads a compiled binary that interacts with the local Docker daemon to bootstrap Kubernetes nodes from specific container images. Installing `kubectl` places the command-line utility in the PATH. The verification steps (`kind --version` and `kubectl version --client`) check the output for binary version validity. At this stage, these commands do not actively connect to a remote API server; instead, they simply validate that the client-side parsing libraries and authentication logic are intact and ready to interface with a Kubernetes control plane once initialized.

### Evidence
Install kind and kubectl:

<img width="802" height="185" alt="4  install kind and kubectl" src="https://github.com/user-attachments/assets/be95f2dd-bf13-4013-b554-9147288f9686" />



Verify kind and kubectl:

<img width="706" height="132" alt="5  verify kind   kubectl" src="https://github.com/user-attachments/assets/4f522b97-e0d6-4c92-ac6a-bb88e533640b" />



---

## 4. Helper Tools (OpenSSL, oathtool)

### Objective
To safely install and verify fundamental cryptographic and authentication suite programs, specifically OpenSSL and oathtool.

### Conceptual Background
OpenSSL is a robust commercial-grade toolkit providing cryptography and secure communication protocols (TLS/SSL). `oathtool` is a highly flexible command-line utility for generating Time-based One-Time Passwords (TOTP). In cloud security, OpenSSL is heavily relied upon for generating RSA/ECC key pairs, creating and signing x.509 certificates, and managing Public Key Infrastructure (PKI) vital for securing endpoints. `oathtool` is essential for scripting Multi-Factor Authentication (MFA), which is a mandatory administrative control for averting unauthorized access to privileged cloud IAM accounts.

### Execution Details
The installation package managers pull these cryptographic binaries and libraries and register them with the host's execution environment. When running the respective version checks, the system not only proves the software exists but verifies successful linking to internal cryptographic cipher suites (for OpenSSL) and valid time-hashing algorithm libraries (for `oathtool`). Behind the scenes, these checks guarantee that the tools are properly initialized to output pseudo-random entropy for certificate generation and mathematically compute precise time-synchronized MFA tokens.

### Evidence
Install and verify helper tools: 

<img width="805" height="235" alt="6  install and verify helper tools" src="https://github.com/user-attachments/assets/2e7b2e21-4cfe-491f-8af7-2c793e3aeab8" />


---

## 5. Start & Stop the Lab Environment (LocalStack & Kubernetes Cluster)

### Objective
To systematically operationalize the lab environment by successfully spinning up and validating the networking and health status of the LocalStack AWS simulator and the `kind` Kubernetes cluster.

### Conceptual Background
LocalStack is an ecosystem that provides a fully functional, localized AWS environment mimicking core cloud services (like S3, IAM, and Lambda) without requiring real internet-bound traffic or incurring cloud costs. Coupling LocalStack with a local Kubernetes cluster enables a comprehensive Infrastructure-as-Code testbed. For a cloud security student, running this simulated infrastructure allows for the safe implementation of Identity Federation, aggressive configuration of security guardrails, and execution of local penetration tests without threatening a live production ecosystem.

### Execution Details
Starting LocalStack typically triggers an underlying Docker command to pull and run its monolithic container image, exposing multiple mocked APIs on an aggregated port (usually TCP/4566). The health check process issues a localized cURL or API request parsing the `/health` endpoint JSON response to confirm that specific mocked services have reached a 'running' state. Similarly, creating the `kind` cluster (`kind create cluster`) instructs Docker to configure new bridged networks and launch isolated containers acting as independent control-plane worker nodes. Verifying the cluster via `kubectl get nodes` forces the kubectl binary to parse the default `~/.kube/config` file for context strings and certificate data, authenticate over HTTPS to the local Docker-mapped API server port, and actively retrieve the operational specifications of the current cluster nodes.

### Evidence
#### LocalStack
LocalStack check:

<img width="807" height="297" alt="8  LocalStack checked" src="https://github.com/user-attachments/assets/148dd87a-21a4-4aa3-bd76-636b3aa1b862" />


LocalStack healthy:

<img width="855" height="170" alt="9  localstack healthy status and port 4566 mapped" src="https://github.com/user-attachments/assets/1d6265e7-0ba3-4a22-b49e-86fcc0bb16a6" />


#### Kubernetes Cluster
Create cluster:

<img width="562" height="312" alt="10  create cluster " src="https://github.com/user-attachments/assets/eaea3283-e855-4a47-ade2-6597dd07b74c" />


Cluster Verification:

<img width="862" height="260" alt="11  cluster was verified" src="https://github.com/user-attachments/assets/6f4e5812-c88c-4314-8aba-c65771fd8387" />



---

## 6. One-Time AWS CLI Configuration

### Objective
To pre-configure the AWS CLI core credentials and configuration profiles with dummy values explicitly geared to route simulated cloud traffic toward the local LocalStack instance instead of live public AWS endpoints.

### Conceptual Background
The AWS CLI determines authorization and routing by referencing credentials (Access Key ID and Secret Access Key), as well as configuration items (Default Region and Output Format), generally stored within the local user's `~/.aws/` directory. Strict credential management is a paramount discipline in cloud security; unintentional leakage of live AWS keys into git repositories has historically led to massive automated compromises. By deliberately substituting these files with dummy values (e.g., `test`) and targeting a localized endpoint (`http://localhost:4566`), we enforce fail-safe security practices that prevent accidental modification or exposure of actual production accounts.

### Execution Details
Running the interactive `aws configure` console program overrides or creates specific plain-text blocks within `~/.aws/credentials` and `~/.aws/config`. Behind the scenes, these inputs formulate a configuration profile (by default `[default]`). Subsequently, whenever an `aws` command is executed, the CLI reads these files, securely signs the HTTP payload utilizing the mock secret key via AWS Signature Version 4 mapping, and structures the request. By later combining this default profile with the `--endpoint-url` flag, the local DNS resolution bypasses standard AWS edge servers, redirecting the fully signed but dummy-authorized payload straight to the localized LocalStack Docker container port interceptor.

### Evidence
Configure AWS CLI

<img width="482" height="427" alt="12  configure AWS CLI for LocalStack" src="https://github.com/user-attachments/assets/6e0e19c3-c5cf-464b-8be8-d4c40e33e46b" />

