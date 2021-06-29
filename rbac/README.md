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
