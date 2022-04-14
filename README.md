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
kubectl exec -it pod/mysql-0 --namespace home-stack -- mysql -u root -p home-stack
````
````
kubectl logs pod/mysql-0 --namespace home-stack
````
````
mysql -u root -p home-stack --host 127.0.0.1 --port 32306
````
Follow the link to configure sqldeveloper in Mac to connect to remote MySQL server - https://cybercafe.dev/setup-mysql-and-sql-developer-on-macos/
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
### Delete Stack
````
kubectl delete namespace home-stack 
````
#### Kubernetes Dashboard - Pod/Deployment/Service
````
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.5.0/aio/deploy/recommended.yaml
````
Change service type to LoadBalancer to access the dashboard externally
````
kubectl edit svc -n kubernetes-dashboard kubernetes-dashboard

	spec:
	  type: LoadBalancer	# this has to be changed to LoadBalancer to access the dashboard externally 
````
Hard code the Node IPs - as below
````
kubectl edit svc -n kubernetes-dashboard kubernetes-dashboard

	spec:
	  type: LoadBalancer	# this has to be changed to LoadBalancer to access the dashboard externally 
	  externalIPs:
	  - 192.168.0.200		# this is needed because public IP cant be assigned automaic
````
````
kubectl delete -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.5.0/aio/deploy/recommended.yaml
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
url: https://jgte (try in Mozilla)
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
