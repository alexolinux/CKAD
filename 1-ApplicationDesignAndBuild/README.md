# Application Design and Build

------------------------------

## Jobs and CronJobs

<https://kubernetes.io/docs/concepts/workloads/controllers/job>

### Running an example Job

Shell example

```shell
kubectl create job my-job --image=busybox --dry-run=client -o yaml -- /bin/sh -c 'echo Hello, world!'
```

YAML example

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: pi
spec:
  template:
    spec:
      containers:
      - name: pi
        image: perl:5.34.0
        command: ["perl",  "-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: Never
  backoffLimit: 4
  activeDeadlineSeconds: 10
```

- `backoffLimit` specifies the number of retries a CronJob should make before considering it as failed.

- `activeDeadlineSeconds` sets a time limit for the Job to execute. If the Job exceeds this time limit, it will be terminated.

### Running an example of CronJob

<https://kubernetes.io/docs/tasks/job/automated-tasks-with-cron-jobs>

Shell example

```shell
kubectl create cronjob my-cronjob --image=busybox --schedule="* * * * *" --dry-run=client -o yaml -- /bin/sh -c 'echo Hello, world!'
```

YAML example

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: hello
spec:
  schedule: "* * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: busybox:1.28
            imagePullPolicy: IfNotPresent
            command:
            - /bin/sh
            - -c
            - date; echo Hello from the Kubernetes cluster
          restartPolicy: OnFailure
      backoffLimit: 4
      activeDeadlineSeconds: 10
```

## Building Multi-Container Pods

<https://kubernetes.io/blog/2015/06/the-distributed-system-toolkit-patterns>

Design Patterns for Multi-Container Pods:

- **Sidecar Pattern**: In this pattern, a secondary container (sidecar) is attached to the main container to provide additional functionality, such as logging, monitoring, or proxying.
- **Ambassador Pattern**: This pattern involves using a proxy container to abstract service discovery and network communication complexities from the main container.
- **Adapter Pattern**: In this pattern, a separate container is used to modify or adapt data from a shared volume before passing it to the main container.

### Example of Sidecar Pattern

<https://kubernetes.io/docs/concepts/workloads/pods/sidecar-containers>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-sidecar-example
  labels:
    app: nginx-sidecar
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
  - name: nginx-sidecar
    image: nginx:latest
    command: ["/bin/sh", "-c"]
    args:
    - "while true; do echo 'HTTP/1.1 200 OK\n\nSidecar Proxy'; done | nc -l -p 8080"
    ports:
    - containerPort: 8080

```

### Ambassador Pattern

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ambassador-example
spec:
  containers:
  - name: main-container
    image: main-container-image:latest
    # Main application container
    ports:
    - containerPort: 8080
  - name: ambassador-container
    image: ambassador-container-image:latest
    # Ambassador container for proxying
```

### Adapter Pattern

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: adapter-example
spec:
  containers:
  - name: main-container
    image: main-container-image:latest
    # Main application container
    ports:
    - containerPort: 8080
  - name: adapter-container
    image: adapter-container-image:latest
    # Adapter container for data transformation
```

## init-containers

Init-containers are specialized containers *that run and complete before the main application containers start*. They are primarily used to perform initialization tasks such as setup, configuration, or data preparation required by the main application containers. Init containers help ensure that the environment or dependencies necessary for the proper functioning of the main application are in place before it starts running. They run to completion and can share data with the main containers through shared volumes.

<https://kubernetes.io/docs/concepts/workloads/pods/init-containers>

YAML configuration for a Pod with an init container:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app.kubernetes.io/name: MyApp
spec:
  containers:
  - name: myapp-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice
    image: busybox:1.28
    command: ['sh', '-c', "until nslookup myservice.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for myservice; sleep 2; done"]
  - name: init-mydb
    image: busybox:1.28
    command: ['sh', '-c', "until nslookup mydb.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for mydb; sleep 2; done"]
```

## Volumes

Volumes in Kubernetes is that they are directory-like structures that exist in a container's filesystem and can be shared among multiple containers in a pod. They enable data to persist beyond the lifetime of a single container and facilitate communication and data exchange between containers within the same pod.

<https://kubernetes.io/docs/concepts/storage/volumes>

### emptyDir Volumes

In the next example, a volume named "my-volume" is defined with an emptyDir type, and it is mounted to the path "/data" in the container named "my-container" running the nginx image:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: my-container
    image: nginx
    volumeMounts:
    - name: my-volume
      mountPath: /data
  volumes:
  - name: my-volume
    emptyDir: {}
```

### ConfigMap Volumes

An example of use to create configMap volumes and access in Pods

- `trauerweide.yaml` (configMap for ENV)

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: trauerweide
data:
  tree: trauerweide
```

- `birke.yaml` (configMap for ENV File)

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: birke
data:
  tree: birke
  level: "3"
  department: park
```

- `pod1.yaml` with these configMaps

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
  name: pod1
spec:
  containers:
  - image: nginx:alpine
    name: pod1
    env:
    - name: TREE1
      valueFrom:
        configMapKeyRef:
          name: trauerweide
          key: tree
    volumeMounts:
    - name: birke-volume
      mountPath: /etc/birke
  volumes:
    - name: birke-volume
      configMap:
        name: birke
```

### Persistent Volume, Persistent Volume Claim

An example that involves creating a PersistentVolume (PV), a PersistentVolumeClaim (PVC), and then mounting the PVC into a Pod in Kubernetes:

- Creating a PersistentVolume `my-pv.yaml`:

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-pv
spec:
  capacity:
    storage: 1Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: manual
  hostPath:
    path: "/mnt/data"
```

- Creating a PersistentVolumeClaim `my-pvc.yaml` that requests storage from the PersistentVolume from `my-pv.yaml`:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: manual
  volumeName: my-pv
```

- Creating a Pod that mounts the PersistentVolumeClaim `my-pvc`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
    - name: my-container
      image: nginx
      volumeMounts:
        - name: my-persistent-storage
          mountPath: /var/www/html
  volumes:
    - name: my-persistent-storage
      persistentVolumeClaim:
        claimName: my-pvc
```
