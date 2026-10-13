# DevOps Interview Questions and Answers

This document provides interview-ready answers to the questions visible in the supplied screenshots. Adapt the project examples to match your actual experience; avoid claiming tools or implementations you have not used.

## Kubernetes & Deployment Strategies

### 1. Explain Kubernetes control-plane and worker-node components. What does each do?

**Answer:**

The Kubernetes control plane manages the cluster, while worker nodes run application workloads.

- **API Server (`kube-apiserver`):** The main entry point for Kubernetes API requests. It validates requests and updates cluster state in etcd.
- **etcd:** A distributed key-value store that holds Kubernetes cluster state and configuration.
- **Scheduler (`kube-scheduler`):** Selects a suitable node for each unscheduled Pod based on resource requests, constraints, affinity, taints, and tolerations.
- **Controller Manager (`kube-controller-manager`):** Runs controllers that continuously compare actual state with desired state and take corrective action. Examples include Deployment/ReplicaSet and node controllers.
- **kubelet:** Runs on each worker node and ensures the Pods assigned to that node are running as specified. It works with the container runtime and reports status to the API server.
- **kube-proxy:** Implements Service networking rules on many Kubernetes installations, using iptables or IPVS where configured. Some clusters use a CNI/eBPF implementation that replaces this role.
- **Container runtime:** Runs containers, commonly containerd or CRI-O, through the Container Runtime Interface (CRI).
- **CNI plugin:** Configures Pod networking and IP connectivity. Examples include Calico, Cilium, and Flannel.

**Typical flow:** A user applies a manifest → API server validates it and stores desired state in etcd → controllers create or update workload objects → scheduler selects a node → kubelet asks the runtime to start containers → CNI configures Pod networking.

### 2. Which deployment strategies have you used? Explain Rolling, Blue-Green, and Canary with project examples.

**Answer:**

- **Rolling update:** Gradually replaces old Pods with new Pods. It is resource-efficient and commonly used for routine releases. Configure `maxSurge` and `maxUnavailable`, and use readiness probes so traffic is sent only to ready Pods.
- **Blue-Green:** Maintains two environments or versions. Deploy and validate the new version in Green while Blue continues serving traffic, then switch the Service or routing target. Rollback is usually a switch back to Blue, provided data and API changes remain compatible.
- **Canary:** Sends a small percentage of traffic to the new version first, monitors errors and latency, and increases traffic if metrics remain healthy. This usually requires an ingress controller, service mesh, or progressive-delivery controller that supports traffic splitting.

**Example phrasing:** “For a Kubernetes application, I would use rolling updates for normal releases. For a high-risk release, I would deploy a parallel version, validate health checks and smoke tests, then shift traffic gradually or switch the Service. I would monitor error rate, latency, restarts, and business metrics before completing the rollout.”

Only describe a strategy as something you personally used if you actually implemented it.

### 3. During a Blue-Green release, what happens to old Pods, ongoing requests, and live traffic when traffic switches?

**Answer:**

When routing is switched to Green, new requests are sent to the new version. Blue Pods do not necessarily stop immediately: they can remain available for rollback until the release is accepted.

For a graceful transition:
1. Update the Service selector or traffic-routing configuration to target Green.
2. Confirm Green Pods are Ready and smoke tests pass.
3. Stop sending new traffic to Blue.
4. Allow in-flight requests to complete using graceful shutdown and an appropriate `terminationGracePeriodSeconds`.
5. Ensure the application handles `SIGTERM`, stops accepting new work, and drains connections correctly.
6. Scale down Blue after the observation window.

Existing connections may behave differently depending on the load balancer, ingress, protocol, and connection pooling. A routing change does not guarantee every existing connection instantly moves to Green. Keep database changes backward-compatible during the rollback window.

### 4. How does node autoscaling work when demand rises or falls? Cluster Autoscaler/Karpenter vs HPA?

**Answer:**

These components solve different scaling problems:

- **HPA (Horizontal Pod Autoscaler):** Changes the replica count of a workload based on CPU, memory, or custom/external metrics. Metrics Server or another metrics provider must supply the required metrics.
- **Cluster Autoscaler:** Adds nodes when Pods cannot be scheduled due to insufficient resources and removes eligible underutilized nodes when workloads can be moved safely. It generally operates with supported node groups or autoscaling groups.
- **Karpenter:** In supported environments, provisions nodes based on unschedulable Pod requirements and can consolidate or replace nodes based on its disruption and scheduling rules.

