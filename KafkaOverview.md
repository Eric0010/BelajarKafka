1. What is Kafka?

Apache Kafka adalah penyimpanan data terdistribusi yang dioptimalkan untuk menyerap dan memproses data streaming secara real-time. 
Data streaming adalah data yang terus-menerus dihasilkan oleh ribuan sumber data, yang biasanya mengirimkan rekaman data secara bersamaan. 
Platform streaming perlu menangani masuknya data secara konstan ini, dan memproses data secara berurutan dan bertahap.

2. What is Kafka used for?

Kafka digunakan untuk membangun jalur data streaming real-time dan aplikasi streaming real-time. 
Jalur data memproses dan memindahkan data dari satu sistem ke sistem lain secara andal, dan aplikasi streaming adalah aplikasi yang menggunakan aliran data. 
Misalnya, jika Anda ingin membuat jalur data yang mengambil data aktivitas pengguna untuk melacak bagaimana orang menggunakan situs web Anda secara real-time, Kafka akan digunakan untuk menyerap dan menyimpan data streaming sambil menyajikan pembacaan untuk aplikasi yang mendukung jalur data tersebut.
Kafka juga sering digunakan sebagai solusi perantara pesan, yaitu platform yang memproses dan memediasi komunikasi antara dua aplikasi.

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

4. Companies that use Kafka:

Banyak perusahaan dan organisasi terkemuka menggunakan Apache Kafka untuk kebutuhan streaming data dan pemrosesan real-time mereka. 

Berikut ini beberapa contoh penting:

. LinkedIn: Kafka awalnya dikembangkan di LinkedIn, dan perusahaan terus menggunakannya secara ekstensif untuk agregasi log, analisis real-time, dan pemrosesan aliran.

. Netflix: Netflix menggunakan Kafka untuk sistem streaming dan pemantauan acara real-time, menangani sejumlah besar data streaming dari berbagai sumber.

. Uber: Uber memanfaatkan Kafka untuk memproses aliran data real-time yang terkait dengan permintaan perjalanan, aktivitas pengemudi, dan metrik operasional lainnya.

. Spotify: Spotify menggunakan Kafka untuk mengelola dan memproses sejumlah besar data acara, termasuk interaksi pengguna dan log aplikasi.

. Airbnb: Airbnb menggunakan Kafka untuk integrasi data dan analisis real-time, yang memungkinkan perusahaan untuk menangani berbagai aliran data dari berbagai layanan.

. Ebay: eBay menggunakan Kafka untuk jalur datanya guna mendukung analisis real-time dan pemrosesan data di seluruh platformnya.

. Twitter: Twitter menggunakan Kafka untuk pemrosesan aliran data secara real-time dan untuk menangani data berthroughput tinggi yang dihasilkan oleh platform media sosialnya.

. Yahoo: Yahoo mengintegrasikan Kafka untuk kebutuhan pemrosesan data berskala besar, termasuk analisis data real-time dan agregasi log.

. Salesforce: Salesforce menggunakan Kafka untuk streaming data dan arsitektur berbasis peristiwa untuk mendukung berbagai aplikasi dan layanan.

. Microsoft: Microsoft menggunakan Kafka dalam layanan cloud Azure untuk streaming peristiwa dan pemrosesan data secara real-time.

Perusahaan-perusahaan ini mengandalkan skalabilitas, toleransi kesalahan, dan kinerja tinggi Kafka untuk mengelola infrastruktur data mereka dan memungkinkan wawasan dan aplikasi secara real-time.

5. Kafka Use Case
