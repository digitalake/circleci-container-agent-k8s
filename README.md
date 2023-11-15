# <h1 align="center">CircleCI container agent in Kubernetes </a>

### Requirenments

The `most effective solution` for CircleCI self-hosted runners is using `Container Agent` in `Kubenetes`. In this case, Kubernetes cluster was deployed.

To install container runners and run jobs, you will need to have root access, and have the following set up and installed:

- Kubernetes 1.12+
- Helm 3.x
- Kubernetes namespace without any other workloads
- allowed outbound connections over port 22 SSH
- CircleCI CLI (optional)

### Adding Namespace and ResourceClass

In order to install self-hosted runners `namespace` and `resource class token` are needed.
```
circleci namespace create <name> --org-id <your-organization-id>
```

`resource class` creation for self-hosted runner’s namespace can be made  using the following command:
```
circleci runner resource-class create <namespace>/<resource-class> <description> --generate-token
```

The `resource class token` is returned after the runner resource class is successfully created.

### Container runner installation

Adding the container runner Helm repo:
```
helm repo add container-agent https://packagecloud.io/circleci/container-agent/helm
```
Updating the container runner Helm repo:
```
helm repo update
```
Creating seperate namespace for Container runner:
```
kubectl create namespace circleci
```

Preparing values.yaml file for the container runner Helm chart:
```
agent:
  resourceClasses:
    namespace/my-rc:
      token: <resource_class_token>
```
Installing Helm chart:
```
helm install container-agent container-agent/container-agent -n circleci -f values.yaml
```
To check if installation was successful:
```
kubectl get all -n circleci
```

### Migrating to self-hosted runners

After the container runner is installed within the cluster, it can be referenced by using `resource class` definition inside the job configuration. The fields needs to be set to run using newly created container runners are:
`
image: <job-base-image>
resource_class: <namespace>/<resource-class>
`
In my case i added `resource_class` configuration as follows to my [CircleCI infra pipeline project](https://github.com/digitalake/webserver-ec2-module-terraform/tree/dev) for dev environment:

```
...
    docker:
      - image: <specific-job-base-image>
    resource_class: digitalake/container-executor
...
```

### Results

`circleci` namespace Kubernetes resources:

<img src="https://github.com/digitalake/circleci-container-agent-k8s/assets/109740456/c7532fad-3fc2-4f31-a287-e6c99c87b80d" width="500">

`container executor` in my CircleCI UI:

![Screenshot from 2023-11-15 12-53-04](https://github.com/digitalake/circleci-container-agent-k8s/assets/109740456/459eab99-1f4b-4efd-8b44-9e77b8481b7f)

`pipeline` workflow:

![Screenshot from 2023-11-14 13-47-16](https://github.com/digitalake/circleci-container-agent-k8s/assets/109740456/4a0591da-3867-47db-a992-b34d5277dbe9)

Also, CircleCI shows lifecycle task for the jobs executed using Kubernetes container agent:

![Screenshot from 2023-11-15 12-56-21](https://github.com/digitalake/circleci-container-agent-k8s/assets/109740456/24f4acda-ee30-41f3-a202-7ad95877247a)










