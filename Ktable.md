# KTable with Java

Kafka KTable adalah jenis aliran data di Kafka yang mewakili tampilan data seperti tabel. Berbeda dengan aliran biasa yang memproses setiap peristiwa saat terjadi, KTable menyimpan nilai TERBARU untuk setiap kEY. Ini berarti ia terus diperbarui saat data baru masuk. Misalnya, jika Anda melacak jumlah barang yang tersedia, KTable hanya akan menyimpan jumlah saat ini untuk setiap produk, memperbarui setiap kali stok berubah. Ini berguna untuk situasi di mana Anda ingin melacak status terbaru dari data Anda.

Sebelum memulai Class KTable, perlu dibuatnya Class TopicProducer seperti berikut:

```

import org.apache.kafka.clients.admin.Admin;
import org.apache.kafka.clients.admin.AdminClientConfig;
import org.apache.kafka.clients.admin.CreateTopicsResult;
import org.apache.kafka.clients.admin.NewTopic;
import org.apache.kafka.clients.producer.Callback;
import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.ProducerConfig;
import org.apache.kafka.clients.producer.ProducerRecord;
import org.apache.kafka.common.serialization.StringSerializer;
import org.apache.kafka.common.errors.TopicExistsException;

import java.io.IOException;
import java.util.List;
import java.util.Properties;
import java.util.concurrent.ExecutionException;
import java.util.stream.Collectors;

public class TopicLoader {
    public static void main(String[] args) throws IOException {
        runProducer();
    }

    public static void runProducer() throws IOException {
        Properties props = new Properties();
        props.put(AdminClientConfig.BOOTSTRAP_SERVERS_CONFIG, "server-eric:9092");

        // Producer configuration
        props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());
        props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class.getName());

        try (Admin adminClient = Admin.create(props);
             KafkaProducer<String, String> producer = new KafkaProducer<>(props)) {

            final String inputTopic = "inputKtable";
            final String outputTopic = "outputKtable";
            var topics = List.of(new NewTopic(inputTopic, 3, (short) 1), new NewTopic(outputTopic, 3, (short) 1));

            // Create topics asynchronously and wait for completion
            CreateTopicsResult result = adminClient.createTopics(topics);
            try {
                result.all().get(); // This will block until the topics are created
                System.out.println("Topics created successfully.");
            } catch (ExecutionException e) {
                if (e.getCause() instanceof TopicExistsException) {
                    System.out.println("Topics already exist, skipping creation.");
                } else {
                    throw new IOException("Error creating topics", e);
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt(); // Restore interrupted status
                throw new IOException("Error creating topics", e);
            }

            Callback callback = (metadata, exception) -> {
                if (exception != null) {
                    System.out.printf("Producing records encountered error %s %n", exception);
                } else {
                    System.out.printf("Record produced - offset - %d timestamp - %d %n", metadata.offset(), metadata.timestamp());
                }
            };

            var rawRecords = List.of(
                    "orderNumber-1001",
                    "bogus-1",
                    "bogus-2",
                    "orderNumber-8500");

            // Use Collectors.toList() for older Java versions
            var producerRecords = rawRecords.stream()
                    .map(r -> new ProducerRecord<String, String>(inputTopic, "order-key", r))
                    .collect(Collectors.toList());

            producerRecords.forEach(pr -> producer.send(pr, callback));
        }
    }
}
```

Kode ini menunjukkan cara membuat topik dan mengirim pesan ke kluster Kafka menggunakan Java. Pertama, kode ini menyiapkan properti, termasuk alamat server, dan menentukan serializer untuk key-value di Kafka. Dengan menggunakan klien ```Admin``` Kafka, dua topik ```inputKtable dan outputKtable``` dibuat secara asynchronous dengan 3 partisi dan faktor replikasi 1. Jika topik-topik tersebut sudah ada, kode ini akan menangani kesalahan dengan melompati pembuatan topik. Setelah itu, produser diinisialisasi untuk mengirim daftar pesan ```orderNumber-1001, bogus-1, bogus-2, orderNumber-8500``` ke topik ```inputKtable```. Untuk setiap pesan, callback didefinisikan untuk memeriksa apakah catatan berhasil dikirim atau jika terjadi kesalahan. Terakhir, produser mengirimkan pesan-pesan tersebut, dan proses berakhir.

Berikut adalah contoh Class KTable:

