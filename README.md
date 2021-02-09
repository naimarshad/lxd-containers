# Provision Kubernetes cluster with lxc containers

####* This repo contains the necessary files to provision kuberenets cluster I will try step by step approach for this.

Note: We need ansible to complete this cluster creation

I am using Neon User (Ubuntu 20-04) for this purpose. First install lxd runtime as below.

* `sudo snap install lxd`
* `sudo snap start lxd`
* `sudo snap enale lxd`

Initialize the container runtime

* `lxd init`

#### Provide default option for all the questions but make sure you choose storage backend as dir.

`Name of the storage backend to use (btrfs, dir, lvm) [default=btrfs]: dir`

####

#### First create profile for k8s cluster

`lxc profile create k8s`

`lxc profile edit k8s`

Copy the contents from lxd-profile into k8s profile.

#### Time to create a new cluster

Launch the containers, i will choose centos 7 for this task

`lxc launch images:centos/7 kube-master --profile k8s`

`lxc launch images:centos/7 kube-w01 --profile k8s`

`lxc launch images:centos/7 kube-w02 --profile k8s`

Once the containers are ready check them they are created

`lxc list`

```
+------------------+---------+------+------+-----------+-----------+
|       NAME       |  STATE  | IPV4 | IPV6 |   TYPE    | SNAPSHOTS |
+------------------+---------+------+------+-----------+-----------+
| kube-master      | STOPPED |      |      | CONTAINER | 0         |
+------------------+---------+------+------+-----------+-----------+
| kube-w01         | STOPPED |      |      | CONTAINER | 0         |
+------------------+---------+------+------+-----------+-----------+
| kube-w02         | STOPPED |      |      | CONTAINER | 0         |
+------------------+---------+------+------+-----------+-----------+

````

Launch the containers 

`lxc start kube-master kube-w01 kube-w02`

Exec into kube-master container by

`lxc exec kube-master bash`

and install openssh server

`yum install openssh-server-y `

`systemctl enable sshd && systemctl start sshd`

set the root password to login through ssh

`passwd `

Do the above step from exec to installation & start off ssh on other containers as well.

To complete this guide and to successfully install docker & kubernetes we need ansible. Please install ansible in your host machine you can sue pythone package manager to install ansible.

`pip3 install ansible`

Once the ansible installed exchage your ssh keys with all the running containers for example my `lxc list` list shows one of container as below

```
+------------------+---------+----------------------+------+-----------+-----------+
|       NAME       |  STATE  |         IPV4         | IPV6 |   TYPE    | SNAPSHOTS |
+------------------+---------+----------------------+------+-----------+-----------+
| kube-master      | RUNNING | 172.25.25.129 (eth0) |      | CONTAINER | 0         |
+------------------+---------+----------------------+------+-----------+-----------+

```

`ssh-copy-id root@172.25.25.129`

Do this with all the other two nodes.

Once done go to the inventory file in the repo and update the IPs under the sections

`[kube_master]`

`[kube_workers]`

save the file & check if ansible can talk to those hosts by sending ping module command.

```
❯ ansible -m ping all  
172.25.25.129 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}
172.25.25.96 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}
172.25.25.190 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}

```

Once you can reach to those containers, run the commands respectively in your host machine.

`ansible-playbook docker.yaml`

Since I have already ran the playbook the out put with your terminal might be differrent but similar.

```
❯ ansible-playbook docker.yaml  

PLAY [Install docker on all hosts] ***************************************************************************************************************************

TASK [Gathering Facts] ***************************************************************************************************************************************
ok: [172.25.25.190]
ok: [172.25.25.96]
ok: [172.25.25.129]

TASK [Install yum utils] *************************************************************************************************************************************
ok: [172.25.25.190]
ok: [172.25.25.96]
ok: [172.25.25.129]

TASK [Install device-mapper-persistent-data] *****************************************************************************************************************
ok: [172.25.25.129]
ok: [172.25.25.190]
ok: [172.25.25.96]

TASK [Install lvm2] ******************************************************************************************************************************************
ok: [172.25.25.129]
ok: [172.25.25.96]
ok: [172.25.25.190]

TASK [Add Docker repo] ***************************************************************************************************************************************
ok: [172.25.25.190]
ok: [172.25.25.129]
ok: [172.25.25.96]

TASK [Install Docker] ****************************************************************************************************************************************
ok: [172.25.25.190]
ok: [172.25.25.129]
ok: [172.25.25.96]

TASK [Start Docker service] **********************************************************************************************************************************
ok: [172.25.25.190]
ok: [172.25.25.129]
ok: [172.25.25.96]

