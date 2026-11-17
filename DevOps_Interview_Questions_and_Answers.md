# DevOps, SRE, Cloud & Platform Engineering Interview Preparation

**Practical scenario-based answers \| AWS, Azure, GCP, Linux, Docker,
Kubernetes, CI/CD, Terraform, Observability and DevSecOps**

> **How to use this guide:** Answer in an interview using the sequence
> **scope impact → gather evidence → isolate the layer → mitigate safely
> → verify recovery → prevent recurrence**. Commands are examples; adapt
> namespaces, resource names, cloud regions and change controls to the
> environment. In production, prefer read-only diagnostics first and
> obtain approval before disruptive actions.

## Contents

1.  Linux and shell scripting
2.  Docker and containers
3.  Kubernetes operations and troubleshooting
4.  CI/CD and Jenkins
5.  Cloud architecture and AWS scenarios
6.  Terraform and infrastructure as code
7.  Monitoring, observability and incident response
8.  Security and DevSecOps
9.  Advanced platform engineering and NVIDIA-style system design
10. Interview response framework

------------------------------------------------------------------------

# 1. Linux + Shell Scripting

## 1. How do you troubleshoot high CPU, memory, disk I/O and load average on a production Linux server?

**Answer:**

I first confirm the scope, start time, affected services and whether the
issue is sustained or a short spike. I avoid restarting services before
collecting evidence.

**CPU**

``` bash
uptime
top
htop
ps -eo pid,ppid,cmd,%cpu,%mem --sort=-%cpu | head
mpstat -P ALL 1 5
```

Check whether CPU is consumed by application processes, system
processes, interrupts or I/O wait. A high load average does not
automatically mean high CPU; compare it with the number of CPU cores and
inspect blocked tasks.

**Memory**

``` bash
free -h
vmstat 1 5
ps aux --sort=-%mem | head
cat /proc/meminfo
dmesg -T | grep -i -E 'oom|killed process'
```

Check available memory, swap activity, cache, memory leaks and
OOM-killer events. On a container host, also inspect cgroup/container
memory limits.

**Disk space and inodes**

``` bash
df -h
df -i
du -xhd1 /var 2>/dev/null | sort -h
find /var/log -type f -size +500M -ls
```

A filesystem can run out of inodes even when free disk space remains.
Identify large files before deleting or truncating anything.

**Disk I/O**

``` bash
iostat -xz 1 5
iotop -o
vmstat 1 5
```

Look at device utilization, latency, queue depth and I/O wait. Correlate
with database queries, backups, log growth and storage performance.

**Recovery:** mitigate the confirmed cause, such as stopping an approved
runaway job, scaling capacity, rotating logs or fixing a query. Preserve
logs and evidence. After recovery, add alerts, capacity thresholds and a
root-cause action.

## 2. Write a Bash script to monitor disk usage, identify threshold breaches and trigger alerts.

**Answer:**

This script checks mounted filesystems, ignores pseudo-filesystems, and
sends an alert when usage reaches the configured threshold.

``` bash
#!/bin/bash

THRESHOLD=80

df -h | grep '^/dev' | while read line
do
    USAGE=$(echo $line | awk '{print $5}' | sed 's/%//')
    DISK=$(echo $line | awk '{print $6}')

    if [ "$USAGE" -ge "$THRESHOLD" ]; then
        echo "WARNING: Disk usage is high on $DISK: $USAGE%"
    else
        echo "Disk usage is normal on $DISK: $USAGE%"
    fi
done
```

Run it with:

``` bash
chmod +x disk-monitor.sh
THRESHOLD=85 ALERT_EMAIL=ops@example.com ./disk-monitor.sh
```

For production, schedule it using cron or a systemd timer, log results,
monitor inode usage separately, and preferably send metrics to
Prometheus/CloudWatch rather than relying only on local email. Test
alert delivery before depending on it.

------------------------------------------------------------------------

# 2. Docker + Containers

## 3. How do you troubleshoot a Docker container that repeatedly exits or enters a restart loop?

**Answer:**

First determine whether the container exits because of the application,
its configuration, resource limits or the host.

``` bash
docker ps -a
docker inspect <container>
docker logs --timestamps --tail 200 <container>
docker events --since 30m
docker stats
```

Check: - Exit code and `State.OOMKilled` in `docker inspect`. -
Entrypoint, command, environment variables and mounted files. - Missing
secrets/configuration, permissions and working directory. - Dependency
availability, DNS and network connectivity. - Health-check failures and
restart policy. - Host memory, disk, inode and daemon events.

For a container that exits too quickly, inspect its logs and
configuration; if safe, run the same image with an overridden entrypoint
for debugging. Do not blindly change the restart policy or repeatedly
restart it. Fix the root cause, deploy the corrected configuration and
verify stable health.

## 4. Explain multi-stage builds, Docker networking, volumes and image optimization techniques.

**Answer:**

**Multi-stage builds** use one stage to compile/build and a smaller
runtime stage to contain only the required binary and runtime
dependencies. This avoids shipping compilers, caches and build tools.

``` dockerfile
FROM golang:1.24 AS build
WORKDIR /src
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 go build -o /out/app ./cmd/app

FROM gcr.io/distroless/static-debian12
COPY --from=build /out/app /app
USER nonroot:nonroot
ENTRYPOINT ["/app"]
```

**Networking** - `bridge`: common default for containers on one Docker
host. - User-defined bridge: provides container-name DNS and
isolation. - `host`: shares the host network namespace; use only when
justified. - `none`: no network. - Overlay: multi-host networking in
supported orchestrators.

**Volumes** - Named volumes are managed by Docker and suit persistent
application data. - Bind mounts map host paths into containers and are
useful for development or specific host integrations. - Container
writable layers are ephemeral; do not store important state only there.

**Optimization:** use a suitable slim runtime base, multi-stage builds,
`.dockerignore`, dependency-layer caching, pinned base images, non-root
users, minimal packages and no build secrets in layers. Scan images and
test the final runtime image.

## 5. Explain Docker image optimization. What would you do if an image grew from 300 MB to 3 GB?

**Answer:**

I compare the old and new image layers and identify which layer
introduced the growth.

``` bash
docker history --no-trunc <image>
docker image inspect <image>
docker buildx du
```

Then I check for: - Build tools, package caches, test data or source
maps copied into runtime. - Large logs, generated files or dependency
directories. - A broad `COPY . .` caused by a missing `.dockerignore`. -
Multiple package installation layers that retain caches. - An
unexpectedly large base image or architecture-specific artifacts.

I introduce multi-stage builds, remove caches in the same layer in which
they are created, copy only runtime artifacts, use `.dockerignore`, and
select a trusted minimal base. I compare image size and startup
behavior, run vulnerability scans and functional tests, then roll out
the new immutable tag. I do not remove libraries blindly if the
application requires them.

## 6. A Docker container works on a developer machine but fails in Kubernetes. What would you check?

**Answer:**

I compare the local runtime with the Kubernetes runtime:

1.  **Image:** confirm the exact digest/tag, registry credentials,
    architecture and image pull policy.
2.  **Command:** inspect `ENTRYPOINT`, `CMD`, arguments and working
    directory.
