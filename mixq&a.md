# Kubernetes 503 Service Unavailable Error

## 📌 What is 503 Error?
A **503 Service Unavailable** error in Kubernetes means:
> The request reached the cluster, but no backend pod is available to handle it.

---

## 🧠 Architecture Flow
Client → Ingress → Service → Pods


**503 occurs when:**
- Service has **no healthy/ready pods**

---

## 🔍 Common Causes

### 1. No Running or Ready Pods
- Pods are in:
  - CrashLoopBackOff
  - Pending
  - Not Ready

**Check:**
```bash
kubectl get pods
kubectl get endpoints <service-name>
```
2. Readiness Probe Failure
Pod is running but not marked as Ready
Removed from service endpoints

Check:

kubectl describe pod <pod-name>

Look for:

Readiness probe failed
3. Service Selector Mismatch
Labels in Service do not match Pod labels

Example:

# Service
selector:
  app: my-app
# Pod
labels:
  app: myapp  # ❌ mismatch
4. Ingress Issues
Backend service not reachable
Wrong service name or port
No endpoints available

Check:

kubectl describe ingress <name>
kubectl get svc
5. Port Mismatch
Container port and Service port mismatch

Check:

kubectl describe svc <service-name>
6. Application Crash / Slow Startup
App not ready when traffic arrives

Solution:

Configure readinessProbe
Use startupProbe
7. Resource Issues (OOMKilled / CPU)
Pod crashes due to insufficient resources
No healthy pods left

Check:

kubectl describe pod <pod-name>



## 🔹 GPU in Kubernetes

* GPU Node → Node with GPU hardware
* Device Plugin → Enables Kubernetes to use GPU
* Pod GPU Usage → Request using `nvidia.com/gpu`
* GPU Partitioning → Split GPU using MIG for better utilization

---

## 🔹 etcd Failure Impact

* Control plane becomes unavailable
* kubectl commands fail
* No new deployments possible
* Running pods may continue temporarily

---

## 🔹 Kubernetes Security

* Enable TLS encryption
* Restrict API server access
* Use RBAC for authorization
* Enable audit logs

---

## 🔹 Jobs & CronJobs

* Job → Runs task once
* CronJob → Runs task on schedule

---

# ☁️ AWS Concepts

## 🔹 CloudWatch Setup

* Create IAM role
* Install CloudWatch agent
* Configure logs & metrics
* Create alarms

---

## 🔹 S3 Cost Optimization

* Move data between storage classes
* Use Lifecycle Policies for automation
* Use Intelligent Tiering for dynamic workloads

---

## 🔹 Reserved Instances

* Billing discount model
* Purchased separately (not inside EC2)
* Applied automatically to matching usage

---

## 🔹 Auto Scaling Warm Pool

* Pre-initialized EC2 instances
* Faster scaling during traffic spikes
* Reduces startup delay

---

# 🧱 Terraform Concepts

## 🔹 Provisioners

* local-exec → runs locally
* remote-exec → runs on remote machine
* file → copies files
* Not recommended for production

---

## 🔹 Connection Block

* Required for remote-exec/file
* Can be defined at resource or provisioner level

---

## 🔹 Built-in Functions

* String → upper, lower, replace
* List → join, element
* Map → lookup, keys
* Numeric → max, min

---

## 🔹 Meta-Arguments

* count → create multiple resources
* for_each → create dynamic resources
* depends_on → define dependencies
* lifecycle → control resource behavior

---

## 🔹 Terraform Incident Learning

* No rollback command
* Can recreate infra but not data
* Use Git for rollback
* Always take backups
* Use `prevent_destroy` for safety

---

# 🤖 ML / LLM Concepts

## 🔹 LLM Inference

* Using trained model to generate output
* No training involved
* Focus on latency and performance

---

# 🔄 DevOps Practices

## 🔹 Zero Downtime Deployment

* Rolling updates
* Blue-Green deployment
* Canary deployment
* Health checks and monitoring
* Fast rollback strategy

---
