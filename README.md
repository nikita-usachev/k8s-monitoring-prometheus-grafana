# Deploying Prometheus and Grafana on Kubernetes

## Introduction:

Monitoring a Kubernetes Cluster is the need of the hour for any application following a microservices architecture. There are a bunch of solutions that one can implement to monitor their Kubernetes workload and one of them is Prometheus and Grafana. This article will help you to deploy Prometheus and Grafana in your kubernetes cluster. There are an opensource monitoring solution. Prometheus and Grafana. These tools is a perfect combination for monitoring. Where Prometheus server is a metric storage or time-series database to store the metrics. Grafana is a visualization tool.

## Deployment of Prometheus on Kubernetes:

This setup collects node, pods, and service metrics automatically using Prometheus service discovery configurations.
Let's get started with the setup.

## Connect to the Kubernetes Cluster:

First we will have our cluster up and running next we will connect to our cluster and make sure we have an admin privileges to create cluster roles.

## Create a Namespace & ClusterRole

First of all we will create a Kubernetes namespace for all our monitoring components. The name for our namespace will be "monitoring".
If you don’t create a dedicated namespace, all the Prometheus kubernetes deployment objects get deployed on the default namespace.
Execute the following command to create a new namespace named monitoring.

```
kubectl create namespace monitoring
```

Prometheus uses Kubernetes APIs to read all the available metrics from Nodes, Pods, Deployments, etc. For this reason, we need to create an RBAC policy with read access to required API groups and bind the policy to the monitoring namespace.

## Create a file named [cluster-role.yaml](./k8s/clusterRole.yaml) with following content

```
kubectl create -f clusterRole.yaml
```

**Note: In the role, given below, you can see that we have added get, list, and watch permissions to nodes, services endpoints, pods, and ingresses. The role binding is bound to the monitoring namespace. If you have any use case to retrieve metrics from any other object, you need to add that in this cluster role.**


## Create a [config-map.yaml](./k8s/configmap.yaml) to externalize Prometheus configurations

```
kubectl apply -f config-map.yaml
```

## Create a file named [prometheus-deployment.yaml](./k8s/prometheus-deployment.yaml) and copy the following contents onto the file

```
kubectl apply -f prometheus-deployment.yaml
```

You can check the created deployment using the following command:

```
kubectl get deployments --namespace=monitoring
```

## Connecting to Prometheus Dashboard

1. Using Kubectl port forwarding
2. Exposing the Prometheus deployment as a service with NodePort or a Load Balancer.
3. Adding an Ingress object if you have an Ingress controller deployed.

In our case, we will use port forwarding to connect our Prometheus server. Using kubectl port forwarding, you can access a pod from your local workstation using a selected port on your localhost. This method is primarily used for debugging purposes.

First, get the Prometheus pod name.

```
kubectl get pods --namespace=monitoring
```

Execute the following command with your pod name to access Prometheus from localhost port 8080.

```
kubectl port-forward prometheus-monitoring-3331088907-hm5n1 8080:9090 -n monitoring
```

Now, if you access http://localhost:8080 on your browser, you will get the Prometheus home page.

## Setting up Kube State Metrics

Kube state metrics service will provide many metrics which is not available by default. Please make sure you deploy Kube state metrics to monitor all your kubernetes API objects like deployments, pods, jobs, cronjobs etc.

## What is Kube State Metrics?

Kube State metrics is a service that talks to the Kubernetes API server to get all the details about all the API objects like deployments, pods, daemonsets, Statefulsets. Kube state metrics service exposes all the metrics on /metrics URI. Prometheus can scrape all the metrics exposed by Kube state metrics.

Following are some of the important metrics you can get from Kube state metrics:

1. Node status, node capacity (CPU and memory)
2. Replica-set compliance (desired/available/unavailable/updated status of replicas per deployment)
3. Pod status (waiting, running, ready, etc)
4. Ingress metrics
5. PV, PVC metrics
6. Daemonset & Statefulset metrics.
7. Resource requests and limits.
8. Job & Cronjob metrics

## Kube State Metrics setup:

You will have to deploy the following Kubernetes objects for Kube state metrics to work:

1. A Service Account
2. Cluster Role – for kube state metrics to access all the Kubernetes API objects.
3. Cluster Role Binding – binds the service account with the cluster role.
4. Kube State Metrics Deployment
5. Service – to expose the metrics

All the above Kube state metrics objects will be deployed in the kube-system namespace.
Create all the objects under [kube-state-metrics/](./k8s/kube-state-metrics/) folder, you can copy all files and just execute following command:

```
kubectl apply -f kube-state-metrics/
```

Check the deployment status using the following command.

```
kubectl get deployments kube-state-metrics -n kube-system
```

## Setting up Grafana for viewing metrics:

Grafana is an open platform for beautiful analytics and monitoring. Using Grafana you can create dashboards from Prometheus metrics to monitor the kubernetes cluster. The best part is, you don’t have to write all the PromQL queries for the dashboards. There are many community dashboard templates available for Kubernetes. You can import it and modify it as per your needs.

## Deploy Grafana on Kubernetes:

**Note: Creating ConfigMap for Grafana which consist of configuration information about data source Prometheus. The following data source configuration is for Prometheus. If you have more data sources, you can add more data sources with different YAMLs under the data section.**

Create the [grafana-datasource-config.yaml](./k8s/grafana/grafana-datesource-config.yaml) using the following command:

```
kubectl create -f grafana-datasource-config.yaml
```

Create the [grafana-deployment.yaml](./k8s/grafana/grafana-deployment.yaml) using the following command:

```
kubectl create -f grafana-deployment.yaml
```

**Note: This Grafana deployment does not use a persistent volume. If you restart the pod all changes will be gone. Use a persistent volume if you are deploying Grafana for your project requirements. It will persist all the configs and data that Grafana uses.**

Create the [grafana-service.yaml](./k8s/grafana/grafana-service.yaml) using the following command.

```
kubectl create -f grafana-service.yaml
```
Now we should be able to access the Grafana dashboard use port forwarding using the following command.

```
kubectl port-forward -n monitoring <grafana-pod-name> 3000 &
```

We will be able to access Grafana a from http://localhost:3000

In our case we will connecting to Grafana Dashboard adding an Ingress object to route the Grafana DNS to the Grafana backend service. We already have existing [grafana-ingress.yaml](./k8s/grafana/grafana-ingress.yaml).

## Create Kubernetes Dashboards on Grafana

Creating a Kubernetes dashboard from the Grafana template is pretty easy. There are many prebuilt Grafana templates available for Kubernetes. You can easily have prebuilt dashboards for ingress controllers, volumes, API servers, Prometheus metrics, and much more.
To know more, see https://grafana.com/grafana/dashboards/?search=kubernetes
I used the dashboard ID for Kubernetes cluster monitoring (via Prometheus) from grafana public template list and import to grafana dashboard.
