3. Kafka Architecture:

Kafka mengatasi kedua model yang berbeda tersebut dengan menerbitkan catatan ke topik yang berbeda. 
Setiap topik memiliki log yang dipartisi, yaitu log komit terstruktur yang melacak semua catatan secara berurutan dan menambahkan yang baru secara real time. 
Partisi ini didistribusikan dan direplikasi di beberapa server, yang memungkinkan skalabilitas tinggi, toleransi kesalahan, dan paralelisme. 
Setiap konsumen diberi partisi di topik, yang memungkinkan banyak pelanggan sekaligus menjaga urutan data. Dengan menggabungkan model pengiriman pesan ini, Kafka menawarkan manfaat keduanya. 
Kafka juga bertindak sebagai sistem penyimpanan yang sangat skalabel dan toleran terhadap kesalahan dengan menulis dan mereplikasi semua data ke disk. 

Secara default, Kafka menyimpan data di disk hingga kehabisan ruang, tetapi pengguna juga dapat menetapkan batas penyimpanan. Kafka memiliki empat API:

API Produsen: digunakan untuk menerbitkan aliran catatan ke topik Kafka.
API Konsumen: digunakan untuk berlangganan topik dan memproses aliran catatannya. 
Streams API: memungkinkan aplikasi berperilaku sebagai pemroses aliran, yang mengambil aliran input dari topik dan mengubahnya menjadi aliran output yang masuk ke topik output yang berbeda. 
Connector API: memungkinkan pengguna untuk mengotomatiskan penambahan aplikasi atau sistem data lain ke topik Kafka mereka saat ini dengan lancar.