3.  **Configuration:** compare environment variables, ConfigMaps,
    Secrets and mounted paths.
4.  **Security:** check `runAsUser`, `runAsGroup`, read-only root
    filesystem, capabilities and filesystem permissions.
5.  **Resources:** inspect requests/limits, OOM kills, CPU throttling
    and ephemeral-storage limits.
6.  **Networking:** verify Service ports, targetPort, NetworkPolicies,
    DNS and dependency reachability.
7.  **Probes:** check readiness, liveness and startup probe paths,
    ports, timing and expected status codes.

Useful commands:

``` bash
kubectl describe pod <pod> -n <namespace>
kubectl logs <pod> -n <namespace> --all-containers
kubectl logs <pod> -n <namespace> --previous
kubectl get events -n <namespace> --sort-by=.lastTimestamp
kubectl get pod <pod> -n <namespace> -o yaml
```

I reproduce the Kubernetes constraints locally where possible, fix the
mismatch and validate in a non-production environment.

## 7. How would you recover if the Docker daemon becomes unavailable on a production server?

**Answer:**

I first establish impact: whether only Docker management is unavailable
or whether existing workloads are also affected. I check host health,
disk space, daemon logs and service status.

``` bash
systemctl status docker
journalctl -u docker --since "30 minutes ago"
df -h
df -i
free -h
ps aux | grep -E 'dockerd|containerd'
```

I check daemon configuration, storage-driver errors, full filesystems,
corrupted metadata, containerd health and recent upgrades. If workloads
are business-critical, I use the orchestrator or failover capacity to
maintain service. I restart Docker only after assessing impact and
following the production change procedure; a daemon restart can disrupt
management and, depending on configuration, workloads.

I do not delete `/var/lib/docker` or prune production data as a first
response. After restoring the daemon, verify containers, networks,
volumes, health checks and application traffic. Preserve logs and
investigate why the daemon failed.

------------------------------------------------------------------------

# 3. Kubernetes Operations & Troubleshooting

## 8. How do you troubleshoot CrashLoopBackOff, ImagePullBackOff and Pending pods?

**Answer:**

Start with:

``` bash
kubectl get pods -A -o wide
kubectl describe pod <pod> -n <namespace>
kubectl get events -n <namespace> --sort-by=.lastTimestamp
```

**CrashLoopBackOff** - Inspect current and previous container logs. -
Check exit code, OOMKilled, command/args, configuration, secrets and
dependencies. - Check probe failures and security-context permissions.

``` bash
kubectl logs <pod> -n <ns> --previous
kubectl describe pod <pod> -n <ns>
```

**ImagePullBackOff** - Verify image name, tag/digest and registry
availability. - Check `imagePullSecrets`, service account and registry
permissions. - Check node DNS, network access and architecture
compatibility.

**Pending** - Inspect scheduler events for insufficient CPU/memory,
taints, affinity, topology constraints, PVC binding or quota.

``` bash
kubectl describe pod <pod> -n <ns>
kubectl get nodes
kubectl get pvc -n <ns>
kubectl describe resourcequota -n <ns>
```

I fix the specific scheduling, image or application issue rather than
deleting pods repeatedly.

## 9. Explain readiness, liveness and startup probes. How do you debug failing probes?

**Answer:**

-   **Startup probe:** gives a slow-starting application time to
    initialize. Until it succeeds, Kubernetes does not run liveness or
    readiness checks.
-   **Readiness probe:** determines whether a pod should receive Service
    traffic. A failed readiness probe removes it from ready endpoints
    but does not restart it.
-   **Liveness probe:** detects a container that is stuck and should be
    restarted.

Example:

``` yaml
startupProbe:
  httpGet:
    path: /health/startup
    port: 8080
  periodSeconds: 10
  failureThreshold: 30

readinessProbe:
  httpGet:
    path: /health/ready
    port: 8080
  periodSeconds: 5
  failureThreshold: 3

livenessProbe:
  httpGet:
    path: /health/live
    port: 8080
  periodSeconds: 10
  failureThreshold: 3
```

I inspect pod events, container logs, probe path/port, response code,
timeout, initial delay and application startup time. I test the endpoint
from inside the pod. I avoid making liveness depend on every external
dependency because a temporary database outage could otherwise restart
every application replica.

## 10. How do you troubleshoot Kubernetes Service connectivity, DNS and pod-to-pod communication?

**Answer:**

I trace the request path: client → Ingress/Gateway/LoadBalancer →
Service → EndpointSlice → pod → dependency.

``` bash
kubectl get svc -n <ns>
kubectl describe svc <service> -n <ns>
kubectl get endpointslice -n <ns>
kubectl get pods -o wide -n <ns>
kubectl get networkpolicy -A
```

Check: - Service selector matches pod labels. - `port` and `targetPort`
are correct. - EndpointSlices contain ready pod IPs. - The application
listens on the expected interface and port, not only `127.0.0.1`. -
NetworkPolicies, CNI health, routing, firewall rules and node security
rules. - DNS resolution from a test pod:

``` bash
nslookup <service>.<namespace>.svc.cluster.local
```

-   Test TCP/HTTP connectivity from a temporary approved debug pod.

For DNS, inspect CoreDNS pods, logs, service IP and upstream resolution.
I isolate whether failure is DNS, routing, policy, endpoint selection or
application-level behavior before changing cluster networking.

## 11. How do you perform zero-downtime deployments and rollbacks using Helm?

**Answer:**

I use a rolling update with multiple replicas, correct readiness probes,
sufficient capacity and backward-compatible application/database
changes.

Before deployment:

``` bash
helm lint ./chart
helm template app ./chart -f values-prod.yaml
helm diff upgrade app ./chart -n <ns> -f values-prod.yaml
```

Deploy:

``` bash
helm upgrade --install app ./chart \
  -n <ns> -f values-prod.yaml \
  --wait --timeout 10m
```

Verify:

``` bash
kubectl rollout status deployment/<deployment> -n <ns>
kubectl get pods,svc,endpoints -n <ns>
helm history app -n <ns>
```

Rollback if necessary:

``` bash
helm rollback app <revision> -n <ns> --wait
```

I also check PodDisruptionBudgets, `maxUnavailable`/`maxSurge`,
readiness, capacity, traffic and database compatibility. A Helm rollback
does not automatically reverse database migrations or external side
effects, so those need a separate safe strategy.

## 12. A pod is stuck in CrashLoopBackOff. How do you troubleshoot it?

**Answer:**

I use a consistent evidence-first sequence:

``` bash
kubectl describe pod <pod> -n <ns>
kubectl logs <pod> -n <ns> --all-containers
kubectl logs <pod> -n <ns> --previous
kubectl get events -n <ns> --sort-by=.lastTimestamp
kubectl get pod <pod> -n <ns> -o yaml
```

I check exit code, termination reason, OOMKilled, command/arguments,
missing environment variables, ConfigMaps, Secret mounts, volume
permissions, probes, resource limits and dependency connectivity. If
logs are empty, the process may exit before logging initializes, fail
before stdout is configured, or be killed by the kernel. I inspect node
and container-runtime events as well. After correcting the root cause, I
verify stable restarts, readiness and application-level transactions.

