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
