### Scaling in Kubernetes

## HPA

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
</details>

Kalau diperhatikan, di limit yaml itu kita bikin resource seperti deployment, service, dan juga ingress. Fokus ke deployment, di sana gua ngeset resource request dan juga limitnya. Dan fokus ke hpa nya, gua set untuk maximal replicaset yaitu 3 jika cpu usage/utilizenya lebih dari 50%.
langsung aja diapply untuk file `limit.yaml` dan juga `hpa.yaml`

Deep dive

```
$ kubectl get deployment      
NAME                READY   UP-TO-DATE   AVAILABLE   AGE
nginx               1/1     1            1           118m

$ kubectl describe deployments nginx
Name:                   nginx
Namespace:              default
....
Pod Template:
  Labels:  name=nginx
  Containers:
   nginx-service:
....
    Limits:
      cpu:  50m
    Requests:
      cpu:        2m
....

$ kubectl get hpa                             
NAME    REFERENCE          TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
nginx   Deployment/nginx   0%/50%    1         3         1          40m

$ kubectl describe hpa nginx
Name:                                                  nginx
Namespace:                                             default
Labels:                                                <none>
Annotations:                                           <none>
CreationTimestamp:                                     Tue, 13 Jul 2021 22:49:33 +0700
Reference:                                             Deployment/nginx
Metrics:                                               ( current / target )
  resource cpu on pods  (as a percentage of request):  0% (0) / 50%
Min replicas:                                          1
Max replicas:                                          3
Deployment pods:                                       1 current / 1 desired
Conditions:
  Type            Status  Reason            Message
  ----            ------  ------            -------
  AbleToScale     True    ReadyForNewScale  recommended size matches current size
  ScalingActive   True    ValidMetricFound  the HPA was able to successfully calculate a replica count from cpu resource utilization (percentage of request)
  ScalingLimited  True    TooFewReplicas    the desired replica count is less than the minimum replica count
Events:
  Type     Reason                        Age   From                       Message
  ----     ------                        ----  ----                       -------
```

sekarang kita pantau penggunaan resource dengan command `kubectl top pods` danjuga pantau `hpa` dengan `kubectl get hpa`
```
$ while true; do kubectl top pods nginx-6cd57464f4-mnhd7; sleep 3; done
NAME                     CPU(cores)   MEMORY(bytes)   
nginx-6cd57464f4-mnhd7   0m           5Mi  

$ while true; do kubectl get hpa; sleep 3; done
NAME    REFERENCE          TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
nginx   Deployment/nginx   0%/50%    1         3         1          45m
```

Bisa di liat bahwa penggunaan masih underutilize, alias masih aman, skrg akan kita coba untuk load testing, di sini gua pakai k6. Siapin script untuk load testnya

<details><summary>limit.yaml</summary>

```
import http from 'k6/http';
import { sleep } from 'k6';

export default function () {
    http.get('http://localhost');
    sleep(1);
}

```
</details>

execute ini
```
$ k6 run --vus 300 --duration 180s loadtest_pod.js
```

Nah sambil nunggu kita pantau untuk penggunaan resource di podnys, bisa di liat ningkat
```
nginx-6cd57464f4-mnhd7   0m           5Mi             
NAME                     CPU(cores)   MEMORY(bytes)   
nginx-6cd57464f4-mnhd7   28m          5Mi             
NAME                     CPU(cores)   MEMORY(bytes)   
nginx-6cd57464f4-mnhd7   28m          5Mi             
NAME                     CPU(cores)   MEMORY(bytes)   
nginx-6cd57464f4-mnhd7   28m          5Mi             
NAME                     CPU(cores)   MEMORY(bytes)   
nginx-6cd57464f4-mnhd7   28m          5Mi             
NAME                     CPU(cores)   MEMORY(bytes)   
nginx-6cd57464f4-mnhd7   28m          5Mi             
NAME                     CPU(cores)   MEMORY(bytes)   
nginx-6cd57464f4-mnhd7   48m          5Mi             
```

Usagenya di hpa juga sudah melebihi treshold yang kita set
```
NAME    REFERENCE          TARGETS     MINPODS   MAXPODS   REPLICAS   AGE
nginx   Deployment/nginx   2450%/50%   1         3         1          52m
```

dan juga liat eventnya dengan describe hpa
```
Events:
  Type     Reason                        Age                From                       Message
  ----     ------                        ----               ----                       -------
  Normal   SuccessfulRescale             96s (x2 over 50m)  horizontal-pod-autoscaler  New size: 3; reason: cpu resource utilization (percentage of request) above target
```

Dan deployment autoscale
```
$ kubectl get deployment
NAME                READY   UP-TO-DATE   AVAILABLE   AGE
nginx               3/3     3            3           130m
```

## VPA

Kita butuh install vpa dulu karena vpa ini custom resource. Stepnya seperti ini
```
$ git clone https://github.com/kubernetes/autoscaler.git
$ git checkout vertical-pod-autoscaler-0.8.0
$ cd vertical-pod-autoscaler/hack
$ ./vpa-up.sh
```