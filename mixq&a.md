### is a combination of ALB vs NLB possible
```
“Yes, combining Application Load Balancer and Network Load Balancer is possible in AWS.

Typically, we place NLB in front of ALB when we need both Layer 4 and Layer 7 capabilities.

NLB provides static IPs, high performance, and handles TCP traffic, while ALB provides advanced routing like path-based and host-based routing.

So the flow becomes:
Client → NLB → ALB → backend services.

This is useful in cases like PrivateLink, fixed IP requirements, or when we need both performance and intelligent routing in the same architecture.”
```
### what are lifecycle policies ?
```
Lifecycle policies are rules that automatically manage the lifecycle of resources—like moving, archiving, or deleting data—based on time or conditions.

In AWS, the most common example is Amazon S3 lifecycle policies, where we can transition objects from Standard storage to cheaper classes like Glacier, or delete them after a certain number of days.”
```

### what is used for streaming and why ?
```
“For streaming in AWS, we commonly use Amazon Kinesis.

It is used to collect, process, and analyze real-time streaming data like logs, clickstreams, or IoT data.

The main reason we use it is because it can handle high-throughput, low-latency data ingestion and allows real-time processing instead of batch processing.”
```
### what are storage classes ? rank them based on cost ?
```
In Amazon S3, storage classes define different tiers of storage optimized for cost, availability, and access frequency.”

💰 Storage Classes Ranked by Cost (High → Low)
S3 Standard (most expensive)
Frequently accessed data
High availability, low latency
S3 Intelligent-Tiering
Automatically moves data between tiers
Slight monitoring cost, but saves money if access pattern is unknown
S3 Standard-IA (Infrequent Access)
Lower storage cost
Retrieval cost applies
S3 One Zone-IA
Cheaper than Standard-IA
Stored in a single AZ (less resilient)
S3 Glacier Instant Retrieval
Archive data but still needs quick access
S3 Glacier Flexible Retrieval
Cheaper, but retrieval takes minutes to hours
S3 Glacier Deep Archive (cheapest)
Lowest cost
Retrieval can take hours (12+)
```
### pods not able to access s3 in eks ? how do you fix it?
```
If pods in Amazon EKS are not able to access Amazon S3, the most common cause is missing or incorrect IAM permissions.

The recommended fix is to use IAM Roles for Service Accounts (IRSA) so that pods can securely access AWS services
. Check IAM Permissions
Verify IRSA iam role for serive account Configuration
Network Check vpc endpoint connected 
```
### how does dns resolution work and where are records stored ?
```
DNS resolution is the process of converting a domain name into an IP address so that a client can connect to the correct server.

For example, when we type a URL, DNS translates it into the server’s IP address.”

🔄 Step-by-Step Flow

Let’s say you enter example.com in a browser:

Local cache check
Browser / OS checks if it already knows the IP.
Recursive resolver
If not, the request goes to a DNS resolver (usually ISP or public DNS).
Root server
Resolver asks root servers: “Where is .com handled?”
TLD server
Root points to the .com TLD server, which responds with the authoritative server.
Authoritative DNS server
Final server returns the actual IP address of the domain.
Response returned
Resolver sends IP back to client → browser connects to server.
📍 Where are DNS records stored?

“DNS records are stored in authoritative DNS servers.

In AWS, this is managed by Amazon Route 53, where we create records like A, CNAME, MX, etc.”

🧠 Types of Records (mention 2–3)
A record → maps domain to IP
CNAME → maps one domain to another
MX → mail server routing
```
### what is route 53 ?
```
“Amazon Route 53 is a highly available and scalable DNS (Domain Name System) service in AWS used to route user traffic to applications by translating domain names into IP addresses.”

🧠 Break it into 3 core functions (interviewers like this)
1. Domain Registration
You can buy and manage domain names (like example.com)
2. DNS Management
Store DNS records (A, CNAME, MX, etc.)
Acts as an authoritative DNS server
3. Traffic Routing
Routes traffic intelligently using routing policies
⚡ Routing Policies (mention 2–3 max)
Simple routing → single resource
Weighted routing → split traffic (e.g., 70/30)
Latency-based routing → lowest latency region
Failover routing → active-passive setup
```
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

  Service
selector:
  app: my-app
 Pod
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

```

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
