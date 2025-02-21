1. Apa Itu IP?
   
IP (Internet Protocol) adalah protokol komunikasi dalam jaringan komputer yang digunakan untuk mengidentifikasi dan menghubungkan perangkat dalam suatu jaringan, termasuk internet. IP berfungsi sebagai alamat unik yang diberikan kepada setiap perangkat untuk memungkinkan pengiriman dan penerimaan data.

Konsep:

IP digunakan dalam jaringan untuk mengidentifikasi perangkat seperti komputer, server, router, atau perangkat IoT.
Setiap perangkat memiliki alamat IP unik dalam suatu jaringan.

IP memungkinkan komunikasi antar perangkat dalam jaringan lokal maupun global (internet).

Contoh & Use Case:

Contoh: Laptop yang terhubung ke Wi-Fi memiliki alamat IP yang diberikan oleh router.

Use Case: Saat mengakses situs web, komputer akan mengirimkan permintaan ke server dengan alamat IP tujuan.

2. Apa Itu IPv4 dan IPv6?

2.1 IPv4 (Internet Protocol version 4)

IPv4 adalah versi keempat dari protokol IP yang paling umum digunakan saat ini. IPv4 menggunakan alamat 32-bit yang terdiri dari empat oktet angka desimal (0-255), misalnya:

```
192.168.1.1
```

IPv4 memiliki sekitar 4,3 miliar alamat unik, tetapi karena pertumbuhan internet, jumlah ini menjadi tidak cukup.

2.2 IPv6 (Internet Protocol version 6)

IPv6 adalah pengembangan dari IPv4 yang menggunakan alamat 128-bit, yang memungkinkan jumlah alamat yang jauh lebih besar. Contohnya:

```
2001:db8::ff00:42:8329
```

IPv6 juga menawarkan peningkatan keamanan dan efisiensi dibanding IPv4.

Perbedaan IPv4 vs IPv6

|Aspek |	IPv4 |	IPv6
|------------------|-------|------|
Panjang alamat	| 32-bit	|128-bit
Contoh alamat	| 192.168.1.1	| 2001:db8::ff00:42:8329
Jumlah alamat unik	| 4,3 miliar	| 340 triliun triliun triliun
Konfigurasi	| Manual atau DHCP	| Otomatis dengan autoconfiguration
Keamanan	| Kurang aman, perlu tambahan seperti IPsec	| Lebih aman, IPsec bawaan

Use Case: IPv4 masih banyak digunakan untuk jaringan tradisional.

IPv6 mulai digunakan di layanan cloud, IoT, dan internet modern untuk menangani peningkatan jumlah perangkat.

3. Perbedaan IP Eksternal dan IP Internal

3.1 IP Eksternal (Public IP)

IP eksternal adalah alamat IP yang dapat diakses dari internet dan digunakan untuk menghubungkan perangkat ke jaringan global.

- Diberikan oleh ISP (Internet Service Provider).
- Digunakan oleh server, website, atau router untuk akses publik.
- Contoh: 103.28.12.45 (alamat IP dari ISP).

3.2 IP Internal (Private IP)

IP internal adalah alamat yang digunakan dalam jaringan lokal (LAN) dan tidak dapat diakses langsung dari internet.

- Digunakan dalam jaringan rumah, kantor, atau perusahaan.
- Ditugaskan oleh router atau server DHCP dalam jaringan lokal.
- Contoh: 192.168.1.10 (alamat komputer dalam jaringan Wi-Fi rumah).

Use Case:

IP Eksternal: Server hosting website memiliki IP eksternal agar dapat diakses dari seluruh dunia.

IP Internal: Laptop di rumah memiliki IP internal untuk berkomunikasi dengan router dan perangkat lain dalam jaringan lokal.

4. Perbedaan IP Internal 172, 192, dan 10.

IP internal memiliki tiga rentang utama berdasarkan RFC 1918, yaitu:

Rentang IP	| Kelas	| Jumlah Alamat	| Contoh Penggunaan
|------------------|-------|------|--------------|
10.0.0.0 - 10.255.255.255	| Kelas A	| 16 juta+	| Jaringan besar (perusahaan)
172.16.0.0 - 172.31.255.255	| Kelas B	| 1 juta+	| Jaringan menengah (perusahaan, kampus)
192.168.0.0 - 192.168.255.255	| Kelas C	| 65 ribu+	| Jaringan kecil (rumah, kantor)

Perbedaan utama:

- 192.168.x.x sering digunakan untuk jaringan rumah atau kantor kecil (misalnya Wi-Fi).

- 172.16.x.x - 172.31.x.x digunakan oleh organisasi menengah yang membutuhkan lebih banyak alamat.

- 10.x.x.x digunakan untuk organisasi besar seperti perusahaan dan data center.

Use Case:

- 192.168.x.x: Router rumah menggunakan alamat ini (misalnya 192.168.1.1).

- 172.16.x.x: Digunakan di kampus atau perusahaan menengah.

- 10.x.x.x: Digunakan di jaringan skala besar, misalnya jaringan internal Google atau Facebook.

5. Bagaimana Cara Internet Bekerja dengan IP dan DNS?

Konsep Dasar

DNS (Domain Name System) mengubah nama domain menjadi alamat IP.

IP digunakan untuk mengirim data antar perangkat melalui internet.

Router dan ISP membantu mengarahkan lalu lintas data ke tujuan yang benar.

Cara Kerja

1. Pengguna mengetikkan www.google.com di browser.

2. Komputer mengirim permintaan ke server DNS untuk menerjemahkan www.google.com menjadi alamat IP (misalnya 142.250.180.206).

3. Dengan alamat IP tersebut, browser mengirimkan permintaan ke server Google.

4. Server Google merespons dengan mengirimkan data website ke IP pengguna.

5. Browser menampilkan halaman Google di layar.

Use Case:

DNS sangat penting karena manusia lebih mudah mengingat nama domain dibanding angka IP.

Tanpa DNS, kita harus mengetik IP setiap kali ingin mengakses situs web.