**Scale-up example:** Traffic increases → HPA creates more Pods → scheduler finds insufficient capacity → Cluster Autoscaler/Karpenter adds nodes → Pods are scheduled when capacity becomes available.

**Scale-down:** HPA reduces replicas when demand falls. Node autoscaling may then consolidate workloads and remove unneeded nodes, subject to PodDisruptionBudgets, scheduling constraints, and provider configuration.

HPA scales **Pods**; Cluster Autoscaler/Karpenter scales **nodes**. They can work together.

### 5. How does a newly created Pod receive its IP address? Which component handles Pod networking (CNI)?

**Answer:**

After the scheduler assigns a Pod to a node, kubelet asks the container runtime to create the Pod sandbox. The runtime invokes the configured CNI plugin to set up the Pod network namespace, interface, routes, and IP address. The exact IP allocation mechanism depends on the CNI implementation.

For example, Calico can allocate Pod IPs from configured IP pools and program routing or policy. The Pod IP is then reported through Kubernetes status. A Service provides a stable virtual endpoint for reaching a changing set of Pods.

### 6. If a subnet runs out of IP addresses, what happens to the Pod's status? How would you investigate a sandbox/network setup failure?

**Answer:**

If the CNI cannot allocate an address or create the network, the Pod may remain in `Pending` or show sandbox-related events such as `FailedCreatePodSandBox`. The exact phase and event depend on where creation failed; it is not always a single fixed status.

**Investigation steps:**

```bash
kubectl get pods -A -o wide
kubectl describe pod <pod-name> -n <namespace>
kubectl get events -n <namespace> --sort-by=.lastTimestamp
kubectl describe node <node-name>
kubectl get nodes
```

On the affected node, inspect kubelet and runtime logs:

```bash
sudo journalctl -u kubelet --since "30 minutes ago"
sudo journalctl -u containerd --since "30 minutes ago"
```

Check CNI configuration and logs, IP pools, available subnet addresses, routes, interface state, and network policies. For Calico, inspect the Calico node Pod logs and IPPool/IPAM usage using the tools available in that cluster. Also check whether node Pod limits or cloud subnet limits have been reached.

**Resolution:** Free or expand the appropriate IP pool/subnet, correct CNI configuration, or repair the CNI/runtime issue. Do not delete production Pods or restart networking components until the impact and recovery plan are understood.

## Observability & CI/CD

### 7. How do you troubleshoot CPU, memory, restarts, latency, and production issues with Prometheus and Grafana?

**Answer:**

I start by identifying the affected service, time window, and customer impact, then correlate metrics, logs, events, and recent deployments.

Useful Kubernetes checks:

```bash
kubectl top nodes
kubectl top pods -n <namespace> --containers
kubectl get pods -n <namespace> -o wide
kubectl describe pod <pod-name> -n <namespace>
kubectl logs <pod-name> -n <namespace> --previous
kubectl get events -n <namespace> --sort-by=.lastTimestamp
```

Useful Prometheus examples:

```promql
# CPU usage by container (cores)
sum by (pod, container) (
  rate(container_cpu_usage_seconds_total{namespace="YOUR_NAMESPACE", container!=""}[5m])
)

# Working-set memory by Pod
sum by (pod) (
  container_memory_working_set_bytes{namespace="YOUR_NAMESPACE", container!=""}
)

# Container restarts over the last hour
increase(kube_pod_container_status_restarts_total{namespace="YOUR_NAMESPACE"}[1h])
```

For request latency, use the application's actual histogram metric names, for example a p95 query with `histogram_quantile(0.95, sum by (le) (rate(http_request_duration_seconds_bucket[5m])))` when that metric exists.

In Grafana, correlate CPU throttling, memory, OOM kills, Pod restarts, request rate, 4xx/5xx rates, p95/p99 latency, node saturation, and deployment timestamps. Check logs in Loki or another log platform and inspect traces if distributed tracing is available.

**Typical causes:**
- High CPU: traffic spike, expensive code, insufficient requests/limits, or CPU throttling.
- High memory: leak, cache growth, large requests, or unsuitable limits.
- Restarts: OOMKilled, failed probes, application crash, or node disruption.
- High latency: database contention, downstream dependency, saturation, network issue, or slow application code.

For production, mitigate safely first, communicate impact, preserve evidence, and verify recovery after the fix.

