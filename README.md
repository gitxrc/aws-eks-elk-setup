# AWS EKS Log collection using infamous Elasticsearch, FluentBit and Kibana (ELK)
Demo of creating a Kibana visualization for EKS cluster using Elasticsearch and FluentBit

### Before setting up FluentBit, Elasticsearch, and Kibana, ensure that:

1. You have a AWS EKS Kubernetes cluster with managed node group using EC2 compute.
2. kubectl and helm is installed.
3. You've setup pvc for storage and set it as default in your EKS cluster.
4. You've additional node capacity for required CPU and memory.

### First Step:
---

Deploy Elastisearch:

1. Add Helm Repository

```
helm repo add elastic https://helm.elastic.co
helm repo update
```

2. Deploy Elasticsearch with Helm

```
helm install elasticsearch elastic/elasticsearch \
  --set replicas=3 \
  --set minimumMasterNodes=2 \
  --set persistence.enabled=true \
  --set resources.requests.cpu=1 \
  --set resources.requests.memory=2Gi
```

3. Check status of Elastisearch Deployment

```
kubectl get pods -n default | grep elasticsearch
```

### Second Step
---

Deploy Kibana:

1. Deploy Kibana using Helm

```
helm install kibana elastic/kibana --set service.type=ClusterIP
```

2. Check status of Kibana Deployment

```
kubectl get pods -n default | grep kibana
```

3. Expose Kibana UI

```
kubectl port-forward svc/kibana-kibana 6661:6661
```

You can access the Kibana UI using URL: `http://localhost:6661`

### Third Step

Deploy Fluentbit as Daemonset in your AWS EKS cluster

You can download the fluent-bit-*.yaml files, cd to your local dir and deploy using kubectl

```
kubectl apply -f fluent-bit-*.yaml
```

Get FluentBit Deloyment

```
kubectl get pods -n kube-system | grep fluentbit
```

[Replace fluentbit-***** with one of your pod names]

Before you start using Kibana to access your logs, you need to configure FluentBit configMap. You can use the following command to edit the FleuntBit configMap

```
kubectl edit configmap fluentbit-config -n kube-system
```

The default username is `elastic` and you can get the password using the following command

```
kubectl get secret elasticsearch-master-credentials -o jsonpath="{.data.password}" | base64 --decode
```

Update the FluentBit configMap as below:

```
[OUTPUT]
  Name  es
  Match *
  Host  elasticsearch-master.default.svc.cluster.local
  Port  9200
  Index kubernetes-logs
  Suppress_Type_Name  On
  HTTP_User  elastic
  HTTP_Passwd  FQUlEoMMReKPDgMB
  tls  On
  tls.verify  Off
```

You can check FluentBit Logs to verify everything is working as expected and logs are being shipped

Check FluentBit logs

```
kubectl logs fluentbit-***** -n kube-system
```
[Replace fluentbit-***** with one of your pod names]

You are now ready to access your Kibana Dashboard via URL: `http://localhost:6661`

Enter the same username and password you used for configMap

Go to Discover, and you will be presented with setup for your kubenetes logs Index Stream.

Select the `kubernetes-logs` and save.

You can create a new deployment to test the log generation and visualization.




