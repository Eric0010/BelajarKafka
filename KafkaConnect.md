# Kafka connect with JDBC MySQL connector

Kafka Connect adalah alat yang membantu memindahkan data antara Apache Kafka dan sistem lainnya. Ini memudahkan streaming data dalam jumlah besar, seperti log, basis data, atau data sensor, ke dalam Kafka atau mengambil data dari Kafka dan mengirimkannya ke sistem lain, seperti basis data, danau data, atau bahkan cluster Kafka lainnya.

Kafka Connect bekerja dengan Connector, yang merupakan plugin atau bagian perangkat lunak yang memberi tahu Kafka ke mana data harus dikirim atau dari mana data harus diambil. Ada dua jenis konektor:

Source Connectors (Konektor Sumber): Konektor ini mengambil data dari sistem lain dan mendorongnya ke Kafka.
Sink Connectors (Konektor Penampung): Konektor ini mengambil data dari Kafka dan mendorongnya ke sistem lain.

Kafka Connect sangat berguna karena:

. Anda tidak perlu menulis banyak kode.
. Dapat dengan mudah diskalakan untuk menangani data dalam jumlah besar.
. Berjalan secara otomatis, sehingga setelah diatur, konektor terus bekerja tanpa banyak perhatian.

Berikut adalah langkah - langkah untuk menggunakan kafka connect:

Disini saya menggunakan JDBC MySQL sebagai connector.

1. Download JDBC (Source & Sink) connector dari web berikut ini:

https://www.confluent.io/hub/confluentinc/kafka-connect-jdbc

Sesampainya di website tersebut, plih Self-Hosted. Namun, dikarenakan tidak bisa meng-copy link dari website ini maka, saya tidak bisa menggunakan command ```wget``` di Ubuntu Terminal.

Solusi yang harus diterapkan adalah, saya harus mendownload file tersebut ke dalam direktori Windows terlebih dahulu dan harus memindahkan file tersebut secara manual ke dalam server Ubuntu Terminal menggunakan tools VSCode.

extract file tersebut dengan menggunakan command ```unzip``` di terminal karena file tersebut berbentuk ```.zip```