## 13. A deployment rollout is stuck and new pods are not becoming Ready. What steps would you follow?

**Answer:**

``` bash
kubectl rollout status deployment/<name> -n <ns>
kubectl rollout describe deployment/<name> -n <ns>
kubectl get rs,pods -n <ns>
kubectl describe pod <new-pod> -n <ns>
kubectl get events -n <ns> --sort-by=.lastTimestamp
```

I check image pull failures, scheduling constraints, insufficient
capacity, failed probes, application startup, configuration, PVCs and
admission-policy errors. I compare the new ReplicaSet with the last
healthy one. If customer impact is increasing, I pause or roll back to
the known-good revision after confirming compatibility:

``` bash
kubectl rollout undo deployment/<name> -n <ns>
```

Then I verify Service endpoints and real requests. I do not delete the
old healthy ReplicaSet until the new version is proven stable.

## 14. Users cannot access an application exposed through Ingress. How do you debug it?

**Answer:**

I test each layer:

1.  DNS resolves to the expected public/private load balancer.
2.  Load balancer is healthy and listeners/ports are correct.
3.  Ingress controller is running and has accepted the Ingress resource.
4.  Ingress class, hostname, path and TLS secret are correct.
5.  Backend Service exists and has correct selector and ports.
6.  EndpointSlices contain ready pods.
7.  NetworkPolicies/firewalls allow the traffic.
8.  Application responds directly through the Service.

``` bash
kubectl get ingress -A
kubectl describe ingress <name> -n <ns>
kubectl get svc,endpointslice -n <ns>
kubectl get pods -n <ingress-namespace>
kubectl logs <controller-pod> -n <ingress-namespace>
```

Use `curl -v` and a Host header to separate DNS from routing issues. For
Gateway API, also inspect Gateway and HTTPRoute status/conditions and
parent references.

## 15. One worker node becomes NotReady. What actions would you take?

**Answer:**

I first check whether workloads are affected and whether other nodes
have enough capacity. I inspect:

``` bash
kubectl get nodes -o wide
kubectl describe node <node>
kubectl get pods -A -o wide
kubectl get events -A --sort-by=.lastTimestamp
```

On the node, check kubelet, container runtime, disk/inodes, memory, CPU,
network, certificates and control-plane connectivity:

``` bash
systemctl status kubelet
systemctl status containerd
journalctl -u kubelet --since "30 minutes ago"
df -h
free -h
```

I cordon the node to prevent new scheduling if it is unstable:

``` bash
kubectl cordon <node>
```

Draining is a separate, potentially disruptive action. I review PDBs,
local storage, DaemonSets and workload replicas before any drain.
Restore the underlying issue, uncordon only after validation, and
confirm pods and traffic are healthy.

## 16. etcd becomes unavailable in a production Kubernetes cluster. What is your recovery strategy?

**Answer:**

etcd is the Kubernetes control plane's source of truth. I treat this as
a high-severity control-plane incident and avoid improvising
data-directory changes.

1.  Determine whether one member or the entire quorum is unavailable.
2.  Check etcd member health, logs, disk latency/space, network
    connectivity, certificates and recent changes.
3.  Preserve data directories and logs; do not delete or reinitialize
    members casually.
4.  If quorum remains, restore failed members using the cluster's
    documented procedure.
5.  If quorum is lost, follow the control-plane vendor/distribution's
    disaster-recovery procedure and restore from a verified, consistent
    etcd snapshot.
6.  Validate API server access, node status, workloads and controllers
    after recovery.

Example checks on a secured control-plane host:

``` bash
ETCDCTL_API=3 etcdctl endpoint health
ETCDCTL_API=3 etcdctl endpoint status --write-out=table
ETCDCTL_API=3 etcdctl member list
```

Snapshot restore commands and flags are version- and topology-sensitive.
I use the exact Kubernetes distribution documentation, restore to the
intended members, and verify cluster identity and certificates. I
maintain scheduled encrypted snapshots and regularly test restoration.

## 17. Pods are frequently evicted due to resource pressure. How do you prevent this?

**Answer:**

I inspect pod events and node conditions for `MemoryPressure`,
`DiskPressure` and ephemeral-storage pressure. I identify which workload
is consuming resources.

``` bash
kubectl describe node <node>
kubectl get pods -A -o wide
kubectl top nodes
kubectl top pods -A --sort-by=memory
kubectl describe pod <pod> -n <ns>
```

Prevention: - Set realistic CPU/memory requests and limits based on
measurements. - Set ephemeral-storage requests/limits and manage logs
and temporary files. - Right-size nodes and autoscaling capacity. - Use
quotas and LimitRanges to prevent uncontrolled consumption. - Apply
PriorityClasses carefully to protect critical workloads. - Spread
replicas across nodes/zones and use PDBs for voluntary disruptions. -
Fix memory leaks and noisy-neighbor workloads. - Monitor image garbage
collection, container logs and node filesystem capacity.

Requests influence scheduling and QoS; limits constrain usage. I avoid
simply increasing every limit because that can move the pressure to the
node.

## 18. A service is running but application requests time out. How would you troubleshoot networking?

**Answer:**

A Running pod only proves that the container process is running; it does
not prove that requests can reach it or that it can respond.

I trace DNS → ingress/load balancer → Service → EndpointSlice → pod →
downstream dependency. I check application latency and logs, connection
pools, timeouts, DNS, NetworkPolicies, CNI, MTU, routing, firewall rules
and node health.

``` bash
kubectl get svc,endpointslice -n <ns>
kubectl get networkpolicy -A
kubectl exec -n <ns> <pod> -- sh -c 'getent hosts <service>'
kubectl logs -n <ns> <pod> --since=15m
```

Test the pod directly and through the Service to locate the failing hop.
Compare connection establishment time with application processing time.
Check saturation, packet drops, retransmits and downstream database/API
latency. Change one layer at a time and validate with a real request.

## 19. Your Kubernetes pods are Running and Ready, but users cannot access the application. How do you troubleshoot end-to-end?

**Answer:**

I verify the complete request path rather than assuming pod health means
application availability.

-   Confirm DNS and the resolved load-balancer address.
-   Test load-balancer listeners, health checks and firewall rules.
-   Inspect Ingress/Gateway class, host/path rules, TLS and route
    status.
-   Verify Service selectors, `port`/`targetPort` and EndpointSlices.
-   Confirm the application listens on the expected interface and port.
-   Check NetworkPolicies, CNI, node routing and external dependencies.
-   Test with `curl` at each layer and correlate request IDs in logs.

A useful isolation sequence is external URL → load balancer →
Ingress/Gateway → Service ClusterIP → pod IP → application dependency.
This identifies whether the issue is in DNS, edge routing, service
discovery, network policy or the application itself.

## 20. A production pod restarts with CrashLoopBackOff, but application logs show no errors. What would you investigate?

**Answer:**

I look beyond application logs:

-   `kubectl describe pod` for exit code, reason, events and probe
    failures.
-   `kubectl logs --previous` and logs from all containers/init
    containers.
