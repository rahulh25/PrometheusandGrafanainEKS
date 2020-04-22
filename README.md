# Deploying Prometheus and Grafana on our Amazon EKS

## Tools used

<ol>
<li>eksctl- Creates a EKS cluster- <a href="https://eksctl.io/">Official Docs</a></li>
<li>kubectl- <a href="https://kubernetes.io/docs/tasks/tools/install-kubectl/">Official Docs</a></li>
<li>helm- packaet manager for our kubernetes - <a href="https://helm.sh/">Official Docs</a></li>
</ol>

## Steps to create a EKS cluster

To install eksctl refer the following document

https://docs.aws.amazon.com/eks/latest/userguide/getting-started-eksctl.html

Once installed run the following command and sit back and get some coffee (It takes roughly 15-20 minutes to create the cluster)

```bash
eksctl create cluster -f cluster.yaml
```
![eksctl command](https://github.com/rahulh25/screenshots/blob/master/grafana/eksctl.png)<br>

## Steps to deploy prometheus on the cluster

To install helm refer the following document

https://helm.sh/docs/intro/install/

Once helm is installed we can use it within our kubernetes. To deploy prometheus on our cluster follow the below steps:

1. Create a namespace

```bash
kubectl create namespace monitoring
```
2. Deploy prometheus on the cluster

```bash
helm install final-prometheus stable/prometheus --namespace monitoring --set alertmanager.persistentVolume.storageClass="gp2",server.persistentVolume.storageClass="gp2",server.service.type=LoadBalancer 
```
![eksctl command](https://github.com/rahulh25/screenshots/blob/master/grafana/prometheus.png)<br>

`Note:` In case the above command gives you any issue run the following commands to troubleshoot it:

```bash
helm repo add stable https://kubernetes-charts.storage.googleapis.com
```

```bash
helm repo update
```

3. Keep in mind the following part in the output generated after deploying prometheus:

```bash
The Prometheus server can be accessed via port 80 on the following DNS name from within your cluster:
final-prometheus-server.monitoring.svc.cluster.local
```

4. Get the external ip for your prometheus service using the following command:

```bash
kubectl get svc -n monitoring -o wide
```
![EXTERNAL IP](https://github.com/rahulh25/screenshots/blob/master/grafana/Get%20prometheus%20address.png)<br>

5. Open the prometheus dashboard on your browser using the external ip. If it gives you a connection error give it a few minutes.<br>

![PROMETHEUS DASHBOARD](https://github.com/rahulh25/screenshots/blob/master/grafana/prometheus.png)<br>

## Steps to deploy grafana on the cluster

1. Run the following command to deploy grafana on our cluster:

```bash
helm install grafana stable/grafana --namespace monitoring --set persistence.storageClassName="gp2" --set adminPassword='EKS!sAWSome' --set datasources."datasources\.yaml".apiVersion=1 --set datasources."datasources\.yaml".datasources[0].name=Prometheus --set datasources."datasources\.yaml".datasources[0].type=prometheus --set datasources."datasources\.yaml".datasources[0].url=http://final-prometheus-server.monitoring.svc.cluster.local --set datasources."datasources\.yaml".datasources[0].access=proxy --set datasources."datasources\.yaml".datasources[0].isDefault=true --set service.type=LoadBalancer
```

`Notice:` how we have used the output from our prometheus mentioned in step 3 above has been used here for "datasources\.yaml".datasources[0].url in the above command. Also we have given the adminPassword here by default.

2. Get the external ip for your grafana service using the following command:

```bash
kubectl get svc -n monitoring -o wide
```

3. Go the external ip and you will see a login page enter the following credentials:<br>

username: <b>admin</b><br>
password: <b>EKS!sAWSome</b><br>

![Grafana login](https://github.com/rahulh25/screenshots/blob/master/grafana/grafana%20login%20page.png)<br>

4. Once you login you can add a grafana dashboard like one shown below (the screenshot uses 8588)

![Grafana dashboard](https://github.com/rahulh25/screenshots/blob/master/grafana/Dashboard.png)<br>

