# Confluent CDC from MySQL, PostgreSQL, Oracle

1. MySQL

Pertama - tama yang saya lakukan adalah memastikan keberhasilan tertambahnya connector debezium

dengan command:
```
confluent-hub install debezium/debezium-connector-mysql:latest
```
Jika sudah berhasil, MySQL connector akan terlihat seperti gambar berikut di Confluent:

![image](https://github.com/user-attachments/assets/eba00f51-2fc3-4fba-aab0-56d5fecdf653)

Selanjutnya, saya akan melakukan konsfigurasi connector untuk menghubungkan Kafka dengan MySQL.

Berikut adalah konfigurasinya:
```
    name = TestdebeziumMysql_New
    connector.class = io.debezium.connector.mysql.MySqlConnector
    tasks.max = 1
    key.converter = io.confluent.connect.avro.AvroConverter
    value.converter = io.confluent.connect.avro.AvroConverter
    topic.prefix = cdc_mysql
    database.hostname = 10.100.13.239
    database.port = 3306
    database.user = kafka
    database.password = P@ssw0rd123!
    database.server.id = 223344
    database.ssl.mode = disabled
    snapshot.mode = initial
    table.include.list = kafka_db.kafka_table, kafka_db.example_table
    include.schema.changes = true
    database.include.list = 
    topic.creation.default.partitions = 1
    schema.history.internal.kafka.topic = cdc_schema
    topic.creation.default.replication.factor = 1
    database.allowPublicKeyRetrieval = true
    topic.creation.default.cleanup.policy = compact
    schema.history.internal.kafka.bootstrap.servers = server-eric:9092
    value.converter.schema.registry.url = http://server-eric:8081
    database.connectionTimeZone = Asia/Jakarta
    key.converter.schema.registry.url = http://server-eric:8081
```

Diatas adalah file .properties. saya hanya menguploadnya di menu confluent. 

Berikutnya adalah pastikan connector sudah running dan jika terdapat error, diharapkan untuk melihat log dan usahakan connector berjalan.

Pastikan mode cleanup.policy pada topik menjadi delete dan pilih Retention time menjadi Infinite

![image](https://github.com/user-attachments/assets/f81c85a4-eb13-424b-b916-53b56572a7d7)

Gambar diatas menunjukann Konfigurasi pada topik yang menyimpan data CDC dari MySQL.

Jika semua sudah berjalan, maka akan terdapat topik yang menampung data yang sudah saya tarik dari server MySQL:

Berikut hasilnya:

![image](https://github.com/user-attachments/assets/da00f49f-75fa-4bdf-9c57-f65ce12393f6)

Gambar diatas menggambarkan penambhan dan perubahan yang terjadi pada data 'Ucok'.

CDC bisa merekam pergantian data secara langsung dari MySQL, termasuk INSERT, UPDATE DAN DELETE.

Berikut adalah contoh CDC merekam DELETE:

![image](https://github.com/user-attachments/assets/f82b757b-e9e3-4016-8dd5-466373552904)

Gambar diatas menunjukan data 'Ucok' sudah ter-DELETE.

2. PostgreSQL

Langkah - langkah yang harus dilakukan untuk membuat connector CDC dari PSQL ke Kafka, hampir sama dengan CDC dari MySQL ke kafka.

Pertama, saya harus melakukan penginstalan connector PSQL sendiri dengan command:

```
confluent-hub install debezium/debezium-connector-postgresql:latest
```

Lalu, setelah selesai melakukan penginstallan, saya harus merestart ulang confluent-kafka-connect dan pastikan connector PSQL sudah masuk di dalam menu connect Confluent seperti pada gambar:

![image](https://github.com/user-attachments/assets/fcb6ea17-c5a4-4b88-bf42-58889cc3b7f6)

Setelah menambahkan connector, saya juga perlu untuk download Postgre sendiri dengan command:

```
sudo apt update
sudo apt install postgresql postgresql-contrib
sudo systemctl start postgresql
sudo systemctl enable postgresql
sudo -u postgres psql
```

Selanjutnya setelah download dan mengaktifkan Postgre, saya perlu menambahkan User dan Database baru di Postgre dengan command:

```
sudo -u postgres createuser --interactive
```

command ini digunakan untuk membuat user baru. Dan ketikan dijalankan, Postgre akan bertanya nama user yang ingin dibuat dan bertanya apakah User tersebut ingin dijadikan main role atau super User di Postgre.
Dalam hal ini, saya menjadikan User saya sebagai kafka dan menjadikan kafka sebagai SuperUser.

Berikut adalah command untuk menambahkan password dan menambahkan privileges tambahan kepada user:
```
sudo -u postgres psql
# In the psql prompt
ALTER USER kafka WITH PASSWORD 'P@ssw0rd123!';
ALTER USER kafka WITH LOGIN;
ALTER USER kafka WITH REPLICATION;
CREATE DATABASE kafka_dab;
GRANT ALL PRIVILEGES ON DATABASE kafka_dab TO kafka;
```

Selanjutnya, saya akan membuat table di dalam database kafka_dab yang sudah saya buat dengan command:
```
CREATE TABLE students (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    hobby VARCHAR(50),
    age INT
);
```
Selanjutnya, saya akan mengubah beberapa konfigurasi dari PSQL yang berada pada 2 file dengan command:
```
sudo nano /etc/postgresql/16/main/postgresql.conf
```
```
sudo nano /etc/postgresql/12/main/pg_hba.conf
```

Di dalam file postgresql.conf, saya ingin mengubah hal berikut:

```
listen_addresses = '*'
wal_level = logical
max_replication_slots = 4
max_wal_senders = 4
```
Mengapa? karena jika listen_addresses tidak diubah PSQL tidak dapat di akses dari ip manapun, tanda * menunjukan bahwa PSQL dapat menerima perubahan dari semua ip yang ingin melakukan perubahan data di dalam PSQL.
Begitu juga dengan wal_level yang awalnya berstatus 'replica', saya ubah menjadi 'logical'. Hal ini memungkinkan replikasi yang bersifat logical, yang dimana memperbolehkan perubahan stream data (hal yang penting dilakukan dalam CDC).
max_replication_slots hanya memberi batas berapa banyak replikasi yang PSQL lakukan.
max_wal_senders juga hanya memberikan batas maksimal berapa banyak sender yang ingin mengirim data ke PSQL.

Di dalam file pg_hba.conf, saya ingin manambahkan hal berikut:

```
hostnossl    kafka_dab      kafka      10.100.12.0/24         md5
```
hal ini diperlukan, karena tidak ada ssl yang saya perlukan di dalam koneksi PSQL ke Kafka. Jadi saya memilih untuk membuat host yang tidak mempunyai akses SSL di dalamnya.

Setelah semua selesai, saya akan menjalankan command:
```
sudo systemctl reload postgresql
sudo systemctl stop postgresql
sudo systemctl daemon-reload
sudo systemctl restart postgresql
sudo systemctl status postgresql
```
Jika Postgre sudah berjalan, saya akan coba login ke PSQL menggunakan command:
```
psql -h 10.100.13.239 -p 5432 -U kafka -d kafka_dab
```
Jika berhasil maka PSQL sudah bisa diakses menggunakan ip yang saya inginkan dan tanpa SSL.

Selanjutnya, saya akan membuat konfigurasi dari PSQL ke Kafka untuk PSQL connector, berikut konfigurasi PSQL connector:

```
name=psql-cdc-connector
connector.class=io.debezium.connector.postgresql.PostgresConnector
tasks.max=1

# Converter configuration
key.converter=io.confluent.connect.avro.AvroConverter
value.converter=io.confluent.connect.avro.AvroConverter
key.converter.schema.registry.url=http://server-eric:8081
value.converter.schema.registry.url=http://server-eric:8081

# PostgreSQL Database Configuration
database.hostname=10.100.13.239
database.port=5432
database.user=kafka
database.password=P@ssw0rd123!
database.dbname=kafka_dab
database.server.name=psqldbserver1

# Plugin for logical replication
plugin.name=pgoutput

# Schema Registry URL
schema.registry.url=http://server-eric:8081
key.converter.schema.registry.url = http://server-eric:8081

# Internal Kafka topics configuration
database.history.kafka.bootstrap.servers=server-eric:9092
database.history.kafka.topic=cdc_psqlschema

# Table to capture changes from
table.whitelist=public.students

# Topic prefix for emitted change events
topic.prefix=cdc_psql_students
```

Pastikan semua URL, Port dan hal lainnya sudah sesuai dengan keperluan untuk memulai PSQL connector.

Jika sudah mengupload file konfigurasi berikut dan menunjukan status RUNNING. cobalah untuk melakukan pengecekan pada topik

Jika sudah berhasil, maka akan terlihat sesuai pada gambar:

![image](https://github.com/user-attachments/assets/b318db2e-111c-4f21-955c-8f55a30b221d)

Pada gambar topik tersebut terbuat secara otomatis dari connector dan merepresentasikan data yang sudah saya isi didalam konfigurasi PSQL connector.

Hasil CDC PSQL:

INSERT

![image](https://github.com/user-attachments/assets/d62d34fa-95e5-4a50-a043-4bfc33102b3d)

UPDATE

![image](https://github.com/user-attachments/assets/83318028-7f5a-466f-adba-d201a362a872)

DELETE

![WhatsApp Image 2024-10-07 at 16 49 41_8bd84b3b](https://github.com/user-attachments/assets/171559b6-54f4-44e4-b756-ded8bb9c6820)







