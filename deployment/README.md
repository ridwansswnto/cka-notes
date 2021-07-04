# DEPLOYEMENT in KUBERNETES

Oke kita belajar komponen dasar dulu yak di kubernetes, biar kita paham.

Di kubernetes itu ada istilah Pods, Replicaset, dan Deployment. Masing-masing istilah tersebut memiliki fungsi dan saling berkaitan. sebagai gambarannya seperti ini 

<p align="center">
  <img width="460" height="300" src="https://github.com/ridwansswnto/cka-notes/blob/main/images/deploy.png">
</p>

Nah istilah2 gambar diatas akan banyak kalian dengar ketika membahas deployment

apa itu deployment? bisa di bilang sebuah management untuk mengatur object seperti replica set dan pods. Kalau belum paham bisa liat dibawah ini untuk perbandingan antara pods, replicaset dan deployment

### PODS
Kubernetes tidak menjalan kan container secara langsung/direct, tapi melalui pods. didalam pods bisa terdapat lebih dari satu container dimana si container ini berbagi resouce seperti network dll tetapi tetap ada batasan untuk isolasi resource

kalau bingung bisa liat di config2 berikut

> Satu Pod
<details><summary>tomcat.yaml</summary>

```
kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: tomcat
spec:
  containers:
  - name: tomcat
    image: tomcat:8.0
    ports:
      - name: port
        containerPort: 7500
    imagePullPolicy: Always
EOF
```
</details>


kita cek containernya
```
kubectl get pods              
NAME     READY   STATUS    RESTARTS   AGE
tomcat   1/1     Running   0          8m17s

kubectl describe pod tomcat
```

> Dua PODS

Ini cocok seperti container dengan sidecar pattern

<p align="center">
  <img width="460" height="300" src="https://github.com/ridwansswnto/cka-notes/blob/main/images/podsidecar.png">
</p>

<details><summary>tomcat.yaml</summary>

```
kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
   name: tomcat2
spec:
  containers:
  - name: tomcat2
    image: tomcat:8.0
    ports:
    - name: tomcat
      containerPort: 7500
    imagePullPolicy: Always
  - name: database
    image: mongo
    ports:
    - name: db
      containerPort: 7501
    imagePullPolicy: Always
EOF
```
</details>

kita cek containernya
```
kubectl get pods              
NAME     READY   STATUS    RESTARTS   AGE
tomcat2   2/2     Running   0          4m23s

kubectl describe pod tomcat2
```

Referensi sidecar pattern:
* https://medium.com/bb-tutorials-and-thoughts/kubernetes-learn-sidecar-container-pattern-6d8c21f873d

### REPLICASET

<p align="center">
  <img width="460" height="300" src="https://github.com/ridwansswnto/cka-notes/blob/main/images/replicaset.png">
</p>


siapkan manifest atau file yaml kayak gini
<details><summary>replicaset.yaml</summary>

```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-proxy
  labels:
    app: nginx-proxy
    tier: frontend
spec:
  # modify replicas according to your case
  replicas: 3
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: nginx
        image: nginx
```
</details>

jalankan dengan command `kubectl apply -f rs.yaml` maka kubernetes akan bikin object replicaset seperti ini `replicaset.apps/nginx-proxy created`

kalau kita cek dengan get pods akan ada 3 pod dengan nama nginx-proxy 
```
kubectl get pods        
NAME                READY   STATUS    RESTARTS   AGE
nginx-proxy-8jrf9   1/1     Running   0          51s
nginx-proxy-c5s58   1/1     Running   0          51s
nginx-proxy-jwmsw   1/1     Running   0          51s
```

nah yang menarik disini, ada sebuah controller untuk memastikan bahwa object / pods yang running tetap sesuai atau misal disni harus ada 3, jadi kalau ada pods yang failed atau mati, replicaset akan create pods baru

```
kubectl get replicaset      
NAME          DESIRED   CURRENT   READY   AGE
nginx-proxy   3         3         3       3m13s
```

nah kalian bisa ngecek ke dalem dengan `descirbe` atau `-o yaml`

<details><summary>describe rs nginx-proxy</summary>

```
kubectl describe rs nginx-proxy                                                    
Name:         nginx-proxy
Namespace:    default
Selector:     tier=frontend
Labels:       app=nginx-proxy
              tier=frontend
Annotations:  <none>
Replicas:     3 current / 3 desired
Pods Status:  3 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  tier=frontend
  Containers:
   nginx:
    Image:        nginx
    Port:         <none>
    Host Port:    <none>
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Events:
  Type    Reason            Age    From                   Message
  ----    ------            ----   ----                   -------
  Normal  SuccessfulCreate  4m58s  replicaset-controller  Created pod: nginx-proxy-8jrf9
  Normal  SuccessfulCreate  4m58s  replicaset-controller  Created pod: nginx-proxy-jwmsw
  Normal  SuccessfulCreate  4m58s  replicaset-controller  Created pod: nginx-proxy-c5s58
</details>
```


Oke karena kita sudah ada istilah nya abstraksi / management pods, kita bisa ubah scale nya misal dari 3 menjadi 2 seperti berikut


```
kubectl scale --replicas 2 replicaset nginx-proxy
replicaset.apps/nginx-proxy scaled
```

maka replica podsnya  akan berubah menjadi 2
```
NAME                READY   STATUS    RESTARTS   AGE
nginx-proxy-c5s58   1/1     Running   0          9m40s
nginx-proxy-jwmsw   1/1     Running   0          9m40s
```

Referensi Replicaset:
* https://congdonglinux.com/how-to-create-a-replicaset-in-kubernetes/

Referensi baca-baca:
* https://www.ibm.com/cloud/architecture/content/course/kubernetes-101/deployments-replica-sets-and-pods/
* https://blog.macstadium.com/blog/how-to-k8s-pods-replicasets-and-deployments
* https://www.cloudsavvyit.com/10107/pods-deployments-and-replica-sets-kubernetes-resources-explained/
* https://www.bmc.com/blogs/kubernetes-deployment/
* https://medium.com/google-cloud/kubernetes-101-pods-nodes-containers-and-clusters-c1509e409e16


ReplicaSet:
* https://www.magalix.com/blog/kubernetes-replicaset-101