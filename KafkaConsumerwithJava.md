# Consumer with Java

# IMPORTANT NOTES

Berikut adalah langkah - langkah yang diperlukan sebelum membangun sebuah program di Java yang ingin dihubungkan ke Kafka:

1. Buat File Java dengan Archetype Maven di dalamnya.

2. Masukan Dependency berikut ke dalam file ```pom.xml```:
```
   <dependencies>
        <dependency>
            <groupId>org.apache.kafka</groupId>
            <artifactId>kafka-clients</artifactId>
            <version>3.6.2</version>
        </dependency>
    </dependencies>
```
3. Masukan Class Consumer berikut
```
      import org.apache.kafka.clients.consumer.ConsumerConfig;
      import org.apache.kafka.clients.consumer.ConsumerRecord;
      import org.apache.kafka.clients.consumer.ConsumerRecords;
      import org.apache.kafka.clients.consumer.KafkaConsumer;
      
      import java.time.Duration;
      import java.util.Arrays;
      import java.util.Properties;
      
      public class Consumer {
      
          public static void main(String[] args) {
              Properties props = new Properties();
              props.setProperty(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "server-eric:9092");
              props.setProperty(ConsumerConfig.GROUP_ID_CONFIG, "test");
              props.setProperty(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, "true");
              props.setProperty("auto.commit.interval.ms", "1000");
              props.setProperty(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringDeserializer");
              props.setProperty(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, "org.apache.kafka.common.serialization.StringDeserializer");
              KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);
              consumer.subscribe(Arrays.asList("firstTopic"));
              while (true) {
                  ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(100));
                  for (ConsumerRecord<String, String> record : records) {
                      System.out.println("Receive data: " + record.value());
                  }
              }
          }
      }
```
Pastikan kembali nama server dan topik sudah sama supaya tidak terjadi kesalahan saat mengconsume data.

Karena saya ingin melakukan konsumsi data terhadap file Class Producer yang sudah kita jalankan sebelumnya, maka saya harus menjalankan Class Producer terlebih dahulu, setelah itu barulah jalankan file Class Consumer.

Maka hasilnya bisa terlihat seperti berikut ini:

![image](https://github.com/user-attachments/assets/0334726f-68b6-4146-b218-b137f901939f)



