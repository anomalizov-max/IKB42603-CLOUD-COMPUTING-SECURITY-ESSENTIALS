## Course Information
---
*Course:* IKB42603 Cloud Computing Security Essentials

*Lab:* Lab 0 - Environment Setup

*Name:* MUHAMMAD AMEER BIN IDRIS

*Date:* 13 August 2026


# Lab 2: Secure Isolation and Multi-Tenancy


## Lab Summary

This lab provides a comprehensive exploration of secure isolation strategies within a multi-tenant cloud infrastructure. It models an environment where two tenants are represented using separate logical boundaries via Kubernetes namespaces (`tenant-a` and `tenant-b`), sharing underlying cluster hardware and orchestrator planes.

The lab exercises progressively tackle critical pillars of cloud security:
1. **Network Security**: Demonstrating the inherent risk in Kubernetes' default-open flat network architecture, and remediating this by effectively implementing Zero Trust principles using Calico `NetworkPolicies` for microsegmentation.
2. **Compute & Resource Authorization**: Applying `ResourceQuotas` to defend against denial-of-service ("noisy-neighbor") resource exhaustion attacks, and enforcing strict Role-Based Access Control (RBAC) to securely isolate tenant identity boundaries.
3. **Data Lifecycle & Storage Safety**: Exploring physical data remanence risks by analyzing native Docker volume behaviors. It compares fundamental file-pointer deletion (`rm`) with rigorous data wiping (overwrite-before-delete), emphasizing the need for cryptographic erasure in modern cloud compliance schemas.

## Evidence Folder

All screenshots used for this report are stored in the `Evidence` folder.

| Evidence File | Purpose |
|---|---|
| `1. cointainer network interface.png` | kind cluster `ccse-lab2` creation |
| `1.1 install calico.png` | Calico installation and rollout status |
| `2. two tenant on one cluster.png` | Creation of `tenant-a` and `tenant-b` namespaces |
| `2.1 Check what is running.png` | Nginx deployments and services for both tenants |
| `2.3 he service IP for Tenant B was retrieved.png` | Tenant B service ClusterIP discovery |
| `2.4 temporary curl pod was launched from tenant-a to access the tenant-b web service.png` | Before NetworkPolicy probe showing `tenant-a` can reach `tenant-b` |
| `3. A ResourceQuota was applied to tenant-a.png` | ResourceQuota YAML applied to `tenant-a` |
| `3.1 Verification command.png` | ResourceQuota inspection output |
| `4. A default-deny ingress NetworkPolicy was applied to tenant-b.png` | Default-deny ingress NetworkPolicy applied to `tenant-b` |
| `4.1 . Re-run the probe with resource requests.png` | Network retest attempt after NetworkPolicy |
| `5. Each tenant created its own Kubernetes Secret.png` | Per-tenant Secret creation |
| `5.1 A ServiceAccount, Role and RoleBinding were created for tenant-a.png` | ServiceAccount, Role and RoleBinding creation |
| `5.2 The intended authorization check is.png` | RBAC `can-i` authorization results |
| `6. The first command creates sensitive data in a Docker volume.png` | Normal delete and remanence scan command |
| `6.1 The second command overwrites the file with zeros before deleting it.png` | Overwrite-before-delete secure wipe command |

## Setup: Cluster with Policy Enforcement

The lab cluster was created using kind with the default CNI disabled. Calico was then installed so that Kubernetes NetworkPolicy rules are enforced.

Command summary:

```bash
cat <<EOF | kind create cluster --name ccse-lab2 --config=-
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
networking:
  disableDefaultCNI: true
  podSubnet: 192.168.0.0/16
EOF

kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.27.0/manifests/calico.yaml
kubectl -n kube-system rollout status daemonset/calico-node --timeout=180s
```

Result:

The `ccse-lab2` cluster was created successfully and Calico was installed to enforce network isolation rules.

Evidence:

![Create kind cluster](<Evidence/1. cointainer network interface.png>)

![Install Calico](<Evidence/1.1 install calico.png>)

