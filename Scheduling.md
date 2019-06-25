# Scheduling (5%)

kubernetes.io > Documentation > Reference > kubectl CLI > [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

kubernetes.io > Documentation > Concepts > Configuration > [Managing Compute Resources for Containers](https://kubernetes.io/docs/concepts/configuration/manage-compute-resources-container/)

kubernetes.io > Documentation > Concepts > Workload > Controllers > [DaemonSets](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/)

kubernetes.io > Documentation > Concepts > Configuration > [Assigning Pods to Nodes](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/)

kubernetes.io > Documentation > Concepts > Configuration > [Taints and Tolerations](https://kubernetes.io/docs/concepts/configuration/taint-and-toleration/)

kubernetes.io > Documentation > Tasks > Administer a Cluster > [Configure Multiple Schedulers](https://kubernetes.io/docs/tasks/administer-cluster/configure-multiple-schedulers/)

kubernetes.io > Documentation > Tasks > Administer a Cluster > Manage Memory, CPU, and API Resources > [Configure Default Memory Requests and Limits for a Namespace](https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/memory-default-namespace/)

kubernetes.io > Documentation > Tasks > Administer a Cluster > Static Pods > [Static Pods](https://kubernetes.io/docs/tasks/administer-cluster/static-pod/)

kubernetes.io > Documentation > Reference > Command line tools reference > [kube-scheduler](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-scheduler/)

### 

### Use label selectors to schedule Pods

<details><summary>show</summary>
<p>

```bash
$ kubectl label nodes node-1 size=Large
$ cat pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  -	name: nginx
    image: nginx
  nodeSelector:
    size: Large

```

</p>
</details>

### Understand the role of DaemonSets

<details><summary>show</summary>
<p>

Create a YAML:

```bash
$ cat daemonsets.yaml
```

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: myapp-rc
spec:
  selector:
    matchLabels: 
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
        - name: nginx-container
          image: nginx
```

```bash
$ kubectl create -f daemonset.yaml
```

</p>
</details>

### Understand how resource limits can affect Pod Scheduling

<details><summary>show</summary>
<p>

```
$ cat namespacequota.yaml

apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
  namespace: default
spec:
  hard:
    pods: "10"
    requests.cpu: "4"
    requests.memory: 4Gi
    limits.cpu: "10"
    limits.memory: 10Gi
    
$ cat podquota.yaml

apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
      - containerPort: 8080
    resources:
      requests:
        memory: "1Gi"
        cpu: 1
      limits:
        memory: "2Gi"
        cpu: 2
```



</p>
</details>

### Understand how to run multiple schedulers and how to configure Pods to use them

<details><summary>show</summary>
<p>

```bash
$ cat schedulerpod.yaml
```

```YAML
apiVersion: v1
kind: Pod
metadata:
  name: my-custom-scheduler
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-scheduler
    - --address=127.0.0.1
    - --kubeconfig=/etc/kubernetes/scsheduler.conf
    - -- leader-elect=true
    - --lock-object-name=my-custom-scsheduler
    image: k8s.gcr.io/kube-scheduler-amd64:v1.11.3
    name: kube-scheduler
```

```bash
$ cat pod-to-schedule-differently.yaml

apiVersion: v1
kind: Pod
metadata:
  name: my-custom-scheduled-pod
spec:
  containers:
  - image: nginx
    name: nginx
  schedulerName: my-custom-scheduled
```

</p>
</details>

### Manually schedule a Pod without a scheduler

<details><summary>show</summary>
<p>

```bash
Store the POD yaml files in /etc/Kubernetes/manifests

Create a static pod named static-busybox that uses the busybox image and the command sleep 1000

$ kubectl run --restart=Never --image=busybox static-busybox --dry-run -o yaml --command -- sleep 1000 > /etc/kubernetes/manifests/static-busybox.yaml

If you are asked to delete a static Pod from a specific node then run $ kubectl get nodes -o wide to get the Node IP and then ssh to it. Kubelet config file might be /var/lib/kubelet/config.yaml. Check the 'staticPodPath:' and go to that directory and delete the YAML file. 
```

</p>
</details>

### Display scheduler events

<details><summary>show</summary>
<p>

```bash
$ kubectl get events
$ kubectl get events --watch
$ kubectl logs kube-scheduler-bk8s-node0 -n kube-system

/var/log/kube-scheduler.log on the control/master node (if schedule is standalone service)
```

</p>
</details>

### Know how to configure the Kubernetes scheduler

<details><summary>show</summary>
<p>

```bash
$ wget "https://storage.googleapis.com/kubernetes-release/release/v1.12.0/bin/linux/amd64/kube-scheduler"

$ chmod +x kube-scheduler 
$ sudo mv kube-scheduler /usr/local/bin/
$ sudo mv kube-scheduler.kubeconfig /var/lib/kubernetes/
$ cat <<EOF | sudo tee /etc/kubernetes/config/kube-scheduler.yaml
apiVersion: componentconfig/v1alpha1
kind: KubeSchedulerConfiguration
clientConnection:
  kubeconfig: "/var/lib/kubernetes/kube-scheduler.kubeconfig"
leaderElection:
  leaderElect: true
EOF
$ cat <<EOF | sudo tee /etc/systemd/system/kube-scheduler.service
[Unit]
Description=Kubernetes Scheduler
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-scheduler \\
  --config=/etc/kubernetes/config/kube-scheduler.yaml \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
$ sudo systemctl daemon-reload
$ sudo systemctl enable kube-scheduler
$ sudo systemctl start kube-scheduler
```

</p>
</details>

