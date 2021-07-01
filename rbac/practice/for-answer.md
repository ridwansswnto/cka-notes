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

## Soal 3

<details><summary>Answer - Imperative</summary>

```
k create clusterrole admin-secret --verb="*" --resource secret
clusterrole.rbac.authorization.k8s.io/admin-secret created

k create clusterrolebinding admin-secret --clusterrole=admin-secret --user=secret@cka.com
clusterrolebinding.rbac.authorization.k8s.io/admin-secret created

TEST IT
k auth can-i create secret --as secret@cka.com
yes

k auth can-i "*" secret --as secret@cka.com
yes

k auth can-i get pod --as secret@cka.com
no
```
</details>

## Soal 4

<details><summary>Answer - Imperative</summary>

```
k create clusterrole admin-deploy --verb="*" --resource=pod --resource-name="compute"
clusterrole.rbac.authorization.k8s.io/admin-deploy created

kubectl describe clusterrole admin-deploy
Name:         admin-deploy
Labels:       <none>
Annotations:  <none>
PolicyRule:
  Resources  Non-Resource URLs  Resource Names  Verbs
  ---------  -----------------  --------------  -----
  pods       []                 [compute]       [*]

k create clusterrolebinding admin-deploy --clusterrole=admin-deploy --user=deploy@cka.com
clusterrolebinding.rbac.authorization.k8s.io/admin-deploy created

k describe clusterrolebinding admin-deploy
Name:         admin-deploy
Labels:       <none>
Annotations:  <none>
Role:
  Kind:  ClusterRole
  Name:  admin-deploy
Subjects:
  Kind  Name             Namespace
  ----  ----             ---------
  User  deploy@cka.com

TEST IT
k auth can-i "*" pod/compute --as deploy@cka.com
yes

k auth can-i "*" pod/computed --as deploy@cka.com
no
```
</details>
