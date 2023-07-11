# Set up the CloudWatch agent to collect cluster metrics

In the following steps, you set up the CloudWatch agent to be able to collect metrics from your clusters.

## Getting started with the installation of CloudWatch agent in EKS
## Steps to be followed

### Step 1: Create a namespace for CloudWatch
Use the following step to create a Kubernetes namespace called amazon-cloudwatch for CloudWatch. You can skip this step if you have already created this namespace.

#### To create a namespace for CloudWatch

```
kubectl apply -f https://raw.githubusercontent.com/aws-samples/amazon-cloudwatch-container-insights/latest/k8s-deployment-manifest-templates/deployment-mode/daemonset/container-insights-monitoring/cloudwatch-namespace.yaml
```

### Step 2: Create a service account in the cluster

Use the following step to create a service account for the CloudWatch agent, if you do not already have one.

#### To create a service account for the CloudWatch agent

- Enter the following command.
```
kubectl apply -f https://raw.githubusercontent.com/aws-samples/amazon-cloudwatch-container-insights/latest/k8s-deployment-manifest-templates/deployment-mode/daemonset/container-insights-monitoring/cwagent/cwagent-serviceaccount.yaml
```
If you didn't follow the previous steps, but you already have a service account for the CloudWatch agent that you want to use, you must ensure that it has the following rules. Additionally, in the rest of the steps in the Container Insights installation, you must use the name of that service account instead of cloudwatch-agent.

```
rules:
  - apiGroups: [""]
    resources: ["pods", "nodes", "endpoints"]
    verbs: ["watch", "list"]
  - apiGroups: [""]
    resources: ["nodes/proxy"]
    verbs: ["get"]
  - apiGroups: [""]
    resources: ["nodes/stats", "configmaps", “events”]
    verbs: ["create"]
  - apiGroups: [""]
    resources: ["configmaps"]
    resourceNames: ["cwagent-clusterleader"]
    verbs: ["get", "update"]
  - nonResourceURLs: ["/metrics"]
    verbs: ["get"]
```

### Step 3: Create a ConfigMap for the CloudWatch agent
Use the following steps to create a ConfigMap for the CloudWatch agent.

#### To create a ConfigMap for the CloudWatch agent

1. Download the ConfigMap YAML to your kubectl client host by running the following command:
```
curl -O https://raw.githubusercontent.com/aws-samples/amazon-cloudwatch-container-insights/latest/k8s-deployment-manifest-templates/deployment-mode/daemonset/container-insights-monitoring/cwagent/cwagent-configmap.yaml
```

2. Edit the downloaded YAML file, as follows:\
- *cluster_name* - In the kubernetes section, replace {{cluster-name}} with the name of your cluster. Remove the {{}} characters. Alternatively, if you're using an Amazon EKS cluster, you can delete the "cluster_name" field and value. If you do, the CloudWatch agent detects the cluster name from the Amazon EC2 tags.

3. (Optional) Make further changes to the ConfigMap based on your monitoring requirements, as follows:

- *metrics_collection_interval* – In the kubernetes section, you can specify how often the agent collects metrics. The default is 60 seconds. The default cadvisor collection interval in kubelet is 15 seconds, so don't set this value to less than 15 seconds.

- *endpoint_override* – In the logs section, you can specify the CloudWatch Logs endpoint if you want to override the default endpoint. You might want to do this if you're publishing from a cluster in a VPC and you want the data to go to a VPC endpoint.

- *force_flush_interval* – In the logs section, you can specify the interval for batching log events before they are published to CloudWatch Logs. The default is 5 seconds.

- *region* – By default, the agent published metrics to the Region where the worker node is located. To override this, you can add a region field in the agent section: for example, "region":"us-west-2".

- *statsd* section – If you want the CloudWatch Logs agent to also run as a StatsD listener in each worker node of your cluster, you can add a statsd section to the metrics section, as in the following example

```
"metrics": {
  "metrics_collected": {
    "statsd": {
      "service_address":":8125"
    }
  }
}
```

A full example of the JSON section is as follows.

```
{
    "agent": {
        "region": "us-east-1"
    },
    "logs": {
        "metrics_collected": {
            "kubernetes": {
                "cluster_name": "MyCluster",
                "metrics_collection_interval": 60
            }
        },
        "force_flush_interval": 5,
        "endpoint_override": "logs.us-east-1.amazonaws.com"
    },
    "metrics": {
        "metrics_collected": {
            "statsd": {
                "service_address": ":8125"
            }
        }
    }
}
```

4. Create the ConfigMap in the cluster by running the following command.
```
kubectl apply -f cwagent-configmap.yaml
```

### Step 4: Deploy the CloudWatch agent as a DaemonSet
To finish the installation of the CloudWatch agent and begin collecting container metrics, use the following steps.

#### To deploy the CloudWatch agent as a DaemonSet
1. - If you do not want to use StatsD on the cluster, enter the following command.
```
kubectl apply -f https://raw.githubusercontent.com/aws-samples/amazon-cloudwatch-container-insights/latest/k8s-deployment-manifest-templates/deployment-mode/daemonset/container-insights-monitoring/cwagent/cwagent-daemonset.yaml
```

   - If you do want to use StatsD, follow these steps:
        
        a. Download the DaemonSet YAML to your kubectl client host by running the following command.
        ```
        curl -O  https://raw.githubusercontent.com/aws-samples/amazon-cloudwatch-container-insights/latest/k8s-deployment-manifest-templates/deployment-mode/daemonset/container-insights-monitoring/cwagent/cwagent-daemonset.yaml
        ```
        b. Uncomment the port section in the cwagent-daemonset.yaml file as in the following:
        ```
        ports:
            - containerPort: 8125
                hostPort: 8125
                protocol: UDP

        ``` 
        c. Deploy the CloudWatch agent in your cluster by running the following command.
            ```
            kubectl apply -f cwagent-daemonset.yaml
            ```

2. Validate that the agent is deployed by running the following command.
```
kubectl get pods -n amazon-cloudwatch
```

When complete, the CloudWatch agent creates a log group named /aws/containerinsights/Cluster_Name/performance and sends the performance log events to this log group. If you also set up the agent as a StatsD listener, the agent also listens for StatsD metrics on port 8125 with the IP address of the node where the application pod is scheduled.


Note: [CloudWatch_Installation_Guide](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/Container-Insights-setup-metrics.html) 
