# Configuration

Di kubernetes untuk config aplikasi dapat di lakukan dengan beberapa jeniis. Oke saat kita bikin container images, akan lebih baik dalam prakteknya nanti menjadi yang namanya reusable image. Artinya satu image bisa di gunakan beberapa environment stage.

## ConfigMap

Kubernetes memungkinkan pemisahan opsi konfigurasi menjadi objek terpisah yang disebut ConfigMap, yang berisi key/value pairs, bisa pendek dan bisa juga membaca dari file.

### Bikin ConfigMap

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
```

```

```