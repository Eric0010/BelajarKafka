# Kafka Admin App with Java

Penggunaan utama dari KafkaAdminClient dalam Apache Kafka untuk melakukan manajemen dan inspeksi topik Kafka, broker, dan objek administratif lainnya. Ini biasanya digunakan untuk membuat, menghapus, memodifikasi topik, partisi, konfigurasi, serta mengelola cluster.

Berikut adalah Class AdminApp dengan Java:

```

import org.apache.kafka.clients.admin.AdminClient;
import org.apache.kafka.clients.admin.CreateTopicsResult;
import org.apache.kafka.clients.admin.DeleteTopicsResult;
import org.apache.kafka.clients.admin.NewTopic;

import java.util.Collections;
import java.util.Properties;
import java.util.concurrent.ExecutionException;

public class AdminApp {
    public static void main(String[] args) {
        // Kafka broker address configuration
        Properties config = new Properties();
        config.put("bootstrap.servers", "server-eric:9092");

        // Create AdminClient
        try (AdminClient adminClient = AdminClient.create(config)) {
            // Delete topic
            deleteTopic(adminClient, "topikbaru");
            // Recreate topic
            createTopic(adminClient, "topikbaru", 1, (short) 1);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    private static void deleteTopic(AdminClient adminClient, String topicName) {
        DeleteTopicsResult deleteTopicsResult = adminClient.deleteTopics(Collections.singletonList(topicName));
        try {
            deleteTopicsResult.all().get();
            System.out.println("Topik berikut " + topicName + " dihapus.");
        } catch (InterruptedException | ExecutionException e) {
            System.err.println("Gagal hapus topik " + topicName);
            e.printStackTrace();
        }
    }

    private static void createTopic(AdminClient adminClient, String topicName, int numPartitions, short replicationFactor) {
        NewTopic newTopic = new NewTopic(topicName, numPartitions, replicationFactor);
        CreateTopicsResult createTopicsResult = adminClient.createTopics(Collections.singletonList(newTopic));
        try {
            createTopicsResult.all().get();
            System.out.println("Topik " + topicName + " jadi dibuat.");
        } catch (InterruptedException | ExecutionException e) {
            System.err.println("Gagal buat topik " + topicName);
            e.printStackTrace();
        }
    }

}
```

Metode Utama (Main Method): Metode utama menginisialisasi konfigurasi broker Kafka menggunakan objek Properties dan menetapkan parameter bootstrap.servers untuk menunjuk ke server Kafka yang berjalan di ```server-eric:9092```. Setelah itu, ia membuat instansi dari AdminClient menggunakan konfigurasi ini. Setelah AdminClient dibuat, kode ini pertama-tama menghapus topik Kafka bernama ```topikbaru``` dengan memanggil metode deleteTopic, kemudian membuat ulang topik yang sama dengan memanggil metode ```createTopic```. Instansi AdminClient secara otomatis ditutup setelah digunakan melalui pernyataan ```try-with-resources```.

Metode deleteTopic: Metode ini menerima instansi AdminClient dan nama topik sebagai argumen, lalu mencoba menghapus topik yang disebutkan. Ia memanggil metode ```deleteTopics``` pada AdminClient untuk menghapus topik tersebut, dan menunggu hasilnya menggunakan metode all().get(), yang akan memblokir eksekusi sampai penghapusan selesai. Jika penghapusan berhasil, pesan sukses akan dicetak; jika gagal, pesan kesalahan akan dicetak dan pengecualian (exception) akan ditangani dan dicatat.

Berikut Tampilan jika topik berhasil dihapus:
![image](https://github.com/user-attachments/assets/aaaac6ff-92f9-4169-9851-bc77df7e62b7)

Metode createTopic: Metode ini membuat topik Kafka baru. Ia menerima AdminClient, nama topik, jumlah partisi, dan faktor replikasi sebagai parameter. Objek NewTopic dibuat dengan detail ini, dan metode ```createTopics``` pada AdminClient digunakan untuk mencoba membuat topik tersebut. Seperti pada ```deleteTopic```, metode ini menunggu hasilnya menggunakan ```all()``` ```.get()```. Jika topik berhasil dibuat, pesan sukses akan dicetak; jika tidak, kesalahan akan ditangani dan pengecualian dicatat.

Berikut Tampilan jika topik berhasil dibuat:
![image](https://github.com/user-attachments/assets/e68921bc-c190-4321-b237-84cbe1de587d)


Setiap metode dalam kode ini mengenkapsulasi operasi asinkron dan menangani kesalahan yang mungkin terjadi selama interaksi dengan Kafka, seperti masalah jaringan atau kesalahan pada broker Kafka.