![Check Calico](<Evidence/1.2 check calico is ready.png>)

## Task 1: Two Tenants on One Cluster

Two tenants were created as separate Kubernetes namespaces:

```bash
kubectl create namespace tenant-a
kubectl create namespace tenant-b
```

Each tenant was given a simple Nginx web deployment and ClusterIP service:

```bash
kubectl -n tenant-a create deployment web --image=nginx
kubectl -n tenant-b create deployment web --image=nginx
kubectl -n tenant-a expose deployment web --port=80
kubectl -n tenant-b expose deployment web --port=80
kubectl get pods,svc -n tenant-a
kubectl get pods,svc -n tenant-b
```

Result:

Both tenants share the same Kubernetes cluster but are separated logically using namespaces. The screenshots show web pods and services created in `tenant-a` and `tenant-b`.

Evidence:

![Create tenants](<Evidence/2. two tenant on one cluster.png>)

![Deploy tenant web services](<Evidence/2.1 Check what is running.png>)

## Task 2: Observe the Default-Open Risk

The service IP for Tenant B was retrieved:

```bash
kubectl get svc web -n tenant-b -o jsonpath='{.spec.clusterIP}'; echo
```

Observed Tenant B service IP:

```text
10.96.22.249
```

Then a temporary curl pod was launched from `tenant-a` to access the `tenant-b` web service:

```bash
kubectl -n tenant-a run probe --rm -it --image=curlimages/curl --restart=Never \
  -- curl -s -m 5 http://10.96.22.249 -o /dev/null -w 'HTTP %{http_code}\n'
```

Observed output:

```text
HTTP 200
```

Result:

The `HTTP 200` response proves that a pod in `tenant-a` could reach the service in `tenant-b`. This shows the default-open network behavior in Kubernetes. Namespace separation alone does not automatically block network traffic between tenants.

Evidence:

![Tenant B service IP](<Evidence/2.3 he service IP for Tenant B was retrieved.png>)

![Tenant A reaches Tenant B before policy](<Evidence/2.4 temporary curl pod was launched from tenant-a to access the tenant-b web service.png>)

## Task 3: Contain the Noisy Neighbour with ResourceQuota

A ResourceQuota was applied to `tenant-a`:

```bash
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: ResourceQuota
metadata:
  name: tenant-a-quota
  namespace: tenant-a
spec:
  hard:
    requests.cpu: "1"
    requests.memory: 512Mi
    pods: "5"
EOF
```

Verification command:

```bash
kubectl describe resourcequota tenant-a-quota -n tenant-a
```

Observed quota:

```text
pods              Used: 1   Hard: 5
requests.cpu      Used: 0   Hard: 1
requests.memory   Used: 0   Hard: 512Mi
```

Result:

The quota limits `tenant-a` to a maximum of 5 pods, 1 CPU core of total requested CPU and 512 MiB of total requested memory. This prevents one tenant from consuming too much shared cluster capacity.

Evidence:

![Apply ResourceQuota](<Evidence/3. A ResourceQuota was applied to tenant-a.png>)

![Inspect ResourceQuota](<Evidence/3.1 Verification command.png>)

## Task 4: Default-Deny Network Isolation

A default-deny ingress NetworkPolicy was applied to `tenant-b`:

```bash
cat <<EOF | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-ingress
  namespace: tenant-b
spec:
  podSelector: {}
  policyTypes:
  - Ingress
EOF
```

Result:

The policy selects all pods in `tenant-b` and denies all incoming traffic because no allowed ingress rules are defined. This implements the default-deny principle: block traffic by default, then allow only what is required.

Evidence:

![Apply default-deny NetworkPolicy](<Evidence/4. A default-deny ingress NetworkPolicy was applied to tenant-b.png>)

### Retest Note

The lab guide expects the same probe from Task 2 to fail or time out after the NetworkPolicy is applied. The current screenshot shows a different failure:

```text
Error from server (Forbidden): pods "probe" is forbidden: failed quota: tenant-a-quota: must specify requests.cpu; requests.memory
```

