# Instructions to deploy a sample nginx app in EKS cluster

In the following steps, you set up a sample nginx application that runs on EKS cluste.

## Getting started with the installation of nginx app in EKS
## Steps to be followed

#### Clone a sample nginx application repo:
```
git clone https://github.com/yadhu621/eks-ingress.git
```


### Step 1: Create a an EKS Cluster:
```
eksctl create cluster eks-cluster.yaml
```
#### Wait for sometime and check for nodes, services runnings
```
kubectl get nodes
kubectl get namespaces
kubectl get pods -A
kubectl get svc -A
```

### Step 2: Install a sampleapp ( of service type LoadBalancer )
```
cd sampleapp
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

```
kubectl get pods -A
NAMESPACE     NAME                       READY   STATUS    RESTARTS   AGE
default       sampleapp-ff9ffdfd-wp9rv   1/1     Running   0          9m14s
kube-system   aws-node-gbn47             1/1     Running   0          20m
kube-system   aws-node-v5nmr             1/1     Running   0          20m
kube-system   aws-node-xt7rj             1/1     Running   0          20m
kube-system   coredns-55bd85dd5d-fnstn   1/1     Running   0          31m
kube-system   coredns-55bd85dd5d-xvtpg   1/1     Running   0          31m
kube-system   kube-proxy-78k6z           1/1     Running   0          20m
kube-system   kube-proxy-mztqb           1/1     Running   0          20m
kube-system   kube-proxy-w7z4j           1/1     Running   0          20m

kubectl get svc -A
NAME            TYPE           CLUSTER-IP      EXTERNAL-IP                                                              PORT(S)        AGE
kubernetes      ClusterIP      10.100.0.1      <none>                                                                   443/TCP        23m
sampleapp-svc   LoadBalancer   10.100.123.96   a17114cf8f43446e7a273783c223bcfe-986239722.eu-west-2.elb.amazonaws.com   80:32053/TCP   7s
```

### step3: Access the sampleapp
Paste the EXTERNAL-URL in the browser. ( You should see Welcome to nginx! )
a17114cf8f43446e7a273783c223bcfe-986239722.eu-west-2.elb.amazonaws.com

### step4: As you access the sampleapp, you should see the following in the logs of the sampleapp pod

```
kubectl logs sampleapp-ff9ffdfd-wp9rv --follow
111.111.11.11 - - [17/Feb/2023:08:18:03 +0000] "GET / HTTP/1.1" 304 0 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/110.0.0.0 Safari/537.36 Edg/110.0.1587.41" "-"
111.111.11.11 - - [17/Feb/2023:08:18:07 +0000] "GET / HTTP/1.1" 304 0 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/110.0.0.0 Safari/537.36 Edg/110.0.1587.41" "-"
111.111.11.11 - - [17/Feb/2023:08:18:07 +0000] "GET / HTTP/1.1" 304 0 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/110.0.0.0 Safari/537.36 Edg/110.0.1587.41" "-"
```