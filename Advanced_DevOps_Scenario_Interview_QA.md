# Advanced DevOps Scenario-Based Interview Questions & Answers

## How to Use These Answers

For scenario-based DevOps interviews, answer in a practical,
first-person style. A good structure is:

> **Identify the problem → Check evidence/metrics/logs → Make the safest
> change → Verify the result → Add preventive measures.**

The answers below are written so they can be spoken directly to the
interviewer.

------------------------------------------------------------------------

# 1. Terraform Drift in Production

### Question

A manual change was made to an AWS resource in production causing
Terraform drift, and the next Terraform apply wants to recreate the
resource. How would you reconcile the infrastructure without downtime?

### Answer

> First, I would not directly run `terraform apply`, because the plan is
> showing that Terraform wants to recreate a production resource.
>
> I would first run `terraform plan` and identify exactly which resource
> has drifted and what attributes have changed.
>
> Then I would compare the actual AWS resource with the Terraform
> configuration and Terraform state.
>
> If the manual change is actually the desired production configuration,
> I would update the Terraform code to reflect that change and then run
> `terraform plan` again to make sure there is no unexpected
> replacement.
>
> If the manual change was incorrect, I would decide whether to revert
> the AWS resource or import/update the resource appropriately in
> Terraform.
>
> Before making any production change, I would make sure the plan shows
> no destructive action. If Terraform still wants to destroy and
> recreate the resource, I would investigate the specific attribute
> causing replacement rather than blindly applying it.
>
> My main priority would be to bring Terraform state and the actual
> infrastructure back into alignment without causing downtime.
>
> Going forward, I would also restrict manual production changes through
> IAM permissions and follow infrastructure-as-code practices so that
> Terraform remains the source of truth.

### Useful Commands

``` bash
terraform plan
terraform state list
terraform state show <resource>
terraform import <resource> <resource-id>
```

------------------------------------------------------------------------

# 2. Kubernetes Namespace Networking Issue

### Question

Pods in one Kubernetes namespace can communicate internally, but pods in
another namespace cannot reach them even though services are exposed.
How would you troubleshoot this?

### Answer

> I would troubleshoot this layer by layer instead of immediately
> assuming that the Kubernetes Service is the problem.
>
> First, I would verify that the destination pods and Service are
> healthy.

``` bash
kubectl get pods -n <namespace>
kubectl get svc -n <namespace>
kubectl get endpoints -n <namespace>
```

> Then I would check whether the Service selector actually matches the
> destination pods.
>
> From the source namespace, I would test DNS resolution:

``` bash
nslookup <service>.<namespace>.svc.cluster.local
```

> Then I would test connectivity directly from a pod:

``` bash
curl http://<service>.<namespace>.svc.cluster.local:<port>
```

> If DNS works but connectivity fails, I would check NetworkPolicies
> because Kubernetes NetworkPolicy can allow communication within one
> namespace while blocking traffic from another namespace.
>
> I would also check the CNI, such as Calico, and inspect whether there
> are any network policies or routing issues.
>
> My troubleshooting flow would be:
>
> **Pod → Service → Endpoints → DNS → NetworkPolicy → CNI/networking →
> application port.**
>
> This helps me identify exactly which layer is causing the
> communication failure rather than changing multiple things at once.

------------------------------------------------------------------------

# 3. Kubernetes Pulling an Old Docker Image

### Question

The CI/CD pipeline successfully builds and deploys a container image,
but Kubernetes keeps pulling the old image. What could be causing this?

### Answer

> The first thing I would check is the image tag being deployed.
>
> If the pipeline is repeatedly using something like `latest` or reusing
> the same tag, Kubernetes may not pull the newly built image depending
> on the `imagePullPolicy` and whether the image already exists on the
> node.
>
> I would first check the Deployment:

``` bash
kubectl describe deployment <deployment> -n <namespace>
```

> Then I would check the actual image configured for the pod:

``` bash
kubectl get pod <pod> -n <namespace> \
  -o jsonpath='{.spec.containers[*].image}'
```

> I would also check the image ID:

``` bash
kubectl describe pod <pod> -n <namespace>
```

> My preferred solution is to use immutable image tags, for example:

``` text
app:v1.0.1
app:v1.0.2
app:git-8f32a1
```

> Then every deployment points to a unique image version.
>
> I would also verify that the CI/CD pipeline is actually updating the
> Kubernetes manifest and that Argo CD or another deployment tool is not
> reverting the change.
>
> So I would check:
>
> 1.  Image tag generated by CI/CD
> 2.  Image pushed to registry
> 3.  Image referenced by Deployment
> 4.  `imagePullPolicy`
> 5.  Actual image running in the pod
> 6.  GitOps synchronization, if applicable
>
> This gives me confidence that the new image is actually deployed.

