# Transform

Transform adalah fitur yang dipakai untuk melakukan modifikasi pada waktu data mengalir dari sumber ke tujuan. Misalnya dari MySQL ke topik kafka. Transform punya tugas untuk melakukan transformasi pada pesan tunggal lewat jalur Connect.

Berikut adalah Fitur - fitur Transform yang sering digunakan:

. InsertField: Menambahkan field baru ke dalam rekaman.

. ReplaceField: Mengubah nama atau nilai dari sebuah field.

. MaskField: Mengaburkan informasi sensitif, seperti mengganti nomor kartu kredit dengan tanda bintang.

. ValueToKey: Mengubah payload pesan menjadi kunci rekaman Kafka.

. RegexRouter: Mengarahkan pesan ke topik yang berbeda berdasarkan pencocokan ekspresi reguler pada nama topik.

Contoh misalnya pada gambar saya mau hilangkin field age.

![image](https://github.com/user-attachments/assets/7e9d4d73-78f5-4938-83af-b81814e9b754)

Didalam case ini, saya akan menggunakan fitur ReplaceField, untuk drop atau menghapus field yang saya mau hilangkan.

Maka di konfigurasi Connect yang saya miliki saya menambahkan:
```
transforms=dropAge
transforms.dropAge.type=org.apache.kafka.connect.transforms.ReplaceField$Value
transforms.dropAge.blacklist=age
```

dropage = name transform.

transforms.dropAge.type = tipe transformation yang merubah value di dalam sebuah record.

transforms.dropAge.blacklist = mewakili field age yang saya mau drop.

Hasilnya dari 

name

id

hobby

age

nantinya akan menjadi

name 

id

hobby

setelah menggunakan transform, seperti pada gambar berikut ini

![image](https://github.com/user-attachments/assets/6f095120-e7f6-45a1-a0e4-e8a1512cf225)


