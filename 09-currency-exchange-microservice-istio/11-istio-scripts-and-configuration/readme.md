# Setting up istio

```
gcloud container clusters get-credentials standard-cluster-1 --zone=europe-west1-b

https://istio.io/latest/docs/setup/getting-started/

kubectl label namespace default istio-injection=enabled    ::::::: This ensures that istio sideCar is injected in every pod
```

# Using Kiali

- http://localhost:20001

```
kubectl port-forward \
    $(kubectl get pod -n istio-system -l app=kiali \
    -o jsonpath='{.items[0].metadata.name}') \
    -n istio-system 20001
```
Create a secret

```
kubectl get secret -n istio-system kiali
kubectl create secret generic kiali -n istio-system --from-literal=username=admin --from-literal=passphrase=admin
```

# Using Graphana to see prometheus metrics
- http://localhost:3000

```
kubectl -n istio-system port-forward \
    $(kubectl -n istio-system get pod -l app=grafana \
    -o jsonpath='{.items[0].metadata.name}') 3000
```

# Using Jaeger

http://localhost:16686

```
kubectl port-forward -n istio-system \
    $(kubectl get pod -n istio-system -l app=jaeger \
    -o jsonpath='{.items[0].metadata.name}') 16686
```