-   OOMKilled status and kernel OOM events on the node.
-   Startup command, entrypoint, signal handling and process exit
    behavior.
-   Liveness/startup probe configuration and timeout thresholds.
-   Secret/config mount failures, filesystem permissions and read-only
    root filesystem.
-   Container runtime and node-level events.

If the process is killed before logging starts, there may be no
application error. I correlate restart timestamps with memory metrics,
node pressure and probe events, then reproduce with the same
image/configuration in a safe environment.

## 21. An application works with 5 replicas but fails at 50 replicas. How do you identify the bottleneck?

**Answer:**

I compare throughput, latency and error rate as replica count increases.
I look for a shared bottleneck rather than assuming Kubernetes cannot
scale.

Potential limits: - Database connection pool or maximum connections. -
External API rate limits. - Shared cache, queue or storage throughput. -
CPU/memory contention, network bandwidth or conntrack limits. - Lock
contention, hot partitions, shared files or session state. - Load
balancer connection limits and uneven traffic distribution.

I use metrics and distributed traces to locate where latency increases.
I test controlled load steps, inspect per-pod traffic and database pool
utilization, and check whether adding replicas increases total
throughput. Remedies may include connection pooling, caching,
partitioning, queue-based smoothing, sharding, rate limiting or
right-sizing. I define capacity limits and load-test before production
scaling.

## 22. HPA is configured, but pods do not scale during high CPU utilization. How do you debug?

**Answer:**

``` bash
kubectl get hpa -A
kubectl describe hpa <name> -n <ns>
kubectl top pods -n <ns>
kubectl get apiservices | grep metrics
kubectl get deployment <name> -n <ns> -o yaml
```

I check: - Metrics API/metrics-server availability and metric
freshness. - CPU requests: utilization-based HPA needs CPU requests to
calculate percentage. - HPA target, min/max replicas and scaling
behavior policies. - Whether the HPA reports `unknown` or missing
metrics. - Deployment selector and whether another controller is
changing replicas. - Cluster capacity, quotas, taints and scheduling
constraints. - Whether CPU is actually the bottleneck; custom/external
metrics may be more suitable.

I fix the metric source or configuration, test with controlled load, and
verify desired versus current replicas and actual throughput.

## 23. Kubernetes nodes become NotReady during peak traffic. How do you restore availability?

**Answer:**

I check node conditions and cluster-wide capacity first. I correlate the
incident with CPU/memory pressure, disk I/O, network saturation,
kubelet/container-runtime health and control-plane connectivity.

I protect service availability by ensuring healthy replicas exist on
other nodes/zones, confirming PDBs and available capacity, and
preventing new scheduling to unhealthy nodes where appropriate. I
inspect autoscaler limits and cloud quotas. I do not mass-drain nodes
during a capacity incident without understanding the impact. Restore
nodes one at a time, validate CNI/runtime/kubelet health, and rebalance
workloads after stability returns.

------------------------------------------------------------------------

# 4. CI/CD + Jenkins

## 24. How do you troubleshoot a Jenkins, GitHub Actions or GitLab pipeline that suddenly starts failing?

**Answer:**

I identify the first failing stage and compare the last successful run
with the failed run.

Check: - Recent code, workflow, shared-library, plugin or runner
changes. - Runner/agent availability, disk space, CPU, memory and
network. - Credentials, token expiry, IAM permissions and secret
scope. - Dependency registry availability, rate limits and artifact
retention. - Tool/runtime versions, cached dependencies and changed base
images. - Test failures, flaky tests, timeouts and parallel execution
conflicts.

I inspect full logs and reproduce the failing command on a clean runner.
I avoid disabling tests or security checks as a permanent fix. If a
release is blocked, I use the approved last-known-good artifact and
deployment process while fixing the pipeline.

## 25. How do you implement secure CI/CD pipelines with approvals, artifact management and rollback?

**Answer:**

I use least-privilege identities, protected branches, code review,
short-lived credentials or OIDC federation, and environment-specific
approvals. Secrets are stored in a secrets manager and never printed in
logs.

The pipeline should: 1. Lint and test source code. 2. Scan dependencies,
source and container images. 3. Build once and publish an immutable
artifact/image. 4. Generate provenance/SBOM and retain build metadata.
5. Promote the same artifact through environments. 6. Require approval
for production where policy requires it. 7. Deploy progressively and run
smoke/health checks. 8. Roll back to a known-good immutable artifact if
validation fails.

Use protected environments, signed artifacts where appropriate, audit
logs and deployment permissions. A rollback plan must also cover
database/schema compatibility and external changes.

## 26. Jenkins master/controller is running out of disk space. What actions would you take?

**Answer:**

First identify which filesystem is full and what is consuming it:

``` bash
df -h
df -i
du -xhd1 "$JENKINS_HOME" | sort -h
du -xhd1 /var/log | sort -h
```

Typical causes include old build histories, archived artifacts,
workspaces, tool caches, plugin logs and system logs. I configure build
discard policies, artifact retention, workspace cleanup after builds,
external artifact storage and log rotation. I check whether agents are
incorrectly building on the controller.

I preserve required audit/build artifacts and backups. I do not manually
delete Jenkins job directories or active workspace files while builds
are running. If urgent capacity is needed, I add storage through the
approved process and then implement retention policies.

## 27. Build artifacts are not uploading to the artifact repository. How do you investigate?

**Answer:**

I identify whether the failure is build output creation, packaging,
authentication, network transfer or repository acceptance.

Check: - Does the artifact exist and is its path/name correct? - Is the
upload URL/repository endpoint correct? - Are credentials valid and
authorized for the target repository? - Are DNS, TLS certificates, proxy
and firewall working? - Are file size limits, quotas or storage capacity
being exceeded? - Does the repository return 401/403, 404, 409, 413 or
5xx? - Are checksum, format, naming or duplicate-version rules violated?

I test a small upload with the same identity, inspect repository-side
logs and verify checksum after upload. I keep artifacts immutable and
avoid silently overwriting release versions.

------------------------------------------------------------------------

# 5. Cloud Architecture + AWS Scenarios

## 28. How do you design highly available AWS/Azure/GCP infrastructure across multiple Availability Zones?

**Answer:**

I remove single points of failure at each layer: - Deploy application
instances/pods across multiple zones. - Put a managed load balancer in
front of healthy targets. - Use autoscaling with health checks and
capacity headroom. - Use managed databases with
multi-zone/high-availability configuration and tested backups. - Use
redundant network paths, NAT capacity and resilient DNS. - Store static
assets in durable object storage and use a CDN where appropriate. - Use
infrastructure as code, monitoring, automated recovery and tested
runbooks.

I distinguish **multi-AZ high availability** from **multi-region
disaster recovery**. I define RTO/RPO with the business, choose
backup/restore, pilot-light, warm-standby or active-active accordingly,
and test failover rather than assuming it works.

## 29. Explain IAM roles, Auto Scaling, Load Balancers and disaster recovery strategies.

**Answer:**

-   **IAM roles:** grant temporary, scoped permissions to workloads,
    users or services. Prefer roles and federation over long-lived
    access keys; apply least privilege and audit access.
