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
#### Dashboard Service - Pod/Deployment/Service
````
kubectl apply --validate=true --dry-run=client --filename=https://github.com/alokkusingh/home-stack/blob/main/yaml/dashboard-service.yaml 
````
````
kubectl apply -f https://github.com/alokkusingh/home-stack/blob/main/yaml/dashboard-service.yaml  --namespace=home-stack
````
````
kubectl delete -f https://github.com/alokkusingh/home-stack/blob/main/yaml/dashboard-service.yaml  --namespace=home-stack
````
### Delete Stack
````
kubectl delete namespace home-stack 
````