![image](https://github.com/user-attachments/assets/ed17f8cc-4fe3-4112-b851-a0d9adcbf8da)

Setelah mengekstrak, pindahkan file ```.jar``` tersebut ke dalam direktori ```libs``` di dalam user kafka.

2. Install MySQL Server

.Perbarui Indeks Paket
Pertama, perbarui indeks paket Anda untuk memastikan Anda memiliki informasi terbaru tentang paket yang tersedia:

```
sudo apt update
```
.Untuk menginstal MySQL, jalankan perintah berikut:

```
sudo apt install mysql-server
```

Ini akan menginstal server MySQL dan dependensinya. Selama proses instalasi, Anda mungkin diminta untuk mengonfigurasi kata sandi root. Jika tidak, Anda dapat mengaturnya nanti.

. Periksa Layanan MySQL
Setelah instalasi, layanan MySQL akan dimulai secara otomatis. Anda dapat memeriksa statusnya dengan:

```
sudo systemctl status mysql
```

Jika tidak berjalan, mulai menggunakan:

```
sudo systemctl start mysql
```

Anda juga dapat mengaktifkannya untuk memulai saat boot:

```
sudo systemctl enable mysql
```

. Instalasi MySQL yang Aman
Untuk meningkatkan keamanan, jalankan skrip keamanan MySQL:

```
sudo mysql_secure_installation
```

Skrip ini akan menanyakan beberapa pertanyaan untuk mengamankan instalasi Anda:

Tetapkan kata sandi daripada username yang sudah dibuat (jika tidak ditetapkan selama instalasi)
Hapus pengguna anonim
Larang login username dari jarak jauh
Hapus basis data pengujian
Muat ulang tabel hak istimewa
Untuk masing-masing pertanyaan ini, ketik Y untuk ya guna menerima pengaturan aman default.

. Masuk ke MySQL
Untuk masuk ke MySQL sebagai pengguna root, gunakan perintah berikut:

```
sudo mysql -u <nama pengguna> -p
```

Masukkan kata sandi root yang Anda tetapkan selama instalasi atau saat menjalankan skrip mysql_secure_installation.

. Buat Pengguna Baru (Opsional)
Merupakan praktik yang baik untuk membuat pengguna baru alih-alih menggunakan akun root untuk basis data Anda:

```
CREATE USER 'newuser'@'localhost' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON *.* TO 'newuser'@'localhost';
FLUSH PRIVILEGES;
```

. Verifikasi Instalasi
Anda dapat memverifikasi versi MySQL yang terinstal dengan menjalankan:

```
mysql --version
```

Ini akan menampilkan versi MySQL yang terinstal.

NOTE: username dan password yang saya gunakan adalah berikut:

```

user=kafka&password=P@ssw0rd

```
.Buat Database dan sebuah Table di dalam MySql server dengan command:

-- Buat Database baru
```
CREATE DATABASE kafka_db;
```
-- Pindah ke Database baru
```
USE kafka_db;
```

-- Buat table baru
```
CREATE TABLE kafka_table (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    email VARCHAR(255),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

-- Cek Table baru
```
SHOW TABLES;
```

NOTE: username dan password yang saya gunakan adalah berikut:

```

dbname=kafka_db&tablename=kafka_table

```
.Insert beberapa data ke dalam Table yang sudah dibuat dengan command:
```
INSERT INTO kafka_table (id, name, email, created_at)
VALUES 
    (NULL, 'Alice Johnson', 'alice.johnson@example.com', '2024-09-21 09:15:45'),
    (NULL, 'Bob Brown', 'bob.brown@example.com', '2024-09-21 10:45:30'),
    (NULL, 'Charlie Davis', 'charlie.davis@example.com', '2024-09-21 11:30:00'),
    (NULL, 'Diana Prince', 'diana.prince@example.com', '2024-09-21 12:00:00'),
    (NULL, 'Edward Norton', 'edward.norton@example.com', '2024-09-21 12:30:00');
```

![image](https://github.com/user-attachments/assets/e4c24935-bbce-4976-b9cd-33b0d9fac860)


3. Buat file ```mysql-jdbc-connector.properties``` di dalam direktori config di dalam user kafka.

   Berikut adalah isi dari file properties yang ingin dibuat:

```
name=mysql-source-connector1
connector.class=io.confluent.connect.jdbc.JdbcSourceConnector
tasks.max=1
connection.url=jdbc:mysql://localhost:3306/kafka_db?user=kafka&password=P@ssw0rd
table.whitelist=kafka_table
mode=incrementing
incrementing.column.name=id
topic.prefix=kafka_db-
```

File berikut, mempunyai beberapa komponen yang penting seperti ```connection.url```, ```table.whitelist``` dan ```topic.prefix```

```connection.url``` = berisikan dimana port my sql berjalan bersama dengan user dan password yang kita gunakan di mysql server.

```table.whitelist``` = mewakilkan table yang akan saya panggil.

```topic.prefix``` = mewkilkan database yang akan saya panggil, dan juga mewakilkan judul topik untuk identifikasi oleh sistem kafka sendiri untuk pengambilan data pada database tersebut.

4. Buat Systemd menggunakan Kafka connect dengan command berikut:
```
sudo nano /etc/systemd/system/kafka-connect.service
```
Lalu isi file tersebut seperti yang ada dibawah ini:
```
[Unit]
Description=Kafka Connect Standalone Service
After=network.target

[Service]
Type=simple
User=kafka
Group=kafka
ExecStart=/home/kafka/kafka/bin/connect-standalone.sh /home/kafka/kafka/config/connect-standalone.properties /home/kafka/kafka/config/mysql-jdbc-connector.properties
Restart=on-failure
RestartSec=5
ExecStop=/bin/kill -TERM $MAINPID

[Install]
WantedBy=multi-user.target
```

intinya execstartnya mewakili semua file yang saya mau run dan destinasinya harus tepat sesuai sama direktori file di kafka kita

.Jalankan kafka connectnya

```
systemctl start kafka-connect.service
```

Cek status Kafka Connect, jika masih status masih failed atau status lain selain Running. Gunakan command berikut:

```
tail -f connect.log
```
Identifikasi masalah dan lakukan perbaikan sesuai dengan log yang ditampilkan.

5. Sebelum memindahkan data, perlu untuk melakukan pengecekan connector dengan command:
```
curl http://localhost:8083/connector-plugins
```

![image](https://github.com/user-attachments/assets/fedf85b6-1042-4a1a-9abe-bdcbdd8795d1)

pengecekan juga bisa dalam bentuk file JSON dengan command:

```
curl http://localhost:8083/connector-plugins | jq
```

![image](https://github.com/user-attachments/assets/fd7befa6-de66-48ce-bfd1-842128149ce7)

6. Jika file diatas semua sudah dijalankan barulah saya akan melakukan consume data dengan command consume seperti berikut:
```
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic kafka_db-kafka_table --from-beginning

```

```kafka_db-kafka_table``` mewakili database dan tabel yang terdapat pada mysql server.

![image](https://github.com/user-attachments/assets/19003d06-d4b1-49e1-a6e9-7e1284db6b4a)


Jika ingin melakukan pengecekan topik di dalam user Kafka:
```
bin/kafka-topics.sh --bootstrap-server server-eric:9092 --lists
```




