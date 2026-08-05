# DevOps Interview Q&A (5 Years Experience)

## 1. How would you troubleshoot a deployment that succeeded but users are receiving 503 errors?

**Answer (Interview Style):** I follow the request path from the user to
the application.

-   Verify whether the application pods are Running and Ready.
-   Check Kubernetes Service endpoints (`kubectl get endpoints`).
-   Verify Ingress/Load Balancer configuration.
-   Check application logs and ingress controller logs.
-   Test connectivity from inside the cluster.
-   Compare the new deployment with the previous working version.
-   Roll back immediately if production is impacted while continuing
    RCA.

------------------------------------------------------------------------

## 2. A Terraform apply failed after provisioning half the infrastructure. How would you recover safely?

**Answer:** - Never delete resources manually. - Run `terraform plan` to
compare state and infrastructure. - Fix the root cause (permissions, API
limits, configuration). - Use `terraform import` if resources exist
outside the state. - Re-run `terraform apply`. - Verify the final
infrastructure and document the incident.

------------------------------------------------------------------------

## 3. Your CI/CD pipeline takes 25 minutes. How would you reduce it to under 5 minutes?

**Answer:** - Run jobs in parallel. - Cache dependencies and Docker
layers. - Build only changed services. - Skip unnecessary tests on
feature branches. - Use pre-built base images. - Deploy only affected
components. - Measure every stage before optimization.

------------------------------------------------------------------------

## 4. How would you perform zero-downtime Kubernetes cluster upgrades?

**Answer:** - Upgrade control plane first. - Upgrade node pools one by
one. - Use rolling node drain. - Ensure multiple replicas. - Configure
PodDisruptionBudgets and readiness probes. - Monitor application health
during upgrade. - Keep rollback plan ready.

------------------------------------------------------------------------

## 5. How would you design a rollback strategy if deployment fails?

**Answer:** - Keep previous container images. - Use rolling or
blue-green deployments. - Automate rollback when health checks fail. -
Restore database only if schema changes require it. - Verify application
after rollback.

------------------------------------------------------------------------

## 6. Application latency suddenly increased after a release. What is your debugging approach?

**Answer:** - Confirm whether latency started after deployment. -
Compare application metrics. - Check CPU, memory and network usage. -
Review logs and distributed tracing. - Compare old vs new
configuration. - Roll back if customer impact is high. - Perform RCA
after stabilization.

------------------------------------------------------------------------

## 7. How do you manage secrets across multiple Kubernetes clusters?

**Answer:** - Store secrets in Vault or cloud secret managers. - Never
commit secrets to Git. - Inject secrets during deployment. - Rotate
secrets regularly. - Enable RBAC and audit logging. - Encrypt Kubernetes
Secrets at rest.

------------------------------------------------------------------------

## 8. How do you investigate intermittent pod restarts when logs show no obvious errors?

**Answer:** - Check `kubectl describe pod`. - Review previous container
logs (`--previous`). - Look for OOMKilled or CrashLoopBackOff. - Verify
liveness/readiness probes. - Check node events and resource pressure. -
Review application exit codes.

------------------------------------------------------------------------

## 9. Cloud bill increased by 40%. How would you investigate?

**Answer:** - Identify which services increased spending. - Compare with
previous month's usage. - Check idle resources. - Analyze storage, data
transfer and compute. - Review autoscaling. - Purchase Reserved
Instances/Savings Plans where appropriate. - Enable cost alerts and
tagging.

------------------------------------------------------------------------

## 10. Explain the most challenging production incident you've handled.

**Sample Answer:** During a production deployment, users received
502/503 errors because the Ingress pointed to pods that were not Ready.

I immediately rolled back the deployment to restore service. Then I
investigated pod readiness, application logs, and health checks. The
issue was caused by a configuration mismatch introduced in the new
release.

After fixing the configuration, I added automated health checks,
deployment validation, and monitoring alerts to prevent recurrence.

------------------------------------------------------------------------

## 11. If you had to redesign your current DevOps platform, what would you do differently?

**Answer:** - GitOps using ArgoCD. - Infrastructure as Code using
Terraform. - Centralized secrets using Vault. - Standard CI/CD
templates. - Better observability using Prometheus, Grafana, Loki and
OpenTelemetry. - Policy enforcement using Kyverno/OPA. - Automated
security scanning. - Progressive delivery using Argo Rollouts.

# Final Tip

Always answer with: 1. Immediate action. 2. Investigation steps. 3. Root
cause. 4. Permanent fix. 5. Lessons learned.