### 8. Explain your actual CI/CD work — commit → build → test → security scan → image push → Kubernetes deployment.

**Answer:**

A typical pipeline is:

1. **Commit and trigger:** A push or pull request triggers GitHub Actions, Jenkins, or Azure DevOps.
2. **Checkout and setup:** Fetch source code and set up the required runtime/tool versions.
3. **Build:** Install dependencies and compile/package the application. Reuse trusted base images and cache dependencies where appropriate.
4. **Test:** Run unit tests, linting, and any integration tests.
5. **Security and quality:** Run tools such as Trivy for image/filesystem vulnerabilities and SonarQube for code-quality analysis, with agreed quality gates.
6. **Build container:** Build the Docker image and tag it with an immutable identifier such as the commit SHA or build number.
7. **Push:** Authenticate to the registry using a short-lived identity or a securely stored token, then push the image.
8. **Deploy:** Update the Kubernetes Deployment image or Helm/Kustomize release, then apply/sync the manifests.
9. **Verify:** Wait for rollout completion, run smoke tests, and inspect readiness, logs, and metrics.
10. **Rollback:** If health checks or verification fail, restore the last known-good version using the team's release process.

Example commands:

```bash
docker build -t registry.example.com/team/app:${GIT_SHA} .
docker push registry.example.com/team/app:${GIT_SHA}
kubectl set image deployment/app app=registry.example.com/team/app:${GIT_SHA} -n <namespace>
kubectl rollout status deployment/app -n <namespace> --timeout=180s
kubectl rollout undo deployment/app -n <namespace>  # only if appropriate
```

Use least-privilege credentials, protect production environments, avoid secrets in logs, and promote the same tested image between environments. The commands above are illustrative; adapt them to your pipeline and deployment method.

### 9. Write a Kubernetes Deployment and Service YAML manifest and explain the main fields.

**Answer:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
  namespace: default
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web-app
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  template:
    metadata:
      labels:
        app: web-app
    spec:
      containers:
        - name: web-app
          image: nginx:1.28
          ports:
            - name: http
              containerPort: 80
          resources:
            requests:
              cpu: "100m"
              memory: "128Mi"
            limits:
              cpu: "500m"
              memory: "256Mi"
          readinessProbe:
            httpGet:
              path: /
              port: http
            initialDelaySeconds: 5
            periodSeconds: 10
          livenessProbe:
            httpGet:
              path: /
              port: http
            initialDelaySeconds: 10
            periodSeconds: 20
---
apiVersion: v1
kind: Service
metadata:
  name: web-app
  namespace: default
spec:
  type: ClusterIP
  selector:
    app: web-app
  ports:
    - name: http
      port: 80
      targetPort: http
```

**Main fields:**
- `apiVersion` and `kind`: API group/version and resource type.
- `metadata`: name, namespace, labels, and annotations.
- `replicas`: desired number of Pod replicas.
- `selector.matchLabels`: identifies Pods managed by the Deployment; it must match the Pod template labels.
- `template`: Pod specification.
- `strategy`: how Deployment updates are rolled out.
- `containers.image`: container image and tag.
- `resources.requests/limits`: scheduling reservation and resource ceilings.
- `readinessProbe`: controls when a Pod is considered ready for Service traffic.
- `livenessProbe`: detects containers that should be restarted.
- Service `selector`: selects backend Pods.
- Service `port` and `targetPort`: Service-facing port and destination Pod port.
- `ClusterIP`: exposes the Service internally in the cluster.

For production, pin images to approved versions or digests and configure probes based on the application's real health endpoints.

## Linux & Python

### 10. Which Linux commands check disk usage and available space?

**Answer:**

```bash
df -h                 # Filesystem capacity and free space
df -hT                # Capacity plus filesystem type
du -sh /path/to/dir   # Total size of a directory
du -h --max-depth=1 /var  # Largest immediate subdirectories on GNU du
lsblk                 # Disks, partitions, and mount points
```

`df` reports filesystem usage; `du` estimates space used by files and directories. If `df` shows space used but `du` cannot account for it, check for deleted-but-open files and mount points.

### 11. How do you access AWS from Python using Boto3 and manage credentials securely?

**Answer:**

Boto3 uses the AWS credential provider chain. Prefer IAM roles for EC2, ECS, EKS workloads, or an appropriate federation/SSO flow for developers. Avoid hardcoding access keys in source code or committing them to Git.

Example:

```python
import boto3

