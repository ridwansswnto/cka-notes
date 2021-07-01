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

## Soal 2

<details><summary>Answer - Imperative</summary>

```shell

kubectl create clusterrole for-seceng-reader --resource=secrets --verb=get,list
kubectl create clusterrolebinding user-seceng-reader --clusterrole=for-seceng-reader --group=security

kubectl get secrets --as wawan --as-group security
NAME                  TYPE                                  DATA   AGE
ckaexam-token-hd8zd   kubernetes.io/service-account-token   3      25h
default-token-fthlh   kubernetes.io/service-account-token   3      32h

kubectl get secrets --as wawan --as-group security --all-namespaces
NAMESPACE         NAME                                             TYPE                                  DATA   AGE
default           default-token-fthlh                              kubernetes.io/service-account-token   3      32h
docker            compose-etcd                                     Opaque                                4      32h
docker            compose-token-knm9w                              kubernetes.io/service-account-token   3      32h
docker            default-token-s2h9h                              kubernetes.io/service-account-token   3      32h
kube-node-lease   default-token-45rq7                              kubernetes.io/service-account-token   3      32h

kubectl get pod --as wawan --as-group security 
Error from server (Forbidden): pods is forbidden: User "wawan" cannot list resource "pods" in API group "" in the namespace "default"

kubectl get secrets --as wawan --as-group dev     
Error from server (Forbidden): secrets is forbidden: User "wawan" cannot list resource "secrets" in API group "" in the namespace "default"
```
</details>