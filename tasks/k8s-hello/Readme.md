# My First Kubernetes Service

1. Write a simple web server in either NodeJS, Python or Golang
1. Web server should render a html page reporting the current hostname
1. Build a docker image from it and push it to a public registry
1. Deploy it to a local Kubernetes cluster via a k8s manifest

## Web Application & Dockerfile

- E.g. simple golang webserver, see [joerx/helloweb-go](https://github.com/joerx/helloweb-go)
- Use [quay.io](https://quay.io/) to set up a build triggered from repository pushes
- Run a container using Docker cli:

```sh
docker run -it --rm -p 8000:8000 quay.io/joerx/helloweb:latest
```

- Query the container:

```sh
curl localhost:8000
```

## Kubernetes Manifest

- Deployment for the actual pods
- Service to expose the pods to the world
- Using `NodePort` to expose it outside the cluster
- To start:

```sh
kubectl apply -f kubernetes/manifest.yaml
```

- Watch pod status as they are being created:

```sh
watch kubectl get pod
```

- Node ports are automatically forwarded when using Docker for Mac
- To query the service (once every 2 seconds, observe change of hostname)

```sh
watch curl localhost:30000
```

- Kill a single pod, watch it coming back:

```sh
kubectl delete pod <pod_name>
```
