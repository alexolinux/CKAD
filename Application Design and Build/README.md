# Application Design and Build

------------------------------

## Jobs and CronJobs

<https://kubernetes.io/docs/concepts/workloads/controllers/job>

### Running an example Job

Shell example

```shell
kubectl get job <job_name> --dry-run=client -o yaml
```

YAML Example

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
```

### Running an example of CronJob

Shell example

```shell
kubectl get cronjob <cronjob_name> --dry-run=client -o yaml
```

YAML Example

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
```

## Building Multi-Container Pods

## init-containers

## Volumes
