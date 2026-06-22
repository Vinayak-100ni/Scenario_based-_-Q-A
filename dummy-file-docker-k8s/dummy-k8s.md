# Kubernetes Deployment and Service for MERN Application

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mern-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: mern-app
  template:
    metadata:
      labels:
        app: mern-app
    spec:
      containers:
      - name: mern-app
        image: your-dockerhub-username/mern-app:latest
        ports:
        - containerPort: 3000

---
apiVersion: v1
kind: Service
metadata:
  name: mern-app-service
spec:
  selector:
    app: mern-app
  ports:
  - port: 80
    targetPort: 3000
  type: ClusterIP
```

## Deploy

```bash
kubectl apply -f mern-app.yaml
```

## Verify

```bash
kubectl get deployments
kubectl get pods
kubectl get services
```
