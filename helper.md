**lxc profile create k8s**

##### Edit the profile & replace the contents in this file with the contents from lxd-profile

lxc profile edit k8s

echo 1 > /proc/sys/net/bridge/bridge-nf-call-iptables

kubeadm config images list
kubeadm config images pull

kubeadm init --pod-network-cidr=10.244.0.0/16

kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

kubeadm join 172.25.25.83:6443 --token ekwimt.bt1na5xf49iyv2ra 
--discovery-token-ca-cert-hash sha256:2c95fb1fabd4c542f11276c851dbbd104d8de339407469f39fe8ce8905316ba8

**If for some reason kube-proxy doesn't start on ubuntu20.04 host vm as in my case. The below command fixed my issue.**

sudo sysctl -w net/netfilter/nf_conntrack_max=524288<<--- This value depends, main which you will get from the logs of the kube-proxy pod which doesn't start.

Also we have to set nf_conntrack hashsize value by
echo 65536 > /sys/module/nf_conntrack/parameters/hashsize # **Depending on value shown in error logs of kube-proxy container

**On alamalinux 8, when trying to load module inside lxd containers it complains. We have to manaully insert the modules like below.**

lxc config edit container_name
linux.kernel_modules: xt_conntrack,overlay, br_netfilter