-   **Auto Scaling:** adjusts capacity based on demand or schedules. Use
    meaningful metrics, health checks, cooldown/warm-up and sufficient
    quotas/capacity.
-   **Load Balancers:** distribute requests across healthy targets.
    Configure listeners, TLS, target health checks, routing and
    timeouts.
-   **Disaster recovery:** choose a strategy based on RTO/RPO:
    -   Backup and restore: lowest standby cost, longest recovery.
    -   Pilot light: core data/services ready, scale on disaster.
    -   Warm standby: reduced-capacity environment ready.
    -   Multi-site/active-active: fastest recovery, highest
        complexity/cost.

Test backups and failover regularly and document decision authority and
rollback/failback.

## 30. An application behind an AWS Application Load Balancer returns HTTP 503. How would you troubleshoot?

**Answer:**

An ALB 503 commonly indicates that the load balancer has no usable
registered targets, or that target availability/forwarding is otherwise
failing. I verify the exact ALB access-log and target-group reason
rather than assuming one cause.

1.  Check target group health and registered targets.
2.  Verify target port, protocol, health-check path, success codes and
    timeout.
3.  Confirm application process is listening on the expected
    interface/port.
4.  Check security groups: ALB must reach targets on the target port;
    network ACLs and routes must permit traffic.
5.  Check target registration, Auto Scaling lifecycle and deployment
    health.
6.  Inspect ALB metrics: `HTTPCode_ELB_5XX_Count`,
    `HTTPCode_Target_5XX_Count`, `HealthyHostCount`,
    `UnHealthyHostCount`, `TargetResponseTime`, `RequestCount`.
7.  Review ALB access logs and application logs.

Useful AWS CLI:

``` bash
aws elbv2 describe-target-health --target-group-arn <target-group-arn>
aws elbv2 describe-rules --listener-arn <listener-arn>
```

I restore healthy capacity, correct the health check or networking
issue, and verify with real requests. I distinguish ELB-generated 503s
from target-generated 5xx responses.

## 31. Users report slow application performance. Which AWS services and metrics would you check first?

**Answer:**

I begin with user-facing latency, error rate and traffic, then follow
the request path.

-   **CloudWatch / ALB:** `TargetResponseTime`, request count, ELB
    4xx/5xx, target 5xx, healthy host count.
-   **EC2 / Auto Scaling:** CPU, memory (via agent), network throughput,
    disk I/O, status checks, instance capacity and scaling activity.
-   **ECS/EKS:** task/pod restarts, CPU/memory, throttling, pending
    pods, node pressure and HPA/cluster autoscaler events.
-   **RDS/Aurora:** CPU, freeable memory, connections, read/write
    latency, IOPS, queue depth, storage and database load/Performance
    Insights.
-   **ElastiCache:** CPU, memory, evictions, cache hit rate and
    connections.
-   **Route 53 / CloudFront / API Gateway:** DNS latency, cache hit
    ratio, origin latency, throttling and 4xx/5xx.
-   **X-Ray / OpenTelemetry:** trace slow spans and downstream
    dependency latency.

I correlate metrics by timestamp and deployment change. I avoid scaling
blindly until I know whether the bottleneck is compute, database,
network, cache or an external service.

## 32. An Auto Scaling Group does not launch new instances during traffic spikes. How do you investigate?

**Answer:**

I inspect the scaling policy, CloudWatch metric, alarm state and ASG
activity history.

``` bash
aws autoscaling describe-scaling-activities --auto-scaling-group-name <asg>
aws autoscaling describe-auto-scaling-groups --auto-scaling-group-names <asg>
```

Check: - Desired, minimum and maximum capacity. - Scaling policy,
metric, threshold, cooldown and instance warm-up. - CloudWatch alarm
state and missing/incorrect metrics. - EC2 service quotas and subnet IP
availability. - Launch template version, AMI, instance type and IAM
instance profile. - Security groups, key configuration, user data and
bootstrap failures. - Capacity shortages for the selected instance
type/AZ. - Health-check failures causing instances to be terminated.

I inspect EC2 launch failures and ASG activity messages, correct the
specific limit/configuration, and test scaling before the next peak.

## 33. An RDS database becomes unavailable during business hours. What is your recovery approach?

**Answer:**

I treat it as a critical incident and confirm the database status,
affected applications, recent changes and failover/backup posture.

1.  Check RDS events, CloudWatch metrics and AWS Health.
2.  Verify connectivity, DNS endpoint resolution, security groups,
    subnet groups and route paths.
3.  Check CPU, freeable memory, connections, storage, IOPS, latency and
    database logs.
4.  Determine whether it is a single-AZ failure, Multi-AZ failover,
    storage issue, connection exhaustion or database engine problem.
5.  If Multi-AZ is configured, verify failover and allow clients to
    reconnect using the DNS endpoint; avoid hardcoding instance IPs.
6.  If recovery is not possible, follow the approved restore/PITR or
    replica promotion plan, considering RPO and data consistency.
7.  Validate application transactions and reconcile any in-flight
    operations.

I communicate impact and recovery estimates, preserve logs, and only
perform a restore or promotion after understanding potential data loss
and obtaining the required authorization.

## 34. How would you perform a zero-downtime deployment in AWS?

**Answer:**

I choose a deployment strategy that keeps healthy capacity serving
traffic:

-   **Rolling:** replace instances/tasks gradually with minimum healthy
    capacity.
-   **Blue/green:** deploy a parallel environment, run tests, then shift
    ALB/Route 53 traffic.
-   **Canary:** send a small percentage of traffic to the new version,
    monitor, then increase.
-   **ECS/EKS:** use rolling or blue/green deployment controllers with
    health checks and capacity planning.

I use immutable artifacts, target-group health checks, readiness,
connection draining, backward-compatible database migrations and
automated smoke tests. I monitor latency, error rate and saturation
during rollout. If thresholds fail, shift traffic back or roll back to
the last known-good version. Database changes need expand-and-contract
compatibility because application rollback alone may not reverse schema
changes.

------------------------------------------------------------------------

# 6. Terraform + Infrastructure as Code

## 35. How do you handle Terraform state locking, state corruption and concurrent deployments?

**Answer:**

I store state remotely in a supported backend and enable its locking
mechanism. For example, an S3 backend can use state locking with a lock
file in supported Terraform versions/configurations; DynamoDB locking is
a legacy pattern and should be used only where required by the existing
setup.

I serialize production applies through CI/CD and restrict who can run
them. I never disable locking to bypass a contention error.

``` bash
terraform init
terraform plan
terraform state list
terraform state pull > state-backup.json
```

For suspected corruption, stop concurrent changes, preserve a copy of
the current state and backend version history, inspect the state and
backend logs, then recover from a known-good version only after
assessing infrastructure changes made since that version. `force-unlock`
is only for a confirmed stale lock and requires the correct lock ID and
team coordination.

## 36. How do you manage remote state, modules, workspaces and infrastructure drift?

**Answer:**

-   **Remote state:** use a secure backend with encryption, access
    controls, versioning and locking.
-   **Modules:** build reusable, versioned modules with clear
    inputs/outputs and validation. Pin module/provider versions.
