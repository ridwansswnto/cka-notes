### baca ini dulu guys

Oke pertama2 kalau lu jalanin kubernetes yang dari docker-desktop, gua jamin lu bakalan pusing karena every serviceaccount yang lu buat gak bakal kena di role yang kalian bikin, karena apa? karena service account yang kalian bikin itu kebaca pake RBAC nya si clusterrolebinding `docker-for-desktop-binding`

nah meskipun lu udah kasih limitasi ke serviceaccount yang lu buat, mereka tetep bisa akses apapun di cluster, cara fixnya yaitu lu patch si crb `docker-for-desktop-binding` pakai ini ya

```
kubectl apply -f - <<EOF                                      
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: docker-for-desktop-binding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: Group
  name: system:serviceaccounts:kube-system
EOF
```

Behaviornya gini bro
```
Response Body: {"kind":"SelfSubjectAccessReview","apiVersion":"authorization.k8s.io/v1","metadata":{"creationTimestamp":null},"spec":{"resourceAttributes":{"namespace":"third-apps","verb":"get","resource":"pods"}},"status":{"allowed":true,"reason":"RBAC: allowed by ClusterRoleBinding \"docker-for-desktop-binding\" of ClusterRole \"cluster-admin\" to Group \"system:serviceaccounts\""}}
```

nanti bakalan berubah dari gini
```
Name:         docker-for-desktop-binding
Labels:       <none>
Annotations:  <none>
Role:
  Kind:  ClusterRole
  Name:  cluster-admin
Subjects:
  Kind   Name                    Namespace
  ----   ----                    ---------
  Group  system:serviceaccounts  kube-system
```

jadi gini
```
Name:         docker-for-desktop-binding
Labels:       <none>
Annotations:  <none>
Role:
  Kind:  ClusterRole
  Name:  cluster-admin
Subjects:
  Kind   Name                                Namespace
  ----   ----                                ---------
  Group  system:serviceaccounts:kube-system
```
