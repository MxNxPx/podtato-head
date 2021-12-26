![Version: 0.1.2](https://img.shields.io/badge/Version-0.1.2-informational?style=flat-square) ![AppVersion: v0.1.0](https://img.shields.io/badge/AppVersion-v0.1.0-informational?style=flat-square)

# Deliver with Helm

Here's how to deliver podtato-head using [Helm](https://helm.sh).

## Prerequisites

- Install Helm ([official instructions](https://helm.sh/docs/intro/install/))

## Deliver

You must clone this repo and install from a local copy of the chart:

```
git clone https://github.com/podtato-head/podtato-head.git && cd podtato-head
helm install podtato-head ./delivery/chart
```

This will install the chart in this directory with release name `podtato-head`.

> NOTE: You can instruct helm to wait for the resources to be ready before
marking the release as successful by adding the `--wait` option to the previous
command.

The installation can be customized by changing the following parameters via
`--set` or a custom `values.yaml` file specified with `--values`:

## Values

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| nameOverride | string | `""` |  |
| fullnameOverride | string | `""` |  |
| replicaCount | int | `1` |  |
| images.repositoryDirname | string | `"ghcr.io/podtato-head"` | Prefix for image repos |
| images.pullPolicy | string | `"IfNotPresent"` | Podtato Head Container pull policy |
| images.pullSecrets | list | `[]` | Podtato Head Pod pull secret |
| entry.repositoryBasename | string | `"entry"` | Leaf part of name of image repo for entry |
| entry.tag | string | `"0.1.0"` | Tag of image repo for entry |
| entry.serviceType | string | `"LoadBalancer"` | Service type for entry |
| entry.servicePort | int | `9000` | Service port for entry |
| entry.extraEnvs | list | `[]` | Extra environment variables for entry |
| hat.repositoryBasename | string | `"hat"` | Leaf part of name of image repo for hat |
| hat.tag | string | `"0.1.0"` | Tag of image repo for hat |
| hat.serviceType | string | `"ClusterIP"` | Service type for hat |
| hat.servicePort | int | `9001` | Service port for hat |
| hat.extraEnvs | list | `[]` | Extra environment variables for hat |
| leftLeg.repositoryBasename | string | `"left-leg"` | Leaf part of name of image repo for left-leg |
| leftLeg.tag | string | `"0.1.0"` | Tag of image repo for left-leg |
| leftLeg.serviceType | string | `"ClusterIP"` | Service type for left-leg |
| leftLeg.servicePort | int | `9002` | Service port for left-leg |
| leftLeg.extraEnvs | list | `[]` | Extra environment variables for left-leg |
| leftArm.repositoryBasename | string | `"left-arm"` | Leaf part of name of image repo for left-arm |
| leftArm.tag | string | `"0.1.0"` | Tag of image repo for left-arm |
| leftArm.serviceType | string | `"ClusterIP"` | Service type for left-arm |
| leftArm.servicePort | int | `9003` | Service port for left-arm |
| leftArm.extraEnvs | list | `[]` | Extra environment variables for left-arm |
| rightLeg.repositoryBasename | string | `"right-leg"` | Leaf part of name of image repo for right-leg |
| rightLeg.tag | string | `"0.1.0"` | Tag of image repo for right-leg |
| rightLeg.serviceType | string | `"ClusterIP"` | Service type for right-leg |
| rightLeg.servicePort | int | `9004` | Service port for right-leg |
| rightLeg.extraEnvs | list | `[]` | Extra environment variables for right-leg |
| rightArm.repositoryBasename | string | `"right-arm"` | Leaf part of name of image repo for right-arm |
| rightArm.tag | string | `"0.1.0"` | Tag of image repo for right-arm |
| rightArm.serviceType | string | `"ClusterIP"` | Service type for right-arm |
| rightArm.servicePort | int | `9005` | Service port for right-arm |
| rightArm.extraEnvs | list | `[]` | Extra environment variables for right-arm |
| serviceAccount.create | bool | `true` | Whether or not to create dedicated service account |
| serviceAccount.annotations | object | `{}` | Annotations to add to a created service account |
| serviceAccount.name | string | `""` |  |
| podAnnotations | object | `{}` | Map of annotations to add to the pods  |
| podSecurityContext | object | `{}` |  |
| securityContext | object | `{}` | Set a securityContext |
| ingress.enabled | bool | `false` | Enables ingress |
| ingress.annotations | object | `{}` | Ingress annotations |
| ingress.hosts[0].host | string | `"chart-example.local"` |  |
| ingress.hosts[0].paths | list | `[]` |  |
| ingress.tls | list | `[]` | Ingress TLS configuration |
| resources | object | `{}` | Resource requests and limits    |
| autoscaling.enabled | bool | `false` | Enable horizontal pod autoscaler |
| autoscaling.minReplicas | int | `1` | Min replicas for autoscaling     |
| autoscaling.maxReplicas | int | `100` | Max replicas for autoscaling     |
| autoscaling.targetCPUUtilizationPercentage | int | `80` |  |
| autoscaling.targetMemoryUtilizationPercentage | int | `80` |  |
| nodeSelector | object | `{}` | Labels for pod assignment |
| tolerations | list | `[]` | List of node taints to tolerate |
| affinity | object | `{}` | Affinity configuration |

## Test

Verify the release succeeded:

```
helm list
kubectl get pods
kubectl get services
```

### Test the API endpoint

To connect to the API you'll first need to determine the correct address and
port.

If using a LoadBalancer-type service for `entry`, get the IP address of the load balancer
and use port 9000:

```
ADDR=$(kubectl get service podtato-entry -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
PORT=9000
```

If using a NodePort-type service, get the address of a node and the service's
NodePort as follows:

```
ADDR=$(kubectl get nodes {NODE_NAME} -o jsonpath={.status.addresses[0].address})
PORT=$(kubectl get services podtato-entry -ojsonpath='{.spec.ports[0].nodePort}')
```

If using a ClusterIP-type service, run `kubectl port-forward` in the background
and connect through that:

> NOTE: Find and kill the port-forward process afterwards using `ps` and `kill`.

```
# Choose below the IP address of your machine you want to use to access application
ADDR=127.0.0.1
# Choose below the port of your machine you want to use to access application
PORT=9000
kubectl port-forward --address ${ADDR} svc/podtato-entry ${PORT}:9000 &
```

Now test the API itself with curl and/or a browser:

```
curl http://${ADDR}:${PORT}/
xdg-open http://${ADDR}:${PORT}/
```

## Update

To update the application version, you can choose one of the following methods :

- update `<service>.tag` in `values.yaml` for each service and run `helm upgrade podtato-head ./delivery/chart`
- run `helm upgrade podtato-head ./delivery/chart --set entry.tag=0.1.1 --set leftLeg.tag=0.1.1 ...`

A new revision is then installed.

> NOTE : to ensure idempotency between the first installation and the following updates, you should use the following command :

```
helm upgrade --install podtato-head ./delivery/chart
```

## Rollback

To rollback to a previous revision, run :

```
# Check revision history
helm history podtato-head

# Rollback to the revision 1
helm rollback podtato-head 1

# Check the revision
helm status podtato-head
```

## Uninstall

```
helm uninstall podtato-head
```

## How to generate this file

```
helm-docs -s file
```

----------------------------------------------
Autogenerated from chart metadata using [helm-docs v1.5.0](https://github.com/norwoodj/helm-docs/releases/v1.5.0)
