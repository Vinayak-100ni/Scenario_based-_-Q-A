### CI/CD + GitOps Flow using Argo CD
```
Code Commit
Developer pushes code to Git repo
Test Code
CI tool (like Jenkins) runs tests
Build Image
Create Docker image from code
Push to Container Repo
Push image to registry (e.g., Docker Hub / Azure Container Registry)
Update Config Repo
Update Kubernetes YAML (image tag change)
Argo CD Sync
Argo CD detects change and syncs
Deploy to K8s
Application deployed to Kubernetes cluster
```
#### 1. What is Argo CD?

```Answer:
Argo CD is a GitOps-based continuous delivery tool for Kubernetes.
It syncs application state from Git repository to Kubernetes cluster automatically.

👉 Git = source of truth
👉 Argo CD = ensures cluster matches Git
```
#### 2. What is GitOps?
```
Answer:
GitOps is a practice where:

Git stores desired infrastructure & app configs
Changes happen via pull requests
Tools like Argo CD apply them to cluster
```
#### 3. Argo CD Architecture
```
Main Components:

API Server – UI, CLI, REST
Repository Server – fetches Git repo
Application Controller – compares desired vs live state
Redis – caching

👉 Interview Tip: Always mention reconciliation loop
```
🔹 4. What is an Application in Argo CD?
```
Answer:
An Application is a custom resource (CRD) that defines:

Git repo
Path
Target cluster
Sync policy
```

🔹 5. What is Sync in Argo CD?

```
Answer:
Sync means applying Git state to Kubernetes cluster.

Types:

Manual Sync
Auto Sync
```
🔹 6. Auto Sync vs Manual Sync
```
Feature	Auto Sync	Manual Sync
Trigger	Automatic	User
Use case	Prod pipelines	Dev/Test
```
🔹 7. What is Drift Detection?
```

Answer:
Argo CD detects if cluster state ≠ Git state.

👉 If mismatch → OutOfSync
```

🔹 8. What is Self-Healing?
```

Answer:
If enabled, Argo CD automatically restores changes if someone manually modifies cluster.
```
🔹 9. What is Pruning?
```

Answer:
Deletes resources that exist in cluster but not in Git.
```
🔹 10. What is Sync Policy?
```

Answer:
Defines how sync happens:

syncPolicy:
  automated:
    prune: true
    selfHeal: true
```
🔹 11. What is Argo CD Rollback?
```

Answer:
Rollback to previous Git commit version.
```
🔹 12. How Argo CD Works (Flow)
```

Developer pushes code → Git
Argo CD detects change
Compares with cluster
Syncs automatically
```
🔹 13. What is ApplicationSet?
```

Answer:
Used to manage multiple applications dynamically.

👉 Example:

Multi-cluster deployment
Multi-environment (dev, qa, prod)
```
🔹 14. What is Helm in Argo CD?
```

Answer:
Argo CD supports Helm charts for templating.

👉 You can deploy Helm charts directly from Git.
```
🔹 15. What is Kustomize?
```

Answer:
A Kubernetes-native configuration management tool supported by Argo CD.
```
🔹 16. Difference: Argo CD vs Jenkins
```

Feature	Argo CD	Jenkins
Type	CD Tool	CI/CD
Approach	GitOps	Pipeline
Deployment	Pull-based	Push-based
```
🔹 17. Argo CD Ports
```

UI/API Server → 443 / 80 (default via ingress)
Internal communication → cluster-based
```
🔹 18. How to Access Argo CD UI
```

kubectl port-forward svc/argocd-server -n argocd 8080:443
```
🔹 19. How to Login Argo CD CLI?
```

argocd login <server>
```
🔹 20. What is Argo CD Sync Status?
```

Synced
OutOfSync
Unknown
```

🔹 21. What is Health Status?
```

Healthy
Degraded
Progressing
```

🔹 22. What is Resource Hooks?
```

Answer:
Used to run scripts during deployment.

👉 Types:

PreSync
Sync
PostSync
```

🔹 23. What is Argo CD RBAC?
```

Answer:
Role-based access control for users.

👉 Example:

Read-only access
Admin access
```

🔹 24. How Argo CD Handles Secrets?
```

Answer:

Use tools like:
Sealed Secrets
HashiCorp Vault
```

🔹 25. Multi-Cluster Deployment?
```

Answer:
Argo CD can manage multiple Kubernetes clusters using contexts.
```

🔹 26. What is Sync Waves?
```

Answer:
Control order of resource deployment.
```

🔹 27. What is Sync Phases?
```

PreSync
Sync
PostSync
```

🔹 28. What is Argo CD CLI?
```

Answer:
Command-line tool to interact with Argo CD.
```

🔹 29. Common Issues & Troubleshooting
```

❓ App OutOfSync
Git vs cluster mismatch
❓ Sync Failed
YAML error
Permission issue
❓ CrashLoopBackOff
Wrong image / config
```

🔹 30. Real-Time Scenario Questions
```

✅ Scenario 1:

Cluster manually changed — what happens?
👉 Drift detected → Auto Sync restores (if enabled)

✅ Scenario 2:

Deleted resource from Git?
👉 If prune enabled → resource deleted from cluster

✅ Scenario 3:

Deployment failed?
👉 Check:

Logs
Events
Health status
✅ Scenario 4:

How to deploy to multiple environments?
👉 Use:

ApplicationSet
Separate branches
```

