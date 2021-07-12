# Configuration

<p align="center">
  <img width="700" height="320" src="https://github.com/ridwansswnto/cka-notes/blob/main/images/configuration.png">
</p>

Di kubernetes untuk config aplikasi dapat di lakukan dengan beberapa jeniis. Oke saat kita bikin container images, akan lebih baik dalam prakteknya nanti menjadi yang namanya reusable image. Artinya satu image bisa di gunakan beberapa environment stage.

* Config Maps: a set of values that can be mapped to a pod as “volume” (to access all config map items as files) or passed as environment variables.
* Secrets: similar to config maps, secrets can be mounted into a pod as a volume to expose needed information or can be injected as environment variables. Secrets are intended to store credentials to other services that a container might need or to store any sensitive information.

## ConfigMap

<p align="center">
  <img width="500" height="350" src="https://github.com/ridwansswnto/cka-notes/blob/main/images/configmap2.png">
</p>

Kubernetes memungkinkan pemisahan opsi konfigurasi menjadi objek terpisah yang disebut ConfigMap, yang berisi key/value pairs, dan bisa membaca dari file.

### Basic ConfigMap

#### 
**Simple ConfigMap**

Oke kali ini kita akan bikin configmap simple ya

```
$ kubectl create configmap contoh-config --from-literal=sleep-interval=25

$ kubectl get configmap      
NAME               DATA   AGE
contoh-config      1      10s

$ kubectl describe configmaps contoh-config 
Name:         contoh-config
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
sleep-interval:
----
25
Events:  <none>

$ kubectl get configmaps contoh-config -o yaml
apiVersion: v1
data:
  sleep-interval: "25"
kind: ConfigMap
metadata:
  creationTimestamp: "2021-07-10T16:54:47Z"
  managedFields:
  - apiVersion: v1
    fieldsType: FieldsV1
    fieldsV1:
      f:data:
        .: {}
        f:sleep-interval: {}
    manager: kubectl-create
    operation: Update
    time: "2021-07-10T16:54:47Z"
  name: contoh-config
  namespace: default
  resourceVersion: "257622248"
  selfLink: /api/v1/namespaces/default/configmaps/contoh-config
  uid: aacb418b-5258-4929-aa0d-1e7bbd87c63d
```

Bisa di liat dari gambar diatas bahwa kita bikin config dengan key/value pairs yaitu sleep-interval=25

#### 
**Multiple K/V ConfigMap**

Bikin configMap dengan banyak kv seperti berikut, perhatikan saat go to detailsnya
```
kubectl create configmap app-config \
--from-literal=application.port=80 \
--from-literal=application.path=/apps \
--from-literal=application.database=localhost \
--from-literal=application.database.port=3017
```

Kita liat detail nya
```
kubectl get configmaps app-config           
NAME         DATA   AGE
app-config   4      7s

kubectl describe configmaps app-config   
Name:         app-config
....

Data
====
application.database.port:
----
3017
application.path:
----
/apps
application.port:
----
80
application.database:
----
localhost
....

kubectl get configmaps app-config -o json 
{
    "apiVersion": "v1",
    "data": {
        "application.database": "localhost",
        "application.database.port": "3017",
        "application.path": "/apps",
        "application.port": "80"
    },
    "kind": "ConfigMap",
    "metadata": {
        ....
        "name": "app-config",
        ....
    }
}
```

Oke configmap bisa juga membaca dari file seperti berikut ini. Bikin dulu filenya dengan nama application.properties
<details><summary>application.properties</summary>

```
application.name=haiho
application.port=80
application.path=/apps
application.database=localhost
application.database.port=3017
```
</details>

Pastiin workdir kalian sudah sama dengan si application.properties seperti berikut ini
```
$ ls
application.properties

$ kubectl create configmap app-properties --from-file=application.properties
configmap/app-properties created
```

Lets deep dive
```
kubectl describe configmaps app-properties                                
Name:         app-properties
....

Data
====
application.properties:
----
application.name=haiho
application.port=80
application.path=/apps
application.database=localhost
application.database.port=3017
....

{
    "apiVersion": "v1",
    "data": {
        "application.properties": "application.name=haiho\napplication.port=80\napplication.path=/apps\napplication.database=localhost\napplication.database.port=3017"
    },
    "kind": "ConfigMap",
    "metadata": {
        ....
        "name": "app-properties",
        ....
    }
}
```

Hmm kalau teman-teman memperhatikan baik-baik, ada perbedaan yaitu saat kita menggunakan literal, kita bikin seperti key/value pairs, nah kalau di atas kita bikin seperti config file biasa. Jika teman-teman menggunakan tools seperti lens akan terlihat seperti ini

<p align="center">
  <img width="300" height="200" src="https://github.com/ridwansswnto/cka-notes/blob/main/images/configmap3.png">
</p>

<p align="center">
  <img width="300" height="200" src="https://github.com/ridwansswnto/cka-notes/blob/main/images/configmap4.png">
</p>

### Configmap As a ENV
Masih menggunakan Configmap `app-config`, sekarang akan kita pair dengan pods kita seperti berikut

<details><summary>config-pod.yaml</summary>