```
import org.apache.kafka.common.serialization.Serdes;
import org.apache.kafka.common.utils.Bytes;
import org.apache.kafka.streams.KafkaStreams;
import org.apache.kafka.streams.StreamsBuilder;
import org.apache.kafka.streams.kstream.Materialized;
import org.apache.kafka.streams.kstream.Produced;
import org.apache.kafka.streams.kstream.KTable;
import org.apache.kafka.streams.state.KeyValueStore;

import java.time.Duration;
import java.util.Properties;
import java.util.concurrent.CountDownLatch;

public class KTableApp {
    // Define topics and filters as constants or configuration
    private static final String inputTopic = "inputKtable";
    private static final String outputTopic = "outputKtable";
    private static final String orderNumberStart = "orderNumber";
    private static final String bogusStart = "bogus";

    public static void main(String[] args) {
        Properties props = new Properties();
        props.put(org.apache.kafka.streams.StreamsConfig.APPLICATION_ID_CONFIG, "ktable-app");
        props.put(org.apache.kafka.streams.StreamsConfig.BOOTSTRAP_SERVERS_CONFIG, "server-eric:9092");
        props.put(org.apache.kafka.streams.StreamsConfig.DEFAULT_KEY_SERDE_CLASS_CONFIG, Serdes.String().getClass().getName());
        props.put(org.apache.kafka.streams.StreamsConfig.DEFAULT_VALUE_SERDE_CLASS_CONFIG, Serdes.String().getClass().getName());

        StreamsBuilder builder = new StreamsBuilder();

        KTable<String, String> firstKTable = builder.table(inputTopic,
                Materialized.<String, String, KeyValueStore<Bytes, byte[]>>as("ktable-store")
                        .withKeySerde(Serdes.String())
                        .withValueSerde(Serdes.String()));

        firstKTable.filter((key, value) -> value.contains(orderNumberStart) || value.contains(bogusStart))
                .mapValues(value -> value.substring(value.indexOf("-") + 1))
                .filter((key, value) -> Long.parseLong(value) > 1)
                .toStream()
                .peek((key, value) -> System.out.println("Outgoing record - key " + key + " value " + value))
                .to(outputTopic, Produced.with(Serdes.String(), Serdes.String()));

        try (KafkaStreams kafkaStreams = new KafkaStreams(builder.build(), props)) {
            final CountDownLatch shutdownLatch = new CountDownLatch(1);

            Runtime.getRuntime().addShutdownHook(new Thread(() -> {
                kafkaStreams.close(Duration.ofSeconds(2));
                shutdownLatch.countDown();
            }));

            // Start the producer to populate the input topic
            TopicLoader.runProducer();

            try {
                kafkaStreams.start();
                shutdownLatch.await();
            } catch (Throwable e) {
                e.printStackTrace();
                System.exit(1);
            }
        } catch (Exception e) {
            e.printStackTrace();
            System.exit(1);
        }
    }
}
```

Kode Java ini menunjukkan penggunaan Kafka Streams untuk memproses dan memfilter data dalam sebuah topik Kafka menggunakan KTables. Pertama, kode ini menyiapkan properti seperti ID aplikasi dan alamat server Kafka. Sebuah ```StreamsBuilder``` digunakan untuk mendefinisikan logika pemrosesan stream. Sebuah KTable dibuat dari topik ```inputKtable``` dan disimpan di dalam sebuah penyimpanan status (state store). KTable ini memfilter catatan yang mengandung "orderNumber" atau "bogus", mengambil bagian angka setelah tanda hubung, dan memfilter lebih lanjut untuk hanya menyimpan catatan yang angkanya lebih besar dari 1. Catatan-catatan yang telah difilter ini dikonversi menjadi stream dan dikirim ke topik ```outputKtable```, sambil mencetak setiap catatan keluar ke konsol.

Instance ```KafkaStreams``` dijalankan untuk memulai pemrosesan stream. Sebuah CountDownLatch digunakan untuk menangani penghentian aplikasi dengan lancar. Selain itu, metode ```TopicLoader.runProducer()``` dipanggil untuk mengirim data uji ke topik input sebelum pemrosesan stream dimulai.

Berikut adalah contoh data yang akan ditest di dalam KTable:

![image](https://github.com/user-attachments/assets/979ffcda-e32f-4d13-8a32-322deb962ff7)

Dan jika sudah menjalankan Class KTable, maka yang kita dapatkan hanya hasil yang terakhir saja, seperti gambar berikut:

![image](https://github.com/user-attachments/assets/255f89b7-fbf4-45ca-8164-5d83d1385f35)

Bisa terlihat pada gambar terakhir, yang di tampilkan oleh Ktable hanya data yang terakhir yaitu: "order-key value 8500"