# Uses the configured AWS credential provider chain.
session = boto3.Session(region_name="ap-south-1")
s3 = session.client("s3")

for bucket in s3.list_buckets()["Buckets"]:
    print(bucket["Name"])
```

For local development, use AWS IAM Identity Center/SSO or a named profile:

```bash
aws configure sso
aws sso login --profile dev
```

Then create the session with `boto3.Session(profile_name="dev", region_name="ap-south-1")`. Grant only the permissions needed, rotate/revoke exposed credentials promptly, and use Secrets Manager or another approved secret store for application secrets.

### 12. Write Python code to check if an S3 bucket exists, print the result, and distinguish 403 from 404 errors.

**Answer:**

```python
import boto3
from botocore.exceptions import ClientError, NoCredentialsError, EndpointConnectionError

def check_bucket(bucket_name: str, region: str = "ap-south-1") -> None:
    s3 = boto3.client("s3", region_name=region)

    try:
        s3.head_bucket(Bucket=bucket_name)
        print(f"Bucket '{bucket_name}' exists and is accessible.")

    except NoCredentialsError:
        print("AWS credentials are missing or unavailable.")

    except EndpointConnectionError as exc:
        print(f"Could not connect to AWS endpoint: {exc}")

    except ClientError as exc:
        error = exc.response.get("Error", {})
        code = str(error.get("Code", ""))
        status = exc.response.get("ResponseMetadata", {}).get("HTTPStatusCode")

        if status == 403 or code in {"403", "AccessDenied"}:
            print("Access denied (403). The bucket may exist, but this identity lacks permission; this does not prove existence.")
        elif status == 404 or code in {"404", "NoSuchBucket", "NotFound"}:
            print("Bucket not found (404), subject to S3's response and permissions.")
        else:
            print(f"Unable to verify bucket. HTTP status={status}, error={code}: {exc}")

if __name__ == "__main__":
    check_bucket("replace-with-your-bucket-name")
```

**Important distinction:** `HeadBucket` can return 403 when the caller lacks permission to determine whether a bucket exists. A 403 must not be treated as proof that the bucket exists. For reliable verification, use the correct account/region and grant the least-privilege permissions needed. A 404 can indicate that the bucket does not exist or is not addressable in the expected way.

## AWS, RDS & S3

### 13. What are the Route 53 routing policies? Give practical use cases.

**Answer:**

Amazon Route 53 supports several routing policy types:

- **Simple:** One record or a set of values without health-based selection; useful for straightforward DNS mapping.
- **Weighted:** Distributes DNS responses by configured weights; useful for gradual traffic shifts or testing a new endpoint.
- **Latency-based:** Directs users toward the AWS Region with the lowest measured network latency among configured records.
- **Failover:** Uses primary/secondary records with health checks for active-passive disaster recovery.
- **Geolocation:** Routes based on the user's geographic location; useful for regional content or compliance requirements.
- **Geoproximity:** Routes based on geographic proximity and optional bias; commonly used with Traffic Flow.
- **IP-based:** Routes based on the source IP's CIDR mapping; useful when source network ranges should map to specific endpoints.
- **Multivalue answer:** Returns multiple healthy record values; useful for simple DNS-level distribution across several resources.

DNS caching and TTLs mean changes are not necessarily instantaneous. Route 53 routing is DNS-based and does not itself migrate existing connections.

### 14. How do you connect EC2 to RDS? Cover VPC, security groups, endpoint, port, and connection command.

**Answer:**

1. Place EC2 and RDS in the same VPC or configure appropriate routing between their networks.
2. Prefer a private RDS endpoint and private subnets unless a documented requirement says otherwise.
3. Configure the RDS security group to allow inbound database traffic **from the EC2 instance's security group**, not from the whole internet.
4. Confirm network ACLs, routes, DNS resolution, database status, and the correct database port.
5. Retrieve the RDS endpoint from the AWS console or CLI. Do not use the DB instance identifier as the hostname.

Common ports:
- PostgreSQL: `5432`
- MySQL: `3306`

Connectivity checks:

```bash
# PostgreSQL
nc -vz <rds-endpoint> 5432
psql "host=<rds-endpoint> port=5432 dbname=<database> user=<username> sslmode=require"

