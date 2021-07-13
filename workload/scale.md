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


<details><summary>limit.yaml</summary>

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      name: nginx
  template:
    metadata:
      labels:
        name: nginx
    spec:
      containers:
        - name: nginx-service
          image: nginx
          ports:
            - containerPort: 30000
              name: rest
          resources:
            limits:
              cpu: 50m
            requests:
              cpu: 2m
---
apiVersion: v1
kind: Service
metadata:
  name: nginx
spec:
  selector:
    name: nginx
  ports:
    - port: 80
      nodePort: 30000
  type: NodePort
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx
spec:
  rules:
  - host: kubernetes.docker.internal
  - http:
      paths:
      - backend:
          service:
            name: nginx
            port:
              number: 80
        path: /
        pathType: Prefix
```
</details>

<details><summary>hpa.yaml</summary>

```
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: nginx
  namespace: default
spec:
  maxReplicas: 3
  minReplicas: 1
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx
  targetCPUUtilizationPercentage: 50
```