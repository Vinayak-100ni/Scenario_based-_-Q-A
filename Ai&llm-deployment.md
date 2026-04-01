## 1. Deployment vs StatefulSet
**Question:** What's the difference between Deployment and StatefulSet?

**Answer:** Deployment is used for stateless applications where pods are
interchangeable and can scale freely. StatefulSet is used for stateful
applications where each pod requires a stable identity, persistent
storage, and ordered deployment.
Use Deployment: - APIs - Microservices - Web apps
Use StatefulSet: - Databases - Kafka - Redis - AI distributed training
------------------------------------------------------------------------
## 2. Node Failure Handling in Kubernetes
**Question:** When a node running your pods goes down, how does
Kubernetes reschedule them?

**Answer:** Kubernetes detects node failure via heartbeats, marks node
NotReady, and reschedules pods on healthy nodes.
To minimize downtime: - Multiple replicas - PodDisruptionBudget -
Readiness probes - Pod anti-affinity - Multi‑AZ deployment
------------------------------------------------------------------------
## 3. GPU Scheduling in Kubernetes
**Question:** How do you ensure pods land only on GPU nodes?

**Answer:** Use: - NVIDIA device plugin - GPU requests/limits - Node
selectors - Taints and tolerations - RuntimeClass - MIG (GPU
partitioning)
Example:
``` yaml
resources:
 limits:
nvidia.com/gpu: 1
```
------------------------------------------------------------------------
## 4. Build Once Promote Everywhere CI/CD
**Question:** How do you promote same image across environments?

**Answer:** Build once → Tag immutable → Push to registry → Promote same
image.
Example: - dev → staging → prod - No rebuild - Same artifact everywhere
Tagging: - commit SHA - version tags - environment tags
------------------------------------------------------------------------
## 5. Approval & Policy Gates
**Question:** How do you implement approvals and policy gates?

**Answer:** Use: - Security scans (Trivy) - SBOM generation - Image
signing - Manual approval - Policy enforcement
Block promotion if: - Vulnerabilities found - Tests fail - Signature
missing
------------------------------------------------------------------------
## 6. Failed Rollout Detection
**Question:** How detect rollout failure?

**Answer:** Signals: - rollout status - readiness probe failure -
crashloop - high error rate
Rollback:
``` bash
kubectl rollout undo deployment
```
------------------------------------------------------------------------
## 7. Reproducible AI Containers
**Question:** How make AI training reproducible?

**Answer:** - Pin base image - Freeze dependencies - Set seeds - Version
dataset - Multi-stage builds
Example:
``` dockerfile
FROM pytorch/pytorch:2.2.0-cuda11.8-runtime
```
------------------------------------------------------------------------
## 8. Dataset Caching in Kubernetes
**Question:** How handle dataset access?

**Answer:** Use: - Object storage - PVC caching - Init containers -
Sidecars - Node local storage
Flow: S3 → Init container → PVC → Training pod
------------------------------------------------------------------------
## 9. GPU Scheduling Settings
**Question:** Kubernetes GPU settings?

**Answer:** - Device plugin - Requests / Limits - Node affinity -
Taints - Runtime class - MIG
------------------------------------------------------------------------
## 10. AWS ECR GPU Images
**Question:** How build GPU images for AWS?

**Answer:** - Pinned CUDA image - Immutable tags - Push to ECR - Layer
caching - Pre-pull images
------------------------------------------------------------------------
## 11. IRSA vs Node Role
**Question:** IAM access for ECR?

**Answer:** Node role: - Simple - Less secure
IRSA: - Fine-grained access - Credential rotation - Recommended
------------------------------------------------------------------------
## 12. IRSA Implementation
**Question:** How implement IRSA?

Steps: 1. Enable OIDC 2. Create IAM policy 3. Create IAM role 4. Create
service account 5. Attach role
Example:
``` yaml
serviceAccountName: ecr-sa
```
------------------------------------------------------------------------