🔹 31. Best Practices
Use Git as single source
Enable auto sync carefully
Use RBAC
Manage secrets securely
Monitor with Prometheus + Grafana


❓ How Argo CD ensures idempotency?
```

👉 Reconciliation loop ensures desired state always maintained
```

❓ Difference between Push vs Pull model?
```

Argo CD → Pull
Jenkins → Push
```

❓ How to secure Argo CD?
```

RBAC
SSO integration
TLS
❓ How Argo CD integrates with CI tools?

👉 CI (like Jenkins) updates Git → Argo CD deploys
```


1. Your application is OutOfSync in Argo CD, but no changes were made in Git. Why?
```
Answer:

Manual changes done in cluster (kubectl edit)
ConfigMap/Secret updated dynamically
Helm values drift
Auto-sync disabled

👉 Fix:

Enable self-heal OR revert manual changes
```
❓ 2. Argo CD is not detecting changes from Git. What will you check?
```
Answer:

Repo connectivity (SSH/HTTPS)
Repo server logs
Webhook configuration
Git branch/path mismatch
Cache issue (repo-server)
```
❓ 3. Sync is failing. How do you debug?
```
Answer:

Check application events
Check controller logs
Validate YAML
Check RBAC permissions
Run:
argocd app logs <app-name>
```
❓ 4. You deleted a resource from Git but it's still running in cluster. Why?
```
Answer:
👉 Prune not enabled

Fix:

syncPolicy:
  automated:
    prune: true
```
❓ 5. Someone manually changed replicas in Kubernetes. What happens?
```
Answer:

Argo CD detects drift
If self-heal ON → revert to Git state
If OFF → stays OutOfSync
```
❓ 6. How do you promote application from dev → prod?
```
Answer:

Use different branches OR folders
Update image tag in Git
Argo CD syncs automatically
```
❓ 7. How do you handle secrets in Argo CD?
```
Answer:
Use:

Sealed Secrets
HashiCorp Vault

Never store plain secrets in Git ❌
```
❓ 8. Argo CD UI is not accessible. What will you check?
```
Answer:

Service type (ClusterIP/NodePort/Ingress)
Port-forward
Pod status
Network policies
```
❓ 9. App is in “Degraded” state. What does it mean?
```
Answer:

Resource not healthy
Pod crash / readiness probe failed
```
❓ 10. How do you rollback deployment in Argo CD?
```
Answer:

Rollback to previous Git commit
Or use UI → History → Rollback
🔥 2. Troubleshooting Questions (Most Asked)
```
❓ 11. Difference between Sync Failed vs Degraded?
```
Sync Failed → deployment failed
Degraded → deployed but unhealthy
```
❓ 12. Argo CD showing Unknown status?
```
Answer:

API server issue
Cluster connectivity problem
```
❓ 13. Pods in CrashLoopBackOff after sync. What will you do?
```
Answer:

Check logs
Check image/tag
Check env variables
```
❓ 14. Argo CD is slow in syncing. Why?
```
Answer:

Large repo
Too many apps
Repo-server bottleneck
No caching optimization
```
❓ 15. How do you scale Argo CD?
```
Answer:

Increase replicas of:
application-controller
repo-server
🔥 3. Architecture-Level Questions
```
❓ 16. Explain Argo CD architecture in interview
```
Answer:

API Server (UI/CLI)
Repo Server (Git fetch)
Application Controller (reconciliation)
Redis (cache)

👉 Always mention reconciliation loop
```
❓ 17. What is reconciliation loop?
```
Answer:
Continuously compares:
👉 Desired state (Git) vs Live state (Cluster)
```
❓ 18. Why Argo CD is called pull-based?
```
Answer:
👉 It pulls changes from Git instead of pushing from CI
```
❓ 19. How Argo CD integrates with CI tools?
```
Answer:
CI tools like Jenkins:

Build image
Update Git
Argo CD deploys
```
❓ 20. How do you manage multiple clusters?
```
Answer:

Add clusters in Argo CD
Use contexts
Use ApplicationSet
```
❓ 21. How do you deploy 100+ applications using Argo CD?
```
Answer:
👉 Use ApplicationSet with generators
```
❓ 22. How do you ensure zero downtime deployments?
```
Answer:

Rolling updates
Health checks
Sync waves
```
❓ 23. What is Sync Wave? Real use case?
```
Answer:

Deploy DB first → backend → frontend
```
❓ 24. Can Argo CD replace Jenkins?
```
Answer:
❌ No
👉 Jenkins = CI
👉 Argo CD = CD
```
❓ 25. How do you secure Argo CD?
```
Answer:

RBAC
SSO
TLS
Private repo access
```
❓ 26. What happens if Git repo is down?
```
Answer:

No new deployments
Existing apps keep running
```
❓ 27. Can Argo CD work without Git?
```
Answer:
❌ No (Git is mandatory source of truth)
```
❓ 28. What is difference between Helm vs Argo CD?
```
Answer:

Helm → templating
Argo CD → deployment
```
❓ 29. How do you debug repo-server issues?
```
Answer:

Check repo-server logs
Test Git access manually\
```
❓ 30. What is biggest limitation of Argo CD?
```
Answer:

Not a CI tool
Git dependency
Complex setup at scale
```