PLAY RECAP ***************************************************************************************************************************************************
172.25.25.129              : ok=7    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
172.25.25.190              : ok=7    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
172.25.25.96               : ok=7    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

```

```
❯ ansible-playbook kube8s.yaml 

PLAY [Install kuberenetes packages on all hosts] *************************************************************************************************************

TASK [Gathering Facts] ***************************************************************************************************************************************
ok: [172.25.25.190]
ok: [172.25.25.129]
ok: [172.25.25.96]

TASK [Copy kubernetes repo file to servers] ******************************************************************************************************************
ok: [172.25.25.190]
ok: [172.25.25.129]
ok: [172.25.25.96]

TASK [Copy k8s.conf file to servers] *************************************************************************************************************************
ok: [172.25.25.190]
ok: [172.25.25.129]
ok: [172.25.25.96]

TASK [Copy docker custom daeom file to servers] **************************************************************************************************************
changed: [172.25.25.96]
changed: [172.25.25.129]
changed: [172.25.25.190]

TASK [Create a directory if it does not exist] ***************************************************************************************************************
ok: [172.25.25.190]
ok: [172.25.25.129]
ok: [172.25.25.96]

TASK [Installing kubeadm, kubelet and kubectl] ***************************************************************************************************************
changed: [172.25.25.129]
changed: [172.25.25.190]
changed: [172.25.25.96]

TASK [Restart & enable kubelet service] **********************************************************************************************************************
changed: [172.25.25.190]
changed: [172.25.25.129]
changed: [172.25.25.96]

TASK [Restart & enable docker service] ***********************************************************************************************************************
changed: [172.25.25.190]
changed: [172.25.25.129]
changed: [172.25.25.96]

PLAY RECAP ***************************************************************************************************************************************************
172.25.25.129              : ok=8    changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
172.25.25.190              : ok=8    changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
172.25.25.96               : ok=8    changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

```

We have succefully installed the docker & the binaries to create kubernetes cluster by running the abvoe mentioned playbooks.

Let's exec into kube-master container & boot strap the cluster.

`lxc exec kube-master bash`

`kubeadm init --pod-network-cidr=10.244.0.0/16 `

```
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 172.25.25.129:6443 --token sntzcn.9rs6sr5fsr1koeju \
    --discovery-token-ca-cert-hash sha256:c811d8e88bbad04b06d892be678e51dfb66639935f361bc379c9ddc11c3375af 
```

Our cluster master is ready, we have to install cni plugin for node connectivity.

`kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml`

```
[root@kube-master ~]# kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml 
podsecuritypolicy.policy/psp.flannel.unprivileged configured
clusterrole.rbac.authorization.k8s.io/flannel unchanged
clusterrolebinding.rbac.authorization.k8s.io/flannel unchanged
serviceaccount/flannel unchanged
configmap/kube-flannel-cfg unchanged
daemonset.apps/kube-flannel-ds unchanged

```

```
[root@kube-master ~]# kubectl get nodes
NAME          STATUS   ROLES    AGE     VERSION
kube-master   Ready    master   5m28s   v1.19.7
[root@kube-master ~]# 
```

Once the plugin is installed & flannel posd is running, now we can exec into other workers to join them to master.

```
[root@kube-master ~]# kubectl get pods -A
NAMESPACE     NAME                                  READY   STATUS    RESTARTS   AGE
kube-system   coredns-f9fd979d6-9v5pb               1/1     Running   0          5m49s
kube-system   coredns-f9fd979d6-m2zbv               1/1     Running   0          5m49s
kube-system   etcd-kube-master                      1/1     Running   0          5m58s
kube-system   kube-apiserver-kube-master            1/1     Running   0          5m58s
kube-system   kube-controller-manager-kube-master   1/1     Running   0          5m58s
kube-system   kube-flannel-ds-db6zd                 1/1     Running   0          66s
kube-system   kube-proxy-4w96c                      1/1     Running   0          5m48s
kube-system   kube-scheduler-kube-master            1/1     Running   0          5m58s
[root@kube-master ~]# 

```

```
kubeadm join 172.25.25.129:6443 --token sntzcn.9rs6sr5fsr1koeju \
    --discovery-token-ca-cert-hash sha256:c811d8e88bbad04b06d892be678e51dfb66639935f361bc379c9ddc11c3375af
```

We will get back to master by `lxc exec kube-master bash` and check if our nodes already joined to master or no.

```
[root@kube-master ~]# kubectl get nodes
NAME          STATUS   ROLES    AGE    VERSION
kube-master   Ready    master   10m    v1.19.7
kube-w01      Ready    <none>   2m6s   v1.19.7
kube-w02      Ready    <none>   33s    v1.19.7
[root@kube-master ~]# 


```
