# Helm chart deployer

This is a simple Ansible playbook that deploys a simple ruby chart to a `minikube` cluster using `Helm 3`.

## Pre-req

This playbook must be used in a machine that has these thins ready:

- Minikube (with ingress and metrics-server addons enabled)
```
minikube addons enable ingress
```
```
minikube addons enable metrics-server
```
- Docker (to build the ruby-server image and also used as a Minikube driver)
- Ansible 3
- Pip
- Nginx Ingress controller(deployed on the Minikube cluster to have route)

## Getting started

Clone this repo and inside the project directory run these commands:

```
ansible-galaxy install -fr requirements.yml
```

After installing the Ansible requirements, run the playbook using:

```
ansible-playbook -i inventory main.yml --ask-become-pass
```

This playbook follows these steps:

1. Checks if `Minikube` is started(otherwise it'll do it with `docker driver`).
2. Installs `Helm 3`.
3. If the ruby server image is not present in the `Minikube`, it'll build the image and then loads it to `Minikube`.
4. Clones the helm chart and installs it to `Minikube`.

## Verification

If there are no errors then you can see the application in [simple-ruby.local](http://simple-ruby.local).

Now this is the default host and you can change it in the chart values both by forking it's repo and then using it in the playbook or locally in `/opt/http_server_chart/values.yaml`.

If you changed the default host, use this inside chart directory in `/opt`:

```
helm upgrade ruby-server . --values values.yaml
```

Also if you're not using local DNS, add the entry:

```
<ingress service ip> ruby-server.local
```

You can get ingress service ip by running:

```
kubectl get svc ingress-nginx-controller -n ingress-nginx
```

## Architecture and Strategy of Replication

This chart deplos a horizontal horizontal autoscaler that makes sure the application(based on load) have at least 2 running instances and at max 5.

It also uses CPU utilization metris gathered by `metrics-server` and when reached to `30%` it'll scale-out the instances.