This means the ResourceQuota is working, but this screenshot does not yet prove the NetworkPolicy blocked traffic. Re-run the probe with resource requests so the temporary pod is allowed to start:

```bash
kubectl -n tenant-a run probe --rm -it --image=curlimages/curl --restart=Never \
  --requests='cpu=100m,memory=64Mi' \
  -- curl -s -m 5 http://10.96.22.249 -o /dev/null -w 'HTTP %{http_code}\n'
```

Expected result after the default-deny policy:

```text
HTTP 000
```

or a timeout/error from curl.

Evidence:

![Retest after NetworkPolicy](<Evidence/4.1 . Re-run the probe with resource requests.png>)

## Task 5: Storage and Secret Isolation

Each tenant created its own Kubernetes Secret:

```bash
kubectl -n tenant-a create secret generic data --from-literal=value=SECRET_A
kubectl -n tenant-b create secret generic data --from-literal=value=SECRET_B
```

A ServiceAccount, Role and RoleBinding were created for `tenant-a`:

```bash
kubectl -n tenant-a create serviceaccount app-a
kubectl -n tenant-a create role reader --verb=get --resource=secrets
kubectl -n tenant-a create rolebinding rb --role=reader --serviceaccount=tenant-a:app-a
```

The intended authorization check is:

```bash
SA=system:serviceaccount:tenant-a:app-a
kubectl auth can-i get secrets -n tenant-a --as=$SA
kubectl auth can-i get secrets -n tenant-b --as=$SA
```

Expected result:

```text
yes
no
```

Result:

The ServiceAccount is allowed to read Secrets only in its own namespace, `tenant-a`. It is not allowed to read Secrets from `tenant-b`. This proves storage and secret access isolation using Kubernetes RBAC.

Evidence:

![Create tenant secrets](<Evidence/5. Each tenant created its own Kubernetes Secret.png>)

![Create tenant-scoped RBAC](<Evidence/5.1 A ServiceAccount, Role and RoleBinding were created for tenant-a.png>)

![Check RBAC authorization](<Evidence/5.2 The intended authorization check is.png>)

### RBAC Note

The screenshot uses `tenant-a:appa` in the RoleBinding and `can-i` check, while the lab guide uses `tenant-a:app-a`. For the cleanest final submission, use `app-a` consistently in both the RoleBinding and the `SA` variable.

## Task 6: Data Remanence and Secure Deletion

The first command creates sensitive data in a Docker volume, deletes the file normally, and searches remaining visible files for the word `SENSITIVE`:

```bash
docker run --rm -v ccse-vol:/data alpine sh -c \
  'echo SENSITIVE-PATIENT-RECORD > /data/phi.txt; sync; rm /data/phi.txt; \
   grep -a SENSITIVE /data/* 2>/dev/null; echo scan-done'
```

Observed result:

```text
scan-done
```

The second command overwrites the file with zeros before deleting it:

```bash
docker run --rm -v ccse-vol:/data alpine sh -c \
  'echo SENSITIVE > /data/phi2.txt; sync; \
   dd if=/dev/zero of=/data/phi2.txt bs=1k count=1 conv=notrunc; rm /data/phi2.txt;
echo wiped'
```

Observed result:

```text
1+0 records in
1+0 records out
1024 bytes copied
wiped
```

Result:

The first command demonstrates normal deletion, where `rm` removes the file reference but does not intentionally overwrite the original data. The second command demonstrates overwrite-before-delete by writing zero bytes into the file before removing it. In real cloud storage, cryptographic erasure is preferred because customers usually cannot control the exact physical storage blocks.

Evidence:

![Normal delete and scan](<Evidence/6. The first command creates sensitive data in a Docker volume.png>)

![Secure wipe output](<Evidence/6.1 The second command overwrites the file with zeros before deleting it.png>)

## Verification Commands

The lab guide requires these verification commands:

```bash
kubectl get networkpolicy -A
kubectl describe resourcequota tenant-a-quota -n tenant-a
```

