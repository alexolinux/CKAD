# Application Deployment

------------------------

## Objectives

* Understand Deployments and their purpose.
* Create, update, and manage a Deployment.
* Scale applications dynamically.
* Perform rolling updates and rollbacks.
* Utilize probes (liveness and readiness) for health checks.

### **Deployment configuration `deployment.yaml`**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web-app
  template:
    metadata:
      labels:
        app: web-app
    spec:
      containers:
      - name: web-container
        image: nginx:1.21
        ports:
        - containerPort: 80

```

### **Applying the manifest**

```shell
kubectl apply -f deployment.yaml
```

**Verify the Deployment**

```shell
kubectl get deployments
kubectl get pods
```

### **Scaling Applications**

```shell
kubectl scale deployment web-app --replicas=5
```

### **Deployment - Update & Rollback**

To update a Deployment, you typically modify the Deployment's specification. This can include changes to the container image, environment variables, resource requests/limits, and more.

**Update Strategies**

Kubernetes supports different update strategies for Deployments:

* Rolling Update (default): Gradually replaces Pods with new ones. This ensures that some Pods are always available during the update process
* Recreate: Terminates all existing Pods before creating new ones. This can lead to downtime.

**Setting Update Strategy**

```yaml
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
```

**Rollback Mechanism**

If an update causes issues, you can easily roll back to a previous version of the Deployment.

Command to rollback

```shell
kubectl rollout undo deployment/web-app
```

Specify a revision to roll back to:

```shell
kubectl rollout undo deployment/my-app --to-revision=2
```

### **Probes: Ensuring Application Health**

Probes allow Kubernetes to monitor the health and readiness of your application.

* Liveness Probe

Ensures the application is running. If the check fails, Kubernetes restarts the container.

```yaml
livenessProbe:
  httpGet:
    path: /healthz
    port: 80
  initialDelaySeconds: 3
  periodSeconds: 5

```

* Readiness Probe

Ensures the application is ready to handle traffic.

```yaml
readinessProbe:
  httpGet:
    path: /ready
    port: 80
  initialDelaySeconds: 5
  periodSeconds: 10

```

### **Managing Rollouts**

View Deployment History

```yaml
kubectl rollout history deployment/web-app
```

Pause a Deployment, if you need to inspect or modify a rollout:

```yaml
kubectl rollout pause deployment/web-app
```

Resume a Deployment

```yaml
kubectl rollout resume deployment/web-app
```

Cleaning Up, when the Deployment is no longer needed:


```yaml
kubectl delete deployment web-app
```
