# Pengertian

---

**Kafka REST Proxy**, atau disebut juga **kafkarest**, adalah API RESTful yang disediakan oleh Apache Kafka untuk berinteraksi dengan cluster Kafka melalui permintaan HTTP, bukan klien Kafka asli. Kafka REST Proxy adalah bagian dari Confluent Platform dan memudahkan penggunaan Kafka dengan memungkinkan aplikasi untuk berkomunikasi dengan topik Kafka melalui panggilan RESTful sederhana.

Berikut adalah poin-poin penting mengenai Kafka REST Proxy:

1. **Tujuan dan Kasus Penggunaan**
   - Kafka REST Proxy memungkinkan produsen dan konsumen berinteraksi dengan topik Kafka melalui HTTP tanpa memerlukan klien Kafka asli.
   - Berguna untuk aplikasi di mana integrasi langsung dengan pustaka Kafka sulit dilakukan (misalnya, sistem lama, aplikasi serverless, atau sistem dengan batasan untuk menjalankan klien Kafka).
   - Cocok untuk transfer data yang ringan dan berdaya rendah serta integrasi dengan sistem yang terutama berkomunikasi melalui HTTP/REST.

2. **Fungsi Dasar**
   - **Mengirim Pesan**: Memungkinkan pengiriman pesan ke topik Kafka dengan mengirim permintaan HTTP POST ke REST Proxy.
   - **Mengkonsumsi Pesan**: Memungkinkan pengambilan pesan dari topik Kafka melalui permintaan HTTP GET.
   - **Akses Metadata**: Menyediakan endpoint API untuk mengambil metadata tentang topik, partisi, dan broker dalam cluster Kafka.

3. **Integrasi Schema Registry**
   - REST Proxy terintegrasi dengan Confluent Schema Registry untuk mengelola skema data pesan yang diserialisasi, mendukung skema Avro dan JSON.
   - Saat mengirim pesan, pengguna dapat menentukan skema, yang memastikan data sesuai dengan struktur dan format yang diharapkan.
   - Fitur ini sangat berguna untuk menjaga konsistensi data dan mencegah masalah evolusi skema dalam arsitektur berbasis Kafka.

4. **Keamanan**
   - Kafka REST Proxy mendukung berbagai konfigurasi keamanan, termasuk enkripsi TLS untuk komunikasi HTTP yang aman, Autentikasi Dasar (Basic Authentication), dan OAuth untuk mengamankan akses ke endpoint.
   - Juga dapat dikonfigurasi untuk mengautentikasi pengguna terhadap layanan otorisasi, memberikan kontrol akses berbasis peran (RBAC).

5. **Konfigurasi dan Penerapan**
   - REST Proxy dapat dikonfigurasi dengan detail cluster Kafka tertentu, dan biasanya dijalankan sebagai layanan mandiri atau sebagai bagian dari Confluent Platform.
   - Mudah untuk disiapkan dan diterapkan di lingkungan kontainer dan cloud, menjadikannya fleksibel untuk berbagai kebutuhan penerapan.

6. **Pertimbangan Performa**
   - Meskipun berguna untuk akses berbasis HTTP, REST Proxy umumnya kurang efektif dibandingkan klien Kafka asli, sehingga lebih cocok untuk aplikasi dengan throughput rendah hingga menengah.
   - Aplikasi yang membutuhkan throughput tinggi dan latensi rendah mungkin tidak cocok karena overhead HTTP dibandingkan dengan protokol asli Kafka.

7. **Kompatibilitas**
   - Kafka REST Proxy kompatibel dengan beberapa versi Kafka, mendukung kompatibilitas mundur (backward compatibility) dengan versi Kafka sebelumnya dalam versi yang didukung oleh Confluent.
   - Biasanya diperbarui sejalan dengan Kafka dan komponen lain di Confluent Platform, memastikan kompatibilitas di seluruh ekosistem.

Sebagai ringkasan, Kafka REST Proxy adalah alat yang berguna untuk mengekspos kapabilitas Kafka melalui HTTP, terutama di lingkungan di mana integrasi pustaka Kafka tidak memungkinkan. Namun, untuk skenario kinerja tinggi, klien Kafka asli umumnya lebih efisien.

--- 

# Produce dan Consume KafkaRest Tanpa Schema

Command yang diperlukan untuk produce message menggunakan KafkaRest adalah:

```
 curl -X POST -H "Content-Type: application/vnd.kafka.json.v2+json" \
     --data '{"records":[{"value": "Hello Kafka"}]}' \
     http://localhost:8082/topics/your_topic_name
```

Message bisa terlihat pada topik: your_topic_name seperti pada gambar

![image](https://github.com/user-attachments/assets/129b313c-090a-41cd-b916-80de9e808a37)

# Produce dan Consume menggunakan Schema

Berikut adalah command - command yang diperlukan untuk produce message di KafkaRest menggunakan Schema:

Pertama, saya ingin mendaftarkan schema di dalam schema registry dengan command:

```
curl -X POST -curl -X POST -H "Content-Type: application/vnd.schemaregistry.v1+json"      --data '{
       "schema": "{\"type\":\"record\",\"name\":\"YourRecordNamee\",\"fields\":[{\"name\":\"field1\",\"type\":\"string\"},{\"name\":\"field2\",\"type\":\"int\"}]}"
     }'      http://localhost:8081/subjects/your_topic_namee-value/versions
```

Bisa dilihat di dalam schema ini, saya mempunyai 2 tipe data yaitu pada field1;string sedangkan pada filed2;integer.

Jalankan command tersebut. Setelah dijalankan, maka terdapat Id Schema yang sudah didaftarkan. Id Schema yang saya daftarkan adalah 77. Id ini diperlukan untuk command selanjutnya dalam mem-produce sebuah message menggunakan KafkaRest

Berikut Commandnya:

```
 curl -X POST -H "Content-Type: application/vnd.kafka.avro.v2+json"      --data '{
       "value_schema_id": 77,
       "records": [
         {"value": {"field1": "example1", "field2": 100}},
         {"value": {"field1": "example2", "field2": 200}}
       ]
     }'      http://localhost:8082/topics/your_topic_namee
```

Kali ini saya membedakan topiknya menjadi your_topic_namee

Maka, jika sudah berhasil. Hasilnya akan terlihat seperti pada gambar:

![image](https://github.com/user-attachments/assets/d0bb9998-f85d-4883-b32f-94646462c0b2)

dengan schema yang sudah didaftarkan:

![image](https://github.com/user-attachments/assets/3e49d6ce-bb52-4a76-9080-9724fe721b1e)

