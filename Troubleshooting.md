# Troubleshooting (10%)

kubernetes.io > Documentation > Reference > kubectl CLI > [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

kubernetes.io > Documentation > Tasks > Monitor, Logging & Debugging > [Determine the Reason for Pod Failure](https://kubernetes.io/docs/tasks/debug-application-cluster/determine-reason-pod-failure/)

kubernetes.io > Documentation > Tasks > Monitor, Logging & Debugging > [Application Introspection and Debugging](https://kubernetes.io/docs/tasks/debug-application-cluster/debug-application-introspection/)

kubernetes.io > Documentation > Tasks > Monitor, Logging & Debugging > [Debug Services](https://kubernetes.io/docs/tasks/debug-application-cluster/debug-service/)

kubernetes.io > Documentation > Tasks > Monitor, Logging & Debugging > [Troubleshoot Applications](https://kubernetes.io/docs/tasks/debug-application-cluster/debug-application/)

kubernetes.io > Documentation > Tasks > Monitor, Logging & Debugging > [Troubleshoot Clusters](https://kubernetes.io/docs/tasks/debug-application-cluster/debug-cluster/)

###  

### Troubleshoot application failure

<details><summary>show</summary>
<p>

```bash
Check Accessibility

$ curl http://web-service-ip:node-port

Check Service Status

$ kubectl describe svc web-service

compare the endpoints and selector on the POD definition

Check the POD

$ kubectl get po
$ kubectl describe po web
$ kubectl logs web
$ kubectl logs web -f
$ kubectl logs web -f --previous

```

</p>
</details>

### Troubleshoot control plane failure

<details><summary>show</summary>
<p>

```bash
Check Node Status

$ kubectl get nodes
$ kubectl get pods

Check Controlplane Pods

$ kubectl get pods -n kube-system

Check Controlplane Services

$ service kube-apiserver status
$ service kube-controller-manager status
$ service kube-scheduler status
$ service kubelet status
$ service kube-proxy status

Check Service Logs

$ kubectl logs kube-apiserver-master -n kube-system
$ sudo journalctl -u kube-apiserver

```

</p>
</details>



### Troubleshoot worker node failure

<details><summary>show</summary>
<p>

```bash
Check Node Status

$ kubectl get nodes
$ kubectl describe node <nodename>
$ top
$ df -h
$ service kubelet status
$ journalctl -u kubelet -f
$ openssl x509 -in /var/lib/kubelet/worker-1.crt -text

```

</p>
</details>

### Troubleshoot networking

<details><summary>show</summary>
<p>

```bash

Make sure you’re connecting to the service’s cluster IP from within the cluster, not from the outside.

Don’t bother pinging the service IP to figure out if the service is accessible (remember, the service’s cluster IP is a virtual IP and pinging it will never work).

If you’ve defined a readiness probe, make sure it’s succeeding; otherwise the pod won’t be part of the service.

To confirm that a pod is part of the service, examine the corresponding Endpoints object with kubectl get endpoints.

If you’re trying to access the service through its FQDN or a part of it (for example, myservice.mynamespace.svc.cluster.local or myservice.mynamespace) and it doesn’t work, see if you can access it using its cluster IP instead of the FQDN.

Check whether you’re connecting to the port exposed by the service and not the target port.

Try connecting to the pod IP directly to confirm your pod is accepting connections on the correct port.

If you can’t even access your app through the pod’s IP, make sure your app isn’t only binding to localhost.

```

</p>
</details>

