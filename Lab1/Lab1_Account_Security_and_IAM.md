## Course Information
---
*Course:* IKB42603 Cloud Computing Security Essentials
*Lab:* Lab1_Account_Security_and_IAM
*Name:* MUHAMMAD AMEER BIN IDRIS
*Student ID:* 52215124748
*Date:* 4 August 2026

# Lab 1: Cloud Account Security, Identity and Access Management

## Lab Summary // Objective

This lab demonstrated account security and access control using two local platforms:

- **LocalStack IAM** was used to simulate AWS IAM users, groups, policies and access keys.
- **Kubernetes RBAC** was used to enforce real authorization decisions using roles and role bindings.

## Evidence

All screenshots used for this report are stored in the same directory.

| Evidence File | Purpose |
|---|---|
| `1. start localstack.png` | Start LocalStack environment |
| `2. confirm localstack is healty.png` | Verify LocalStack is running healthy |
| `3.create shell variable.png` | Create endpoint shell variable for AWS CLI |
| `4. see all listed policies attached.png` | View all available IAM policies |
| `5. aws lkist attached group policies.png` | Verify AdministratorAccess policy is attached |
| `6. create your personal admin user.png` | Create personal admin user |
| `7. put user in the group.png` | Add personal admin user to Admins group |
| `8. verify the membership.png` | Verify personal admin user membership in Admins group |
| `9. create read only user.png` | Create read-only Analyst user |
| `10. attach a scoped , read only policy.png` | Attach AmazonS3ReadOnlyAccess policy |
| `11. list what the user can do.png` | Verify attached policies for the Analyst user |
| `12. create an access key for the ananlyst.png` | Create access key for the Analyst user |
| `13. list access key.png` | Verification of created access key |
| `14. Create a throwaway cluster.png` | Create a local kind Kubernetes cluster |
| `15. Confirm it is up 1.png` | Get Kubernetes cluster info |
| `16. Confirm it is up 2.png` | Get Kubernetes cluster nodes |
| `17. Separate Environments with Namespaces.png` | Create and list active namespaces (`dev` and `prod`) |
| `18. Create Service Account.png` | Create service account `dev-user` |
| `19. Create Pod Reader Role.png` | Create role `pod-reader` |
| `20. Create RoleBinding.png` | Create RoleBinding |
| `21. test access control.png` | Set up for testing RBAC |
| `22. list pod in dev.png` | Test listing pods in dev |
| `23. delete pods in dev.png` | Test deleting pods in dev |
| `24. list pods in prod.png` | Test accessing prod namespace |
| `25. RBAC verification command.png` | View RoleBinding definition in YAML |

## Task 1: Map the Cloud Identity Landscape

| Concept | AWS Term | Purpose |
|---|---|---|
| All-powerful owner | Root user | The original account owner with full control over all resources and billing. It should be protected and not used for daily administration. |
| Human/app identity | IAM User | A named identity for a person, application or service that needs credentials to access cloud resources. |
| Permission bundle | IAM Policy | A JSON permission document that defines which actions are allowed or denied on specific resources. |
| Collection of users | IAM Group | A way to manage permissions for multiple users together by attaching policies to the group. |
| Temporary identity | IAM Role | An identity that can be assumed temporarily to grant short-lived permissions without long-term user credentials. |

## Session A: LocalStack IAM

### Environment Setup

The AWS CLI was pointed to LocalStack using:

```bash
EP='--endpoint-url=http://localhost:4566'
```

This means AWS CLI commands were sent to the local LocalStack endpoint instead of real AWS.

Evidence:
start localstack:

<img width="510" height="76" alt="1  start localstack" src="https://github.com/user-attachments/assets/54353e8b-5cce-439a-9d24-7046af9a7c3d" />

confirm localstack is healty:

<img width="856" height="282" alt="2  confirm localstack is healty" src="https://github.com/user-attachments/assets/b8e0408b-93cc-4a31-86a3-c4a6697faee7" />

create shell variable:

