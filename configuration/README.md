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

### Bikin ConfigMap
Oke kali ini kita akan bikin configmap simple ya

```
$ kubectl create configmap contoh-config --from-literal=sleep-interval=25

$ kubectl get configmap      
NAME               DATA   AGE
contoh-config      1      10s

$ kubectl describe configmaps contoh-config Name:         contoh-config
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