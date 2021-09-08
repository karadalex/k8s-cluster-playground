Kubernetes Cluster Playground
=======================

Kubernetes Cluster with 1 master node and 2 worker nodes provisioned with Vagrant. Installed with Callico 

## Requirements

Vagrant installed

## Instructions

Create and provision the Kubernetes cluster 
```bash
vagrant up
```

Login to manager node 
```bash
vagrant ssh manager1
```

You can configure memory and number of cpus in the Vagrantfile and then reload the machine with the following command
```bash
vagrant reload [vm-name]
```

To connect the cluster to the kubectl outside of the vagrannt nodes
```bash
vagrant ssh master
cat /home/vagrant/.kube/config
```
and copy the yaml data to the corresponding parts on you local kubectl file and then
```bash
kubectl config use-context kubernetes-admin@kubernetes
```
and verify whether you are in the correct cluster
```bash
kubectl get svc
kubectl get nodes
```

To install the dashboard, via your local kubectl
```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.3.1/aio/deploy/recommended.yam
kubectl proxy
```
To get a bearer token follow this guide [https://github.com/kubernetes/dashboard/blob/master/docs/user/access-control/creating-sample-user.md](https://github.com/kubernetes/dashboard/blob/master/docs/user/access-control/creating-sample-user.md) or run the following from this diretory
```bash
kubectl apply -f admin-user.yaml
kubectl -n kubernetes-dashboard get secret $(kubectl -n kubernetes-dashboard get sa/admin-user -o jsonpath="{.secrets[0].name}") -o go-template="{{.data.token | base64decode}}"
```
and copy the output token to the browser


## Troubleshooting

- A worker (e.g. worker1) hasn't joined the cluster successfully during provisioning
```bash
vagrant ssh worker1
sudo kubeadm reset
sudo kubeadm join 10.0.0.10:6443 --token <TOKEN> --discovery-token-ca-cert-hash <SHA256>
```

## Resources

- https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/
- https://medium.com/@raj10x/multi-node-kubernetes-cluster-with-vagrant-virtualbox-and-kubeadm-9d3eaac28b98
- https://gist.github.com/danielepolencic/ef4ddb763fd9a18bf2f1eaaa2e337544
- https://k21academy.com/docker-kubernetes/three-node-kubernetes-cluster/?utm_source=youtube&utm_medium=referral&utm_campaign=kubernetes30_march21_docker_kubernetes_with_k21academy
- https://www.tigera.io/project-calico/