# SOAL RBAC

## Soal 1 

Ada 3rd party application yang butuh akses untuk nge describe `ingress` object yang ada di dalem namespace `third-apps`. Buatkan seperti berikut:
* Bikin namespace `third-apps`
* Bikin service account dengan nama `third-reader` untuk namespace `third-apps`
* bikin role dengan nama `for-3threader` yang bisa untuk get dan list object
* bikin role binding dengan nama `user-3threader` untuk si service account dan role yang sudah di bikin
* pastiin si service account bisa get object yang dibutuhkan saja


## Soal 2

Ada security engineer yang pengen bisa liat apakah semua secret sudah sesuai dengan standard mereka, jadi security engineer ini bisa ngeliat secret di semua namespaces (kita anggap setiap aplikasi memiliki namespace sendiri)

* Bikin clusterrole dengan nama for-seceng-reader untuk bisa get, list, secret
* bikin clusterrolebinding dengan nama user-seceng-reader untuk group security
* impersonate salah satu user misal si wawan(orang security engineer )dengan group security


## Soal 3
Create a ClusterRole and ClusterRoleBinding so that user secret@cka.com can only access and manage secrets. Test it


## Soal 4

Create a ClusterRole and ClusterRoleBinding so that user deploy@cka.com can only deploy and manage pods named compute. Test it
