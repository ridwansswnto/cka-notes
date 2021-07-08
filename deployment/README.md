# DEPLOYEMENT in KUBERNETES

### Daftar Isi
<!--ts-->
   * [DEPLOYEMENT in KUBERNETES](#deployment-in-kubernetes)
   * [PODS](#pods)
   * [REPLICASET](#replicaset)
   * [DEPLOYMENT](#deployment)
<!--te-->

Oke kita belajar komponen dasar dulu yak di kubernetes, biar kita paham.

Di kubernetes itu ada istilah Pods, Replicaset, dan Deployment. Masing-masing istilah tersebut memiliki fungsi dan saling berkaitan. sebagai gambarannya seperti ini 

<p align="center">
  <img width="500" height="350" src="https://github.com/ridwansswnto/cka-notes/blob/main/images/deploy.png">
</p>

Nah istilah2 gambar diatas akan banyak kalian dengar ketika membahas deployment

apa itu deployment? bisa di bilang sebuah management untuk mengatur object seperti replica set dan pods. Kalau belum paham bisa liat dibawah ini untuk perbandingan antara pods, replicaset dan deployment

## PODS
Kubernetes tidak menjalan kan container secara langsung/direct, tapi melalui pods. didalam pods bisa terdapat lebih dari satu container dimana si container ini berbagi resouce seperti network dll tetapi tetap ada batasan untuk isolasi resource

kalau bingung kita praktek ke basic pods aja yaa

### Basic Pods
Seperti yang sebelumnya di bilang, dalam sebuah pods bisa memiliki lebih dari 1 container, tergantung kebutuhan dan pattern yang akan di terapkan.

Berikut ini contoh untuk satu pods

#### 
**Satu Pod**

---

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
# kubectl get pods              
> NAME     READY   STATUS    RESTARTS   AGE
> tomcat   1/1     Running   0          8m17s

# kubectl describe pod tomcat
```

####
**Dua Pod**

---

Nah untuk lebih dari satu pods teman-teman bisa menerapkan pattern yang sesuai dengan kebutuhan. Berikut gambar referensinya

<p align="center">
  <img width="640" height="480" src="https://github.com/ridwansswnto/cka-notes/blob/main/images/containerpattern.jpg">
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
# kubectl get pods              
> NAME     READY   STATUS    RESTARTS   AGE
> tomcat2   2/2     Running   0          4m23s

# kubectl describe pod tomcat2
```

#### 
**Basic Operation of Pods**

1. Cek log pod

Pertama jalanin command ini dulu
```
kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: pod-satu
spec:
  containers:
  - name: log
    image: busybox
    command: ["printenv"]
    args: ["HOSTNAME", "KUBERNETES_PORT"]
EOF
```

liat logs dengan cara command berikut
```
# kubectl logs pod-satu  
> pod-satu
> tcp://10.96.0.1:443
```

2. Exec to pods
Pertama jalanin ini dulu 
```
kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: pod-dua
spec:
  containers:
  - name: log
    image: busybox
    command: 
      - /bin/sh
      - "-c"
      - "sleep 5m"
EOF
```

lanjut untuk masuk ke pods dengan command
```
kubectl exec -it pod-dua sh
printenv 
KUBERNETES_SERVICE_PORT=443
KUBERNETES_PORT=tcp://10.96.0.1:443
HOSTNAME=pod-dua
```

### Pods lifecyle Fase
Ketika pods di bikin akan memiliki fase seperti gambar dibawah ini

<p align="center">
  <img width="460" height="350" src="https://github.com/ridwansswnto/cka-notes/blob/main/images/pod2.png">
</p>

Berikut penjelasan detail fase-fasenya:
* Pending = The Pod has been accepted by the Kubernetes system, but one or more of the container images has not been created.
* Running = At least one container is still running, or is in the process of starting or restarting.
* Succeeded = All containers in the Pod terminated successfully.
* Failed = Containers in the Pod terminated, as least one failed with an error.
* Unknown = The state of Pod could not be obtained.


Referensi sidecar pattern:
* https://medium.com/bb-tutorials-and-thoughts/kubernetes-learn-sidecar-container-pattern-6d8c21f873d

## REPLICASET

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
# kubectl get pods        
> NAME                READY   STATUS    RESTARTS   AGE
> nginx-proxy-8jrf9   1/1     Running   0          51s
> nginx-proxy-c5s58   1/1     Running   0          51s
> nginx-proxy-jwmsw   1/1     Running   0          51s
```

nah yang menarik disini, ada sebuah controller untuk memastikan bahwa object / pods yang running tetap sesuai atau misal disni harus ada 3, jadi kalau ada pods yang failed atau mati, replicaset akan create pods baru

```
# kubectl get replicaset      
> NAME          DESIRED   CURRENT   READY   AGE
> nginx-proxy   3         3         3       3m13s
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
```
</details>

Oke karena kita sudah ada istilah nya abstraksi / management pods, kita bisa ubah scale nya misal dari 3 menjadi 2 seperti berikut

```
# kubectl scale --replicas 2 replicaset nginx-proxy
> replicaset.apps/nginx-proxy scaled
```

maka replica podsnya  akan berubah menjadi 2
```
NAME                READY   STATUS    RESTARTS   AGE
nginx-proxy-c5s58   1/1     Running   0          9m40s
nginx-proxy-jwmsw   1/1     Running   0          9m40s
```

Referensi Replicaset:
* https://congdonglinux.com/how-to-create-a-replicaset-in-kubernetes/
* https://theithollow.com/2019/01/28/kubernetes-replica-sets/

## DEPLOYMENT

<p align="center">
  <img width="460" height="300" src="https://github.com/ridwansswnto/cka-notes/blob/main/images/deploy2.png">
</p>

Deployment, hmm topic yang menaric nic. bisa di liat dengan analogi gambar diatas. Deployment sudah mencakup untuk object seperti replicaset dan pods.

<details><summary>deployment.yaml</summary>

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: contoh-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: contoh
  template:
    metadata:
      labels:
        app: contoh
    spec:
      containers:
      - name: contoh-testing-container
        image: debian:buster-slim
        command: ["bash", "-c", "while true; do echo \"Hello\"; echo \"EXAMPLE_ENV: $EXAMPLE_ENV\"; sleep 5; done"]
        env:
        - name: EXAMPLE_ENV
          value: Hello Bro
```
</details>

kita liat semua object dengan command `kubectl get all`
```
NAME                                     READY   STATUS    RESTARTS   AGE
pod/contoh-deployment-55954f5855-st6fx   1/1     Running   0          6m24s

NAME                                READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/contoh-deployment   1/1     1            1           6m25s

NAME                                           DESIRED   CURRENT   READY   AGE
replicaset.apps/contoh-deployment-55954f5855   1         1         1       6m24s
```

> Scaling a Deployment

nah kali ini semisal kita butuh menambahkan jumlah pods kita bisa naikin replica seperti ini 

```
# kubectl scale --replicas 5 deployment contoh-deployment
> deployment.apps/contoh-deployment scaled
```

nah kita bisa pantau rolling updatenya dengan command `kubectl rollout status deployment contoh-deployment`
```
Waiting for deployment "contoh-deployment" rollout to finish: 1 of 5 updated replicas are available...
Waiting for deployment "contoh-deployment" rollout to finish: 2 of 5 updated replicas are available...
Waiting for deployment "contoh-deployment" rollout to finish: 3 of 5 updated replicas are available...
Waiting for deployment "contoh-deployment" rollout to finish: 4 of 5 updated replicas are available...
deployment "contoh-deployment" successfully rolled out
```

Nah misalkan gua pengen update image dari `debian:busterslim` jadi `debian:bullseyeslim` yaitu dengan cara command seperti dibawah
```
# kubectl set image deployment contoh-deployment contoh-testing-container=debian:bullseye-slim --record

Waiting for deployment "contoh-deployment" rollout to finish: 1 old replicas are pending termination...
Waiting for deployment "contoh-deployment" rollout to finish: 1 old replicas are pending termination...

# kubectl get pods contoh-deployment-67ccbff8df-9pfbg -o yaml | grep image
> image: debian:bullseye-slim
```



Referensi baca-baca deployment:
* https://kubernetes.io/docs/concepts/workloads/controllers/deployment/
* https://www.cloudops.com/blog/kubernetes-deployments-101/

Referensi baca-baca:
* https://www.ibm.com/cloud/architecture/content/course/kubernetes-101/deployments-replica-sets-and-pods/
* https://blog.macstadium.com/blog/how-to-k8s-pods-replicasets-and-deployments
* https://www.cloudsavvyit.com/10107/pods-deployments-and-replica-sets-kubernetes-resources-explained/
* https://www.bmc.com/blogs/kubernetes-deployment/
* https://medium.com/google-cloud/kubernetes-101-pods-nodes-containers-and-clusters-c1509e409e16
* https://www.infoq.com/articles/kubernetes-effect/


ReplicaSet:
* https://www.magalix.com/blog/kubernetes-replicaset-101