------------------------------------------------------------------------

# 4. HPA Not Scaling

### Question

The application is under heavy traffic but Kubernetes HPA is not scaling
pods even though CPU usage is above the configured threshold. How would
you debug it?

### Answer

> First, I would verify whether the HPA itself is healthy.

``` bash
kubectl get hpa -n <namespace>
kubectl describe hpa <hpa-name> -n <namespace>
```

> I would specifically check the current CPU utilization, target
> utilization, minimum and maximum replicas, and whether there are any
> HPA events or errors.
>
> Then I would verify that Kubernetes has metrics available:

``` bash
kubectl top pods -n <namespace>
kubectl top nodes
```

> If `kubectl top` is not working, I would check the Metrics Server.
>
> I would also verify that the Deployment has CPU requests configured
> because HPA utilization is normally calculated relative to the CPU
> request.

Example:

``` yaml
resources:
  requests:
    cpu: "250m"
```

> I would then check whether the HPA is pointing to the correct
> Deployment and whether the metrics configuration is correct.
>
> I would also verify whether the HPA has already reached `maxReplicas`.
>
> My troubleshooting flow would be:
>
> **HPA → Metrics Server → CPU requests → Deployment → current replicas
> → maxReplicas → HPA events.**
>
> Once I identify the issue, I would fix the configuration and monitor
> the scaling behavior under load.

------------------------------------------------------------------------

# 5. Terraform Module Dependency Conflicts

### Question

Multiple Terraform modules across repositories depend on each other and
a deployment fails because of dependency conflicts. How would you design
a reliable deployment strategy?

### Answer

> I would avoid tightly coupling all Terraform modules together.
>
> I would define clear ownership and dependency boundaries between
> modules.
>
> For example, I might separate infrastructure into layers such as:
>
> **Networking → Security → Shared Services → Compute → Application**
>
> Each layer would expose only the outputs required by the next layer.
>
> I would also version reusable Terraform modules instead of allowing
> different repositories to consume uncontrolled changes.
>
> In CI/CD, I would run:
>
> 1.  `terraform fmt`
> 2.  `terraform validate`
> 3.  `terraform plan`
> 4.  Review/approval
> 5.  `terraform apply`
>
> I would also use remote state with locking so that two pipelines
> cannot modify the same infrastructure simultaneously.
>
> The main goal would be to make dependencies explicit, versioned and
> predictable rather than allowing one repository to unexpectedly break
> another.

------------------------------------------------------------------------

# 6. EKS Nodes Joined but Pods Are Pending

### Question

New worker nodes successfully join an EKS cluster, but pods remain in
Pending state. What could be the possible reasons?

### Answer

> The fact that the nodes have joined the cluster doesn't necessarily
> mean they can schedule workloads.
>
> First, I would check the pod's scheduling events:

``` bash
kubectl describe pod <pod> -n <namespace>
```

> The events usually give me the first indication of the problem.
>
> Then I would check the nodes:

``` bash
kubectl get nodes
kubectl describe node <node>
```

> I would check several things:
>
> -   Node CPU and memory capacity
> -   Node taints
> -   Pod tolerations
> -   Node selectors
> -   Affinity rules
> -   Resource requests
> -   Available IP addresses in the VPC
> -   EKS networking/CNI status
> -   Pod quotas
>
> For example, if the node has a taint and the pod doesn't have the
> corresponding toleration, the scheduler will not place the pod there.
>
> Similarly, if the pod requests 4 CPUs but the node only has 2 CPUs
> available, it will remain Pending.
>
> So I would always start with `kubectl describe pod` because the
> scheduler events usually tell me why the pod cannot be scheduled.

------------------------------------------------------------------------

# 7. Zero-Downtime Microservice Deployment

### Question

You need to deploy a new version of a critical microservice in
production without downtime. Which deployment strategy would you
implement?

### Answer

> For a critical production microservice, I would normally prefer a
> rolling deployment, provided the application supports backward
> compatibility.
>
> I would run multiple replicas and configure Kubernetes with a proper
> readiness probe, liveness probe and rolling-update strategy.
>
> For example:

``` yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 0
    maxSurge: 1
```

