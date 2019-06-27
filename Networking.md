# Networking (11%)

kubernetes.io > Documentation > Reference > kubectl CLI > [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

kubernetes.io > Documentation > Concepts > Cluster Administration > [Cluster Networking](https://kubernetes.io/docs/concepts/cluster-administration/networking/)

kubernetes.io > Documentation > Concepts > Services, Load Balancing, and Networking > [Service](https://kubernetes.io/docs/concepts/services-networking/service/)

kubernetes.io > Documentation > Tasks > Access Applications in a Cluster > [Create an External Load Balancer](https://kubernetes.io/docs/tasks/access-application-cluster/create-external-load-balancer/)

kubernetes.io > Documentation > Concepts > Services, Load Balancing, and Networking > [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/)

kubernetes.io > Documentation > Concepts > Services, Load Balancing, and Networking > [DNS for Services and Pods](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/)

kubernetes.io > Documentation > Concepts > Cluster Administration > [Installing Addons](https://kubernetes.io/docs/concepts/cluster-administration/addons/)



###  

### Understand the Networking configuration of the cluster nodes

<details><summary>show</summary>
<p>

```bash
$ ip addr
$ ip link
$ ip link show ens3
$ arp node01
$ ip link show docker0 
$ ip route show default
$ netstat -nplt
$ netstat -anp | grep etcd

```

</p>
</details>

### Understand Pod networking concepts

<details><summary>show</summary>
<p>



```bash
$ ip netns add white
$ ip netns
$ ip netns exec white ip link
$ ip -n red link
$ ip netns exec white arp
$ ip netns exec white route
$ ip link set veth-white netns white
$ ip -n white addr add 192.168.1.1 dev veth-white
$ ip -n white link set veth-white up
$ ip link add v-net-0 type bridge
$ ip link set dev v-net-0 up
$ ip link add veth-white type veth peer name veth-white-br
$ ip link set veth-white netns white
$ ip link set veth-white-br master v-net-0
$ ip -n white addr add 192.168.1.1 dev veth-white
$ ip -n white link set veth-white up
$ docker network ls
$ docker inspect <network ns>

```



![POD Communication](https://github.com/stretchcloud/cka-lab-practice/blob/master/pod-networking.jpg)



</p>
</details>

### Understand service networking

<details><summary>show</summary>
<p>

```
$ ps aux | grep kube-api

--service-cluster-ip-range=10.0.0.0/24

$ iptables -L -t net | grep <service name>
$ cat /var/log/kube-proxy.log
$ kubectl logs weave-net-cwpbj weave -n kube-system

check for ipalloc-range:

$ kubectl logs <kube-proxy-pod> -n kube-system

Check for "Flag proxy-mode="" unknown, assuming iptables proxy"
```



</p>
</details>

### Deploy and configure network load balancer

<details><summary>show</summary>
<p>

```bash
$ cat influxdbpod.yaml

apiVersion: v1
kind: Pod
metadata:
  name: influxdb
  labels:
    name: influxdb
spec:
  containers:
     - name: influxdb
       image: influxdb
       ports:
         - containerPort: 8086

$ cat influxdbservice.yaml

kind: Service
apiVersion: v1
metadata:
  name: influxdb
spec:
  type: LoadBalancer
  ports:
    - port: 8086
  selector:
    name: influxdb

```



</p>
</details>

### Know how to use Ingress rules

<details><summary>show</summary>
<p>

```bash
$ cat ingress-controller.yaml

---

kind: Namespace
apiVersion: v1
metadata:
  name: ingress-space

---

kind: ConfigMap
apiVersion: v1
metadata:
  name: nginx-configuration
  namespace: ingress-space

---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: nginx-ingress-controller
  namespace: ingress-space
spec:
  replicas: 1
  selector:
    matchLabels:
      name: nginx-ingress
  template:
    metadata:
      labels:
        name: nginx-ingress
    spec:
      serviceAccountName: nginx-ingress-serviceaccount
      containers:
        - name: nginx-ingress-controller
          image: quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.21.0
          args:
            - /nginx-ingress-controller
            - --configmap=$(POD_NAMESPACE)/nginx-configuration
            - --default-backend-service=app-space/default-http-backend
          env:
            - name: POD_NAME
              valueFrom:
                fieldRef:
                  fieldPath: metadata.name
            - name: POD_NAMESPACE
              valueFrom:
                fieldRef:
                  fieldPath: metadata.namespace
          ports:
            - name: http
              containerPort: 80
            - name: https
              containerPort: 443

---
apiVersion: v1
kind: Service
metadata:
  name: ingress-service
  namespace: ingress-space
spec:
  type: NodePort
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
    nodePort: 30080
    name: http
  - port: 443
    targetPort: 443
    protocol: TCP
    name: https
  selector:
    name: nginx-ingress


---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: nginx-ingress-serviceaccount
  namespace: ingress-space
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx

---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRole
metadata:
  name: nginx-ingress-clusterrole
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
rules:
  - apiGroups:
      - ""
    resources:
      - configmaps
      - endpoints
      - nodes
      - pods
      - secrets
    verbs:
      - list
      - watch
  - apiGroups:
      - ""
    resources:
      - nodes
    verbs:
      - get
  - apiGroups:
      - ""
    resources:
      - services
    verbs:
      - get
      - list
      - watch
  - apiGroups:
      - "extensions"
    resources:
      - ingresses
    verbs:
      - get
      - list
      - watch
  - apiGroups:
      - ""
    resources:
      - events
    verbs:
      - create
      - patch
  - apiGroups:
      - "extensions"
    resources:
      - ingresses/status
    verbs:
      - update

---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: Role
metadata:
  name: nginx-ingress-role
  namespace: ingress-space
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
rules:
  - apiGroups:
      - ""
    resources:
      - configmaps
      - pods
      - secrets
      - namespaces
    verbs:
      - get
  - apiGroups:
      - ""
    resources:
      - configmaps
    resourceNames:
      # Defaults to "<election-id>-<ingress-class>"
      # Here: "<ingress-controller-leader>-<nginx>"
      # This has to be adapted if you change either parameter
      # when launching the nginx-ingress-controller.
      - "ingress-controller-leader-nginx"
    verbs:
      - get
      - update
  - apiGroups:
      - ""
    resources:
      - configmaps
    verbs:
      - create
  - apiGroups:
      - ""
    resources:
      - endpoints
    verbs:
      - get

---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: RoleBinding
metadata:
  name: nginx-ingress-role-nisa-binding
  namespace: ingress-space
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: nginx-ingress-role
subjects:
  - kind: ServiceAccount
    name: nginx-ingress-serviceaccount


---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: nginx-ingress-clusterrole-nisa-binding
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: nginx-ingress-clusterrole
subjects:
  - kind: ServiceAccount
    name: nginx-ingress-serviceaccount
    namespace: ingress-space


$ cat ingress-resource.yaml

---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-wear-watch
  namespace: app-space
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
spec:
  rules:
  - http:
      paths:
      - path: /wear
        backend:
          serviceName: wear-service
          servicePort: 8080
      - path: /watch
        backend:
          serviceName: video-service
          servicePort: 8080

$ kubectl get ingress
$ kubectl describe ingress --namespace app-space
$ kubectl create ns ingress-space
$ kubectl create configmap nginx-configuration --namespace ingress-space
$ kubectl create serviceaccount ingress-serviceaccount --namespace ingress-space
$ kubectl get roles,rolebindings --namespace ingress-space
$ kubectl expose deployment -n ingress-space ingress-controller --type=NodePort --port=80 --name=ingress --dry-run -o yaml > ingress.yaml

```



![Ingress Flow](https://github.com/stretchcloud/cka-lab-practice/blob/master/Ingress-Flow.png)



</p>
</details>

### Know how to configure and use the cluster DNS

<details><summary>show</summary>
<p>

```bash
$ curl http://web-service.apps.svc.cluster.local
$ curl http://10-10-10-5.apps.pod.cluster.local
$ cat /etc/coredns/Corefile
$ kubectl get configmap -n kube-system
$ kubectl get service -n kube-system
$ ps aux | grep coredns

-conf /etc/coredns/Corefile

$ kubectl exec <coredns pod> -n kube-system ps
$ kubectl describe configmap coredns -n kube-system
$ kubectl set env deployment/webapp DB_Host=mysql.payroll
$ kubectl exec -it hr nslookup mysql.payroll > /root/nslookup.out

```

</p>
</details>

### Understand CNI

<details><summary>show</summary>
<p>

```bash
$ cat /etc/system/system.d/kubelet.service

--network-plugin=cni \\
--cni-bin-dir=/opt/cni/bin \\
--cni-conf-dir=/etc/cni/net.d \\

$ ps -aux | grep -i kubelet
$ cat /etc/cni/net.d/net-script.conf

{
	"cniversion": "0.2.0",
	"name": "mynet",
	"type": "net-script",
	"bridge": "cni0",
	"isGateway": true,
	"ipMasq": true,
	"ipam": {
		"type": "host-local",
		"subnet": "10.10.0.0/16",
		"routes": [
		{
			"dst": "0.0.0.0/0"
		}
		]
	}
}

$ kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"

Weave CNI Range -> 10.32.0.0/12 (10.32.0.1 - 10.47.255.254)

$ ip addr show weave
```

</p>
</details>