<img width="552" height="65" alt="3 create shell variable" src="https://github.com/user-attachments/assets/6be96987-99f0-4e6f-9895-927db3869250" />


Verification command:

```bash
aws --endpoint-url=http://localhost:4566 sts get-caller-identity
```

Output:

```json
{
    "UserId": "000000000000",
    "Account": "000000000000",
    "Arn": "arn:aws:iam::000000000000:root"
}
```

The account ID `000000000000` confirms the commands were executed against LocalStack.

## Task 2: Create a Least-Privilege Admin

### Step 2.1: Create Admins Group

Command:

```bash
aws $EP iam create-group --group-name Admins
```

Result:

The group `Admins` was created successfully.

### Step 2.2: Attach Administrator Policy to Group

Command:

```bash
aws $EP iam attach-group-policy --group-name Admins \
    --policy-arn arn:aws:iam::aws:policy/AdministratorAccess
```

Evidence:
see all listed policies attached:

<img width="595" height="222" alt="4  see all listed policies attached" src="https://github.com/user-attachments/assets/764b75c4-8525-4374-88d6-25b41b85b90b" />


Verification command:

```bash
aws $EP iam list-attached-group-policies --group-name Admins
```

Output:

```json
{
    "AttachedPolicies": [
        {
            "PolicyName": "AdministratorAccess",
            "PolicyArn": "arn:aws:iam::aws:policy/AdministratorAccess"
        }
    ]
}
```

This proves that the `AdministratorAccess` policy was attached to the `Admins` group.

Evidence:
aws list attached group policies:

<img width="615" height="97" alt="5  aws lkist attached group policies" src="https://github.com/user-attachments/assets/20de862f-0cea-4251-8c7a-aa3fa7332258" />



### Step 2.3: Create Personal Admin User

Command:

```bash
aws $EP iam create-user --user-name CloudAdmin_dani
```
*(User identity may vary based on student setup, but it functions as a personal admin)*

Result:

The user was created successfully.

Evidence:
create your personal admin user:

<img width="720" height="236" alt="6  create your personal admin user" src="https://github.com/user-attachments/assets/ee14a0e2-805e-4ef4-ae31-8248cd22595d" />


### Step 2.4: Add User to Admins Group and Verify Membership

Command:

```bash
aws $EP iam add-user-to-group --group-name Admins \
    --user-name CloudAdmin_dani
```

Evidence:
put user in the group:

<img width="596" height="85" alt="7  put user in the group" src="https://github.com/user-attachments/assets/8661672d-caac-408c-abb6-6353d8a68ddb" />


Verification command:

```bash
aws $EP iam get-group --group-name Admins
```

Output summary:

```json
{
    "Users": [
        {
            "UserName": "CloudAdmin_dani",
            "Arn": "arn:aws:iam::000000000000:user/CloudAdmin_dani"
        }
    ],
    "Group": {
        "GroupName": "Admins",
        "Arn": "arn:aws:iam::000000000000:group/Admins"
    }
}
```

This proves that the personal admin is a member of the `Admins` group. The admin permission is inherited from the group rather than attached directly to the user.

Evidence:
verify the membership:

<img width="790" height="417" alt="8  verify the membership" src="https://github.com/user-attachments/assets/3ff19975-d065-48c9-b2c0-0885c10c82f0" />


## Task 3: Enforce Least Privilege with a Scoped Policy

### Step 3.1: Create Read-Only Analyst User

Command:

```bash
aws $EP iam create-user --user-name Analyst_jiha
```

Result:

The user was created successfully.

Evidence:
create read only user:

<img width="673" height="242" alt="9  create read only user" src="https://github.com/user-attachments/assets/738bb07c-1d0c-4b19-9ce3-48f3d5f41c90" />


### Step 3.2: Attach S3 Read-Only Policy

Command:

```bash
aws $EP iam attach-user-policy --user-name Analyst_jiha \
    --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess
```

Evidence:
attach a scoped , read only policy:

