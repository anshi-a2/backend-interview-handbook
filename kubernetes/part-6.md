# 06_Storage_Security_Configuration.md

> **Goal**
>
> Learn how Kubernetes manages persistent storage, application configuration, secrets, security, authentication, authorization, and production best practices.

---

# Table of Contents

1. Why Storage & Configuration?
2. Volumes
3. Persistent Volumes (PV)
4. Persistent Volume Claims (PVC)
5. StorageClass
6. ConfigMaps
7. Secrets
8. Environment Variables vs Volume Mounts
9. Service Accounts
10. Authentication
11. Authorization (RBAC)
12. Security Context
13. Network Policies
14. Production Best Practices
15. Common Production Issues
16. Debugging Commands
17. Interview Summary

---

# 1. Why Storage & Configuration?

Containers are **ephemeral**.

Suppose your application stores uploaded files inside the container.

```
Container

↓

uploads/

↓

Container Crashes

↓

New Container

↓

uploads/ Missing
```

Everything stored inside the container is lost.

Similarly,

hardcoding values like

```
Database Password

Redis URL

Kafka URL
```

inside code is not scalable.

Kubernetes solves these problems using

- Volumes
- Persistent Volumes
- ConfigMaps
- Secrets

---

# 2. Volumes

A Volume provides shared storage for containers inside a Pod.

Example

```
Pod

├── Spring Boot
├── Fluent Bit
└── Shared Volume
```

Both containers can read/write the same files.

Common volume types

- emptyDir
- hostPath
- ConfigMap
- Secret
- Persistent Volume

---

## emptyDir

Created when Pod starts.

Deleted when Pod dies.

Use cases

- Temporary cache
- Intermediate files
- Shared files between containers

---

## hostPath

Mounts a directory from the worker node.

```
Worker Node

↓

/data

↓

Mounted into Pod
```

Not recommended in production because Pods may move to another node.

---

# 3. Persistent Volume (PV)

A Persistent Volume represents **actual storage**.

Examples

- AWS EBS
- Azure Disk
- Google Persistent Disk
- NFS
- SAN

Think of PV as

```
Hard Disk

↓

Managed by Kubernetes
```

A PV exists independently of Pods.

---

# 4. Persistent Volume Claim (PVC)

Applications should not directly request storage devices.

Instead,

they request storage through a PVC.

Flow

```
Application

↓

PVC

↓

PV

↓

Storage
```

Example

```
Need

20 GB SSD
```

PVC requests it.

Kubernetes binds a matching PV.

---

## Why PVC?

Developer doesn't care whether storage comes from

- AWS
- Azure
- Local SSD
- NFS

Only requests

```
20 GB

ReadWriteOnce
```

---

# 5. StorageClass

StorageClass enables **dynamic provisioning**.

Without StorageClass

```
Admin Creates PV

↓

Developer Creates PVC

↓

Binding
```

With StorageClass

```
PVC

↓

StorageClass

↓

Automatically Creates PV
```

Production clusters almost always use StorageClasses.

---

# Access Modes

| Mode | Meaning |
|------|----------|
| ReadWriteOnce (RWO) | One node can read/write |
| ReadOnlyMany (ROX) | Multiple nodes read only |
| ReadWriteMany (RWX) | Multiple nodes read/write |

---

# 6. ConfigMaps

ConfigMaps store **non-sensitive configuration**.

Examples

```
Database URL

Redis Host

Kafka Topic

Application Properties
```

Instead of rebuilding Docker images,

configuration changes independently.

---

Example

```
ConfigMap

↓

application.properties

↓

Mounted inside Pod
```

---

Benefits

- Environment-specific configs
- No image rebuild
- Easier deployments

---

# 7. Secrets

Secrets store sensitive information.

Examples

```
Passwords

API Keys

JWT Secret

TLS Certificates

OAuth Tokens
```

Secrets are stored separately from application code.

Can be injected as

- Environment variables
- Mounted files

---

Important

Secrets are only **Base64 encoded**, not encrypted by default.

For production

Enable

```
Encryption at Rest
```

inside etcd.

---

# ConfigMap vs Secret

| ConfigMap | Secret |
|------------|---------|
| Non-sensitive | Sensitive |
| Plain text | Base64 encoded |
| Public configs | Passwords, Tokens |

---

# 8. Environment Variables vs Volume Mounts

## Environment Variables

Example

```
DB_HOST

DB_PORT

REDIS_HOST
```

Application reads them during startup.

Good for

- Small configuration

---

## Volume Mount

Entire configuration files mounted.

Example

```
application.yml

logback.xml

nginx.conf
```

Useful when configuration is large.

---

# 9. Service Accounts

Pods communicate with Kubernetes using Service Accounts.

Every Pod gets one.

```
Pod

↓

Service Account

↓

API Server
```

Example

Application wants

```
Read ConfigMaps
```

It authenticates using its Service Account.

---

# 10. Authentication

Authentication answers

```
Who are you?
```

Methods

- Client Certificate
- Bearer Token
- Service Account
- OIDC
- Cloud IAM

