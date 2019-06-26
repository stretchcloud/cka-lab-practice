# Cluster (11%)

kubernetes.io > Documentation > Reference > kubectl CLI > [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

kubernetes.io > Documentation > Reference > Setup tools reference > Kubeadm > [kubeadm upgrade](https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-upgrade/)

kubernetes.io > Documentation > Tasks > Administer a Cluster > Administration with Kubeadm > [Upgrading kubeadm clusters from v1.13 to v1.14](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade-1-14/)

kubernetes.io > Documentation > Tasks > Administer a Cluster > [Cluster Management](https://kubernetes.io/docs/tasks/administer-cluster/cluster-management/#maintenance-on-a-node)

kubernetes.io > Documentation > Tasks > Administer a Cluster > [Operating etcd clusters for Kubernetes](https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/)

etcd recovery > [etcd recovery github](https://github.com/etcd-io/etcd/blob/master/Documentation/op-guide/recovery.md)



### Understand Kubernetes Cluster upgrade process

<details><summary>show</summary>
<p>

```bash
Allowed combination of kubernetes binary versions:

kube-apiserver - version X
controller-manager - version X-1
kube-scheduler - version X-1
kubelet - version X-2
kube-proxy - version X-2
kubectl - version X+1 > X-1

At any point in time X-2 is supported.

Master Upgrade
***************
$ apt-get upgrade -y kubeadm=1.12.0-00
$ kubectl upgrade plan
$ kubectl upgrade apply v1.12.0
$ apt-get upgrade -y kubelet=1.12.0-00
$ systemctl restart kubelet

Node Upgrade
**************
$ kubectl drain node01
$ apt-get upgrade -y kubeadm=1.12.0-00
$ apt-get upgrade -y kubelet=1.12.0-00
$ kubeadm upgrade node config --kubelet-version v1.12.0
$ systemctl restart kubelet
$ kubectl uncordon node01
```

</p>
</details>

### Facilitate OS upgrades

<details><summary>show</summary>
<p>

```bash
$ kubectl drain node01 --ignore-daemonsets

Apply patches now on the Node01 and once it comes back up. Make it schedulable agaian.

If you are running a Pod on the node which is not part of a replicaset or controller then you need to force the eviction and that Pod will be lost forever.

$ kubectl drain node01 --ignore-daemonsets --force
$ kubectl uncordon node01

If you just want to make the node unschedulable but don't want to evict the running Pod then just cordon the node.

$ kubectl cordon node01
```



</p>
</details>

### Implement backup and restore methodologies

<details><summary>show</summary>
<p>

```
$ kubectl get all --all-namespaces -o yaml > all-services.yaml

etcd save it's all data here:

$ cat /etc/system.d/system/etcd.service

--data-dir=/var/lib/etcd

$ ETCDCTL_API=3 etcdctl snapshot save snapshot.db \
--endpoints=https://127.0.0.1:2379 \
--cacert=/etc/etcd/ca.crt \
--cert=/etc/etcd/etcd-server.crt \
--key=/etc/etcd/etcd-server.key

$ ETCDCTL_API=3 etcdctl snapshot status snapshot.db

$ service kube-apiserver stop
$ ETCDCTL_API=3 etcdctl \
snapshot restore snapshot.db \
--name=master \
--cert=/etc/kubernetes/pki/etcd/server.crt \
--key=/etc/kubernetes/pki/etcd/server.key \
--data-dir /var/lib/etcd-from-backup \
--initial-cluster master-1=https://127.0.0.1:2380 \
--initial-cluster-token etcd-cluster-1 \
--initial-advertise-peer-urls https://127.0.0.1:2380

$ cat /etc/system/system.d/etcd.service

--initial-cluster-token etcd-cluster-1
--data-dir /var/lib/etcd-from-backup

$ systemctl daemon-reload
$ service etcd restart
$ service kube-apiserver start

```



</p>
</details>



