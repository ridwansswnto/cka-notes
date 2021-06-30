# SOAL RBAC

## Soal 1 

Ada 3rd party application yang butuh akses untuk nge describe `ingress` object yang ada di dalem namespace `third-apps`. Buatkan seperti berikut:
* Bikin namespace `third-apps`
* Bikin service account dengan nama `third-reader` untuk namespace `third-apps`
* bikin role dengan nama `for-3threader` yang bisa untuk get dan list object
* bikin role binding dengan nama `user-3threader` untuk si service account dan role yang sudah di bikin
* pastiin si service account bisa get object yang dibutuhkan saja
