# Exercises

-----------

## Deploying a redis with volume
  
  You are a Kubernetes administrator tasked with deploying a Redis that requires persistent storage for storing user data. You need to configure a PersistentVolume (PV) and a PersistentVolumeClaim (PVC) to provide storage for the Redis pod.

  1. Create a PersistentVolume (PV) named "redis-pv" with the following specifications:

- Storage: 1Gi
- Access mode: ReadWriteOnce
- Storage Class: Use the default storage class
- HostPath: /mnt/data

  2. Create a PersistentVolumeClaim (PVC) named "redis-pvc" that requests storage from the "redis-pv" PV with the following specifications:

- Requested storage: 500Mi
- Access mode: ReadWriteOnce

  3. Finally, deploy a Redis pod named "redisdb" using the following YAML definition (`redisdb.yaml`):

  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: redisdb
  spec:
    containers:
    - name: redisdb
      image: redis:latest
      volumeMounts:
      - name: data
        mountPath: /data
    volumes:
    - name: data
      persistentVolumeClaim:
        claimName: redis-pvc
  ```

  4. Validating the persistent data storage.

- Access the pod via kubectl exec and create some data using `redis-cli`:

  ```shell
  kubectl exec -it redisdb -- redis-cli
  ```

  ```shell
  set createdby redisdb
  ```

- Delete this Pod, after that recreate it replacing the name for redisdb2 (edit your `redisdb.yaml`). Then, access Pod redisdb2 to run the following redis query:

  ```shell
  kubectl exec -it redisdb2 -- redis-cli
  ```

  ```shell
  get createdby
  ```

  > The output must have the key:value created using the first redisdb Pod.

- Create all resources in the default namespace.
- Ensure that the PV and PVC are created before deploying the backend pod.
- Do not modify the provided YAML definitions for the redisdb pod.
- Ensure that the PV and PVC are appropriately configured according to the specified requirements.

## Solution

<details>
  <summary>pv-redis.yaml</summary>
  
  ```yaml
  apiVersion: v1
  kind: PersistentVolume
  metadata:
    name: redis-pv
  spec:
    storageClassName: standard
    capacity:
      storage: 1Gi
    accessModes:
      - ReadWriteOnce
    hostPath:
      path: "/mnt/data"
  ```

</details>

<details>
  <summary>pvc-redis.yaml</summary>
  
  ```yaml
  apiVersion: v1
  kind: PersistentVolumeClaim
  metadata:
    name: redis-pvc
  spec:
    storageClassName: standard
    accessModes:
      - ReadWriteOnce
    resources:
      requests:
        storage: 500Mi
  ```

</details>

-----------

## Create a Pod with volume

Create a persistent volume called `log-volume`.
It should make use of a *storage class name* **manual**.
It should use `RWX` as the access mode, and have a size of `1Gi`.
The volume should use the **hostPath** `/opt/volume/nginx`.

Next, create a PVC called `log-claim`, requesting a minimum of `200Mi` of storage.
This PVC should bind to `log-volume`.

Mount this in a Pod called **logger** at the location `/var/www/nginx`.
This Pod should use the image `nginx:alpine`.

## Solution

<details>
  <summary>pv-log.yaml</summary>
  
  ```yaml
  apiVersion: v1
  kind: PersistentVolume
  metadata:
    name: log-volume
    labels:
      app: logger
  spec:
    storageClassName: manual
    capacity:
      storage: 1Gi
    accessModes:
      - ReadWriteMany
    hostPath:
      path: "/opt/volume/nginx"
  ```

</details>

<details>
  <summary>pvc-log.yaml</summary>
  
  ```yaml
  apiVersion: v1
  kind: PersistentVolumeClaim
  metadata:
    name: log-claim
    labels:
      app: logger
  spec:
    volumeName: log-volume
    storageClassName: manual
    accessModes:
      - ReadWriteMany
    resources:
      requests:
        storage: 200Mi
  ```

</details>

<details>
  <summary>pod-logger.yaml</summary>
  
  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: logger
    labels:
      app: logger
  spec:
    volumes:
      - name: log-volume
        persistentVolumeClaim:
          claimName: log-claim
    containers:
      - image: nginx:alpine
        name: logger
        volumeMounts:
          - mountPath: "/var/www/nginx"
            name: log-volume
    restartPolicy: Never
  ```

</details>

-----------

## Deploy and Configure a Web Application

You are given the task to deploy a simple web application called my-web-app that serves static content. You need to perform the following tasks:

1. Deployment

* Create a Deployment named `my-web-app` that uses the Docker image **nginxdemos/hello** (which serves a simple **"Hello World"** webpage).

* The Deployment should have **3** replicas.

2. Service Exposure

* Expose this Deployment using a Service of type `ClusterIP`.

* The Service should route traffic to port `80` on the Deployment.

3. Configure Readiness and Liveness Probes

* Ensure that the Deployment includes both readiness and liveness probes that check if the application is responsive. The probes should perform HTTP GET requests on the root path `/`.

4. Scaling

* After your Deployment is created, scale the number of replicas to **5**.

## Solution

<details>
  <summary>Deploy web-app.yaml</summary>
  
  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: my-web-app
    labels:
      app: my-web-app
  spec:
    replicas: 3
    selector:
      matchLabels:
        app: my-web-app
    template:
      metadata:
        labels:
          app: my-web-app
      spec:
        containers:
          - image: nginxdemos/hello
            name: hello
            ports:
              - containerPort: 80
            readinessProbe:
              httpGet:
                path: /
                port: 80
              initialDelaySeconds: 5
              periodSeconds: 10
            livenessProbe:
              httpGet:
                path: /
                port: 80
              initialDelaySeconds: 10
              periodSeconds: 20
  ```

</details>

<details>
  <summary>SVC web-app.yaml</summary>
  
  ```yaml
  apiVersion: v1
  kind: Service
  metadata:
    name: svc-web-app
    labels:
      app: my-web-app
  spec:
    selector:
      app: my-web-app
    ports:
      - port: 80
        protocol: TCP
        targetPort: 80
  ```

</details>

* Scaling replicas

```shell
kubectl scale --current-replicas=3 --replicas=5 deployment/my-web-app
```
