# kub-autoscaling-example

An example on how to setup auto-scaling in a kubernetes cluster.

Parts of this example is inspired by this repository which shows the functionality to autoscale with a rabbitMQ queue.
```
https://github.com/onfido/k8s-rabbit-pod-autoscaler
```

First we need to download docker tools, minikube, kubectl, helm and put them in our path.

```
https://github.com/docker/toolbox/releases
https://kubernetes.io/docs/tasks/tools/install-minikube/
https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl-on-windows
https://github.com/helm/helm/releases
```

Putting your installation in the path on windows and on my system
they are in the tools directory so I will need to run the command
below.
```
set PATH=%PATH%;c:\tools
```

First we need to start minikube so we have the system created and running.
```
minikube start
```

In order to install the metrics server we use to follow how much resourses our pods are using we use the command below.
```
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.3.6/components.yaml
```

Alternative with minikube there is a addon called metrics server so we don't need any extra setup.
```
minikube addons list
minikube addons enable metrics-server
```

Next up we install our application, here we have a very simple application that will generate some work using a square root function.
```
kubectl apply -f https://k8s.io/examples/application/php-apache.yaml
```

Next up we setup an auto-scaler manually with a minimum of 1 pod and a maximum of 10 pods changing when a pod have more than 50% CPU utilization.
```
kubectl autoscale deployment php-apache --cpu-percent=50 --min=1 --max=10
```

Using kubectl we can look at the scaling functionallity.
```
kubectl get hpa
```

Next up we will create some load and see the scaler working.
```
kubectl run -it --rm load-generator --image=busybox /bin/sh

Hit enter for command prompt

while true; do wget -q -O- http://php-apache; done
```

If you want to read the original guide from kubernetes you have that guide below.
```
https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/
```

Last example we will look into is how to scale your pods depending on a rabbitMQ queue. Creating and starting the administration GUI we can do that following the commands below.
```
kubectl create namespace rabbit
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install mu-rabbit bitnami/rabbitmq --namespace rabbit
kubectl port-forward --namespace rabbit svc/mu-rabbit-rabbitmq 15672:15672
```

Next up we need the password to login. We can retrieve it with the command below.
```
kubectl get secret --namespace rabbit mu-rabbit-rabbitmq -o jsonpath="{.data.rabbitmq-password}" | base64 --decode
```

In order to build and deploy the rabbit example we will configure our docker environment, build the docker image and deploy it in our cluster.
```
minikube docker-env
docker build -t kalaspuffar/kub-autoscaling-example .
kubectl -n kube-system apply -f deploy.yml
```
