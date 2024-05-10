# deploy on k8s 
(-> kns)


## Reflect on dn <img src="https://assets-global.website-files.com/646218c67da47160c64a84d5/6463443b4648fab8dc73c49f_45.png" height="20" width="20">
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



## this reflect ~~not~~ available ~~yet~~

> 1. What is the difference between Rolling Update and Recreate deployment strategy?

RollingUpdate deployment strategy updates the servers one-by-one, while the server keeps running with the servers available. This means during the update, there will a mix of servers with old and new versions.

Recreate deployment strategy kills all the servers first, then creates new servers with the update. This effectively takes down the server, since no server will be alive to handle the requests. However, the benefit is that there will be no moment in which old and new servers exist together.

>2. Try deploying the Spring Petclinic REST using Recreate deployment strategy and document your attempt.

To change deployment strategy, I created a yaml file to patch the strategy type, seen here

`patch-recreate.yaml`
```yaml
spec:
  strategy:
    $retainKeys:
    - type
    type: Recreate
```
Then, I run this command to apply the patch.
```shell
kubectl patch deployments spring-petclinic-rest --patch-file ./patch-recreate.yaml
```
After rolling out the update, this is the output of `kubectl rollout status deployments/spring-petclinic-rest`
```
Waiting for deployment "spring-petclinic-rest" rollout to finish: 0 of 4 updated replicas are available...
Waiting for deployment "spring-petclinic-rest" rollout to finish: 1 of 4 updated replicas are available...
Waiting for deployment "spring-petclinic-rest" rollout to finish: 2 of 4 updated replicas are available...
Waiting for deployment "spring-petclinic-rest" rollout to finish: 3 of 4 updated replicas are available...
deployment "spring-petclinic-rest" successfully rolled out
```
As seen here, the deployment starts at 0 replicas available, this means all of the replicas were 


>3. Prepare different manifest files for executing Recreate deployment strategy.

```shell
kubectl get deployments/spring-petclinic-rest -o yaml > deployment-recreate.yaml
```
In this repository, there are 2 deployment files, [deployment.yaml](deployment.yaml) for deploying with RollingUpdate strategy, and [deployment-recreate.yaml](deployment-recreate.yaml), for deploying with Recreate strategy.


>4. What do you think are the benefits of using Kubernetes manifest files? Recall your experience in deploying the app manually and compare it to your experience when deploying the same app by applying the manifest files (i.e., invoking `kubectl apply -f` command) to the cluster.

Using manifest files makes it easier and faster to deploy the app, since we don't need to set up the deployment again.
We only need to run 1 command to apply all the settings needed.
Manifest files also provide a backup to use after we delete the minikube Kubernetes cluster.
This makes it faster to get the deployment up and running after restarting a cluster.




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

### Create Another Deployment
1. 
```shell
kubectl create deployment spring-petclinic-rest --image=docker.io/springcommunity/spring-petclinic-rest:3.0.2
```

### Create Another Service
1.
```shell
kubectl expose deployment spring-petclinic-rest --type="LoadBalancer" --port 9966
```

### Scale the App
3.
```shell
kubectl scale deployments/spring-petclinic-rest --replicas=4
```

### Perform a Rolling Update
2.
```shell
kubectl set image deployments/spring-petclinic-rest spring-petclinic-rest=docker.io/springcommunity/spring-petclinic-rest:3.2.1
```

5.
```shell
kubectl set image deployments/spring-petclinic-rest spring-petclinic-rest=docker.io/springcommunity/spring-petclinic-rest:4.0
```

### Write Kubernetes Manifest Files
1.
```shell
kubectl get deployments/spring-petclinic-rest -o yaml > deployment.yaml
```