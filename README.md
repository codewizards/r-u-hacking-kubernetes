# CodeWizards Kubernetes demo

This is a very top level introduction to the most basic Kubernetes resources. You should be able to deploy a workload to a cluster with the information given here. There is so much more to kubernetes than what is contained here, but have fun getting your feet wet :)

## Requirements

- [minikube](https://minikube.sigs.k8s.io/docs/start/) (local kubernetes cluster)
- [kubectl](https://kubernetes.io/docs/tasks/tools/) (command line interface)
- [helm](https://helm.sh/docs/intro/install/) (kubernetes template/package manager)

## Setup

Initialise minikube:

```bash
minikube config set profile minikube
minikube config set memory 8192
minikube config set cpus 4
minikube start --extra-config=kubelet.housekeeping-interval=10s
minikube addons enable ingress
minikube addons enable metrics-server
minikube addons enable dashboard
```

## Basic commands

`kubectl api-resources` shows you all the resource types the cluster knows about - there are _many_ in here, and more can be added but for the purpose of this example you only need worry about:

- pods
- services (svc)
- deployments (deploy)
- ingresses (ing)
- configmaps (cm)
- secrets
- horizontalpodautoscalers (hpa)

`kubectl get RESOURCE_TYPE` is the "listing" operator on various resources.

`kubectl describe RESOURCE_TYPE RESOURCE_NAME` gets detailed information on the desired resource

`kubectl apply -f MANIFEST_FILE` will create the resources in the given file

`kubectl delete -f MANIFEST_FILE` will delete the resources in the given file

Examples

`kubectl get pods` will list the pods in the current namespace

`kubectl describe pod blue-app` will give you info on the `blue-app` pod

## Adding resources to Kubernetes via manifests

There is an official web dashboard to get a view on what is going on, however it is not recommended for production. If you aren't very comfortable with the cli then it's a good jumping off point.

To view the dashboard

```bash
minikube dashboard --url
```

Now we want an IP to talk to the cluster with

```bash
minikube ip
```

Alter your hosts file with a few new hosts

OSX/Linux: `/etc/hosts`
In windows: Alter the hosts file at `c:\windows\system32\drivers\etc\hosts`

```
YOUR.MINIKUBE.IP.ADDRESS 	green.local
YOUR.MINIKUBE.IP.ADDRESS 	blue.local
```

You can now apply each manifest file with the following syntax (except helm!)

```bash
kubectl apply -f PATH_TO_FILE
```

You can delete resources in the same way

```bash
kubectl delete -f PATH_TO_FILE
```

Install a copy of the helm template with:

```bash
helm install NAME_OF_APP ./helm-example
```

Make sure you add a host entry

```
YOUR.MINIKUBE.IP.ADDRESS 	NAME_OF_APP.local
```

Run a load test!

```bash
kubectl run -i --tty load-generator --rm --image=busybox:1.28 --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://NAME_OF_APP.local; done"
```

Watch with the CLI or the dashboard

```bash
kubectl get hpa NAME_OF_APP-scaler --watch
```

## Example wordpress install

Setup another minikube cluster:

```bash
minikube delete -p wordpress
minikube start -p wordpress
minikube config set profile wordpress
minikube update-context wordpress
minikube addons enable ingress
```

Install wordpress:

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install my-site bitnami/wordpress \
	--set service.type=ClusterIP \
	--set ingress.enabled=true \
	--set ingress.ingressClassName=nginx \
	--set ingress.hostname=wordpress.local \
	--set ingress.pathType=Prefix \
	--set ingress.path=/ \
	--set persistence.storageClass=standard \
	--set wordpressPassword=password \
	--set mariadb.auth.rootPassword=password \
	--set mariadb.auth.password=password \
	--version 13.1.16
```

The above will take some time to get ready, you can check the status of the installation with:

```bash
kubectl get pods
```

Once thats up, add it to your hosts

In Linux/OSX:

```bash
echo "$(minikube ip)	wordpress.local" | sudo tee -a /etc/hosts > /dev/null
```

In windows: Alter the hosts file at `c:\windows\system32\drivers\etc\hosts`

## Further reading

- Service meshes eg [linkerd](https://linkerd.io/2.11/overview/)/[Istio](https://istio.io/latest/about/service-mesh/) - very handy for [canary releases](https://blog.getambassador.io/cloud-native-patterns-canary-release-1cb8f82d371a)
- [Nginx Ingress](https://kubernetes.github.io/ingress-nginx/) - youll probably have to deal with nginx at some point!
- [Prometheus](https://prometheus.io/) - metric aggregation
- [DataDog](https://www.datadoghq.com/) - log collection
- [Grafana](https://grafana.com/) - incredible dashboarding tool
- [OAuth2 Proxy](https://oauth2-proxy.github.io/oauth2-proxy/) - versatile auth barrier that plugs into most federated identity providers allowing you to secure dashboards to authenticated authorised users (eg the grafana ui!)
- [Tailscale](https://tailscale.com/) - wireguard based vpn to secure cluster access
- [Operators](https://kubernetes.io/docs/concepts/extend-kubernetes/operator/) - how to extend Kubernetes
- [Pager Duty](https://www.pagerduty.com/index/) - alerting tool to tell you when things have gone wrong
- [ArgoCD](https://argo-cd.readthedocs.io/en/stable/) - have your cluster look at a git repository to make its updates