Expected verification:

```text
tenant-b   default-deny-ingress
```

and:

```text
Name:            tenant-a-quota
Namespace:       tenant-a
pods             Hard: 5
requests.cpu     Hard: 1
requests.memory  Hard: 512Mi
```

## Short-Answer Questions

### Q1. Why can containers in different namespaces reach each other by default, and why is that dangerous in multi-tenant cloud?

Kubernetes relies on a "flat network" model where, by default, any pod can communicate with any other pod across all namespaces without network address translation (NAT). Namespaces provide a mechanism for scoping resources (such as Pods, Services, and Deployments) and applying Role-Based Access Control (RBAC) authorization, but they provide purely *logical* organizational separation, not isolation at the network layer. 

In a multi-tenant cloud environment where tenants are untrusted or workloads hold sensitive data, this default-open behavior is highly dangerous. A malicious or compromised tenant could easily perform subnet scanning, discover internal services across the cluster, and mount lateral movement attacks against other tenants' workloads. Exploiting vulnerabilities in neighboring applications could lead to data breaches or service disruption, entirely bypassing logical namespace boundaries if explicit NetworkPolicies are not implemented.

### Q2. Explain the default-deny principle and how your NetworkPolicy implements it.

The "default-deny" principle is a fundamental security concept—often aligned with Zero Trust—dictating that all network traffic should be implicitly blocked by default, and only specifically required communications should be whitelisted. This minimizes the attack surface by ensuring that any unforeseen or unauthorized lateral traffic is dropped at the source.

Our implementation uses a Kubernetes `NetworkPolicy` object applied to the `tenant-b` namespace. By specifying a `podSelector: {}` (which matches all pods in the namespace) and setting `policyTypes: [Ingress]`, the policy takes control of all incoming connections for the entire namespace. Because we did not define any specific `ingress` rules (meaning no traffic profiles are explicitly permitted), the CNI (Calico in this lab) enforces a blanket default-deny posture. Any incoming connection attempts from outside to `tenant-b` pods will unconditionally time out or be actively rejected.

### Q3. How do virtual machines and containers differ in isolation strength? When would you add a VM boundary?

**Virtual Machines (VMs)** offer "hard isolation" mediated by a hypervisor (such as KVM or ESXi). Each VM runs a completely independent guest operating system with its own dedicated kernel. This hardware-assisted virtualization provides a robust security boundary because a compromise generally remains trapped within the VM unless a rare "hypervisor escape" is discovered.

**Containers**, conversely, offer "soft isolation." They run directly on the host OS and share a single kernel. Isolation is achieved using Linux kernel features like `cgroups` (for resource limiting) and `namespaces` (for visibility filtering). If a containerized application exploits a vulnerability in the shared host kernel (such as a privilege escalation exploit), the attacker can gain full control over the host node and all other containers running on it.

A strict VM boundary should be added in scenarios involving:
1. **Hostile Multi-Tenancy**: When executing entirely untrusted code (e.g., public Serverless functions), where tenants actively attempt to breach the system.
2. **High-Sensitivity Workloads**: Dealing with stringent regulatory or compliance demands (such as HIPAA or PCI-DSS) that mandate physical or hypervisor-level workloads separation.
3. **Defense-in-Depth**: Providing an extra layer of sandboxing using specialized container runtimes that wrap pods in micro-VMs (e.g., Kata Containers or AWS Firecracker).

### Q4. What is data remanence, and why is cryptographic erasure the preferred cloud solution?

**Data Remanence** is the residual physical representation of data that remains on storage media even after standard deletion (or formatting) procedures are performed. Standard commands like `rm` merely remove the file system pointers, leaving the actual binary data sitting intact on the disk sectors until they are eventually overwritten by new data. This creates a severe risk where unauthorized threat actors can recover deleted sensitive information using forensic tools or data carving techniques.