```
apiVersion: v1
kind: Pod
metadata:
  name: configured-pod
spec:
  containers:
  - image: nginx:1.19.0
    name: app
    envFrom:
    - configMapRef:
        name: app-config
```
</details>

Kita check2 ke pods
```
$ kubectl describe pods configured-pod  
Name:         configured-pod
....
    Environment Variables from:
      app-config  ConfigMap  Optional: false
....

$ kubectl exec -it configured-pod -- env            
....
application.database.port=3017
application.path=/apps
application.port=80
application.database=localhost
....
```

Reassigning environment variable keys for ConfigMap entries

```
apiVersion: v1
kind: Pod
metadata:
  name: configured-pod
spec:
  containers:
  - image: nginx:1.19.0
    name: app
    env:
    - name: DATABASE_URL
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: application.database
    - name: PORT
      valueFrom:
        configMapKeyRef:
          name: backend-config
          key: application.port
```

### Mounting a ConfigMap as Volume
Pertama bikin manifest dulu untuk podsnya seperti ini
<details><summary>config-pod2.yaml</summary>

```
apiVersion: v1
kind: Pod
metadata:
  name: configured-pod-2
spec:
  containers:
  - image: nginx:1.19.0
    name: app
    volumeMounts:
    - name: properties-volume
      mountPath: /config
  volumes:
  - name: properties-volume
    configMap:
      name: app-properties
```
</details>

Lanjut kita buat podsnya
```
$ kubectl create -f config-pod2.yaml
pod/configured-pod-2 created
```
Lets deep dive
```
$ kubectl describe pods configured-pod-2
Name:         configured-pod-2
....
    Mounts:
      /config from properties-volume (rw)
....
Volumes:
  properties-volume:
    Type:      ConfigMap (a volume populated by a ConfigMap)
    Name:      app-properties
    Optional:  false
....

$ k exec -it configured-pod-2 sh
# ls config
application.properties
# cat config/application.properties
application.name=haiho
application.port=80
application.path=/apps
application.database=localhost
application.database.port=3017

```

## Secret

<p align="center">
  <img width="500" height="350" src="https://github.com/ridwansswnto/cka-notes/blob/main/images/secret.png">
</p>

simple secret with
```
kubectl create secret generic db-cred --from-literal=passwd=s3cre!
```


yuk liat detail
```
$ kubectl get secret                                                    
NAME                  TYPE                                  DATA   AGE
db-cred               Opaque                                1      33m

$ kubectl get secret db-cred -o json                                    
{
    "apiVersion": "v1",
    "data": {
        "passwd": "czNjcmUh"
    },
    "kind": "Secret",
    ....
}
```

Nah itu di decode ya, cara nya gini
```
$ kubectl get secret db-cred -o json | jq '.data | map_values(@base64d)'                             
{
  "passwd": "s3cre!"
}
```

### Dari file

```
db.username=test
db.password=testingaja
mq.username=test
mq.password=testingaja
```

kalau sudah bikin secretnya
```
$ kubectl create secret generic app-secret --from-file=application.secret
secret/app-secret created
```

deep dive
```
$ kubectl get secret
NAME                  TYPE                                  DATA   AGE
app-secret            Opaque                                1      5s

$ kubectl describe secrets app-secret                                      
Name:         app-secret
Namespace:    default
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data
====
application.secret:  79 bytes

$ kubectl get secret app-secret -o yaml                                    
apiVersion: v1
data:
  application.secret: ZGIudXNlcm5hbWU9dGVzdApkYi5wYXNzd29yZD10ZXN0aW5nYWphCm1xLnVzZXJuYW1lPXRlc3QKbXEucGFzc3dvcmQ9dGVzdGluZ2FqYQ==
kind: Secret

$ kubectl get secret app-secret -o json | jq '.data | map_values(@base64d)'
{
  "application.secret": "db.username=test\ndb.password=testingaja\nmq.username=test\nmq.password=testingaja"
}
```
### As Env Vars
```
kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: configured-pod
spec:
  containers:
  - image: nginx:1.19.0
    name: app
    envFrom:
    - secretRef:
        name: db-cred
EOF

$ kubectl exec -it configured-pod -- env
....
passwd=s3cre!
....
```

### As a Mounting Volume
```
kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: configured-pod-2
spec:
  containers:
  - image: nginx:1.19.0
    name: app
    volumeMounts:
    - name: secret-volume
      mountPath: /var/app
      readOnly: true
  volumes:
  - name: secret-volume
    secret:
      secretName: app-secret
EOF
```

Deep dive
```
$ kubectl get pods                                                         
NAME                                 READY   STATUS    RESTARTS   AGE
configured-pod-2                     1/1     Running   0          48s

$ kubectl describe pods configured-pod-2           
Name:         configured-pod-2
....
Containers:
....
    Mounts:
      /var/app from secret-volume (ro)
....
Volumes:
  secret-volume:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  app-secret
    Optional:    false

$ kubectl exec -it configured-pod-2 -- ls /var/app/
application.secret

$ kubectl exec -it configured-pod-2 -- cat /var/app/application.secret
db.username=test
db.password=testingaja
mq.username=test
mq.password=testingaja
```