<img width="685" height="83" alt="10  attach a scoped , read only policy" src="https://github.com/user-attachments/assets/f79c59af-82bc-41b7-83c4-30043bf82a37" />


### Step 3.3: Verify Analyst Permissions

Verification command:

```bash
aws $EP iam list-attached-user-policies --user-name Analyst_jiha
```

Output:

```json
{
    "AttachedPolicies": [
        {
            "PolicyName": "AmazonS3ReadOnlyAccess",
            "PolicyArn": "arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess"
        }
    ]
}
```

This proves that the analyst only has the `AmazonS3ReadOnlyAccess` policy attached.

Evidence:
list what the user can do:

<img width="778" height="213" alt="11  list what the user can do" src="https://github.com/user-attachments/assets/f277f800-3b9b-4659-8460-a76ad66ef6cd" />

### Least Privilege Explanation

- **Containment of Compromise**: If the `Analyst_jiha` account credentials were stolen or inadvertently exposed, the potential damage to the infrastructure would be strictly contained. Because the account is explicitly granted only the `AmazonS3ReadOnlyAccess` policy, the unauthorized party is confined to viewing and retrieving S3 bucket data.
- **Prevention of Privilege Escalation**: The attacker would not possess inherent administrative privileges. Without IAM modification rights, they are unable to create new privileged users, alter existing IAM policies, or grant themselves access to other critical infrastructure components like EC2 instances, RDS databases, or VPC networks.
- **Minimization of Blast Radius**: This architectural design effectively minimizes the "blast radius" or the extent of damage from a security breach. By enforcing least privilege, the compromised identity is restricted strictly to the explicit actions granted by its meticulously scoped policy, thereby preserving the integrity and availability of the broader cloud environment.
- **Protection Against Data Modification and Destruction**: Due to the read-only constraint, an adversary is fundamentally restricted from modifying, overwriting, or deleting crucial data assets within S3, maintaining the data's integrity against potential sabotage or ransomware attempts.

## Task 4: Credential Hygiene and Access Keys

### Step 4.1: Create Access Key

Command:

```bash
aws $EP iam create-access-key --user-name Analyst_jiha
```

Result:

An access key was created for the analyst.

Evidence:
create an access key for the ananlyst:

<img width="730" height="246" alt="12  create an access key for the ananlyst" src="https://github.com/user-attachments/assets/70ebb17b-c862-4ce0-901c-430ad722e494" />


Security note: the secret access key is not repeated in this report. In real cloud environments, access keys must not be committed to repositories, shared in screenshots or stored in plaintext.

### Step 4.2: List Access Keys

Command:

```bash
aws $EP iam list-access-keys --user-name Analyst_jiha
```

Output:

```json
{
    "AccessKeyMetadata": [
        {
            "UserName": "Analyst_jiha",
            "AccessKeyId": "LKIAQAAAAAAANMJV6XA3",
            "Status": "Inactive",
            "CreateDate": "2026-07-29T05:29:06.789002+00:00"
        }
    ]
}
```

Evidence:
list access key:

<img width="732" height="265" alt="13  list access key" src="https://github.com/user-attachments/assets/0424accf-ce53-4ab5-8f99-a5164d24a031" />

### Step 4.3: Rotate and Deactivate Old Key

Command:

```bash
aws $EP iam update-access-key --user-name Analyst_jiha \
    --access-key-id LKIAQAAAAAAANMJV6XA3 --status Inactive
```

Result:

The access key status is now `Inactive`, which demonstrates key rotation/deactivation.

## Session B: Kubernetes RBAC

### Setup: Create Local Kubernetes Cluster

Command:

```bash
kind create cluster --name ccse-lab1
kubectl cluster-info --context kind-ccse-lab1
kubectl get nodes
```

Result:

The local kind cluster `ccse-lab1` was created and kubectl was configured to use context `kind-ccse-lab1`.

Evidence:
Create a throwaway cluster:

<img width="857" height="335" alt="14  Create a throwaway cluster" src="https://github.com/user-attachments/assets/12aa1ab3-5dae-4fab-807e-db8e9c00b3d3" />

