# Fluentd installation guide to EKS

Set up Fluentd as a DaemonSet to send logs to CloudWatch Logs
In the following steps, you set up Fluentd as a DaemonSet to send logs to CloudWatch Logs. When you complete this step, Fluentd creates the following log groups if they don't already exist.

| Log group name              | Log source |
| :---------------- | :------: |
| /aws/containerinsights/Cluster_Name/application      |   All log files in /var/log/containers   |
| /aws/containerinsights/Cluster_Name/host         |   Logs from /var/log/dmesg, /var/log/secure, and /var/log/messages   |
| /aws/containerinsights/Cluster_Name/dataplane   |  The logs in /var/log/journal for kubelet.service, kubeproxy.service, and docker.service.   |

## Getting started with the installation of Fluentd in EKS

# Steps to execute:

#### Clone the repo:
```
git clone https://github.com/SathyajithPuttaiah/argocd-tools-installation-eg.git
cd /argocd-tools-installation-eg/fluentd-installation/yaml_manifests
```

1. Create a namespace for CloudWatch
```
kubectl apply -f cloudwatch-namespace.yaml
```
```
[ec2-user@ip ~]$ kubectl apply -f cloudwatch-namespace.yaml
namespace/amazon-cloudwatch configured
```
2. Install Fluentd

#### To install Fluentd

1. Create a ConfigMap named cluster-info with the cluster name and the AWS Region that the logs will be sent to
```
kubectl create configmap cluster-info \
--from-literal=cluster.name=<cluster_name> \
--from-literal=logs.region=<region_name> -n amazon-cloudwatch
```

2. Deploy the Fluentd DaemonSet to the cluster by running the following command

*Note : Changes to be done for fluentd.yaml:*

- Change the parse type from json to cri (container runtime interface)
- update the fluentd image with latest one : fluent/fluentd-kubernetes-daemonset:v1.16.1-debian-cloudwatch-1.2

And then apply the yaml:
```
[ec2-user@ip- fluentd]$ kubectl apply -f fluentd.yaml -n amazon-cloudwatch
serviceaccount/fluentd configured
clusterrole.rbac.authorization.k8s.io/fluentd-role configured
clusterrolebinding.rbac.authorization.k8s.io/fluentd-role-binding configured
configmap/fluentd-config configured
daemonset.apps/fluentd-cloudwatch configured
```

3. Validate the deployment by running the following command. Each node should have one pod named fluentd-cloudwatch-*
```
kubectl get pods -n amazon-cloudwatch
```
4. Verify the Fluentd setup

In the list of log groups in the Region, you should see the following:
- /aws/containerinsights/Cluster_Name/application
- /aws/containerinsights/Cluster_Name/host
- /aws/containerinsights/Cluster_Name/dataplane