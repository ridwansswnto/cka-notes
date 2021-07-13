### Scaling in Kubernetes

Kali ini gua akan bahas mengenai HPA, jadi di kubernetes untuk scaling kita bisa manual dengan menggunakan replicaset, dan juga ada yang autoscale yaitu salah satunya hpa.

Nah untuk detail prakteknya temen2 prepare dulu manifest-manifest nya sebagai berikut

Enable nginx ingress
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.41.2/deploy/static/provider/cloud/deploy.yaml
```

Enable metric-server, copy paste file `components.yaml`
```
kubectl apply -f components.yaml
```


<details><summary>components.yaml</summary>
