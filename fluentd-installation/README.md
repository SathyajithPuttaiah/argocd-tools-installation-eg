# Fluentd installation guide to EKS

Set up Fluentd as a DaemonSet to send logs to CloudWatch Logs
In the following steps, you set up Fluentd as a DaemonSet to send logs to CloudWatch Logs. When you complete this step, Fluentd creates the following log groups if they don't already exist.

| Log group name              | Log source |
| :---------------- | :------: |
| /aws/containerinsights/Cluster_Name/application      |   All log files in /var/log/containers   |
| /aws/containerinsights/Cluster_Name/host         |   Logs from /var/log/dmesg, /var/log/secure, and /var/log/messages   |
| /aws/containerinsights/Cluster_Name/dataplane   |  The logs in /var/log/journal for kubelet.service, kubeproxy.service, and docker.service.   |

## Getting started with the installation of Fluentd in EKS
## Steps to be followed

### Step 1: Create a namespace for CloudWatch
Use the following step to create a Kubernetes namespace called amazon-cloudwatch for CloudWatch. You can skip this step if you have already created this namespace.

#### To create a namespace for CloudWatch

```
kubectl apply -f https://raw.githubusercontent.com/aws-samples/amazon-cloudwatch-container-insights/latest/k8s-deployment-manifest-templates/deployment-mode/daemonset/container-insights-monitoring/cloudwatch-namespace.yaml
```

### Step 2: Install Fluentd

Start this process by downloading Fluentd. When you finish these steps, the deployment creates the following resources on the cluster:

- A service account named fluentd in the amazon-cloudwatch namespace. This service account is used to run the Fluentd DaemonSet.
- A cluster role named fluentd in the amazon-cloudwatch namespace. This cluster role grants get, list, and watch permissions on pod logs to the fluentd service account.
- A ConfigMap named fluentd-config in the amazon-cloudwatch namespace. This ConfigMap contains the configuration to be used by Fluentd.

#### To install Fluentd
1. Create a ConfigMap named cluster-info with the cluster name and the AWS Region that the logs will be sent to. Run the following command, updating the placeholders with your cluster and Region names.
```
kubectl create configmap cluster-info \
--from-literal=cluster.name=cluster_name \
--from-literal=logs.region=region_name -n amazon-cloudwatch
```

2. Download and deploy the Fluentd DaemonSet to the cluster by running the following command.
```
kubectl apply -f https://raw.githubusercontent.com/aws-samples/amazon-cloudwatch-container-insights/latest/k8s-deployment-manifest-templates/deployment-mode/daemonset/container-insights-monitoring/fluentd/fluentd.yaml
```

3. Validate the deployment by running the following command. Each node should have one pod named fluentd-cloudwatch-*.
```
kubectl get pods -n amazon-cloudwatch
```

### Step 3: Verify the Fluentd setup
To verify your Fluentd setup, use the following steps.

To verify the Fluentd setup for Container Insights
1. Open the CloudWatch console at [console](https://console.aws.amazon.com/cloudwatch/)
2. In the navigation pane, choose Log groups. Make sure that you're in the Region where you deployed Fluentd to your containers.

In the list of log groups in the Region, you should see the following:

/aws/containerinsights/Cluster_Name/application
/aws/containerinsights/Cluster_Name/host
/aws/containerinsights/Cluster_Name/dataplane

If you see these log groups, the Fluentd setup is verified.


$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$

# Steps to execute:

1. Create a namespace for CloudWatch
```
kubectl apply -f https://raw.githubusercontent.com/aws-samples/amazon-cloudwatch-container-insights/latest/k8s-deployment-manifest-templates/deployment-mode/daemonset/container-insights-monitoring/cloudwatch-namespace.yaml
```
```
[ec2-user@ip-172-31-0-85 ~]$ kubectl apply -f https://raw.githubusercontent.com/aws-samples/amazon-cloudwatch-container-insights/latest/k8s-deployment-manifest-templates/deployment-mode/daemonset/container-insights-monitoring/cloudwatch-namespace.yaml
namespace/amazon-cloudwatch configured
```
2. Install Fluentd

#### To install Fluentd

    1. reate a ConfigMap named cluster-info with the cluster name and the AWS Region that the logs will be sent to
    ```
    kubectl create configmap cluster-info \
    --from-literal=cluster.name=cluster_name \
    --from-literal=logs.region=region_name -n amazon-cloudwatch
    ```

    2. Download and deploy the Fluentd DaemonSet to the cluster by running the following command

    *Note* : Changes to be done for fluentd.yaml:

    - Change the parse type from json to cri (container runtime interface)
    - update the fluentd image with latest one : fluent/fluentd-kubernetes-daemonset:v1.16.1-debian-cloudwatch-1.2

    And then apply the yaml:
    ```
    [ec2-user@ip-172-31-0-85 fluentd]$ kubectl apply -f fluentd.yaml -n amazon-cloudwatch
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