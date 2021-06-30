## Soal 1
<details><summary>Answer - Imperative</summary>

```shell
kubectl create namespace third-apps

kubectl create serviceaccount third-reader -n third-apps

kubectl create role for-3threader --verb=get,list --resource=ingresses -n third-apps

kubectl create rolebinding user-3threader --role=for-3threader --serviceaccount=third-apps:third-reader -n third-apps

kubectl create rolebinding user-3threader --role=for-3threader --serviceaccount=system:serviceaccount:third-apps:third-reader -n third-apps

kubectl auth can-i "get" ingress --as system:serviceaccount:third-apps:third-reader -n third-apps
=== yes

kubectl auth can-i "get" pods --as system:serviceaccount:third-apps:third-reader -n third-apps
=== no
```
</details>