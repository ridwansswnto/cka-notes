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

Referensi baca-baca:
* https://www.ibm.com/cloud/architecture/content/course/kubernetes-101/deployments-replica-sets-and-pods/
* https://blog.macstadium.com/blog/how-to-k8s-pods-replicasets-and-deployments
* https://www.cloudsavvyit.com/10107/pods-deployments-and-replica-sets-kubernetes-resources-explained/
* https://www.bmc.com/blogs/kubernetes-deployment/
* https://medium.com/google-cloud/kubernetes-101-pods-nodes-containers-and-clusters-c1509e409e16


ReplicaSet:
* https://www.magalix.com/blog/kubernetes-replicaset-101