# MySQL
nc -vz <rds-endpoint> 3306
mysql -h <rds-endpoint> -P 3306 -u <username> -p
```

Use the database's required TLS settings, keep passwords out of command history where possible, and store credentials in Secrets Manager or another approved secret-management service.

### 15. Which RDS engine did you work with (MySQL/PostgreSQL), and what are your daily database tasks?

**Answer template — personalize honestly:**

“I have worked with **[PostgreSQL/MySQL—choose what you actually used]**. My infrastructure-side responsibilities include provisioning or configuring database resources, managing security-group access, checking connectivity, monitoring CPU/storage/connections, reviewing logs and alarms, validating backups, and coordinating maintenance. I work with database administrators or developers for schema changes, query tuning, and application-level issues when those are outside my ownership.”

Typical DevOps tasks include:
- Monitor CPU, memory-related indicators, free storage, connection count, latency, and database logs.
- Check backup status, retention, snapshots, and restore procedures.
- Review parameter groups, maintenance windows, encryption, TLS, and access controls.
- Investigate connection failures, exhausted connection pools, slow queries, and storage pressure.
- Test recovery procedures and document operational changes.

Do not claim DBA duties or a specific engine unless they reflect your actual work.

### 16. How do you make RDS highly available? Explain Multi-AZ, automatic failover, backup, and monitoring.

**Answer:**

- **Multi-AZ DB instance deployment:** AWS maintains a standby in another Availability Zone for supported configurations. During certain failures or maintenance events, RDS can fail over to the standby. The standby is not normally used to serve read traffic.
- **Multi-AZ DB cluster:** A different deployment option with a writer and readable instances; behavior differs from a traditional Multi-AZ DB instance.
- **Backups:** Configure automated backups and retention, and take manual snapshots before important changes when appropriate.
- **Recovery:** Understand point-in-time recovery and snapshot restoration. Test recovery and record the achievable RPO/RTO.
- **Monitoring:** Use CloudWatch metrics and alarms for CPU, free storage, connections, latency, and relevant database metrics. Review RDS events and logs.
- **Application resilience:** Use the RDS DNS endpoint, retry transient connection failures with bounded exponential backoff, and ensure the connection pool can recover after failover.

Multi-AZ improves availability but does not replace backups, tested recovery, monitoring, or application retry logic.

### 17. How is S3 used in your project — logs, application artifacts, backups, and Terraform state?

**Answer:**

S3 can be used for:
- **Artifacts:** Store versioned build artifacts or release packages where appropriate.
- **Logs and exports:** Store archived logs, reports, or exports with lifecycle policies and access controls.
- **Backups:** Keep backup copies subject to retention, encryption, and restore testing.
- **Terraform state:** Store remote state in a dedicated, restricted bucket, typically with versioning and state locking configured using a supported locking mechanism.

For Terraform state, use a dedicated bucket/prefix, enable versioning, block public access, encrypt data, restrict access with IAM and bucket policies, and configure locking to prevent concurrent state writes. State may contain sensitive values, so treat it as confidential. Do not casually share the state bucket with public application artifacts.

**Example phrasing:** “I use S3 for centrally stored artifacts or backups where required. For Terraform, remote state provides a shared source of truth for the team; access is restricted, versioning is enabled, and locking is configured to reduce the risk of concurrent changes.”

Only state the specific S3 uses you have actually implemented.

## Final Interview Question

### 18. “Do you have any questions for me?” What would you ask about infrastructure, deployment, and on-call responsibilities?

**Answer:**

Choose two or three questions based on the conversation:

1. “Could you describe your current cloud and Kubernetes architecture, and the main reliability challenges the team is working on?”
2. “How is your CI/CD pipeline structured, and what quality and security gates must pass before production deployment?”
3. “How do you handle progressive delivery, monitoring, and rollback when a release causes problems?”
4. “What does the on-call rotation look like, and how are incidents escalated and reviewed?”
5. “What would success look like for the person joining this role in the first 90 days?”

## Quick interview technique

For scenario questions, structure your response as:

1. **Context:** What system and impact were involved?
2. **Diagnosis:** Which metrics, logs, events, and commands did you inspect?
3. **Action:** What change did you make, and why was it safe?
4. **Verification:** How did you prove the issue was resolved?
5. **Prevention:** What alert, test, automation, or runbook improvement followed?

Use real examples from your experience and be clear about whether you implemented a solution yourself, supported it, or are describing the recommended approach.
