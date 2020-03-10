                    **Monasca deployment on Kubernetes Instructions**


**Introduction**

This chart bootstraps Monasca deployment on a Kubernetes cluster using the Helm Package manager.

**Prerequisites**

Kubernetes 1.4+ with 1 master + 3 worker nodes


**1. Kafka Installation :**

This chart will do the following:

Implement a dynamically scalable kafka cluster using Kubernetes StatefulSets

Implement a dynamically scalable zookeeper cluster as another Kubernetes StatefulSet required for the Kafka cluster above


Installing the Chart

To install the chart with the release name standalone-kafka in the monasca namespace:
```
# helm repo add incubator http://storage.googleapis.com/kubernetes-charts-incubator
# mkdir monasca-install
# cd monasca-install
# git clone https://github.com/cloud-operation/monasca-helm.git
# cd monasca-helm/charts
# helm install -f kafka_values.yaml --name standalone-kafka --namespace monasca incubator/kafka
# kubectl -n monasca get sts
NAME                         READY   AGE
standalone-kafka             3/3     5m26s
standalone-kafka-zookeeper   3/3     5m26s
# kubectl -n monasca get jobs
NAME                               COMPLETIONS   DURATION   AGE
standalone-kafka-config-8145cacc   0/1           5m44s      5m44s
```

**2. Kafka manager Installation :**
```
# helm install -f kafka-manager-values.yaml --name kafka-manager --namespace monasca stable/kafka-manager
# kubectl -n monasca get pods
NAME                                                   READY   STATUS      RESTARTS   AGE
kafka-manager-cf9d4bd4b-rm55z                          1/1     Running     0          53s
kafka-manager-kafka-manager-bootstrap-09113f4d-hnx2b   0/1     Completed   3          53s
standalone-kafka-0                                     2/2     Running     1          11m
standalone-kafka-1                                     2/2     Running     0          9m49s
standalone-kafka-2                                     2/2     Running     0          8m12s
standalone-kafka-config-8145cacc-5cl7z                 0/1     Completed   5          11m
standalone-kafka-zookeeper-0                           1/1     Running     0          11m
standalone-kafka-zookeeper-1                           1/1     Running     0          11m
standalone-kafka-zookeeper-2                           1/1     Running     0          9m58s
```
This will host kafka manager on http://kafka-manager.brilliant.com.bd using ingress routing. The default seven monasca topics and one default topic (consumer_offset) should also have been created.


**3. Monasca Installation :**

This chart bootstraps a Monasca deployment on a Kubernetes cluster using the Helm Package manager.


Installing the Chart:
```
# helm repo add monasca http://monasca.io/monasca-helm
# helm install -f mysql/values.yaml --name standalone-monasca-mysql --namespace monasca monasca/monasca
# helm install -f influxdb/values.yaml --name standalone-monasca-influxdb --namespace monasca monasca/monasca
# helm install -f monasca/values.yaml --name monasca --namespace monasca monasca/monasca
```
Deploying Ingress for Monasca API:
```
# kubectl apply -f monasca-api-ingress.yaml
```
This will host monasca api on http://mon-api.brilliant.com.bd using ingress routing

Modify monasca-client deployment to point to external keystone endpoint:
```
# kubectl -n monasca edit deploy monasca-api
...
        env:
        - name: OS_AUTH_URL
          value: http://cloud.brilliant.com.bd:5000/v3
...
```
This will point monasca-client to retrive token from http://cloud.brilliant.com.bd:5000/v3 (external keystone)


**4. Patch Persistent Volumes to retain:**
It is very important to update the PVs created and set them to "Retain" from "Delete". Delete setting will delete the PVs once the pod gets deleted.
```

# kubectl patch pv -p '{"spec":{"persistentVolumeReclaimPolicy":"Retain"}}' pvc-372af896-6818-11e9-8d26-02c0777b771b
persistentvolume/pvc-372af896-6818-11e9-8d26-02c0777b771b patched
# ... (Repeat for all other PVs)
```
**5. Deploy grafana dashboard for monasca**
```
# kubectl apply -f grafana/configmap.yaml
# kubectl apply -f grafana/deployment.yaml
# kubectl apply -f grafana/service.yaml
# kubectl apply -f grafana/grafana-init.yaml
# kubectl apply -f grafana/grafana-ingress.yaml

# kubectl -n monasca get svc
kafka-manager                         ClusterIP   10.102.130.56    <none>        9000/TCP                     94m
monasca-api                           ClusterIP   10.111.198.134   <none>        8070/TCP                     92m
monasca-grafana                       ClusterIP   10.108.32.60     <none>        3000TCP                      28m
monasca-influxdb                      ClusterIP   10.97.0.249      <none>        8086/TCP                     92m
monasca-memcached                     ClusterIP   10.98.28.187     <none>        11211/TCP                    92m
monasca-mysql                         ClusterIP   10.105.211.107   <none>        3306/TCP                     92m
standalone-kafka                      ClusterIP   10.98.138.137    <none>        9092/TCP                     101m
standalone-kafka-headless             ClusterIP   None             <none>        9092/TCP                     101m
standalone-kafka-zookeeper            ClusterIP   10.108.75.69     <none>        2181/TCP                     101m
standalone-kafka-zookeeper-headless   ClusterIP   None             <none>        2181/TCP,3888/TCP,2888/TCP   101m

# kubectl -n monasca get ing
```

Open grafana on http://grafana-mon.brilliant.com.bd with keystone user "monasca" and password.
