# Install CP via Package Manager with all services (zookeeper, kafka, schema registry, kafka connect, ksqldb, kafka rest, Control center) with security enabled (SASL_SSL)

Karena saya menggunakan Ubuntu, maka yang akan saya jelaskan adalah Instalasi CP via Ubuntu. Untuk Instalasi Device yang lain bisa dilihat dalam web berikut:

https://docs.confluent.io/platform/current/installation/installing_cp/zip-tar.html 

1. Untuk melakukan Instalasi CP. berikut adalah command yang harus dijalankan:

   ```
   curl -O https://packages.confluent.io/archive/7.7/confluent-7.7.1.zip
   ```
   NOTE:
   1.pastikan anda berada di direktori user anda. Jangan melakukan instalasi di dalam sebuah folder yang sudah mempunyai fasilitas yang berjalan di sebuah port.
   2. Matikan semua fasilitas yang sedang berjalan di Linux.
   
   Lalu, yang perlu dilakukan adalah unzip file tersebut dengan command:

   ```
   unzip confluent-7.7.1.zip
   ```

   Setelah Unzip, saya perlu melakukan export Plugin-path confluent yang sudah saya install dengan menjalankan

   ```
   nano ~/.bashrc
   ```

  Di baris paling bawah, saya menambahkan command berikut ini:

  ```
  export CONFLUENT_HOME=~/confluent-7.7.0
  export PATH=$PATH:$CONFLUENT_HOME/bin
  ```
  lalu ketik 
  
  ```
  source ~/.bashrc
  ```

  Sesudah menambahkan Plugin-path Confluent, barulah saya akan melakukan penginstallan Confluent dengan command:

  ```
  sudo apt-get update && \
  sudo apt-get install confluent-platform && \
  sudo apt-get install confluent-security
  ```
  Sesudah selesai install, saya mencoba run fasilitas yang diebrikan confluent dari yang paling kecil dengan Command:

  ```
  systemctl start confluent-<fasilitas yang ingin dijalankan>
  ```
  Berikut adalah urutan fasilitas dari yang paling kecil atau yang pertama kali harus dijalankan.

  zookeeper, kafka, schema registry, kafka connect, ksqldb, kafka rest, Control center

  Untuk mengetahui fasilitas tersebut berjalan tau tidak, bisa menggunakan command:
  
  ```
  systemctl status confluent-<fasilitas yang ingin dijalankan>
  ```
  Jika Failed, bisa melihat dan merubah ExecStart dari fasilitas yang failed tersebut, di direktori berikut

  ```
  /home/kafka/confluent-7.7.0/lib/systemd/system
  ```

  2. Jika semua fasilitas sudah berjalan, cobalah untuk pergi ke browser dan ketik address
  ```
  localhost:9021
  ```
  saya menggunakan   ```server-eric:9021```, karena saya melakukan prubahan di Listener Confluent Control Center.

  Jika berhasil, maka tampilan C3 adalah seperti berikut:
  
  ![image](https://github.com/user-attachments/assets/a998e86b-bbd0-4397-9abb-ea108bd84da8)

  3. Buat dan Hapus Topic dengan C3

     Untuk membuat sebuah topic di dalam confluent sudah dimudahkan. Dibandingkan kafka, confluent sudah menyediakan sebuah tombol di menu topic:
     
     ![add_topic](https://github.com/user-attachments/assets/587e1aa6-326c-47eb-802a-3adc48f160c7)

     Sesudah saya menekan tombol Add topic, saya akan membuat nama topik dan partisi yang saya butuhkan di dalam topik tersebut.

     ![image](https://github.com/user-attachments/assets/f6c6f400-46c6-4263-964c-011aa4f15786)

     Setelah selesai, saya hanya tinggal menekan create by default dan topik sudah berhasil dibuat.

     ![image](https://github.com/user-attachments/assets/490e48f0-3827-46e6-a2d3-16303c449e21)

     Selanjutnya adalah bagaimana caranya untuk menghapus topik.

     saya sudah berhasil membuat topic_75. Maka yang perlu saya lakukan untuk menghapus topik adalah hanya menekan topic_75 sendiri dan ke menu configuration.

     ![image](https://github.com/user-attachments/assets/6d2e4e9e-4ca2-4387-83ac-e2ca50665e60)

     Dan tekan Delete Topic dan lakukan pengetikan ulang nama topik, maka topik akan berhasil dihapus.

  4. Produce dan Consume via C3

     Untuk Memproduce dan Consume sebuah message pada C3 sungguh lah mudah. Bisa dilihat pada gambar, anda bisa menekan "Produce a new message to this topic".

     Masukan Pesan yang anda ingin masukan. Lalu, tekan produce. Maka pesan akan langsung bisa dilihat pada C3.

     ![image](https://github.com/user-attachments/assets/e669b0c3-857e-4a3f-bac8-c510d7f398da)


     ![image](https://github.com/user-attachments/assets/a506c5b6-a149-4d29-b312-545f6731b4bc)








