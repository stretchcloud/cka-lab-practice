# Installation, Configuration & Validation (12%)

kubernetes.io > Documentation > Reference > kubectl CLI > [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

kubernetes.io > Documentation > Tutorials > [Using Minikube to Create a Cluster](https://kubernetes.io/docs/tutorials/kubernetes-basics/create-cluster/)

kubernetes.io > Documentation > Getting Started > Production Environment > Installing Kubernetes with deployment tools > Bootstrapping clusters with kubeadm > [Creating a single control-plane cluster with kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/)

kubernetes.io > Documentation > Concepts > Cluster Administration > [Cluster Networking](https://kubernetes.io/docs/concepts/cluster-administration/networking/#how-to-implement-the-kubernetes-networking-model)

kubernetes.io > Documentation > Tasks > TLS > [Manage TLS Certificates in a Cluster](https://kubernetes.io/docs/tasks/tls/managing-tls-in-a-cluster/)

kubernetes.io > Documentation > Getting Started > Production Environment > Installing Kubernetes with deployment tools > Bootstrapping clusters with kubeadm > [Creating Highly Available clusters with kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/high-availability/)

kubernetes.io > Documentation > Getting Started > [Release notes and version skew](https://kubernetes.io/docs/setup/release/)

Provision underlying infrastructure to deploy a Kubernetes Cluster > [Kubernetes the hard way](https://github.com/kelseyhightower/kubernetes-the-hard-way/blob/f9486b081f8f54dd63a891463f0b0e783d084307/docs/01-infrastructure-gcp.md)

Kubernetes end-to-end Testing > [End to end Test](https://kubernetes.io/blog/2019/03/22/kubernetes-end-to-end-testing-for-everyone/)

###  

### Design a Kubernetes cluster

<details><summary>show</summary>
<p>

```bash
Purpose
	Education
		Minikube
	Dev & Test
	Prod
On Premise or Cloud
Workloads
	Type
		Web
		DB
		Analytics
	Volume
	Resource Requirement
		CPU Hogging
		Memory Hogging
	Network Pattern
		Heavy
		Burst
		

```

</p>
</details>

### Install Kubernetes masters and nodes

<details><summary>show</summary>
<p>

```bash
If you want to do it through Kubeadm then follow these steps:

Run these on all nodes to prepare them:

$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
$ sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
$( l sb_r el ease - cs) \ st abl e"
$ curl -s
https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
$ cat <<EOF| sudo tee
/etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
$ sudo apt-get update
$ sudo apt-get install -y docker-ce=18.06.1~ce~3-0~ubuntu kubelet=1.13.5-00 kubeadm=1.13.5-00 kubectl=1.13.5-00
$ sudo apt-mark hold docker-ce kubelet kubeadm kubectl
$ echo "net.bridge.bridge-nf-call-iptables=1" | sudo tee -a /etc/sysctl.conf
$ sudo sysctl -p


Run these on Master and install the Master Components:

$ sudo kubeadminit --pod-network-cidr=10.244.0.0/16 (assuming you will use Flannel where 10.244.0.0/16 is mandatory requirement as POD network)
$ mkdir -p $HOME/.kube
$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
$ sudo chown $(id -u):$(id -g) $HOME/.kube/config
$ kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/bc79dd1505b0c8681ece4de4c0d86c5cd2643275/Documentation/kube-flannel.yml

Run this command to join the worker node with Master

$ sudo kubeadm join
$ kubectl get nodes -o wide


```



</p>
</details>

### Configure secure cluster communications

<details><summary>show</summary>
<p>

```
# Create private key for CA
$ openssl genrsa -out ca.key 2048

# Create CSR using the private key
$ openssl req -new -key ca.key -subj "/CN=KUBERNETES-CA" -out ca.csr

# Self sign the csr using its own private key
$ openssl x509 -req -in ca.csr -signkey ca.key -CAcreateserial  -out ca.crt -days 1000

# Geenrate private key for admin user
$ openssl genrsa -out admin.key 2048

# Generate CSR for admin user. Note the OU.
$ openssl req -new -key admin.key -subj "/CN=admin/O=system:masters" -out admin.csr

# Sign certificate for admin user using CA servers private key
$ openssl x509 -req -in admin.csr -CA ca.crt -CAkey ca.key -CAcreateserial  -out admin.crt -days 1000

Generate the kube-controller-manager client certificate and private key:

$ openssl genrsa -out kube-controller-manager.key 2048
$ openssl req -new -key kube-controller-manager.key -subj "/CN=system:kube-controller-manager" -out kube-controller-manager.csr
$ openssl x509 -req -in kube-controller-manager.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out kube-contro

Generate the kube-proxy client certificate and private key:

$ openssl genrsa -out kube-proxy.key 2048
$ openssl req -new -key kube-proxy.key -subj "/CN=system:kube-proxy" -out kube-proxy.csr
$ openssl x509 -req -in kube-proxy.csr -CA ca.crt -CAkey ca.key -CAcreateserial  -out kube-proxy.crt -days 1000

Generate the kube-scheduler client certificate and private key:

$ openssl genrsa -out kube-scheduler.key 2048
$ openssl req -new -key kube-scheduler.key -subj "/CN=system:kube-scheduler" -out kube-scheduler.csr
$ openssl x509 -req -in kube-scheduler.csr -CA ca.crt -CAkey ca.key -CAcreateserial  -out kube-scheduler.crt -days 1000

The Kubernetes API Server Certificate

cat > openssl.cnf <<EOF
[req]
req_extensions = v3_req
distinguished_name = req_distinguished_name
[req_distinguished_name]
[ v3_req ]
basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
subjectAltName = @alt_names
[alt_names]
DNS.1 = kubernetes
DNS.2 = kubernetes.default
DNS.3 = kubernetes.default.svc
DNS.4 = kubernetes.default.svc.cluster.local
IP.1 = 10.96.0.1
IP.2 = 192.168.5.11
IP.3 = 192.168.5.12
IP.4 = 192.168.5.30
IP.5 = 127.0.0.1
EOF

Generates certs for kube-apiserver

$ openssl genrsa -out kube-apiserver.key 2048
$ openssl req -new -key kube-apiserver.key -subj "/CN=kube-apiserver" -out kube-apiserver.csr -config openssl.cnf
$ openssl x509 -req -in kube-apiserver.csr -CA ca.crt -CAkey ca.key -CAcreateserial  -out kube-apiserver.crt -extensions v3_req -extfile openssl.cnf -days 1000


The ETCD Server Certificate

cat > openssl-etcd.cnf <<EOF
[req]
req_extensions = v3_req
distinguished_name = req_distinguished_name
[req_distinguished_name]
[ v3_req ]
basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
subjectAltName = @alt_names
[alt_names]
IP.1 = 192.168.5.11
IP.2 = 192.168.5.12
IP.3 = 127.0.0.1
EOF

Generates certs for ETCD

$ openssl genrsa -out etcd-server.key 2048
$ openssl req -new -key etcd-server.key -subj "/CN=etcd-server" -out etcd-server.csr -config openssl-etcd.cnf
$ openssl x509 -req -in etcd-server.csr -CA ca.crt -CAkey ca.key -CAcreateserial  -out etcd-server.crt -extensions v3_req -extfile openssl-etcd.cnf -days 1000

Generate the service-account certificate and private key:

$ openssl genrsa -out service-account.key 2048
$ openssl req -new -key service-account.key -subj "/CN=service-accounts" -out service-account.csr
$ openssl x509 -req -in service-account.csr -CA ca.crt -CAkey ca.key -CAcreateserial  -out service-account.crt -days 1000


```



</p>
</details>

### Configure a Highly-Available Kubernetes cluster

<details><summary>show</summary>
<p>

```bash
If you want to distribute the ETCD Server across multiple instances then follow these steps:

$ kubectl get endpoints kube-scheduler -n kube-system -o yaml
$ kube-controller-manager --leader-elect true 
													--leader-elect-lease-duration 15s 
													--leader-elect-renew-deadline 10s 
													--leader-elect-retry-period 2s
													

$ cat /etc/systemd/system/kube-apiserver.service

--etcd-servers=https://IP:2379, https://IP:2379

$ wget -q --https-only "https://github.com/coreos/etcd/releases/download/v3.3.9/etcd-v3.3.9-linux-amd64.tar.gz"
$ tar -xvf etcd-v3.3.9-linux-amd64.tar.gz
$ mv etcd-v3.3.9-linux-amd64/etcd* /usr/local/bin
$ mkdir -p /etc/etcd /var/lib/etcd
$ cp ca.pem kubernetes-key.pem kubernetes.pem /etc/etcd/

$ etcd.service

--initial-cluster peer-1=https://${PEER1_IP}:2380,peer-2=https://${PEER2_IP}:2380

$ export ETCDCTL_API=3

Initialize the cluster with stacked etcd

$ sudo kubeadm init --config=kubeadm-config.yaml


If you just want to have multiple Kube-API Server then follow these steps:

$ cat kube-config.yaml

apiVersion: kubeadm.k8s.io/v1beta2
kind: ClusterConfiguration
kubernetesVersion: stable
controlPlaneEndpoint: "LOAD_BALANCER_DNS:LOAD_BALANCER_PORT"

$ sudo kubeadm init --config=kubeadm-config.yaml

```



</p>
</details>

### Know where to get the Kubernetes release binaries

<details><summary>show</summary>
<p>

```bash
Kubernetes Main Github Repository -> https://github.com/kubernetes/kubernetes
 
$ wget https://github.com/kubernetes/kubernetes/releases/download/v1.13.5/kubernetes.tar.gz
$ tar -xzvf kubernetes.tar.gz
$ cd kubernetes

For downloading actual binary for your cluster OS, run this:

$ cluster/get-kube-binaries.sh
$ cd server
$ tar -xzvf kubernetes-server-linux-amd64.tar.gz
$ ls kubernetes/server/bin

```

</p>
</details>



### Choose a network solution

<details><summary>show</summary>
<p>

```bash
Network Plug-in aka CNI extend the functionality of Kubernetes. Use this link to see the various different Plug-ins available:

https://kubernetes.io/docs/concepts/cluster-administration/addons/#networking-and-network-policy

```

</p>
</details>



### Run end-to-end tests on your cluster

<details><summary>show</summary>
<p>

```bash
Verify that you can run these checked items:

1. Deployments can run
2. Pods can run
3. Pods can be directly accessed
4. Logs can be collected
5. Commands run from Pod
6. Services can provide access
7. Nodes are healthy
8. Pods are healthy

$ kubectl run nginx --image=nginx
$ kubectl get deployments
$ kubectl get pods
$ kubctl get pods -n kube-system
$ kubectl port-forward nginx 8081:80
$ curl --head http://127.0.0.1:8081
$ kubectl logs nginx
$ kubectl exec -it nginx --nginx -v
$ kubectl expose deployment nginx --port 80 --type NodePort
$ kubectl get services
$ curl -I localhost:<node port>
$ kubectl get nodes
$ kubectl describe nodes
$ kubectl describe pods

```

</p>
</details>

### Analyse end-to-end tests results

<details><summary>show</summary>
<p>

```bash
$ go get -u k8s.io/test-infra/kubetest
$ kubetest --extract=v1.11.3
$ export KUBE_MASTER_IP="IP ADDRESS"
$ export KUBE_MASTER=<master host>
$ cd kubernetes
$ kubetest --test --provider=skeleton > output.txt

For Conformance Test run this:

$ kubetest --test --provider=skeleton --test_args="--ginkgo.focus=\[Conformance\]" > output.txt

```

</p>
</details>

### Run Node end-to-end tests

<details><summary>show</summary>
<p>

```bash
$ kubectl get pods
$ kubctl get pods -n kube-system
$ service kube-apiserver status
$ service kube-controller-manager status
$ service kube-scheduler status
$ service kubelet status
$ service kube-proxy status
$ kubectl run nginx --image=nginx
$ kubectl scale replicas=3 deploy/nginx

Kubernetes Test Suite is located here -> https://github.com/kubernetes/test-infra

```

</p>
</details>

