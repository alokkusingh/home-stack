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

## Services 

| Application | Description | Interface Type | Deployment or StatefulSet | URL | Comments |
| --- | --- | --- | --- | --- | --- |
| Stmt Parser | Springboot Service| ClusterIP (Headless) | StatefulSet | | NA |
| Home Dashboard | ReactJS App on Nginx| NodePort | Deployment | http://jgte:30080 | - For multinode deployment Interface has to be changed to ClusterIP and put behind Ingress |
| Database | MySQL | NodePort | StatefulSet | | - NodePort because I want to access SQL from outside of the cluster |
| Kubernetes Dashboard |  | LoadBalancer (static IP) | Deployment | https://jgte/ | |
| Kubernetes Matrix | Generating resource utilization matrix | ClusterIP | Deployment | NA | |
| Kubernetes Matrix Scraper | Matrix scrapper from pods | ClusterIP | Deployment | NA | |
| Jaeger Dashboard |  | NodePort | Deployment | http://jgte:31686/ | |

### Deployment of home-stack Kubernetes Stack
#### Create Namespaces
````
kubectl apply -f yaml/namespace.yaml
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
kubectl apply --validate=true --dry-run=client -f yaml/jaeger-all-in-one-template.yaml 
````
````
kubectl apply -f yaml/jaeger-all-in-one-template.yaml  --namespace=home-stack
````
````
kubectl delete -f yaml/jaeger-all-in-one-template.yaml  --namespace=home-stack
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
## Service Mesh - Istio
### Install

---
>To be explored - seems microk8s isteo addon not supported for ARMx64 architecture. Where the same is supported for minikube.
---