Confirm it is up 1:

<img width="865" height="113" alt="15  Confirm it is up 1" src="https://github.com/user-attachments/assets/baa355f5-b95a-426e-bea0-c18e85127b67" />

Confirm it is up 2:

<img width="865" height="113" alt="16  Confirm it is up 2" src="https://github.com/user-attachments/assets/d8d83674-a94e-4816-b850-2c35bceb8753" />

## Task 5: Separate Environments with Namespaces

Commands:

```bash
kubectl create namespace dev
kubectl create namespace prod
kubectl get namespaces
```

Result:

The namespaces `dev` and `prod` were created and listed as `Active`.

Evidence:
Separate Environments with Namespaces:

<img width="855" height="286" alt="17  Separate Environments with Namespaces" src="https://github.com/user-attachments/assets/47a14672-b4a7-4305-885b-1c7b1503eacf" />

## Task 6: Define a Role and Bind It

### Step 6.1: Create Service Account

Command:

```bash
kubectl create serviceaccount dev-user -n dev
```

Result:

The service account `dev-user` was created in the `dev` namespace.

Evidence:
Create Service Account:

<img width="862" height="73" alt="18  Create Service Account" src="https://github.com/user-attachments/assets/2a56848b-21e5-4cd1-b3a4-d498ec1138d4" />

### Step 6.2: Create Pod Reader Role

Command:

```bash
kubectl create role pod-reader -n dev \
    --verb=get,list,watch --resource=pods
```

Result:

The Role `pod-reader` was created in the `dev` namespace. It allows only `get`, `list` and `watch` actions on pods.

Evidence:
Create Pod Reader Role:

<img width="860" height="92" alt="19  Create Pod Reader Role" src="https://github.com/user-attachments/assets/3d1d16b4-e5b5-46c2-b198-66c7b6f4b747" />

### Step 6.3: Create RoleBinding

Command:

```bash
kubectl create rolebinding dev-user-binding -n dev \
    --role=pod-reader --serviceaccount=dev:dev-user
```

Result:

The RoleBinding `dev-user-binding` binds the `pod-reader` Role to the `dev-user` service account.

Evidence:
Create RoleBinding:

<img width="862" height="97" alt="20  Create RoleBinding" src="https://github.com/user-attachments/assets/4e7e7fca-4e40-4736-b515-f19f1b0a71f3" />


## Task 7: Test Access Control

The service account identity was stored in a shell variable:

```bash
SA=system:serviceaccount:dev:dev-user
```

This represents the Kubernetes service account `dev-user` in the `dev` namespace.

Evidence:
test access control:

<img width="861" height="57" alt="21  test access control" src="https://github.com/user-attachments/assets/6241ef24-df61-444f-b074-0ce02d3bc66b" />

### Test 1: List Pods in Dev

Command:

```bash
kubectl auth can-i list pods -n dev --as=$SA
```

Result:

```text
yes
```

Explanation:

The service account can list pods in `dev` because the `pod-reader` Role allows `list` on pods in the `dev` namespace.

Evidence:
list pod in dev:
<img width="867" height="62" alt="22  list pod in dev" src="https://github.com/user-attachments/assets/04e34b6a-93dd-4053-a19a-4e3ee4b4a8c0" />

### Test 2: Delete Pods in Dev

Command:

```bash
kubectl auth can-i delete pods -n dev --as=$SA
```

Result:

```text
no
```

Explanation:

The service account cannot delete pods because the Role only grants `get`, `list` and `watch`. Delete permission was not granted.

Evidence:
delete pods in dev:

<img width="861" height="72" alt="23  delete pods in dev" src="https://github.com/user-attachments/assets/f6a36e64-17ab-4b22-a28c-66130ff8082c" />

### Test 3: List Pods in Prod

Command:

```bash
kubectl auth can-i list pods -n prod --as=$SA
```

Result:

```text
no
```

Explanation:

