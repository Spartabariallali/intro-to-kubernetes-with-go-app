## Small Kubernetes cluster architecture

#### Structure of local multi-node Kubernetes cluster architecture
- There will one Kubernetes master node (which will run the Kubernetes control plane) and two worker nodes (which will run the workloads)
- The master node will run the kube-apiserver (which exposes Kubernetes API), kube scheduler (which schedules workloads on nodes), and etcd (which stores Kubernetes's data)
- The worker nodes will run kubelet (an agent that makes sure the right containers are running on the right pods) and a container runtime - in our case, Docker.

### Vagrantfile creates the local IaC
- The vagrantfile will set the default vms that will be created. Each VM will have 2GB of ram and 2 virtual CPUs cores
- each vm will have `geerlingguy/debian10` base

### Building a Kubernetes cluster with ansible

- Describing hosts with an inventory

```
[kube]

kube1 ansible_host=192.168.7.2 kubernetes_role=master
kube2 ansible_host=192.168.7.3 kubernetes_role=node
kube3 ansible_host=192.168.7.4 kubernetes_role=node

[kube:vars]
ansible_ssh_user=vagrant
ansible_ssh_private_key_file=~/.vagrant.d/insecure_private_key
```
