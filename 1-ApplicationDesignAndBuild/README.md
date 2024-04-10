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

Design Patterns for Multi-Container Pods:

- **Sidecar Pattern**: In this pattern, a secondary container (sidecar) is attached to the main container to provide additional functionality, such as logging, monitoring, or proxying.
- **Ambassador Pattern**: This pattern involves using a proxy container to abstract service discovery and network communication complexities from the main container.
- **Adapter Pattern**: In this pattern, a separate container is used to modify or adapt data from a shared volume before passing it to the main container.

### Sidecar Pattern

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sidecar-example
spec:
  containers:
  - name: main-container
    image: main-container-image:latest
    # Main application container
    ports:
    - containerPort: 8080
  - name: sidecar-container
    image: sidecar-container-image:latest
    # Sidecar container for logging
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

## Volumes
