# k8s-intro-eks

Assets for the [Introduction to Kubernetes with Amazon EKS](https://dev.to/donaldsebleung/introduction-to-kubernetes-with-amazon-eks-1nj6) article on Dev.to

## Quick start

It is assumed you already have a Kubernetes cluster up and running, and that you can connect to it with `kubectl`. If not, see:

- [Minikube](https://minikube.sigs.k8s.io/docs/start/), [kind](https://kind.sigs.k8s.io/) for setting up a local, single-node cluster
- [kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/) for a self-managed, multi-node cluster
- [Various managed Kubernetes offerings by cloud providers](https://dev.to/cloudtech/comparing-managed-kubernetes-services-eks-vs-aks-vs-gke-4l8d) for a production-grade, multi-node cluster without the headache of self-management as in kubeadm

Now clone this repo and make it your working directory:

```bash
$ git clone https://github.com/DonaldKellett/k8s-intro-eks.git && cd k8s-intro-eks
```

### Deploy a single Pod and expose it within the cluster via a ClusterIP Service

Create our namespace, deploy the Pod and deploy the Service:

```bash
$ kubectl apply -f namespace.yaml
$ kubectl apply -f pod.yaml
$ kubectl apply -f clusterip-service.yaml
```

The pod is running an HTTPS web server so we'll refer to it as our website subsequently.

Take note of the cluster IP of our Service with the following command:

```bash
$ kubectl get svc -n donaldsebleung-com
```

Export it to an environment variable `CLUSTER_IP`. Afterwards, spawn a shell within our pod and query our website with `wget` through our cluster IP:

```bash
$ kubectl exec \
  -n donaldsebleung-com \
  -it donaldsebleung-com \
  -- /bin/bash -c "wget -qO - https://$CLUSTER_IP --no-check-certificate"
```

### Deploy a Deployment with 3 replicas; expose to the outside world using a LoadBalancer Service

First delete our existing pod and service, if any:

```bash
$ kubectl delete -f clusterip-service.yaml
$ kubectl delete -f pod.yaml
```

Now ensure an appropriate load balancer controller is deployed to your Kubernetes cluster. For example, with Amazon EKS, you need an [AWS Load Balancer Controller](https://kubernetes-sigs.github.io/aws-load-balancer-controller/v2.3/).

Now deploy our Deployment and Service:

```bash
$ kubectl apply -f deployment.yaml
$ kubectl apply -f loadbalancer-service.yaml
```

Discover the external IP (or DNS name) of your service:

```bash
$ kubectl get svc -n donaldsebleung-com
```

Now visit that external IP / DNS name using your favorite web browser, prefixing it with `https://` if necessary, and ignore any warnings from your browser about a self-signed certificate.

### Rolling update

Apply a modified configuration for our Deployment, which uses a newer container image for the underlying pods:

```bash
$ kubectl apply -f deployment-patched.yaml
```

Now visit the website again over the next few minutes. You should eventually notice that the slogan on the homepage has changed from "IT consultant by day, software developer by night" to "Cloud, virtualization and open source enthusiast".

### Autoscaling with HPA

Ensure a [metrics server](https://github.com/kubernetes-sigs/metrics-server) is deployed to your Kubernetes cluster. Now set resource limits on containers in pods within our deployment, then deploy a horizontal pod autoscaler, often abbreviated HPA:

```bash
$ kubectl apply -f deployment-patched-with-limit.yaml
$ kubectl apply -f hpa.yaml
```

Wait for a few moments (maybe a minute or two), then query our HPA to see that we now have 1 replica instead of 3:

```bash
$ kubectl get hpa -n donaldsebleung-com
```

Bombard our website with requests - you might want to do this in multiple terminal windows / tabs. This assumes the `EXTERNAL_IP` environment variable is set accordingly:

```bash
$ while true; do wget -qO - "https://$EXTERNAL_IP" --no-check-certificate > /dev/null; done
```

Wait for a few more minutes, then notice that the number of replicas is scaled up to 10 (or some other number greater than 1, depending on the amount of load you produced):

```bash
$ kubectl get hpa -n donaldsebleung-com
```

Stop the bombardment, the wait a few minutes yet again and see the number of replicas scaled back down to 1:

```bash
$ kubectl get hpa -n donaldsebleung-com
```

### Cleanup

Delete the namespace. This also delete all resources within that namespace:

```bash
$ kubectl delete -f namespace.yaml
```

You may also want to decommission the cluster to save costs, if you are using a managed pay-as-you-go Kubernetes offering by a cloud provider.

## License

[MIT](./LICENSE)
