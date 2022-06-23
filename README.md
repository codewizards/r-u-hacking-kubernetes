# CodeWizards Kubernetes demo

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

```
YOUR.MINIKUBE.IP.ADDRESS 	green.local
YOUR.MINIKUBE.IP.ADDRESS 	blue.local
YOUR.MINIKUBE.IP.ADDRESS	wordpress.local
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