-   **Workspaces:** useful for multiple instances of the same
    configuration, but not a substitute for strong production
    environment/account isolation.
-   **Environment separation:** commonly use separate
    accounts/subscriptions/projects, state keys and CI/CD permissions
    for dev, UAT and production.
-   **Drift:** run scheduled plans and review changes. Use cloud audit
    logs to identify manual modifications, then decide whether to
    import, update code or revert the manual change.

I never share one state file between unrelated environments or allow
unreviewed production applies.

## 37. Terraform state file is accidentally deleted. How would you recover?

**Answer:**

First stop all Terraform applies and prevent further state changes.
Determine whether the state object is actually deleted or merely
inaccessible.

1.  Check backend versioning, object history, backups and replication.
2.  Restore the latest known-good state version to the correct backend
    key using the backend's documented recovery procedure.
3.  Pull and securely preserve a copy before further operations.
4.  Run `terraform state list` and a normal `terraform plan`.
5.  Compare state with actual cloud resources and audit changes since
    the backup.
6.  If some resources are missing from state, import them carefully
    using their real resource IDs.
7.  Review the plan line by line before any apply.

Do not run `terraform apply` against an empty state; Terraform may
attempt to recreate existing resources. If no backup exists, reconstruct
state by importing resources into the configuration in a controlled
sequence.

## 38. Terraform apply fails midway after creating some resources. What are your next steps?

**Answer:**

Terraform state is updated as operations complete, so I do not assume
the whole apply was rolled back.

1.  Stop additional applies and preserve the error output and state.
2.  Check backend state and lock status.
3.  Inspect actual cloud resources and compare them with
    `terraform state list`.
4.  Run `terraform plan` to understand remaining changes.
5.  Fix the root cause, such as permissions, quota, dependency, API
    failure or invalid configuration.
6.  Import any created resource that exists remotely but is not tracked
    in state.
7.  Re-run plan and obtain review before applying.

I avoid manual deletion of partially created production resources unless
the recovery plan explicitly requires it.

## 39. Multiple engineers modify infrastructure simultaneously. How do you handle state locking?

**Answer:**

Use a remote backend with locking and one controlled production
deployment pipeline. Locking prevents simultaneous state operations, but
it does not replace code review or coordination.

I use pull requests, plan artifacts, environment approvals and a
serialized apply stage. If a lock is held, identify the owner and active
operation. Never remove a lock simply because it is inconvenient. Use
force-unlock only after confirming the original process is dead and the
lock is stale.

## 40. Someone manually changes AWS infrastructure and Terraform shows drift. How do you resolve it?

**Answer:**

I first identify what changed, who changed it and why using
CloudTrail/configuration history. Then I compare actual infrastructure
with Terraform configuration and state.

``` bash
terraform plan -refresh-only
terraform plan
```

I decide: - If the manual change is desired, update code to match and
review the plan. - If it is unauthorized/unwanted, revert it through
Terraform or an approved change. - If the resource is unmanaged, import
it and write matching configuration. - If a sensitive attribute changed,
verify provider behavior and replacement risk.

I do not blindly run apply. I review destructive actions, especially
database, networking and identity resources, and use lifecycle
protections where appropriate.

## 41. Terraform plan shows that a critical production database will be destroyed and recreated. What do you do?

**Answer:**

I stop the deployment. I do not approve or run the apply.

I inspect the plan and identify the attribute forcing replacement,
provider/version changes, resource address changes, `count`/`for_each`
key changes, immutable properties and state drift. I confirm the actual
database identity and backup/PITR status.

Possible fixes include correcting configuration, using a `moved` block
or state move for an address refactor, importing the correct resource,
or adjusting an immutable setting through a supported migration.
`prevent_destroy` can provide an additional guard, but it is not a
substitute for review. I require a second reviewer and a tested
migration/rollback plan before any production change.

## 42. Two engineers run Terraform apply simultaneously against production. How do you prevent conflicts?

**Answer:**

Use remote state locking, but also enforce a single-writer process.
Production applies should run through a protected CI/CD environment with
concurrency controls, approvals and an auditable plan. Engineers submit
changes through pull requests and review the same environment plan.
Locking serializes state operations; it does not guarantee two
independently prepared plans are semantically compatible, so plans
should be generated and applied in a controlled workflow.

## 43. Terraform apply accidentally modifies critical production resources. How do you recover safely?

**Answer:**

I stop further applies and assess the actual change using Terraform
state, cloud audit logs and service health. I identify which resources
changed and whether the change caused data loss or availability impact.

I restore service using the fastest safe method: revert the Terraform
code and apply a reviewed corrective plan, restore a resource from
backup/PITR if data was affected, or follow the cloud service's failover
procedure. I do not blindly restore an old state file because that can
make state inconsistent with real infrastructure. After recovery, I add
plan review, policy checks, approval gates, protected resources and
tested backups.

------------------------------------------------------------------------

# 7. Monitoring + Observability

## 44. How do you design monitoring and alerting using Prometheus and Grafana?

**Answer:**

I define service-level indicators first: availability, request rate,
error rate and latency, plus resource saturation. I instrument
applications and collect infrastructure/container metrics using
Prometheus exporters and service discovery.

-   Prometheus scrapes metrics and evaluates alert rules.
-   Alertmanager groups, routes, deduplicates and silences alerts.
-   Grafana provides dashboards and investigation views.
-   Labels identify service, environment, cluster and instance without
    creating unbounded cardinality.

I alert on user impact and actionable symptoms, not every transient
metric spike. I use SLO/error-budget alerts where appropriate, document
runbooks, test alert delivery and review noisy alerts. I retain metrics
at a suitable resolution and secure endpoints and dashboards.

## 45. How do you use OpenTelemetry, logs, metrics and traces to troubleshoot production incidents?

**Answer:**

I use the three telemetry signals together:

-   **Metrics** show what changed and when: latency, request rate,
    errors, saturation and resource utilization.
-   **Logs** provide detailed event context, ideally structured and
    correlated with trace/request IDs.
-   **Traces** show a request's path across services and identify slow
    or failing spans.

OpenTelemetry provides common instrumentation and collection for traces,
metrics and logs. I start with the affected service and time window,
find the relevant trace, inspect slow spans and correlate them with logs
and infrastructure metrics. I check sampling, clock synchronization,
missing instrumentation and cardinality/cost. This helps distinguish
application processing from database, network or third-party dependency
latency.

------------------------------------------------------------------------

# 8. Security + DevSecOps

## 46. How do you implement secrets management, IAM least privilege and vulnerability scanning in CI/CD?

**Answer:**

I store secrets in a managed secrets service such as AWS Secrets
Manager/SSM Parameter Store, Azure Key Vault or a dedicated vault.
Pipelines obtain short-lived credentials through workload identity/OIDC
federation where possible. I avoid hardcoded secrets, long-lived access
keys and printing secret values in logs.

IAM permissions are scoped to the exact resources and actions required,
separated by environment and reviewed periodically. I scan source,
dependencies, IaC and container images, define severity-based gates,
produce SBOMs and patch vulnerable base images. Secrets are rotated and
access is audited. Emergency exceptions are time-limited, approved and
tracked.

