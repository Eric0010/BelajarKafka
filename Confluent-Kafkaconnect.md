# Buat Connector Source/Sink via C3

Pertama - tama saya perlu memastikan Confluent Kafka Connect sudah berjalan dengan baik atau belum. dengan command:
```
  systemctl status confluent-kafka-connect
```
Selanjutnya saya ingin melakukan perubahan di systemD Confluent-kafka-connect. Berikut perubahannya:

```
[Unit]
Description=Apache Kafka Connect - distributed
Documentation=http://docs.confluent.io/
After=network.target confluent-server.target

[Service]
Type=simple
User=cp-kafka-connect
Group=connect-default
ExecStart=/home/kafka/confluent-7.7.0/bin/connect-distributed /home/kafka/confluent-7.7.0/etc/kafka/connect-distributed.properties
TimeoutStopSec=180
Restart=no

[Install]
WantedBy=multi-user.target
```

Setelah saya merubah systemd dari confluent-kafka-connect, saya akan merubah sesuatu di dalam file connect-ditributed.properties berikut perubahannya:
```
bootstrap.servers = server-eric:9092
#unique name for the cluster, used in forming the Connect cluster group. Note that this must not conflict with consumer group IDs
group.id = connect-cluster
#The converters specify the format of data in Kafka and how to translate it into Connect data. Every Connect user will
#need to configure these based on the format they want their data in when loaded from or stored into Kafka
#key.converter = org.apache.kafka.connect.json.JsonConverter
#value.converter = org.apache.kafka.connect.json.JsonConverter
#Converter-specific settings can be passed in by prefixing the Converter's setting with the converter we want to apply
#it to
key.converter=io.confluent.connect.avro.AvroConverter
value.converter=io.confluent.connect.avro.AvroConverter
key.converter.schemas.enable = false
value.converter.schemas.enable = false
listeners=http://0.0.0.0:8083
plugin.path = /usr/share/java,/usr/share/confluent-hub-components, /home/kafka/confluent-7.7.0/share/java/kafka
```
Disini plugin.path yang saya miliki beberapa destinasi. Destinasi ini menggambar file .jar atau plugin yang saya gunakan untk JDBC Connector (Source dan Sink) yang bisa di download di link berikut:

https://www.confluent.io/hub/confluentinc/kafka-connect-jdbc - Connector

https://dev.mysql.com/downloads/connector/j/ - MYSQL Driver

NOTE: 2 file tersebut harus terinstall dan jelas alamat direktori anda menginstallnya.

Jika file ditas semua sudah sesuai, restart confluent-kafka-connect:

```
  systemctl restart confluent-kafka-connect
```

![confluent_connector](https://github.com/user-attachments/assets/b565e800-e917-45c3-afc3-2157316ba184)

NOTE: Jika masih belum terlihat, restart semua Fasilitas seperti di catatan Confluent sebelumnya
# Source

Untuk menambahkan Sebuah connector kita perlu sebuah konfigurasi dalam bentuk file .properties atau .json. Seperti contoh berikut:
```
name = JdbcSourceConnector
connector.class = io.confluent.connect.jdbc.JdbcSourceConnector
tasks.max = 1
key.converter = io.confluent.connect.avro.AvroConverter
value.converter = io.confluent.connect.avro.AvroConverter
connection.url = jdbc:mysql://10.100.13.239:3306/kafka_db
connection.user = kafka
connection.password = P@ssw0rd
table.whitelist = kafka_table
dialect.name = MySqlDatabaseDialect
mode = bulk
topic.prefix = kafka_db-
value.converter.schema.registry.url = http://server-eric:8081
key.converter.schema.registry.url = http://server-eric:8081
```
Saya akan mengambil data dari MySQL. menggunakan JDBC Source

![image](https://github.com/user-attachments/assets/02d5dfd2-df35-48c4-a23b-34f5e0e8059c)

Disini saya sudah mengupload file .properties ditas melalui confluent.

Berikut hasil yang ditampilkan dari table kafka_table:

Kafka Source:
Sudah berhasil, saya bisa melihat message yang saya buat dari mySQL di Confluent

![image](https://github.com/user-attachments/assets/b79492c9-5005-469d-a0fd-3b870fa5d06d)

# Sink

Untuk Sink juga membutuhkan file .properties untuk menjalankannya. berikut file .propertiesnya:

```
name = mysql_sink
connector.class = io.confluent.connect.jdbc.JdbcSinkConnector
tasks.max = 1
key.converter = io.confluent.connect.avro.AvroConverter
value.converter = io.confluent.connect.avro.AvroConverter
topics = inventory_python
connection.url = jdbc:mysql://10.100.13.239:3306/kafka_db
connection.user = kafka
connection.password = P@ssw0rd
dialect.name = MySqlDatabaseDialect
insert.mode = insert
table.name.format = ${topic}
auto.create = false
auto.evolve = false
value.converter.schema.registry.url = http://server-eric:8081
key.converter.schema.registry.url = http://server-eric:8081
```

Disini saya akan mengambil data dari producer dengan bahasa pemrograman Python, Java dan Golang:

Saya akan menjelaskan sedikit tentang Kafka connect Sink ini:
1. ```topics = inventory_python``` topik menentukan pengambilan message yang ingin diambil. jika penginputan topic salah maka message tidak berhasil diambil.
2. ```auto.create = false``` fitur ini bisa digunakan jika anda belum create table di Mysql. fitur ini akn membuat secara otomatis table/schema yang dihasilkan producer pada mysql sendiri beserta dengan pesan - pesannya. (jika value = true)

NOTE: Pada awalnya saya melakukan sink pada producer python dan membuat table di mysql secara manual. dan bekerja. 
tetapi pada saat saya menarik message dari topik Java, pesan saya tidak bisa di sink karena auto.create is disabled terlihat pada log, padahal saya sudah create table secara manual di mysql. Bila hal ini terjadi pada anda, segera ubah value ```auto.create=``` menjadi true

Berikut adalah hasil dari Sink yang saya lakukan dari ke 3 Producer:

Kafka Sink:
![image](https://github.com/user-attachments/assets/7ce4834b-78fc-4c63-b97b-3577794301e3)

Message Python:

![image](https://github.com/user-attachments/assets/f3f415b3-44d3-430c-a0e2-ffd5a0b054ee)

Message Java:

![image](https://github.com/user-attachments/assets/dc3a213b-d6c0-4b46-8212-d61976fc7c64)


Message Client Golang. 


![image](https://github.com/user-attachments/assets/586e3532-9370-41d9-9e22-ffa04104268b)