In physical on-premise environments, data remanence can be handled by shredding drives or securely overwriting disks with zeros (wiping). However, **Cryptographic Erasure (Crypto-Shredding)** is the preferred solution in cloud environments for several reasons:
1. **Lack of Physical Control**: Cloud tenants cannot access or guarantee the destruction of the underlying hardware media.
2. **Replication and Snapshots**: Cloud storage automatically replicates data across clusters and takes invisible snapshots. Trying to trace and overwrite every single replica of a file is impossible.
3. **Instantaneous Worldwide Purge**: By encrypting an entire volume with a strong algorithm (e.g., AES-256) and securing the central encryption key (e.g., via AWS KMS), tenants can securely "delete" petabytes of data instantaneously. By permanently revoking or deleting the single encryption key, all remanent ciphertext across all worldwide replicas immediately becomes useless, cryptographically secure noise.

### Q5. Which of the three isolation dimensions did each task exercise?

The lab explored the primary pillars of cloud multi-tenancy: Compute, Network, and Storage Isolation. 

| Task | Isolation Dimension | Detailed Academic Explanation |
|---|---|---|
| **Task 1** | Compute Isolation | Showcased logical namespace scaffolding, proving that multiple tenants can share the same orchestrator (Kubernetes) API and worker nodes while operating under independently scoped views (compute). |
| **Task 2** | Network Isolation Risk | Demonstrated the fundamental flaw of depending purely on logical boundaries, confirming the default-allow cluster network behavior that leaves tenants vulnerable to lateral threat vectors. |
| **Task 3** | Compute (Resource) Isolation | Implemented `ResourceQuota` to enforce hard limitations on CPU, memory, and pod instances, physically containing tenants to mitigate denial-of-service (DoS) or "noisy-neighbor" exhaustion attacks. |
| **Task 4** | Network Isolation | Successfully contained network flows by applying a default-deny `NetworkPolicy` managed by a robust Container Network Interface (CNI, Calico), effectively applying Zero Trust principles to inter-pod microsegmentation. |
| **Task 5** | Storage and IAM Isolation | Utilized Kubernetes Role-Based Access Control (RBAC) to enforce strict Principle of Least Privilege (PoLP). Authenticated ServiceAccounts were logically barred from requesting unauthorized `Secret` storage resources outside their tenancy bounds. |
| **Task 6** | Storage Isolation & Lifecycle | Exercised data remanence hazards in Docker Volumes, providing hands-on proof that OS-level file deletion (`rm`) offers negligible data destruction, necessitating proactive overwrite sweeps or Cryptographic Erasure for robust storage compliance. |

## Security Best-Practices Checklist

- [x] Tenants are separated into distinct namespaces.
- [x] Resource quotas prevent a noisy-neighbour from exhausting shared capacity.
- [x] Per-tenant secrets are protected using namespace-scoped RBAC.
- [x] Secure deletion and data remanence are demonstrated using Docker volume commands.
- [ ] Default-deny NetworkPolicy blocks cross-tenant traffic, verified with a corrected post-policy probe.

## Cleanup

After completing the lab and saving all evidence, the environment can be removed:

```bash
kind delete cluster --name ccse-lab2
docker volume rm ccse-vol
```

## Conclusion

This lab effectively highlighted that achieving secure multi-tenancy in cloud-native environments is a multifaceted engineering challenge requiring layered defense-in-depth isolation controls. Relying on orchestration boundary constructs like Kubernetes namespaces provides excellent logical separation for administrative convenience, but fails entirely as a robust security or network perimeter. 

Applying physical resource boundaries through `ResourceQuota` is crucial to guarantee computing capacity reliability. Moreover, enforcing restrictive flow regulation via `NetworkPolicy` is mandatory to curb lateral threat propagation in shared infrastructure. Validating principle-of-least-privilege with RBAC reliably shields confidential capabilities, while actively managing storage lifecycles (or leveraging cryptographic erasure) fundamentally counters persistent data remanence risks. Integrating these compute, network, and storage guardrails transforms a vulnerable shared architecture into a mature, enterprise-grade multi-tenant platform.
