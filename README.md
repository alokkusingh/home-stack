# home-stack

<div class="warning" style='padding:0.1em; background-color:#E9D8FD; color:#69337A'>
<span>
<p style='margin-top:1em; text-align:center'>
<b>Home Project Stack</b></p>
<p style='margin-left:1em;'>
The stack is deployed using Kubernetes cluster enabled using microk8s. microk8s is installed using snap package manger. Package is provided by Canonical (publisher of Ubuntu).<br>
- Resources: quad-core ARMx64 processor with 8GB RAM<br>
- Kernel: GNU/Linux 5.4.0-1058-raspi aarch64<br>
- OS: Ubuntu 20.04.4<br><br>
As of now it is deployed on single node cluster.
</p>
<p style='margin-bottom:1em; margin-right:1em; text-align:right; font-family:Georgia'> <b>- Alok Singh</b> 
</p></span>
</div>

### Deployment of home-stack Kubernetes Stack
#### Create Namespaces
````
kubectl apply -f yaml/namespace.yaml
````
#### Create ConfigMap
````
kubectl apply -f yaml/config-map.yaml
````
#### Create Secrets
````
kubectl apply -f yaml/secrets.yaml
````
#### Create Network policy
````
kubectl apply -f yaml/networkpolicy.yaml
````
#### MySQL Service - Pod/Deployment/Service
````
kubectl apply --validate=true --dry-run=client -f yaml/mysql-service.yaml 
````
````
kubectl apply -f yaml/mysql-service.yaml  --namespace=home-stack
````
````
kubectl delete -f yaml/mysql-service.yaml  --namespace=home-stack
````
````
kubectl exec -it pod/mysql-0 --namespace home-stack -- mysql -u root -p home-stack
````
````
kubectl logs pod/mysql-0 --namespace home-stack
````
````
mysql -u root -p home-stack --host 127.0.0.1 --port 32306
````
---
**Note:**
>[Follow the link to configure sqldeveloper on Mac to connect to MySQL server remotely](https://cybercafe.dev/setup-mysql-and-sql-developer-on-macos/ "https://cybercafe.dev/setup-mysql-and-sql-developer-on-macos/")
---
#### Home API Service - Pod/Deployment/Service
````
kubectl apply --validate=true --dry-run=client -f yaml/home-api-service.yaml 
````
````
kubectl apply -f yaml/home-api-service.yaml  --namespace=home-stack
````
````
kubectl delete -f yaml/home-api-service.yaml  --namespace=home-stack
````
````
kubectl exec -it pod/home-api-deployment-0 --namespace home-stack -- bash
````
````
kubectl exec -it pod/home-api-deployment-0 --namespace home-stack -- tail -f /opt/logs/application.log
````
````
kubectl logs pod/home-api-deployment-0 --namespace home-stack
````
````
kubectl rollout restart statefulset.apps/home-api-deployment -n home-stack
````
#### Home ETL Service - Pod/Statefulset/Service
````
kubectl apply --validate=true --dry-run=client -f yaml/home-etl-service.yaml 
````
````
kubectl apply -f yaml/home-etl-service.yaml  --namespace=home-stack
````
````
kubectl delete -f yaml/home-etl-service.yaml  --namespace=home-stack
````
````
kubectl exec -it pod/home-etl-deployment-0 --namespace home-stack -- bash
````
````
kubectl exec -it pod/home-etl-deployment-0 --namespace home-stack -- tail -f /opt/logs/application.log
````
````
kubectl logs pod/home-etl-deployment-0 --namespace home-stack
````
````
kubectl rollout restart statefulset.apps/home-api-deployment -n home-stack
````
#### Home GIT Commit CronJob
````
kubectl apply --validate=true --dry-run=client -f yaml/git-commit-cronjob.yaml 
````
````
kubectl apply -f yaml/git-commit-cronjob.yaml  --namespace=home-stack
````
````
kubectl delete -f yaml/git-commit-cronjob.yaml  --namespace=home-stack
````
#### Statement Parser Service - Pod/Deployment/Service
````
kubectl apply --validate=true --dry-run=client -f yaml/stmt-parser-service.yaml 
````
````
kubectl apply -f yaml/stmt-parser-service.yaml  --namespace=home-stack
````
````
kubectl delete -f yaml/stmt-parser-service.yaml  --namespace=home-stack
````
````
kubectl exec -it pod/stmtparser-deployment-0 --namespace home-stack -- bash
````
````
kubectl exec -it pod/stmtparser-deployment-0 --namespace home-stack -- tail -f /opt/logs/spring-batch.log
````
````
kubectl logs pod/stmtparser-deployment-0 --namespace home-stack
````
````
kubectl rollout restart statefulset.apps/stmtparser-deployment -n home-stack
````
#### Dashboard Service - Pod/Deployment/Service
````
kubectl apply --validate=true --dry-run=client -f yaml/dashboard-service.yaml 
````
````
kubectl apply -f yaml/dashboard-service.yaml  --namespace=home-stack
````
````
kubectl delete -f yaml/dashboard-service.yaml  --namespace=home-stack
````
````
kubectl exec -it deployment.apps/dashboard-deployment --namespace home-stack -- /bin/sh
````
````
kubectl logs deployment.apps/dashboard-deployment --namespace home-stack
````
#### Jaeger Service
````
kubectl apply --validate=true --dry-run=client -f yaml/jaeger-all-in-one-template.yml 
````
````
kubectl apply -f yaml/jaeger-all-in-one-template.yml  --namespace=home-stack
````
````
kubectl delete -f yaml/jaeger-all-in-one-template.yml  --namespace=home-stack
````
#### Delete Stack
````
kubectl delete namespace home-stack 
````
### Kubernetes Dashboard - Pod/Deployment/Service
````
kubectl apply -f yaml/kubernetes-dashboard.yaml
````
````
kubectl delete -f yaml/kubernetes-dashboard.yaml
````
````
kubectl get all --namespace kubernetes-dashboard
````
````
kubectl get secrets -n kubernetes-dashboard
````
````
kubectl get secret kubernetes-dashboard-token-wtmbt -n kubernetes-dashboard -o jsonpath="{.data.token}" | base64 --decode
````
### Kubernetes Metrics Server
````
kubectl apply -f yaml/metrix-server.yaml
````
````
kubectl delete -f yaml/metrix-server.yaml
````
````
kubectl get deployment metrics-server -n kube-system
````
````
kubectl top nodes
````
### Horizon Autoscaling
#### Create HorizonTalPodAutoscaler
````
kubectl autoscale deployment dashboard-deployment --min=2 --max=3 -n home-stack
````
````
kubectl get hpa --namespace home-stack
````

#### Update Scale to 1
````
kubectl edit hpa dashboard-deployment --namespace home-stack
````
````
kubectl scale -n home-stack deployment dashboard-deployment --replicas=1
````

### Miscellaneous commands
#### Get all 
````
kubectl get all --namespace home-stack
````
#### Get Pod Log
````
kubectl logs pod/dashboard-deployment-65cf5b8858-7x8z8 --namespace home-stack
````
#### Describe a Pod
````
kubectl describe pod/dashboard-deployment-65cf5b8858-7x8z8  --namespace=home-stack
````
#### Get All Pods under All Namespaces
````
kubectl get -A pods
````
#### Describe a spec
````
kubectl explain --api-version="batch/v1beta1" cronjobs.spec
````
## Service Mesh - Istio
### Install

---
>To be explored - seems microk8s isteo addon not supported for ARMx64 architecture. Where the same is supported for minikube.
---

## Deployment Architecture
![alt text](https://github.com/alokkusingh/home-stack/blob/main/draw-io/image/HomeStack.drawio.png)

### Services

| Application               | Description                                 | Service Type             | Deployment/StatefulSet/CronJob/DamonSet | URL                | Comments                                                                                                                                     |
|---------------------------|---------------------------------------------|--------------------------|-----------------------------------------|--------------------|----------------------------------------------------------------------------------------------------------------------------------------------|
| Home ETL Service          | ETL for bank statement and other sources    | ClusterIP (Headless)     | StatefulSet                             | /home/etl          | NA                                                                                                                                           |
| Home API Service          | API for Bank/Expnse/Tax/Invetsment/etc...   | ClusterIP                | Deployment                              | /home/api          | GraalVM based native Image                                                                                                                   |
| Home Dashboard            | ReactJS App on Nginx                        | NodePort                 | Deployment                              | http://jgte:30080  | - For multinode deployment Interface has to be changed to ClusterIP and put behind Ingress - externalTrafficPolicy: Local to disable SNATing |
| Home GIT Chronjob         | Cronjob to update GIT with uplaod statement | None                     | CronJob                                 | NA                 | NA                                                                                                                                           |
| Database                  | MySQL                                       | NodePort                 | StatefulSet                             |                    | - NodePort because I want to access SQL from outside of the cluster                                                                          |
| Kubernetes Dashboard      |                                             | LoadBalancer (static IP) | Deployment                              | https://jgte/      |                                                                                                                                              |
| Kubernetes Matrix         | Generating resource utilization matrix      | ClusterIP                | Deployment                              | NA                 |                                                                                                                                              |
| Kubernetes Matrix Scraper | Matrix scrapper from pods                   | ClusterIP                | Deployment                              | NA                 |                                                                                                                                              |
| Jaeger Dashboard          |                                             | NodePort                 | Deployment                              | http://jgte:31686/ |                                                                                                                                              |