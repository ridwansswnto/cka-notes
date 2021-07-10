# Configuration

Di kubernetes untuk config aplikasi dapat di lakukan dengan beberapa jeniis. Oke saat kita bikin container images, akan lebih baik dalam prakteknya nanti menjadi yang namanya reusable image. Artinya satu image bisa di gunakan beberapa environment stage.

## ConfigMap

Kubernetes memungkinkan pemisahan opsi konfigurasi menjadi objek terpisah yang disebut ConfigMap, yang berisi key/value pairs, bisa pendek dan bisa juga membaca dari file.

### Bikin ConfigMap

```
kubectl create configmap fortune-config --from-literal=sleep-interval=25
```