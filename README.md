# home-stack
Home Project Stack

## Services and their ports
1. Statement Parser - 8081
2. Dashboard (Nginx) - 80

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
kubectl exec -it deployment.apps/mysql --namespace home-stack -- mysql -u root -p home-stack
````
````
kubectl logs deployment.apps/mysql --namespace home-stack
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
kubectl exec -it deployment.apps/stmtparser-deployment --namespace home-stack -- bash
````
````
kubectl exec -it deployment.apps/stmtparser-deployment --namespace home-stack -- tail -f /opt/logs/spring-batch.log
````
````
kubectl logs deployment.apps/stmtparser-deployment --namespace home-stack
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
### Delete Stack
````
kubectl delete namespace home-stack 
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
