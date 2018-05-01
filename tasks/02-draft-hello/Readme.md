# Development on K8S using Draft and Helm

Draft is a tool to simplify local development with Kubernetes. It is intended to assist.
_local development_, not for production deployments. Since it's based on Helm, it is a good
resource to get started with it.

Draft uses Helm charts from predefined "[packs][2]" to deploy into a local (or remote) Kubernetes
cluster

_Note: Draft is heavily WIP and may change over time. Information here might not always be
up-to-date_

## What Is Helm

- Helm is a package and release manager for Kubernetes maintained by the [CNCF][5]
- It helps to define, deploy and maintain complex Kubernetes applications in so called 'Charts'
- The project maintains [an extensive repository][4] of Helm charts for various applications

## Set up Draft

- When using Docker for Mac with Kubernetes we will be ignoring anything related to Minikube
- If you're Minikube, just follow the instructions [here][1]

Install draft:

```sh
brew tap azure/draft
brew install draft
```

Set up Draft by running this command:

```sh
draft init
```

Install Helm in your cluster (if not done already)

```sh
helm init
```

## Getting Started

### Setting up your Project

- Tested with [joerx/helloweb-go](https://github.com/joerx/helloweb-go), YMMV
- Initalise your app:

```sh
draft create
```

- Take some time to explore the helm chart in `charts`
- Make sure the port your app is listening on and the `internalPort` in `values.yaml` are equal
- Build and deploy your app as docker image using `draft up`:

```sh
draft up
```

- Expore pods, deployments, services, helm releases:

```sh
$ kubectl get pod
NAME                           READY     STATUS    RESTARTS   AGE
helloweb-go-867c9bdb47-m2l56   1/1       Running   0          6m

$ kubectl get deployment
NAME          DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
helloweb-go   1         1         1            1           6m

$ kubectl get service
NAME          TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
helloweb-go   ClusterIP   10.99.51.140   <none>        8080/TCP   7m
```

Connect to the running service, take note of the target port:

```sh
$ draft connect
Connect to go:8080 on localhost:49667
```

In another terminal window, query the service using that target:

```sh
curl localhost:49667
```

Alternatively, use kubernetes port forwarding to connect to your pod:

```sh
$ kubectl port-forward helloweb-go-746c5f9f86-dxlzl 31000:8080
Forwarding from 127.0.0.1:31000 -> 8080
```

In another terminal, query the service again:

```sh
curl localhost:31000
```

### Change Service Type

Using Docker for Mac, ports on Kubernetes are forwarded to `localhost:<port>` by default. We can
take advantage of this by using a service of type [NodePort][3] instead.

Change service type in `values.yaml`:

```yaml
service:
  name: golang
  type: NodePort
  externalPort: 8080
  internalPort: 8080
```

Update the deployed application:

```sh
draft up
```

List services, take note of the target port in the `PORT(S)` column. It's a random free port by 
default, `31832` in the example:

```sh
$ kubectl get service
NAME          TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
helloweb-go   NodePort    10.99.51.140   <none>        8080:31832/TCP   11m
```

Query the web server using that port:

```sh
curl localhost:31832
```

## Note: Minikube vs. Docker for Mac

- Draft guides refer to "Minikube" as the go-to solution for running k8s locally
- The only dependency is that draft won't push images into a registry by default
- Hence, both k8s and draft need to use the same docker daemon to for docker images
- In effect, Docker for Mac with k8s "just works" since it runs k8s on the local `dockerd`

[1]:https://github.com/Azure/draft/blob/master/docs/install-minikube.md
[2]:https://github.com/Azure/draft/tree/master/packs
[3]:https://kubernetes.io/docs/concepts/services-networking/service/#type-nodeport
[4]:https://github.com/kubernetes/charts
[5]:https://www.cncf.io/
