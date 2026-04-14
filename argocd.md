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
