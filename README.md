# deploy on k8s 
(-> kms)


## Reflect on dn <img src="6463443b4648fab8dc73c49f_45.png" height="20" width="20">
> 1. Compare the application logs before and after you exposed it as a Service. Try to open the app several times while the proxy into the Service is running. What do you see in the logs? Does the number of logs increase each time you open the app?

Before exposing it as a Service, the logs are like this:
```
I0510 06:03:36.423550       1 log.go:195] Started HTTP server on port 8080
I0510 06:03:36.423783       1 log.go:195] Started UDP server on port  8081
```
After exposing it as a Service, the logs are like this:
```
I0510 06:03:36.423550       1 log.go:195] Started HTTP server on port 8080
I0510 06:03:36.423783       1 log.go:195] Started UDP server on port  8081
I0510 06:12:40.428417       1 log.go:195] GET /
I0510 06:12:40.473999       1 log.go:195] GET /
I0510 06:13:28.497326       1 log.go:195] GET /
...
```
The difference is the `GET /` lines, which indicate HTTP packets sent to the node.
This means that the exposing and the proxy was successful, and that HTTP packets sent to the minikube container is successfully sent to and received from the individual nodes.


> 2. Notice that there are two versions of `kubectl get` invocation during this tutorial section. The first does not have any option, while the latter has `-n` option with value set to `kube-system`. What is the purpose of the `-n` option and why did the output not list the pods/services that you explicitly created?

The option `-n` in `kubectl get` is used to set the namespace of the get command.
That means the get only returns objects contained within that namespace.
Usually there are 4 namespaces, of which 2 are used here.
The first is `default`, which is what it gets set to if no option is provided.
This namespace is the default namespace associated to created objects.
Since we do not provide a namespace when creating the nodes and services, their namespace is set to `default`.
The second is `kube-system`, which is associated automatically to objects created by the Kubernetes system (wow).
This namespace usually contains internal systems, like `kube-dns`, `kube-proxy`, `kubernetes-dashboard`.

So, when we run `kubectl get -n kube-system`, it only returns the internal objects and not the pods/services we created.



## this reflect not available yet




## copyable
outside these can be copied, check output

### Create a Deployment
1. 
```shell
kubectl create deployment hello-node --image=registry.k8s.io/e2e-test-images/agnhost:2.39 -- /agnhost netexec --http-port=8080
```

6.
jalanin `kubectl get pods` outputnya 
```
NAME                          READY   STATUS    RESTARTS   AGE
hello-node-55fdcd95bf-ncbql   1/1     Running   0          116s
```
<POD_NAME> = string dibawah kolom name\
contoh:
```shell
kubectl logs hello-node-55fdcd95bf-ncbql
```

### Create a Service
1.
```shell
kubectl expose deployment hello-node --type=LoadBalancer --port=8080
```