## 47. How do you integrate SAST, DAST, container scanning and dependency scanning into deployment pipelines?

**Answer:**

-   **SAST:** scans source code for insecure patterns before merge.
-   **Dependency/SCA scanning:** detects vulnerable third-party
    libraries and license risks.
-   **IaC scanning:** checks Terraform/Kubernetes configuration for
    misconfiguration.
-   **Container scanning:** scans OS and application packages in the
    built image.
-   **DAST:** tests a running application in a test/staging environment
    from the outside.

I place fast checks early and deeper tests before release. I define
severity thresholds, ownership and remediation SLAs, suppress only with
documented justification, and rescan after fixes. I publish
reports/SBOMs and ensure the exact tested immutable artifact is the one
promoted to production.

------------------------------------------------------------------------

# 9. Advanced Platform Engineering & NVIDIA-Style System Design

## 48. Explain your CI/CD pipeline and how you run large test suites fast.

**Answer:**

I use a pipeline with source validation, dependency caching, parallel
tests, build, security checks, artifact publication and progressive
deployment.

To make large test suites fast: - Split tests into independent shards
and run them in parallel. - Use dependency/build caches with safe cache
keys. - Run fast unit tests before expensive integration/end-to-end
suites. - Detect changed components in a monorepo and run only relevant
tests, while retaining scheduled full-suite runs. - Use ephemeral,
right-sized runners and avoid overloading shared agents. - Quarantine
flaky tests only with ownership and a plan to fix them. - Publish test
reports and timings to identify slow suites.

I optimize for reliable feedback, not merely skipping tests. I measure
queue time, execution time, cache hit rate and failure rate.

## 49. How do you build and optimize container images for GPU workloads?

**Answer:**

I separate build-time tooling from runtime, use a trusted
CUDA-compatible base image, pin compatible CUDA/cuDNN/framework versions
and include only required runtime libraries. I avoid embedding large
datasets, model checkpoints or credentials in the image unless there is
a deliberate immutable-artifact design.

I validate host driver and container CUDA compatibility, use multi-stage
builds, layer caching and `.dockerignore`, and scan/sign images. I test
the image on the target GPU architecture and verify GPU access at
runtime. For Kubernetes, I ensure the NVIDIA device plugin or GPU
Operator is correctly installed and that workloads request GPU
resources.

## 50. How do you schedule GPU workloads on Kubernetes?

**Answer:**

I expose GPUs to Kubernetes using the NVIDIA device plugin or NVIDIA GPU
Operator. GPU workloads request extended resources such as
`nvidia.com/gpu`, rather than treating GPUs as ordinary CPU/memory.

I use node labels, taints/tolerations, node affinity and topology
constraints to target compatible GPU nodes. I consider GPU model,
memory, driver/CUDA compatibility, NUMA locality and data locality. For
multi-GPU jobs, I account for topology and communication requirements. I
use quotas, priorities and fair scheduling to avoid one team
monopolizing the cluster. I monitor GPU utilization, memory,
temperature, power and job queue time.

## 51. How do you autoscale compute-heavy workloads cost-effectively?

**Answer:**

I scale on workload demand, not CPU alone. For batch/ML jobs, queue
depth, pending jobs, GPU utilization and job latency can be better
signals. I separate latency-sensitive services from interruptible batch
workloads, use right-sized instances and consider Spot/preemptible
capacity with checkpointing and retry.

I set minimum/maximum capacity, scale-down safeguards, warm pools where
justified and cluster autoscaler limits. I measure cost per job/request
and throughput. I avoid scaling faster than dependencies or data
pipelines can handle, and keep enough on-demand capacity for critical
workloads.

## 52. Explain your Terraform module structure for multi-account setups.

**Answer:**

I separate reusable modules from environment/account composition.
Modules represent capabilities such as VPC, IAM, Kubernetes, database
and observability, with typed inputs, outputs, validation and
versioning.

Each account/environment has its own root configuration, provider
alias/account context, backend state key and CI/CD permissions. I use
separate state and credentials per account, and assume roles through
short-lived federation. Shared organization-level policies are managed
separately from application workloads. I pin provider/module versions
and test modules before promotion.

## 53. How do you detect and fix configuration drift at scale?

**Answer:**

I run scheduled, read-only Terraform plans across environments and use
cloud configuration/audit services such as AWS Config and CloudTrail to
detect manual changes. I centralize results and alert on unexpected
drift.

For each drift event, I identify the actor, reason and impact. If the
change is desired, update IaC; if unauthorized, revert through
controlled IaC; if unmanaged, import it. I avoid automatic applies for
critical infrastructure. Policy-as-code, restricted console permissions,
pull-request workflows and periodic access reviews reduce future drift.

## 54. How do you manage secrets across many clusters?

**Answer:**

I use a central secrets-management system with cluster/workload identity
and short-lived authentication. Each workload receives only the secrets
it needs. Depending on the platform, secrets can be synced through an
external-secrets controller or mounted through a secrets CSI
driver/agent.

I separate environments and tenants, rotate credentials, audit reads,
encrypt in transit/at rest and avoid storing plaintext secrets in Git or
container images. I plan for central-vault outage, cache/lease behavior
and renewal failures. Cluster access is scoped through RBAC and network
policy.

## 55. How do you debug a Kubernetes node that keeps going NotReady?

**Answer:**

I inspect node conditions, kubelet events and runtime health, then
correlate with disk/memory pressure, network loss, certificate expiry
and control-plane reachability. I check CNI and container runtime logs,
kernel messages and recent node upgrades.

I cordon an unstable node, assess workload/PDB impact, restore node
health and only then return it to scheduling. If the node is
unrecoverable, replace it through the managed node-group or cluster
procedure rather than manually rebuilding it without preserving required
configuration.

## 56. How do you monitor GPU utilization and cluster health?

**Answer:**

I collect GPU metrics using NVIDIA DCGM Exporter and scrape them with
Prometheus. Useful metrics include GPU utilization, memory used,
temperature, power, clocks, ECC errors and XID errors. I combine them
with Kubernetes node/pod metrics, pending GPU requests, job queue time,
node readiness and application throughput.

Grafana dashboards should show utilization by GPU model, node, namespace
and workload. Alerts should detect GPU errors, thermal/power anomalies,
low utilization with a long queue, and unavailable GPU capacity. I
correlate telemetry with driver, CUDA and workload versions.

## 57. Design a multi-region, multi-cluster platform for heavy compute.

**Answer:**

I start with business requirements: RTO/RPO, data residency, latency,
throughput, GPU/CPU types and cost. I deploy independent Kubernetes
clusters in multiple regions and zones, with consistent
infrastructure-as-code and GitOps.

A global traffic layer routes user-facing services to healthy regions. A
workload control plane or scheduler places jobs based on capacity, GPU
type, data locality, priority and cost. Container images and artifacts
are replicated securely. Data replication strategy depends on
consistency and recovery requirements; not every dataset should be
synchronously replicated.

I standardize observability, identity, policy, secrets, network
segmentation and cluster lifecycle. I test regional failure, cluster
upgrades and restore procedures. Avoid a single global control-plane
dependency that can take down all regions.

