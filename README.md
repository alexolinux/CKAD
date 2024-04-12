# CKAD

Brief Journey to achieve the CKAD Certification

-----------------------------------------------

[The Linux Foundation - Domains & Competencies](DOMAINS.md)

## My Study Plan

1 - [Application Design And Build](https://github.com/alexolinux/CKAD/tree/main/1-ApplicationDesignAndBuild)

2 - [Application Deployment](https://github.com/alexolinux/CKAD/tree/main/2-ApplicationDeployment)

3 - [Application Observability And Maintenance](https://github.com/alexolinux/CKAD/tree/main/3-ApplicationObservabilityAndMaintenance)

4 - [Application Environment Configuration And Security](https://github.com/alexolinux/CKAD/tree/main/4-ApplicationEnvironmentConfigurationAndSecurity)

5 - [Services And Networking](https://github.com/alexolinux/CKAD/tree/main/5-ServicesAndNetworking)

## Tips & Shortcuts

- DryRun alias:

  ```shell
  alias kdr="kubectl -o yaml --dry-run=client"
  ```

  - Pod example:

    ```shell
    kdr run mypod --image=busybox > busybox.yaml
    ```

  - Deploy example:

    ```shell
    kdr create deploy nginx --image=nginx:latest --replicas=2 > nginx-deploy.yaml
    ```

  - ConfigMap example:

    ```shell
    kdr create cm troy --from-literal=device=horse > troy-cm.yaml
    ```

## Creating Services

In Kubernetes, you can create services using either the `kubectl create service` command or the `kubectl expose` command. While both commands can be used to create services, they have slightly different functionalities and use cases.

1. kubectl create service

```text
- This command is primarily used to create basic services quickly.
- It supports creating services of type ClusterIP, NodePort, and LoadBalancer.
- It automatically creates a service definition based on the specified parameters (service type, port mappings, etc.).
- It does not require a pre-existing service definition YAML file.
```

  Example usage:
  
  ```shell
  kubectl create service clusterip my-service --tcp=80:8080
  ```

2. kubectl expose

```text
- This command is more versatile and flexible compared to kubectl create service.
- It can be used to expose existing resources, including pods, deployments, and replication controllers, as services.
- It allows for more advanced customization of service configurations through flags and options.
- It can expose services of any type (ClusterIP, NodePort, LoadBalancer), and it can also update existing services.
- It requires a pre-existing resource (such as a deployment or pod) to expose as a service.
- It accepts a wider range of options and configurations, such as labels, annotations, and service types.
```

  Example usage:

  ```shell
  kubectl expose deployment my-deployment --port=80 --target-port=8080 --type=NodePort --name=my-service
  ```
  