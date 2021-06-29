# Role Based Access Control (RBAC)
Pertama, gua bilang RBAC di kubernetes merupakan topic yang susah untuk di mengerti. So, that's why lu butuh waktu untuk fokus mempelajari ini. Lanjut ..

Sebelum ke RBAC, di kubernetes itu ada yang namanya [API ACCESS CONTROL](https://kubernetes.io/docs/reference/access-authn-authz/), ini di bagi menjadi beberapa stage seperti:
 * Authenticating
 * Authorization -> (RBAC)
 * Admission Controllers

![alttest](/images/rbac.png)
![alttest](/images/rbac1.png)

Nah kita lanjut pembahasan mengenai si RBAC ini, jadi ketika lu atau kita bahas mengenai RBAC akan ada beberapa istilah dan field yang akan sering lu denger yaitu:

fields:
* apiGroups - ini gua kurang yakin apaan tapi lu bisa liat2 dimari gan `kubectl api-resources -o wide`
* verbs - ini gua bisa bilang kayak permission, disini lu ngedefine nih si x bisa apa aja, apakah list, delete, create, watch dan DLL    
* resources - nah ini resource di k8s kayak pods, deployment, services dll apa aja yang bisa di akses
* subject - ini kayak lu nge define service account atau user apa yang mau di bikinin akses nya

dan ini istilah lain yang bakal lu sering denger:
* Roles dan ClusterRoles - which specify which verbs can be performed on which resources.
* RolesBinding dan ClusterRolesBinding - which bind the above roles to specific users, groups, or ServiceAccounts

Udah pusing? kalau pusing liat diagram dibawah

kayak gini mainnya:

![alttest](/images/rbac2.png)

Roles menentukan permission apa?, RolesBinding menentukan untuk siapa?

Satu lagi biar gampang

Roles and RoleBindings are namespaced; ClusterRoles and ClusterRoleBindings aren’t.

![alttest](/images/rbac3.png)

udah ada lah ya sedikit gambaran mengenai RBAC ini, yuk kita lanjut ke babak selanjutnya yaitu bermain-main, akhirnya main juga guys

untuk playground gua pakai kubernetes yang ada di docker desktop gak tau istilahnya apaan

## Bermain dengan Roles dan RolesBinding
### Skenario 1
Di namespaces service-a, user jongjong@klapans.com bisa list,get dan watch deployment yang ada, tapi gak bisa hapus

1. bikin namespace service-a dan simple deployment
```
kubectl create namespace service-a
```
```
kubectl create deployment hello-node --image=k8s.gcr.io/echoserver:1.4
```

2. bikin Roles verb list,get,watch deployment di ns service-a
```
kubectl create role for-reader --verb=get,list,watch --resource=deployment -n service-a
```

3. Bikin rolebinding untuk jongjong@klapans.com
```
kubectl create rolebinding user-reader --role=for-reader --user=jongjong@klapans.com -n service-a
```

4. run as a jongjong@klapans.com
```
k auth can-i "get" deployment --as jongjong@klapans.com -n service-a
k auth can-i "list" deployment --as jongjong@klapans.com -n service-a
k auth can-i "delete" deployment --as jongjong@klapans.com -n service-a
k auth can-i "get" pods --as jongjong@klapans.com -n service-a
```

### Skenario 2
nah kita udah bikin si user jongjong@klapans.com bisa nge list deployment, tapi ternyata dia butuh juga buat list pods nih, karena sebelumnya kita hanya assign permission list ke resource deployment doang, jadi lanjutt..

1. edit role for-reader, tambahin resource pods
```
kubectl edit role for-reader -n service-a
```

2. tambahin resources pods dengan bikin rule baru, lho kenapa rules baru ? karena kalau lu liat di apigroups itu deployment masuk ke extensions/apps, sedangkan pods itu gak
```
rules:
- apiGroups:
  - extensions
  resources:
  - deployments
-rules:
- apiGroups:
  - ''
  resources:
  - pods
  verbs:
  - get
  - list
  - watch
```

3. kalau mau check pakai describe atau get -o yaml

```
kubectl describe role for-reader -n service-a
Name:         for-reader
Labels:       <none>
Annotations:  <none>
PolicyRule:
  Resources               Non-Resource URLs  Resource Names  Verbs
  ---------               -----------------  --------------  -----
  pods                    []                 []              [get list watch]
  deployments.extensions  []                 []              [get list watch]
```

```
kubectl get role for-reader -n service-a -o yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  creationTimestamp: "2021-06-29T08:57:37Z"
  name: for-reader
  namespace: service-a
  resourceVersion: "21597"
  selfLink: /apis/rbac.authorization.k8s.io/v1/namespaces/service-a/roles/for-reader
  uid: 0d7ba82e-d8b8-11eb-a810-025000000001
rules:
- apiGroups:
  - extensions
  resources:
  - deployments
  verbs:
  - get
  - list
  - watch
- apiGroups:
  - ""
  resources:
  - pods
  verbs:
  - get
  - list
  - watch
```


4. run as is a jongjong@klapans.com
```
k auth can-i "list" pods --as jongjong@klapans.com -n service-a       
=== yes
```

### Skenario 3
ooh ternyata si jongjong ini udah jadi deployer (tukang deploy2), dia butuh akses buat ngeliat svc, ingress dll. Gass tambah yukk

1. kita as is is as dulu si jongjong bisa get svc atau ingress gak
```
k auth can-i "get" svc --as jongjong@klapans.com -n service-a
=== no

k auth can-i "get" ingress --as jongjong@klapans.com -n service-a
=== no
```

2. oke ternyata gak bisa, kita tambahin aja resources svc sama ingress ke roles for-reader, oiya pastiin lu tau yak ingress sama svc ini pake apiGroups apa
```
rules:
- apiGroups:
  - extensions
  resources:
  - deployments
  - ingresses
-rules:
- apiGroups:
  - ''
  resources:
  - pods
  - services
  verbs:
  - get
  - list
  - watch
```

3. oke biar ada pencerahan kita describe 
```
kubectl describe role for-reader -n service-a              
Name:         for-reader
Labels:       <none>
Annotations:  <none>
PolicyRule:
  Resources               Non-Resource URLs  Resource Names  Verbs
  ---------               -----------------  --------------  -----
  pods                    []                 []              [get list watch]
  services                []                 []              [get list watch]
  deployments.extensions  []                 []              [get list watch]
  ingresses.extensions      []                 []              [get list watch]
```

4. kita test permission si jongjong udah bisa get ke ingress dan svc belum
```
k auth can-i "get" svc --as jongjong@klapans.com -n service-a    
=== yes

k auth can-i "get" ingress --as jongjong@klapans.com -n service-a           
=== yes

```

## Bermain dengan ClusterRole dan ClusteRoleBinding
### Skenario 1
oke kalau sebelumnya si jongjong itu udah dapat permission get workload kayak pods, deployment, svc, dan ingress. Tapii, itu cuma di dalem namespace service-a doang nih. Nah yg kayak gua bilang tadi, si jongjong ini deployer yang bisa aja ngedeploy service-b, anggap aja iya. Gimana dong? apa iya gua harus bikinin di setiap namespace yang ada? lama cuy kayak gitu. 

Nah ini lah gunanya ClusterRole dan ClusterRoleBinding, dari nama nya aja udah cluster, fungsinya udah otomatis bisa cross namespace, gitu lah kira2 guys

1. si jongjong perlu liat semua service(aplikasi) yang ada di dlm cluster nih kita nih, jadi misal di cluster kita ada namespace service-a, service-b, service-c. Nah si jongjong perlu permision buat kesana juga. Jadi kita setup ClusterROle dulu. 

As admin
```
kubectl get pods --all-namespacesNAMESPACE     NAME                                     READY   STATUS    RESTARTS   AGE
xxx
xxx
service-a     hello-node-655d746f47-5s6tg              1/1     Running   0          3h49m
service-b     hello-node-655d746f47-4xllk              1/1     Running   0          10s
service-c     hello-node-655d746f47-bmwxq              1/1     Running   0          9s
```

As jongjong
```
k auth can-i "get" pods --as jongjong@klapans.com --all-namespaces        
===no
```

Setup jos 

```
kubectl create clusterrole for-deployer --verb=get,list,watch --resource=pods,services,ingresses,deployments

```

2. Kita liat clusterrole yang udah di buat dengan describe
```
kubectl describe clusterrole for-deployer                                                                   
Name:         for-deployer
Labels:       <none>
Annotations:  <none>
PolicyRule:
  Resources               Non-Resource URLs  Resource Names  Verbs
  ---------               -----------------  --------------  -----
  pods                    []                 []              [get list watch]
  services                []                 []              [get list watch]
  deployments.extensions  []                 []              [get list watch]
  ingresses.extensions    []                 []              [get list watch]
```

3. Kita bikin clusterrolebinding nya guyss
```
kubectl create clusterrolebinding user-deployer --clusterrole=for-deployer --user=jongjong@klapans.com
```

4. kita describe si crb nya
```
Name:         user-deployer
Labels:       <none>
Annotations:  <none>
Role:
  Kind:  ClusterRole
  Name:  for-deployer
Subjects:
  Kind  Name                  Namespace
  ----  ----                  ---------
  User  jongjong@klapans.com  
```

5. Test as is as jongjong guys
```
k auth can-i "get" pods --as jongjong@klapans.com --all-namespaces
=== yes
```

Kalau kalian bingung trus gua harus pake Roles atau ClusterRoles?

simplenya

If you want to define a role within a namespace, use a Role; if you want to define a role cluster-wide, use a ClusterRole

Nah kira-kira seperti itu guyss mengenai RBAC di kubernetes yakkk, semoga memberikan anda-anda sekalian yang membacanya


Referensi RBAC
* https://kubernetes.io/docs/reference/access-authn-authz/rbac/
* https://medium.com/@ishagirdhar/rbac-in-kubernetes-demystified-72424901fcb3
* https://dev.to/alcide/kubernetes-rbac-moving-from-it-s-complicated-to-in-a-relationship-1bbm
* https://blog.aquasec.com/kubernetes-rbac
* https://alibaba-cloud.medium.com/getting-started-with-kubernetes-access-control-a-security-measure-in-kubernetes-f1fc93e6020a
* https://theithollow.com/2019/05/20/kubernetes-role-based-access/
* https://searchitoperations.techtarget.com/tutorial/Be-selective-with-Kubernetes-RBAC-permissions
* https://medium.com/@HoussemDellai/rbac-with-kubernetes-in-minikube-4deed658ea7b

