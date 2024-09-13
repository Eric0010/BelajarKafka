# Kafka Streams with Java

Kafka Streams adalah library untuk memproses data secara real-time yang disediakan oleh Apache Kafka. Dengan Kafka Streams, kita bisa membangun aplikasi yang dapat mengonsumsi, memproses, dan menghasilkan Semua data dari Kafka secara langsung.

Di dalam panduan [Apache Kafka Streams API](https://kafka.apache.org/38/documentation/streams/tutorial), terdapat 3 aplikasi Kafka Streams yaitu sebagai pada gambar:

![image](https://github.com/user-attachments/assets/62a900f4-96fd-4e2d-ae38-f03b6176cb1f)

Dari gambar diatas, saya aka memilih Class WordCount, Berikut adalah contoh Class WordCount:

```

import org.apache.kafka.common.serialization.Serdes;
import org.apache.kafka.common.utils.Bytes;
import org.apache.kafka.streams.KafkaStreams;
import org.apache.kafka.streams.StreamsBuilder;
import org.apache.kafka.streams.StreamsConfig;
import org.apache.kafka.streams.Topology;
import org.apache.kafka.streams.kstream.KStream;
import org.apache.kafka.streams.kstream.Materialized;
import org.apache.kafka.streams.kstream.Produced;
import org.apache.kafka.streams.state.KeyValueStore;

import java.util.Arrays;
import java.util.Properties;
import java.util.concurrent.CountDownLatch;

public class WordCountApplication {
    public static void main(String[] args) throws Exception{

        Properties props = new Properties();
        props.put(StreamsConfig.APPLICATION_ID_CONFIG, "streams-wordcount");
        props.put(StreamsConfig.BOOTSTRAP_SERVERS_CONFIG, "server-eric:9092");
        props.put(StreamsConfig.DEFAULT_KEY_SERDE_CLASS_CONFIG, Serdes.String().getClass());
        props.put(StreamsConfig.DEFAULT_VALUE_SERDE_CLASS_CONFIG, Serdes.String().getClass());

        final StreamsBuilder builder = new StreamsBuilder();

        KStream<String, String> source = builder.stream("streams-test-wordcount-input");
        source.flatMapValues(value -> Arrays.asList(value.toLowerCase().split("\\W+")))
                .groupBy((key, value) -> value)
                .count(Materialized.<String, Long, KeyValueStore<Bytes, byte[]>>as("counts-store"))
                .toStream()
                .to("streams-test-wordcount-output", Produced.with(Serdes.String(), Serdes.Long()));

        final Topology topology = builder.build();
        final KafkaStreams streams = new KafkaStreams(topology, props);
        final CountDownLatch latch = new CountDownLatch(1);

        Runtime.getRuntime().addShutdownHook(new Thread("streams-shutdown-hook") {
            @Override
            public void run() {
                streams.close();
                latch.countDown();
            }
        });

        try {
            streams.start();
            latch.await();
        } catch (Throwable e) {
            System.exit(1);
        }
        System.exit(0);


    }

}
```

Kode ini adalah aplikasi Kafka Streams yang menghitung jumlah kemunculan kata dalam aliran data teks. Aplikasi ini membaca dari topik input Kafka (streams-test-wordcount-input), memproses setiap pesan dengan memecahnya menjadi kata-kata, dan mengonversi setiap kata menjadi huruf kecil. Setelah itu, aplikasi mengelompokkan kata-kata dan menghitung jumlah kemunculannya menggunakan fitur pemrosesan stateful Kafka (melalui penyimpanan key-value yang disebut counts-store). Hasil dari perhitungan jumlah kata tersebut kemudian dikirimkan ke topik output Kafka (streams-test-wordcount-output). Aplikasi ini dirancang untuk berjalan hingga dihentikan, dengan menggunakan shutdown hook untuk menangani penutupan klien Kafka Streams secara bersih.
