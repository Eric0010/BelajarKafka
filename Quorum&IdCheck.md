# 1. How to check Zookeeper Quorum

Berikut adalah caara melakukan pengcekan Zookeeper Quorum:

1. Masuk ke file zookeeper.properties dengan command

   ```
   sudo nano ~/kafka/config/zookeeper.properties
   ```
  
3. Lalu masukan command 4 whitelist zookeeper command supaya command 4 huruf bisa digunakan
   ```
    4lw.commands.whitelist=*
   ```
2. stop dan start kembali zookeeper service.

   langkah ini diperlukan untuk menjamin 4 whitelist zookeeper berjalan dengan baik

   ```
    ./bin/zookeeper-server-stop.sh  ~/kafka/config/zookeeper.properties
   ```
   ```
    ./bin/zookeeper-server-start.sh  ~/kafka/config/zookeeper.properties
   ```
4. buka terminal baru, terus jalanin command:
    ```
    echo stat | nc localhost 2181
    ```
    atau ada cara lain yaitu dengan command

   ```
   telnet localhost 2181
   ```
   lalu setelah command tersebut berjalan ketik command ```stat```
   
#2. How to check Kafka cluster Id

1. Langkah pertama yang harus dijalankan adalah ketik command

   ```
   ls
   ```

2. Lalu jalankan command
   
   ``` 
   ls kafka/bin/zookeeper-shell.sh
   ```
3.  Setelah itu, jalankan command
   ```
   ./kafka/bin/zookeeper-shell.sh localhost:2181
   ```
4. Setelah command tersebut berjalan, ketik command
   ```
   ls /
   ```
5. Terakhir tambahkan command
   ```
   get /cluster/id
   ```








   
