# 1. Local test
````
docker compose up --build
````
# 2. K8s — apply in this order:

````
kubectl apply -f k8s/base/namespace.yaml
kubectl apply -f k8s/base/configmap.yaml
kubectl apply -f k8s/base/secrets.yaml
kubectl apply -f k8s/storage/postgres-storage.yaml
kubectl apply -f k8s/services/postgres.yaml
kubectl apply -f k8s/services/auth-service.yaml
kubectl apply -f k8s/services/account-service.yaml
kubectl apply -f k8s/services/frontend.yaml
kubectl apply -f k8s/ingress/ingress.yaml
kubectl apply -f k8s/monitoring/hpa.yaml
````
