# Exercises

-----------

- Persistent **Volume and Persistent Volume Claim** in Kubernetes
  
  - Scenario:
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

## Instructions

- Create all resources in the default namespace.
- Ensure that the PV and PVC are created before deploying the backend pod.
- Do not modify the provided YAML definitions for the redisdb pod.
- Ensure that the PV and PVC are appropriately configured according to the specified requirements.

## Solution

<details>
  <summary>PV redis-pv</summary>
  
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
  <summary>PVC redis-pvc</summary>
  
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
