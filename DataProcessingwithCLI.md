# 1. Make a topic with Kafka

Sebelum membuat topik pertama yang harus kita pastikan adalah server Zookeeper dan server Kafka harus dinyalakan terlebih dahulu.

Berikut adalah command untuk membuat topik di Kafka

```
~/kafka/bin/kafka-topics.sh --bootstrap-server localhost:9092 --create --topic <judul topikmu>
```
![image](https://github.com/user-attachments/assets/592988e4-2084-4b8a-8ddd-7d3c106e3034)

Setelah membuat sebuah topik, lakukan pengecekan apakah topik tersebut sudah sukses dibuat?

```
~/kafka/bin/kafka-topics.sh --list --bootstrap-server localhost:9092
```
![image](https://github.com/user-attachments/assets/b1041446-f750-496b-b3e7-32552fec9d86)

# 2. Erase a topic with Kafka

Berikut adalah command untuk menghapus sebuah topik di Kafka

```
bin/kafka-topics.sh --delete --topic <nama topik yang ingin dihapus> --bootstrap-server localhost:9092
```
![image](https://github.com/user-attachments/assets/16e97064-fa86-4b11-ac81-915770d1ce9a)

Setelah command tersbut berjalan, berikut adalah command untuk melakukan pengecekan terhadap list yang sudah kita hapus

```
~/kafka/bin/kafka-topics.sh --list --bootstrap-server localhost:9092
```

# 3. Produce data with Kafka

Untuk menghasilkan sebuah data, hal pertama yang harus dibuat adalah topik.

Setelah membuat topik barulah kita bisa menghasilkan data yang ingin kita buat dengan command

```
~/kafka/bin/kafka-console-producer.sh --bootstrap-server localhost:9092 --topic firstTopic
```

Di dalam code yang berjalan seperti diatas user berlaku sebagai produsen yang bertugas untuk menghasilkan data dan akan diterima oleh konsumen.

# 4. Consume data with Kafka

Setelah menghasilkan data sebagai produsen, langkah selanjutnya adalah user akan berlaku sebagai konsumen data.

Untuk menerima sebuah data sebagai konsumen, berikut adalah command yang diperlukan

```
~/kafka/bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic firstTopic --from-beginning
```

Barulah user bisa melihat data yang sudah dihasilkan oleh produsen data.