Example

```
kubectl

↓

Certificate

↓

API Server
```

---

# 11. Authorization (RBAC)

Authorization answers

```
What can you do?
```

Kubernetes primarily uses

```
RBAC

(Role Based Access Control)
```

---

RBAC Components

```
Role

↓

RoleBinding

↓

User / Service Account
```

---

Example

Developer

Can

- Read Pods

Cannot

- Delete Nodes

---

ClusterRole

Works across entire cluster.

Role

Works within one namespace.

---

# 12. Security Context

Defines security settings for Pods.

Examples

```
Run as non-root

↓

Read-only filesystem

↓

Linux Capabilities

↓

Privilege Escalation
```

Production recommendation

Never run applications as root.

---

Useful fields

- runAsUser
- fsGroup
- readOnlyRootFilesystem
- allowPrivilegeEscalation

---

# 13. Network Policies

By default

Every Pod can communicate with every Pod.

Sometimes this is unsafe.

Example

```
Frontend

↓

Backend

↓

Database
```

Database should not be accessible from every Pod.

Network Policy restricts communication.

Example

```
Allow

Frontend

↓

Backend

Deny

Frontend

↓

Database
```

Useful for Zero Trust networking.

---

# 14. Production Best Practices

## Storage

✔ Use StorageClasses

✔ Avoid hostPath

✔ Backup Persistent Volumes

---

## Configuration

✔ Keep configs in ConfigMaps

✔ Keep secrets in Secrets

✔ Never hardcode passwords

---

## Security

✔ Enable RBAC

✔ Use Service Accounts

✔ Run containers as non-root

✔ Enable Encryption at Rest

✔ Apply Network Policies

✔ Scan container images

---

# 15. Common Production Issues

## PVC Pending

Reasons

- No matching StorageClass
- Storage unavailable
- Incorrect access mode

---

## Secret Missing

Application fails during startup.

Check

```
kubectl describe pod
```

---

## ConfigMap Updated

Running Pods do not automatically reload configuration.

Usually requires

```
Deployment Restart
```

or

Spring Cloud Kubernetes / Config Reload mechanism.

---

## Permission Denied

Usually

RBAC issue.

Check

```
Role

RoleBinding

ClusterRole
```

---

## Volume Mount Failure

Possible reasons

- Missing PVC
- Storage backend unavailable
- Incorrect mount path

---

# 16. Debugging Commands

Storage

```bash
kubectl get pv

kubectl get pvc
```

Storage Details

```bash
kubectl describe pvc
```

ConfigMaps

```bash
kubectl get configmap
```

Secrets

```bash
kubectl get secret
```

Describe Secret

```bash
kubectl describe secret
```

RBAC

```bash
kubectl auth can-i create deployment

kubectl auth can-i get pods
```

Service Accounts

```bash
kubectl get serviceaccounts
```

Network Policies

```bash
kubectl get networkpolicy
```

---

# Production Debugging Flow

### PVC Pending

```
PVC

↓

StorageClass

↓

PV

↓

Storage Backend
```

---

### Application Cannot Read Secret

Check

```
Secret Exists?

↓

Mounted?

↓

Correct Key?

↓

Correct Namespace?
```

---

### Access Denied

Check

```
Service Account

↓

Role

↓

RoleBinding

↓

RBAC
```

---

### Database Not Reachable

Check

```
Service

↓

DNS

↓

Network Policy

↓

Application Logs
```

---

# Interview Summary

## Explain PV and PVC

"A Persistent Volume (PV) represents the actual storage resource available to the cluster, while a Persistent Volume Claim (PVC) is a request for storage made by an application. The application only interacts with the PVC, and Kubernetes binds it to a suitable PV. This abstraction allows the same application to work across different storage providers without code changes."

---

## Explain ConfigMap vs Secret

"A ConfigMap stores non-sensitive configuration such as database hosts, feature flags, or application properties. A Secret stores sensitive data such as passwords, API keys, and certificates. Both can be mounted as files or injected as environment variables, but Secrets should additionally be protected using etcd encryption and proper RBAC."

---

## Explain RBAC

"RBAC controls what authenticated users or Service Accounts can do within a Kubernetes cluster. Authentication verifies identity, while authorization determines permissions. Roles define permissions within a namespace, ClusterRoles define cluster-wide permissions, and RoleBindings associate these permissions with users or Service Accounts."

---

# Key Takeaways

- Containers are **ephemeral**; persistent data should use **Persistent Volumes**.
- **PVCs** abstract storage requests from physical storage implementations.
- **StorageClasses** enable dynamic storage provisioning.
- **ConfigMaps** store non-sensitive configuration.
- **Secrets** store sensitive information and should be encrypted at rest.
- **Service Accounts** provide identities for Pods when communicating with the Kubernetes API.
- **Authentication** verifies identity; **RBAC** authorizes actions.
- **Security Contexts** enforce secure container execution.
- **Network Policies** restrict Pod-to-Pod communication for better security.
- Production clusters should follow the principle of **least privilege**, separating configuration, secrets, storage, and security concerns.