> The readiness probe is particularly important because Kubernetes
> should only send traffic to a pod after the new version is actually
> ready.
>
> For a higher-risk release, I would consider a blue-green deployment or
> canary deployment.
>
> With canary deployment, I would initially send a small percentage of
> traffic to the new version, monitor metrics and logs, and gradually
> increase the traffic if everything looks healthy.
>
> I would monitor:
>
> -   Error rate
> -   Response time
> -   CPU/memory
> -   Application logs
> -   HTTP 4xx/5xx
> -   Business metrics
>
> If there is an issue, I would immediately roll back to the previous
> version.
>
> So the strategy depends on the risk: **rolling for normal releases,
> canary or blue-green for critical/high-risk releases.**

------------------------------------------------------------------------

# 8. Vulnerable Dependency Found in Docker Image

### Question

A vulnerable dependency was detected in a Docker image after deployment.
What steps would you take?

### Answer

> First, I would assess the severity and determine whether the
> vulnerability is actually exploitable in our environment.
>
> If it is a critical vulnerability, I would consider taking the
> affected workload out of service or reducing exposure depending on the
> risk.
>
> Then I would identify whether the vulnerability comes from the
> application dependency or the base image.
>
> I would update the dependency or base image, rebuild the Docker image
> and run a security scan again.
>
> For example, I would use tools such as Trivy in the CI/CD pipeline:

``` bash
trivy image <image>
```

> I would make security scanning part of the pipeline rather than
> waiting until after deployment.
>
> The pipeline would ideally follow something like:
>
> **Build → Unit Test → Security Scan → Image Push → Deployment →
> Verification**
>
> I would also avoid deploying images with critical vulnerabilities
> unless there is an approved exception.
>
> Finally, I would redeploy the patched image using a new immutable tag
> and monitor the production workload.

------------------------------------------------------------------------

# 9. Intermittent Failures Across Microservices

### Question

Users report intermittent failures across multiple microservices, but
individual service logs look normal. How would you identify the root
cause?

### Answer

> Because multiple services are affected and individual application logs
> look normal, I would suspect a shared dependency or infrastructure
> component rather than immediately focusing on one application.
>
> I would start by identifying exactly when the failures occur and
> correlate requests across services.
>
> I would check:
>
> -   Ingress/load balancer
> -   DNS
> -   Network connectivity
> -   Service-to-service communication
> -   Database
> -   Redis/cache
> -   API gateway
> -   CPU and memory
> -   Network latency
> -   Connection limits
>
> I would also use centralized logging and monitoring to correlate the
> failures.
>
> For example, with Prometheus and Grafana I could check error rates,
> latency, CPU, memory and network metrics across the services.
>
> If distributed tracing is available, I would trace one failed request
> across the entire request path.
>
> I would compare successful and failed requests and look for a common
> dependency.
>
> The key here is that **normal individual application logs don't
> necessarily mean the overall system is healthy**. The problem could
> exist between the services or in a shared infrastructure component.

------------------------------------------------------------------------

# 10. Multi-Region AWS Disaster Recovery

### Question

Your production workload runs in a single AWS region, and management
asks you to design a multi-region disaster recovery strategy with
minimal downtime. How would you approach it?

### Answer

> First, I would understand the business requirements, especially the
> required RTO and RPO.
>
> RTO tells me how quickly the application needs to recover, while RPO
> tells me how much data loss is acceptable.
>
> Based on those requirements, I would choose the appropriate disaster
> recovery architecture.
>
> I would provision the infrastructure in a second AWS region using
> Terraform so that both regions are reproducible.
>
> For the application layer, I could maintain a warm standby or
> active-passive environment depending on the required RTO.
>
> For the database, I would use an appropriate cross-region replication
> or managed database disaster recovery mechanism.
>
> I would also replicate required storage and configuration and make
> sure secrets are available in the DR region.
>
> For traffic management, I could use Route 53 health checks and
> failover routing so that traffic moves to the DR region when the
> primary region becomes unavailable.
>
> I would also monitor both regions and regularly test the failover
> process.
>
> My approach would be:
>
> **Define RTO/RPO → Build secondary region → Replicate data → Automate
> infrastructure → Configure failover → Monitor → Regularly test DR.**
>
> I would not consider the DR strategy complete just because the
> infrastructure exists. I would perform actual DR drills to verify that
> the application can be restored within the required RTO and with the
> expected RPO.

------------------------------------------------------------------------

# Quick Interview Framework

For scenario-based DevOps questions, use this structure:

> **"First, I would identify the problem. Then I would check the
> relevant metrics, logs and configuration. Once I identify the root
> cause, I would make the smallest safe change. After that, I would
> verify the result and finally put a preventive measure in place so
> that the issue doesn't happen again."**

This approach demonstrates:

-   Production troubleshooting
-   Root-cause analysis
-   Safe change management
-   Monitoring and observability
-   Automation
-   Rollback planning
-   Preventive engineering
-   Infrastructure-as-code practices
