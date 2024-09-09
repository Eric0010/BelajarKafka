# How to deploy Kafka Services using SystemD

Langkah pertama yang harus dilakukan adalah:

1. Menambahkan User baru dengan nama Kafka dengan command. Menambahkan User baru lebih menjaga data seorang user dibandingkan menjalankan kafka di main terminal Ubuntu.

   ```
   sudo adduser kafka

   ```

![image](https://github.com/user-attachments/assets/7829d6dc-efba-4f4f-93d1-72cbd6565ae4)

   
2. User Kafka harus ditambahkan ke sudo group supaya mempunyai hak khusus dalam instalasi Kafka dengan menggunakan command berikut ini:

   ```
   sudo adduser kafka sudo

   ```

Hak khusus ini diperlukan untuk seorang user untuk menggunakan dependency dari kafka sendiri sebagai sudo-er.
   
3. Log in ke akun Kafka yang sudah dibuat

   ```
   su -l kafka

   ```
4. Lakukan instalasi Java development Kit

   ```
   sudo apt update

   ```
5. Ketik penambahan perintah untuk menginstall Java openjdk versi 11

   ```
   sudo apt install openjdk-11-jdk

   ```
6. Setelah selesai Download Java openjdk, segera download Kafka

   ```
   mkdir ~/downloads
   cd ~/downloads
   wget https://archive.apache.org/dist/kafka/3.4.0/kafka_2.12-3.4.0.tgz
   ```
7. Setelah download selesai, lakukan pengekstrakan file Kafka

   ```
   cd ~
   tar -xvzf ~/downloads/kafka_2.12-3.4.0.tgz
   ```
8. Supaya penaman file tidak berbelit, lakukan penamaan file ulang menjadi kafka

   ```
   mv kafka_2.12-3.4.0/ kafka/
   ```
9. Setelah semua selesai, saatnya start server Zookeeper dan Kafka

    ```
    ~/bin/zookeeper-server-start.sh  ~/kafka/config/zookeeper.properties
    ```
    ```
    ~/kafka/bin/kafka-server-start.sh  ~/kafka/config/server.properties
    ```

10. Untuk Efisiensi, beberapa unit file dari systemD harus dibuat dan juga menggunakan systemctl.

    Unit file untuk Zookeeper

    ```
    sudo nano /etc/systemd/system/zookeeper.service
    ```
    Sesampainya di dalam file, kita perlu mengisi file tersebut dengan file berikut ini:

    ```
    [Unit]
    Description=Apache Zookeeper Service
    Requires=network.target                 
    After=network.target                 
    
    [Service]
    Type=simple
    User=kafka
    ExecStart=/home/kafka/kafka/bin/zookeeper-server-start.sh /home/kafka/kafka/config/zookeeper.properties        
    ExecStop=/home/kafka/kafka/bin/zookeeper-server-stop.sh
    Restart=on-abnormal
    
    [Install]
    WantedBy=multi-user.target
    ```
    Unit file untuk Kafka
    ```
    sudo nano /etc/systemd/system/kafka.service
    ```
    Sesampainya di dalam file, kita perlu mengisi file tersebut dengan file berikut ini:

    ```
    [Unit]
    Description=Apache Kafka Service that requires zookeeper service
    Requires=zookeeper.service
    After=zookeeper.service
    
    [Service]
    Type=simple
    User=kafka
    ExecStart= /home/kafka/kafka/bin/kafka-server-start.sh /home/kafka/kafka/config/server.properties                            
    ExecStop=/home/kafka/kafka/bin/kafka-server-stop.sh
    Restart=on-abnormal
    
    [Install]
    WantedBy=multi-user.target
    ```
    Penjelasan singkat mengenai ```ExecStart``` yang berisikan ``` /home/kafka/kafka/bin/zookeeper-server-start.sh /home/kafka/kafka/config/zookeeper.properties``` dan ``` /home/kafka/kafka/bin/kafka-server-start.sh /home/kafka/kafka/config/server.properties ``` :

    Setiap ingin memulai sebuah server zookeeper dan kafka, perlu dibutuhkan sebuah file properties. Di dalam file properties ini, terdapat sebuah alamat port dan tujuan file direktori yang digunakan zookeeper dan kafka untuk memulai sebuah server dan menyimpan beberapa metadata.

    Setelah semua proses ini selesai, server Kafka sudah bisa dijalankan menggunakan command

    ```
    sudo systemctl start kafka
    ```
    Command berikut juga bisa dipakai, jika seorang user sudah login pada user yang sudah dibuat sebelumnya:
    ```
    systemctl start kafka
    ```
11. Lakukan pengecekan status pada server Kafka:

    ```
    sudo systemctl status kafka
    ```
    ![image](https://github.com/user-attachments/assets/f2a3fdef-70f8-432b-9cb3-5764cf6411e8)

       