## 58. How do you implement GitOps across a fleet of clusters?

**Answer:**

I keep desired state in version-controlled repositories and use a GitOps
controller such as Argo CD or Flux in each cluster. I use a hierarchy:
shared platform baseline, cluster/region configuration and
application/environment overlays.

I define ownership boundaries, sync waves/dependencies, health checks,
RBAC and promotion policies. Production changes go through pull requests
and approvals. I monitor reconciliation drift and controller health.
Secrets are delivered securely through an external secrets mechanism
rather than committed to Git. I canary changes to a small cluster group
before fleet-wide rollout and keep rollback commits/releases available.

## 59. How do you do zero-downtime upgrades of large Kubernetes clusters?

**Answer:**

I review version-skew policy, API deprecations, CNI/CSI/ingress
compatibility and workload disruption budgets. I back up
etcd/control-plane state where applicable and test the upgrade in a
representative non-production cluster.

I upgrade control-plane components according to the distribution's
supported sequence, then roll worker nodes gradually. I use surge
capacity or replacement node pools, cordon/drain one node at a time,
respect PDBs and verify workloads after each batch. I monitor error
rate, scheduling, DNS, storage and network behavior. I pause or roll
back/replace nodes if health degrades. A PDB alone cannot guarantee zero
downtime if the application has insufficient replicas or stateful
dependencies.

## 60. How do you handle a noisy-neighbor problem on shared nodes?

**Answer:**

I identify the noisy workload through per-container CPU, memory, disk
and network metrics. I use requests/limits, QoS classes, quotas,
PriorityClasses and topology/affinity rules to isolate critical
workloads. For stronger isolation, use dedicated node pools,
taints/tolerations or separate clusters.

For GPU workloads, schedule by GPU model and memory, and avoid
oversubscription unless the sharing mechanism is explicitly supported
and measured. I validate fairness and performance under load, then
establish tenant-level capacity and cost reporting.

## 61. How do you reduce cloud cost without hurting throughput?

**Answer:**

I establish a baseline for cost per request/job and identify idle or
overprovisioned resources. I right-size compute, autoscale on useful
metrics, use commitments for predictable baseline demand and Spot
capacity for fault-tolerant work. I optimize storage tiers, snapshots,
data transfer, NAT and log retention.

For Kubernetes, I review requests, bin-packing, node-pool sizing and
unused persistent volumes/load balancers. For GPU workloads, I improve
utilization through batching, scheduling and checkpointing. I test
changes against throughput and latency SLOs and avoid cost reductions
that create a capacity or resilience risk.

## 62. How do you secure the software supply chain for container images?

**Answer:**

I use trusted base images, pinned dependencies, isolated build runners
and least-privilege CI identities. I scan source, dependencies, IaC and
images; generate an SBOM; sign images and verify signatures at
deployment. I retain provenance linking source commit, builder,
dependencies and artifact digest.

Images are promoted by immutable digest, not mutable tags alone.
Registry access is restricted, secrets are not passed through build
layers, and base images are rebuilt regularly. Admission policies can
enforce signature and vulnerability requirements.

## 63. Walk me through a large-scale incident you resolved.

**Answer (sample structure---replace with a real incident):**

"During a production release, users began seeing elevated 5xx responses.
I declared an incident, identified affected services and regions, and
checked the deployment timeline, load balancer health, pod readiness and
application metrics. The evidence showed that the new version was
failing its readiness checks because of a configuration mismatch. I
stopped further rollout and restored the last known-good version to
recover traffic. I verified target/pod health and successful user
transactions, then communicated recovery to stakeholders. Afterward, we
added configuration validation, a canary stage, automated smoke tests
and an alert on error-rate increase."

Do not claim this exact incident as your own unless it happened. Use the
same structure with your genuine example, measurable impact and your
individual contribution.

------------------------------------------------------------------------

# 10. Additional Scenario Answer Patterns

## 64. A production Kubernetes application becomes unavailable immediately after deployment. How do you investigate and restore service?

**Answer:**

Check the deployment history and correlate the outage start with the
release. Inspect rollout status, ReplicaSets, pods, events, readiness,
Service endpoints, Ingress/Gateway status and application logs. Compare
the new manifest/configuration/image with the last healthy release. If
impact is ongoing, roll back to the known-good revision after checking
database compatibility. Verify external requests and key business
transactions, then preserve evidence and add a deployment gate to
prevent recurrence.

## 65. Write a Bash script to check application health and automatically restart a failed service.

**Answer:**

For a Linux host using systemd, a simple example is:

``` bash
#!/usr/bin/env bash
set -euo pipefail

SERVICE="${SERVICE:-myapp}"
HEALTH_URL="${HEALTH_URL:-http://127.0.0.1:8080/health}"
TIMEOUT="${TIMEOUT:-5}"

if ! curl --fail --silent --show-error \
     --max-time "$TIMEOUT" "$HEALTH_URL" >/dev/null; then
  logger -t app-health "Health check failed for ${SERVICE}; attempting restart"

  if systemctl restart "$SERVICE"; then
    sleep 5
    if curl --fail --silent --max-time "$TIMEOUT" "$HEALTH_URL" >/dev/null; then
      logger -t app-health "${SERVICE} recovered"
      exit 0
    fi
  fi

  logger -p user.err -t app-health "${SERVICE} did not recover"
  exit 1
fi
```

Run with least privilege and an appropriate systemd policy. In
production, add restart rate limits, alerting, dependency checks and
monitoring. For Kubernetes-managed applications, prefer
readiness/liveness probes and controller reconciliation rather than
running a host script that fights Kubernetes.

## 66. How do you troubleshoot a Jenkins build that cannot upload an artifact?

**Answer:**

Confirm the artifact was created and the upload step references the
correct path. Check repository URL, credentials/permissions, TLS, proxy,
network connectivity, quota, file-size limits, naming rules and
repository response code. Review both pipeline and repository logs, test
with a small file using the same identity, and verify the uploaded
checksum. Keep release artifacts immutable and make retries idempotent.

------------------------------------------------------------------------

# Interview Answer Framework

For most scenario questions, use this six-step structure:

1.  **Impact:** What users/services/environments are affected? Is it
    regional or global?
2.  **Evidence:** Which metrics, logs, events and recent changes are
    relevant?
3.  **Isolation:** Which layer is failing---application, container,
    orchestration, network, cloud service or IaC?
4.  **Mitigation:** What is the safest way to restore service without
    increasing risk?
5.  **Validation:** How will you prove the service is healthy from the
    user's perspective?
6.  **Prevention:** What monitoring, automation, testing or process
    change prevents recurrence?

## Production safety reminders

-   Prefer read-only commands before making changes.
-   Do not delete Kubernetes pods, PVCs, etcd data or Terraform state as
    a first troubleshooting step.
-   Do not run `terraform apply` when the plan contains unexpected
    replacement or destruction.
-   Check PodDisruptionBudgets and capacity before draining nodes.
-   Preserve logs, state backups and audit evidence during incidents.
-   Confirm database backup/PITR and data-loss implications before
    restore or failover.
-   Communicate impact, actions and recovery status clearly.

**End of guide.**
