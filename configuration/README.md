# Configuration

<p align="center">
  <img width="500" height="350" src="https://github.com/ridwansswnto/cka-notes/blob/main/images/configuration.png">
</p>

Di kubernetes untuk config aplikasi dapat di lakukan dengan beberapa jeniis. Oke saat kita bikin container images, akan lebih baik dalam prakteknya nanti menjadi yang namanya reusable image. Artinya satu image bisa di gunakan beberapa environment stage.

## ConfigMap

<p align="center">
  <img width="500" height="350" src="https://github.com/ridwansswnto/cka-notes/blob/main/images/configmap.gif">
</p>

Kubernetes memungkinkan pemisahan opsi konfigurasi menjadi objek terpisah yang disebut ConfigMap, yang berisi key/value pairs, bisa pendek dan bisa juga membaca dari file.

### Bikin ConfigMap
Oke kali ini kita akan bikin configmap simple ya

```
$ kubectl create configmap contoh-config --from-literal=sleep-interval=25

$ kubectl get configmap      
NAME               DATA   AGE
contoh-config      1      10s
kube-root-ca.crt   1      10d

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

```

```