The service account cannot list pods in `prod` because the Role and RoleBinding are namespaced to `dev`. The permission does not extend to the `prod` namespace.

Evidence:
list pods in prod:

<img width="863" height="66" alt="24  list pods in prod" src="https://github.com/user-attachments/assets/adb3fe7a-2184-4902-9afa-a1c050743852" />

### Authentication vs Authorization

**Authentication** is the process of verifying identity. In this lab, the service account successfully passes the authentication phase because the Kubernetes API server validates and recognizes the identity as `system:serviceaccount:dev:dev-user` based on its presented token.

**Authorization**, conversely, determines what the authenticated identity is actually permitted to do. After successful authentication, the requested actions (e.g., executing a command) are intercepted by the Kubernetes Role-Based Access Control (RBAC) authorization module. 
- **Authorized Actions**: Listing pods within the `dev` namespace is permitted because the explicit `dev-user-binding` RoleBinding explicitly grants the `pod-reader` Role permissions to this user. 
- **Unauthorized Actions**: Attempting to delete pods in the `dev` namespace, or attempting to list pods in the `prod` namespace, are forcefully blocked by the authorization module. In a true least-privilege RBAC system, anything not explicitly allowed is denied by default. Since no permissions were ever provisioned extending to the `prod` namespace or authorizing destructive actions like deletion, the API server returns a definitive 'forbidden' response.

## RBAC Verification Command

Required verification command:

```bash
kubectl get rolebinding dev-user-binding -n dev -o yaml
```

Output:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  creationTimestamp: "2026-07-29T05:48:38Z"
  name: dev-user-binding
  namespace: dev
  resourceVersion: "701"
  uid: 91124053-fdc5-418a-a916-ec078374971c
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: pod-reader
subjects:
- kind: ServiceAccount
  name: dev-user
  namespace: dev
