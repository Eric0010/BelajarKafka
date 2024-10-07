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




