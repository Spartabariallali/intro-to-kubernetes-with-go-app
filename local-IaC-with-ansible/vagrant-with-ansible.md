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
### Building a server with roles - using ansible galaxy
For an indiviudal Kubernetes cluster server, there a few basic things you have to configure:
- Configure basic security such as SSH 
- Disbale swap memory 
- Install Docker 
- Install Kuberentes 

``` 
Create a requirements.yml file which lists each of the roles 
---
roles:
   - name: geerlingguy.security
   - name: geerlingguy.swap
   - name: geerlingguy.docker
   - name: geerlingguy.Kubernetes
```

- Create a ansible.cfg file defining the roles_path so the role will be stored in the project directory 

```
[defaults]

roles_path = ./roles
nocows = 1
host_key_checking = False 
```
- Download the roles from Ansible Galaxy 

```
ansible-galaxy install -r requirements.yml 
```

- One the roles are available locally we can include them in our main.yml file 

```
  roles:
    - geerlingguy.security
    - geerlingguy.swap
    - geerlingguy.docker
    - geerlingguy.Kubernetes
```

### Role Configuration 

- create a vars directory with a main variables file inside `vars/main.ym` and reference it inside the main.yml playbook.

```
vars_files:
    - vars/main.yml
``` 
- In the vars directory we need to establish the two Docker role override variables 

```
---
# docker configuration
docker_install_compose: false
docker_users:
  - vagrant
```

- Next we need to disable swap (by default its enabled) we do that by setting `swap_file_status` to absent and providing the path to the swap file:

```
# swap configuration
swap_file_state: absent
swap_file_path: /dev/mapper/packer--debian--10--amd64--vg-swap_1
```
- Kubernetes configuration 

```
# Kubernetes configuration
kubernetes_version: '1.16'
kubernetes_allow_pods_on_master: false
kubernetes_apiserver_advertise_address: '192.168.7.2'
kubernetes_kubelet_extra_args: '--node-ip={{ ansible_host }}'
```
### Running the cluster build playbook 

```
ansible-playbook -i inventory main.yml 
```