```

Evidence:

RBAC verification command:

<img width="857" height="362" alt="25  RBAC verification command" src="https://github.com/user-attachments/assets/62805017-3c13-4fa1-b471-999842d59df9" />

This confirms that the `dev-user-binding` RoleBinding connects the `dev-user` service account to the `pod-reader` Role in the `dev` namespace.

## Short-Answer Questions

### Q1. Why is attaching policies to groups better than attaching them directly to users?

Attaching policies to IAM groups instead of individual users is considered a cloud security best practice for several crucial reasons. Firstly, it drastically simplifies identity governance, administration, and auditing. In enterprise environments where numerous individuals belong to the same organizational department or role (e.g., developers, analysts), attaching policies at the group level means permissions only need to be defined once. Whenever access requirements change, updating the group policy cascades the updated permissions instantly and uniformly to all group members. Secondly, it reduces the risk of human error (such as "permission drift," where users accidentally retain legacy permissions) by preventing localized, ad-hoc policy attachments that are difficult to track. Group-level assignments ensure a standardized, scalable, and highly maintainable access control framework.

### Q2. What is the difference between an IAM User and an IAM Role?

An **IAM User** represents a specific, long-term identity (often representing a human operator, an admin, or an application). They traditionally rely on long-lived security credentials such as console passwords or static access keys. Because these credentials persist until manually rotated or revoked, they present a higher risk of compromise if inadvertently leaked.

An **IAM Role**, by contrast, is an assumable, non-permanent identity that does not have long-term credentials associated with it. Instead, trusted entities (like federated users, AWS services, or applications) temporarily "assume" the role and are dynamically issued short-lived STS (Security Token Service) credentials. Relying on roles is considerably safer for cloud workloads because it entirely mitigates the risk of hardcoded, permanent access keys and rigorously enforces the principle of granting permissions only for a definitive, necessary timeframe.

### Q3. Explain least privilege using the Analyst account, and how it reduces blast radius if compromised.

The Principle of Least Privilege dictates that an entity must only be granted the minimum necessary permissions required to fulfill their designated job function. The `Analyst_jiha` account perfectly illustrates this principle by being strictly constrained to the `AmazonS3ReadOnlyAccess` managed policy, rather than broader administrative permissions. If a malicious actor successfully compromised this account's access key, the resulting "blast radius" (the span of potential impact of a cybersecurity incident) is aggressively contained. The adversary is technologically restricted to performing read-only queries against S3 buckets. They are expressly barred from performing high-impact, destructive operations—such as permanently deleting cloud resources, corrupting operational databases, exfiltrating non-S3 data, or escalating their privileges by modifying underlying IAM policies.

### Q4. In Kubernetes, what is the difference between a Role and a RoleBinding?

In Kubernetes RBAC architecture, authorization is split between defining permissions and assigning them. A **Role** essentially represents a discrete set of standardized permissions (a rule set); it specifically defines exactly *what* actions (verbs like `get`, `list`, `watch`, `create`, `delete`) are allowed on *which* specific cluster resources (like `pods`, `services`, or `deployments`) within the boundaries of a single namespace. 

A **RoleBinding**, on the other hand, performs the association—it officially binds the defined Role, determining *who* operates with those permissions. It maps the designated Role directly to specific "subjects," which can be human users, user groups, or automated ServiceAccounts. In this practical lab, the `pod-reader` Role conceptually bundles the pod-monitoring permissions, whereas the `dev-user-binding` RoleBinding functionally grants that exact bundle strictly to the `dev-user` service account.

### Q5. Why did the developer service account fail to access prod, and which security principle does that demonstrate?

The developer service account forcefully failed to interact with the `prod` namespace because standard Kubernetes RBAC defaults to an implicit denial of all requests. The user's Role and corresponding RoleBinding were explicitly deployed and localized strictly within the purview of the `dev` namespace. Because RBAC permissions are intrinsically namespace-scoped, zero authorization rules existed granting this identity any visibility or control over `prod` resources. This practical failure strongly demonstrates both the **Principle of Least Privilege** (providing sufficient access to perform testing in development without extraneous access) as well as the **Principle of Separation of Environments/Duties**. By strictly segregating logical environments, an organization actively ensures that a compromise or human error within a development sector cannot inadvertently spill over and disrupt critical production workloads.

## Security Best-Practices Checklist

- [x] Root user is not used for daily tasks because a dedicated admin identity exists.
- [x] Permissions are granted through the `Admins` group instead of attaching administrator permissions directly to the admin user.
- [x] A least-privilege read-only identity was created and assigned `AmazonS3ReadOnlyAccess`.
- [x] Access keys were created, listed and deactivated to demonstrate rotation.
- [x] Kubernetes RBAC blocked unauthorized actions: deleting pods in `dev` and listing pods in `prod`.

## Conclusion

This comprehensive laboratory reliably modeled fundamental cloud identity and access governance, effectively demonstrating how rigorous identity management and the Principle of Least Privilege are operationally enforced. 

In the AWS-simulated LocalStack IAM environment, we exhibited modern architectural best practices by provisioning robust administrative rights procedurally through an IAM Group, deliberately preventing direct policy attachments to isolated users. The implementation of an Analyst profile tightly restricted to `AmazonS3ReadOnlyAccess` showcased how reducing excess permissions radically contains the blast radius of hypothetical system breaches. Additionally, the lab emphasized critical credential hygiene and lifecycle management by generating, auditing, and subsequently rotating (deactivating) IAM access keys, closing potential vectors for long-term credential leakage.

In the concurrent Session B, Kubernetes Role-Based Access Control (RBAC) was heavily leveraged to strictly govern intra-cluster authentication and authorization. Real-world validation verified that the `dev-user` service account successfully passed identity authentication but faced stringent, resource-specific authorization protocols. The user successfully executed non-destructive operations (listing pods) localized entirely to the `dev` namespace, yet was actively blocked from executing destructive directives (deleting pods) or trespassing into isolated administrative boundaries (the `prod` namespace). These verified restrictions decisively proved that modern RBAC configurations, when correctly mapped via precise Roles and RoleBindings, uphold uncompromising least privilege architectures and vital environment separation.
