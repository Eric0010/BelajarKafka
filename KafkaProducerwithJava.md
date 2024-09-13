# Kafka Producer with Java

# IMPORTANT NOTES

Berikut adalah langkah - langkah yang diperlukan sebelum membangun sebuah program di Java yang ingin dihubungkan ke Kafka:

1. Buat File Java dengan Archetype Maven di dalamnya.

1. Masukan Dependency berikut ke dalam file ```pom.xml```:
```
   <dependencies>
        <dependency>
            <groupId>org.apache.kafka</groupId>
            <artifactId>kafka_2.13</artifactId>
            <version>3.6.0</version>
        </dependency>
        <dependency>
            <groupId>org.apache.kafka</groupId>
            <artifactId>kafka-clients</artifactId>
            <version>3.6.2</version>
        </dependency>
    </dependencies>
```
2. Masukan Class Produser Kafka berikut ke dlaam projek:
   ```
   
    import org.apache.kafka.clients.producer.KafkaProducer;
    import org.apache.kafka.clients.producer.ProducerConfig;
    import org.apache.kafka.clients.producer.ProducerRecord;
    import org.apache.kafka.common.serialization.StringSerializer;
    
    import java.util.Properties;
    
    public class Producer{

    public static void main(String[] args) {
        Properties properties = new Properties();
        properties.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "server-eric:9092");
        properties.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
        properties.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());

        KafkaProducer<String, String> producer = new KafkaProducer<String, String>(properties);

        for(int i = 0; i < 20; i++) {
            ProducerRecord<String, String> record = new ProducerRecord<>("firstTopic", "Data ke " + i);
            producer.send(record);
        }

        producer.close();
    }
    }
    ```
Pastikan server dan topik di dalam ```Class Producer``` ini sama pada saat pemanggilan command consumer untuk mengconsume data.

   Di dalam Code berikut ini saya ingin melakukan pengiriman pesan 1 sampai pesan ke 20
   
  ```
       for(int i = 0; i < 20; i++) {
                ProducerRecord<String, String> record = new ProducerRecord<>("firstTopic", "Data ke " + i);
                producer.send(record);
            }
            producer.close();
  ```
   
Lalu setelah melakukan produksi pesan, saya kan menangkap pesan tersebut di kafka melalui command ```~/kafka/bin/kafka-console-consumer.sh --bootstrap-server server-eric:9092 --topic firstTopic --from-beginning```
   ![image](https://github.com/user-attachments/assets/01330d9a-a4e3-4464-85ac-daebcf298c20)
   
Jika sudah berhasil, Consumer di Kafka akan menangkap pesan yang dikirim Produser sebagai berikut.
