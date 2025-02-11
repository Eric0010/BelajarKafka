# Windowing TVF Flink

Windowing Table-Valued Functions (Windowing TVFs) - Batch & Streaming

Windows adalah konsep utama dalam pemrosesan stream yang tidak terbatas (infinite). Windows membagi stream menjadi "buckets" dengan ukuran terbatas (finite), di mana kita bisa menerapkan berbagai operasi komputasi.

Apache Flink menyediakan beberapa window table-valued functions (TVF) untuk membagi elemen dalam tabel menjadi beberapa window, termasuk:

- Tumble Windows (Fixed-size, tidak overlap)
- Hop Windows (Overlapping windows, setiap elemen bisa masuk ke lebih dari satu window)
- Cumulate Windows (Window dengan ukuran yang bertambah)
- Session Windows (Hanya didukung di streaming mode saat ini, berbasis aktivitas user/session)

Catatan: Satu elemen bisa masuk ke lebih dari satu window, tergantung jenis TVF yang digunakan. Misalnya, Hop Windows menciptakan window yang saling overlap, sehingga satu data bisa muncul di beberapa window.

Apa Itu Windowing TVF?
Windowing TVF adalah Polymorphic Table Functions (PTF) yang didefinisikan oleh Flink. PTF ini merupakan bagian dari standar SQL 2016, yaitu tipe khusus table-function yang bisa menerima table sebagai parameter.

Dengan TVF, kita bisa mengubah bentuk tabel sesuai kebutuhan. Karena TVF diperlakukan seperti tabel dalam SQL, pemanggilannya dilakukan di dalam FROM clause dari perintah SELECT.

Keuntungan Windowing TVF dibandingkan Grouped Window Functions:

✅ Lebih sesuai dengan standar SQL

✅ Mendukung lebih banyak operasi (misalnya Window TopN, Window Join)

✅ Lebih fleksibel daripada Grouped Window Functions, yang hanya mendukung Window Aggregation

# Jenis-Jenis Windowing TVF di Flink SQL

Flink menyediakan 4 built-in TVF untuk windowing:

1. TUMBLE (Tumbling Window)

- Membagi data ke dalam window dengan ukuran tetap (fixed size).
  
- Tidak ada overlap antar window.

Misalnya, jika kita menetapkan tumbling window 5 menit, maka Flink akan membuat window baru setiap 5 menit tanpa overlap.

2. HOP (Hopping Window)

- Mirip dengan Tumbling Window, tapi memungkinkan overlap antar window.

- Berguna untuk analisis dengan sliding time windows.

3. CUMULATE (Cumulative Window)

- Window dengan ukuran bertambah (cumulative).

- Digunakan untuk analisis yang membutuhkan aggregasi bertahap.

4. SESSION (Session Window)

- Berdasarkan aktivitas user/session, bukan waktu tetap.

- Hanya tersedia dalam streaming mode saat ini.

Struktur hasil dari TVF akan memiliki 3 kolom tambahan:

- window_start → Waktu mulai window
- window_end → Waktu akhir window
- window_time → Atribut waktu window (di batch mode = TIMESTAMP, di streaming mode = time attribute)

Catatan: Nilai window_time selalu window_end - 1ms.

----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# Contoh Penggunaan TUMBLING WINDOW

![image](https://github.com/user-attachments/assets/6574df09-c1f6-433c-b08f-64f213a67961)

Misalkan kita memiliki tabel Bid dengan struktur seperti berikut:

```
Flink SQL> DESC Bid;
```

|        name |                   type | null | key | extras |                       watermark |
|-------------|------------------------|------|-----|--------|---------------------------------|
|     bidtime | TIMESTAMP(3) *ROWTIME* | true |     |        | `bidtime` - INTERVAL '1' SECOND |
|       price |         DECIMAL(10, 2) | true |     |        |                                 |
|        item |                 STRING | true |     |        |                                 |


Dan datanya sebagai berikut:

```
Flink SQL> SELECT * FROM Bid;
```

|          bidtime | price | item |
|------------------|-------|------|
| 2020-04-15 08:05 |  4.00 | C    |
| 2020-04-15 08:07 |  2.00 | A    |
| 2020-04-15 08:09 |  5.00 | D    |
| 2020-04-15 08:11 |  3.00 | B    |
| 2020-04-15 08:13 |  1.00 | E    |
| 2020-04-15 08:17 |  6.00 | F    |

Membagi Data dengan TUMBLING WINDOW 10 Menit

Kita bisa menggunakan fungsi TUMBLE() untuk membagi data berdasarkan waktu (bidtime) dengan window 10 menit:

```
Flink SQL> SELECT * FROM TUMBLE(TABLE Bid, DESCRIPTOR(bidtime), INTERVAL '10' MINUTES);
```

Atau menggunakan parameter dengan nama yang lebih eksplisit:

```
Flink SQL> SELECT * FROM 
   TUMBLE(
     DATA => TABLE Bid,
     TIMECOL => DESCRIPTOR(bidtime),
     SIZE => INTERVAL '10' MINUTES);
```

Hasilnya:

|          bidtime | price | item |     window_start |       window_end |            window_time  |
|------------------|-------|------|------------------|------------------|-------------------------|
| 2020-04-15 08:05 |  4.00 | C    | 2020-04-15 08:00 | 2020-04-15 08:10 | 2020-04-15 08:09:59.999 |
| 2020-04-15 08:07 |  2.00 | A    | 2020-04-15 08:00 | 2020-04-15 08:10 | 2020-04-15 08:09:59.999 |
| 2020-04-15 08:09 |  5.00 | D    | 2020-04-15 08:00 | 2020-04-15 08:10 | 2020-04-15 08:09:59.999 |
| 2020-04-15 08:11 |  3.00 | B    | 2020-04-15 08:10 | 2020-04-15 08:20 | 2020-04-15 08:19:59.999 |
| 2020-04-15 08:13 |  1.00 | E    | 2020-04-15 08:10 | 2020-04-15 08:20 | 2020-04-15 08:19:59.999 |
| 2020-04-15 08:17 |  6.00 | F    | 2020-04-15 08:10 | 2020-04-15 08:20 | 2020-04-15 08:19:59.999 |


Menggunakan Window untuk Aggregation

Jika kita ingin menjumlahkan harga (price) dalam tiap window, kita bisa menggunakan GROUP BY window_start, window_end:

```
Flink SQL> SELECT window_start, window_end, SUM(price) AS total_price
  FROM TUMBLE(TABLE Bid, DESCRIPTOR(bidtime), INTERVAL '10' MINUTES)
  GROUP BY window_start, window_end;
```

Hasilnya:

|     window_start |       window_end | total_price |
|------------------|------------------|-------------|
| 2020-04-15 08:00 | 2020-04-15 08:10 |       11.00 |
| 2020-04-15 08:10 | 2020-04-15 08:20 |       10.00 |

--------------------------------------------------------------------------------------------------------------------------------------------------------------------

# HOP Window:

HOP adalah fungsi untuk membagi data ke dalam jendela waktu (window) yang bergeser (sliding). Mirip dengan fungsi TUMBLE, tetapi HOP memungkinkan tumpang tindih antara jendela jika slide lebih kecil dari ukuran jendela.

🏷 Konsep Dasar HOP
- Size (Ukuran Window) → Berapa lama periode window, misalnya 10 menit.
- Slide (Periode Pergeseran Window) → Seberapa sering window baru dimulai, misalnya setiap 5 menit.
- Overlap (Tumpang Tindih) → Jika slide lebih kecil dari size, satu data bisa masuk ke beberapa window.

![image](https://github.com/user-attachments/assets/3aa77bd4-91ba-4ea1-82f5-f3a5c7d342ed)

Misalnya, dengan size = 10 menit dan slide = 5 menit, setiap 5 menit akan dibuat window baru yang mencakup data dari 10 menit sebelumnya.

📌 Contoh

Misalkan ada data transaksi lelang (Bid) seperti ini:

bidtime	|         price	| item
|---------------|-------|-------------|
2020-04-15 08:05	| 4.00	| C
2020-04-15 08:07	| 2.00	| A
2020-04-15 08:09	| 5.00	| D
2020-04-15 08:11	| 3.00	| B
2020-04-15 08:13	| 1.00	| E
2020-04-15 08:17	| 6.00	| F

Jika kita menerapkan HOP dengan size 10 menit dan slide 5 menit, hasilnya:

bidtime	| price	| window_start | window_end
|-----------------|-------|-------|---------|
2020-04-15 08:05	| 4.00	| 08:00	| 08:10
2020-04-15 08:05	| 4.00	| 08:05	| 08:15
2020-04-15 08:07	| 2.00	| 08:00	| 08:10
2020-04-15 08:07	| 2.00	| 08:05	| 08:15
2020-04-15 08:09	| 5.00	| 08:00	| 08:10
2020-04-15 08:09	| 5.00	| 08:05	| 08:15
2020-04-15 08:11	| 3.00	| 08:05	| 08:15
2020-04-15 08:11	| 3.00	| 08:10	| 08:20

Setiap data bisa masuk ke beberapa window jika masih masuk dalam jangkauan size window.

📌 Format Penulisan SQL

Fungsi HOP bisa digunakan seperti ini:

```
SELECT * FROM HOP(TABLE Bid, DESCRIPTOR(bidtime), INTERVAL '5' MINUTES, INTERVAL '10' MINUTES);
```

Penjelasan:

- TABLE Bid → Sumber data
- DESCRIPTOR(bidtime) → Kolom waktu yang digunakan
- INTERVAL '5' MINUTES → Slide setiap 5 menit
- INTERVAL '10' MINUTES → Window berdurasi 10 menit

Bisa juga menggunakan named parameters:

```
SELECT * FROM 
    HOP(
      DATA => TABLE Bid,
      TIMECOL => DESCRIPTOR(bidtime),
      SLIDE => INTERVAL '5' MINUTES,
      SIZE => INTERVAL '10' MINUTES
    );
```
 
📊 Contoh Agregasi Data per Window

Misalnya, kita ingin menghitung total harga (SUM(price)) dalam tiap window:

```
SELECT window_start, window_end, SUM(price) AS total_price
FROM HOP(TABLE Bid, DESCRIPTOR(bidtime), INTERVAL '5' MINUTES, INTERVAL '10' MINUTES)
GROUP BY window_start, window_end;
```

Hasilnya:

window_start	| window_end	| total_price
|-----------------|---------------------|-------|
2020-04-15 08:00	| 2020-04-15 08:10	| 11.00
2020-04-15 08:05	| 2020-04-15 08:15	| 15.00
2020-04-15 08:10	| 2020-04-15 08:20	| 10.00
2020-04-15 08:15	| 2020-04-15 08:25	| 6.00

Kesimpulan

HOP digunakan untuk membuat jendela waktu (window) dengan ukuran tetap tetapi bergeser dalam interval tertentu.

------------------------------------------------------------------------------------------------------------------------------

# Memahami CUMULATE dalam Windowing Aggregation:

Fungsi CUMULATE berguna ketika kita ingin melakukan agregasi kumulatif seiring waktu dengan window yang tumpang tindih (overlapping). Ini berbeda dari TUMBLING window yang ukurannya tetap, karena CUMULATE memperbesar window dalam langkah-langkah tertentu.

Bagaimana CUMULATE Bekerja?

- Fixed Start Time (Waktu Mulai Tetap)

Setiap window selalu mulai dari waktu tetap dalam interval tertentu. Misalnya, 00:00 setiap hari.

- Expanding Window Size (Ukuran Window Bertambah)

Daripada memiliki ukuran window tetap, window bertambah secara bertahap sesuai step yang ditentukan (contoh: setiap 2 menit) hingga mencapai ukuran maksimum (contoh: 10 menit).

- Overlapping Windows (Window Tumpang Tindih)

Karena setiap window selalu mulai dari waktu yang sama tetapi terus bertambah ukurannya, maka satu event bisa masuk ke beberapa window yang berbeda.

![image](https://github.com/user-attachments/assets/9657c2f1-cc04-4a76-a365-4438548281a4)

Contoh SQL Query

Misalnya, kita punya Tabel Bid dan ingin membuat cumulative window dengan step 2 menit dan max window size 10 menit:

```
SELECT * FROM 
    CUMULATE(TABLE Bid, DESCRIPTOR(bidtime), INTERVAL '2' MINUTES, INTERVAL '10' MINUTES);
```

Step: INTERVAL '2' MINUTES → Window bertambah setiap 2 menit

Max Size: INTERVAL '10' MINUTES → Window bisa mencapai maksimal 10 menit

Setiap record bid (bidtime) akan masuk ke beberapa window, bertambah setiap 2 menit sampai maksimal 10 menit.

Bagaimana Window Terbentuk?

Misalkan kita punya event bid pada 08:05, maka event ini akan masuk ke beberapa window seperti berikut:

bidtime	price |	item	| window_start	| window_end
|-------------|-------|-------------|-------------
08:05	|4.00	| C	| 08:00	| 08:06
08:05	|4.00	| C	| 08:00	| 08:08
08:05	|4.00	| C	| 08:00	| 08:10

➡ Penjelasan:

Window mulai dari 08:00

Window pertama berakhir di 08:06 (karena step = 2 menit)

Window berikutnya berakhir di 08:08, lalu 08:10 (karena terus bertambah 2 menit sampai maksimal 10 menit)

Jadi, event yang sama bisa muncul di beberapa window yang berbeda.

- Menggunakan Agregasi dalam Cumulative Windows

Setelah window terbentuk, kita bisa melakukan agregasi pada window yang telah dibuat.

```
SELECT window_start, window_end, SUM(price) AS total_price
FROM CUMULATE(TABLE Bid, DESCRIPTOR(bidtime), INTERVAL '2' MINUTES, INTERVAL '10' MINUTES)
GROUP BY window_start, window_end;
```

👉 Query ini menghitung total price untuk setiap window kumulatif.

Hasil Output Setelah Agregasi:

window_start	| window_end	| total_price
|-------------|-------------|-------------|
08:00	| 08:06	| 4.00
08:00	|08:08	| 6.00
08:00	| 08:10	| 11.00
08:10	| 08:12	| 3.00

Kesimpulan (Key Takeaways):

✅ Use case (Kegunaan): Berguna untuk real-time dashboard yang menunjukkan total kumulatif sampai titik waktu tertentu.

✅ Perbedaan dengan Tumbling Window:

- TUMBLING window ukurannya tetap dan tidak overlap

- CUMULATE window memperbesar ukuran secara bertahap dan bisa overlap

✅ Fleksibilitas Agregasi:

Bisa melakukan analisis incremental tanpa harus menunggu window besar selesai.

Window bisa tumpang tindih, sehingga satu data bisa masuk ke beberapa window.

------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# SESSION Window

Apa Itu SESSION Window?

- SESSION window digunakan untuk mengelompokkan event berdasarkan aktivitas.
- Tidak seperti TUMBLING atau HOPPING window yang memiliki ukuran tetap, SESSION window tidak memiliki start dan end time yang tetap.
- Window akan tetap terbuka selama ada aktivitas, dan akan tertutup jika tidak ada aktivitas dalam waktu tertentu (gap inactivity).

Kapan SESSION Window Ditutup?
- Window akan tetap terbuka jika ada event baru yang terjadi dalam periode gap tertentu (contoh: 10 menit).
- Jika tidak ada event baru setelah 10 menit, maka window akan ditutup dan data akan dikirim ke downstream.
- Jika event baru muncul setelah window ditutup, event tersebut akan masuk ke window baru.

Bagaimana SESSION Window Bekerja?

- Menggunakan Kolom Waktu (Time Attribute Column)

Kolom waktu yang digunakan harus merupakan event time atau processing time.

Contohnya: bidtime dalam tabel Bid.

- Partitioning Data (Opsional)

Data bisa dipartisi berdasarkan key tertentu seperti item, sehingga window hanya berlaku dalam kelompok (partition) tertentu.

- Session Gap (Durasi Inactivity untuk Menutup Window)

Setiap event akan memulai atau memperpanjang sesi window selama tidak ada gap lebih dari durasi yang ditentukan (contoh: INTERVAL '5' MINUTES).

Jika gap lebih dari 5 menit, event baru akan masuk ke window baru.

Contoh Data Tabel

Kita memiliki tabel Bid dengan kolom bidtime, price, dan item:

bidtime |	price |	item
|-------|-------|--------|
2020-04-15 08:07	| 4.00	| A
2020-04-15 08:06	| 2.00	| A
2020-04-15 08:09	| 5.00	| B
2020-04-15 08:08	| 3.00	| A
2020-04-15 08:17	| 1.00	| B

Jika kita set SESSION dengan gap 5 menit, maka event yang terjadi kurang dari 5 menit dari event sebelumnya akan masuk ke window yang sama.

Jika tidak ada event baru setelah lebih dari 5 menit, window akan ditutup.

Menggunakan SESSION dalam SQL

1️⃣ SESSION Window dengan Partition Key

Jika kita ingin membuat SESSION window berdasarkan item, kita bisa pakai query berikut:

```
SELECT * 
FROM SESSION(
    TABLE Bid PARTITION BY item, 
    DESCRIPTOR(bidtime), 
    INTERVAL '5' MINUTES);
```

Output yang dihasilkan:

bidtime	| price |	item	| window_start	| window_end |	window_time
|-----------------|-------|---|------------|------------|--------------|
2020-04-15 08:07	| 4.00	| A	| 2020-04-15 08:06	| 2020-04-15 08:13	| 2020-04-15 08:12:59.999
2020-04-15 08:06	| 2.00	| A	| 2020-04-15 08:06	| 2020-04-15 08:13	| 2020-04-15 08:12:59.999
2020-04-15 08:08	| 3.00	| A	| 2020-04-15 08:06	| 2020-04-15 08:13	| 2020-04-15 08:12:59.999
2020-04-15 08:09	| 5.00	| B	| 2020-04-15 08:09	| 2020-04-15 08:14	| 2020-04-15 08:13:59.999
2020-04-15 08:17	| 1.00	| B	| 2020-04-15 08:17	| 2020-04-15 08:22	| 2020-04-15 08:21:59.999

💡 Penjelasan:

Semua item A yang terjadi dalam selisih kurang dari 5 menit masuk ke window yang sama (08:06 - 08:13).

Semua item B yang terjadi dalam selisih kurang dari 5 menit masuk ke window yang sama (08:09 - 08:14).

Event 2020-04-15 08:17 (B) memulai window baru karena sudah lebih dari 5 menit sejak event 2020-04-15 08:09 (B).

2️⃣ SESSION Window dengan Agregasi 

Kita bisa melakukan agregasi seperti SUM(price) untuk mengetahui total harga dalam setiap window:

```
SELECT window_start, window_end, item, SUM(price) AS total_price
FROM SESSION(
    TABLE Bid PARTITION BY item, 
    DESCRIPTOR(bidtime), 
    INTERVAL '5' MINUTES)
GROUP BY item, window_start, window_end;
```

Hasilnya:

window_start	| window_end	| item	| total_price
|-------------|-------------|-------|----------|
2020-04-15 08:06	| 2020-04-15 08:13	| A | 9.00
2020-04-15 08:09	| 2020-04-15 08:14	| B	| 5.00
2020-04-15 08:17	| 2020-04-15 08:22	| B	| 1.00

💡 Penjelasan:

Semua event A (harga 4.00, 2.00, 3.00) dikumpulkan dalam satu session window dengan total price = 9.00.

Semua event B (harga 5.00) berada dalam window tersendiri.

Event B (harga 1.00) berada di window lain karena terjadi setelah gap lebih dari 5 menit.

3️⃣ SESSION Window Tanpa Partition Key

Jika kita tidak ingin mempartisi berdasarkan item, kita cukup jalankan:

```
SELECT * 
FROM SESSION(TABLE Bid, DESCRIPTOR(bidtime), INTERVAL '5' MINUTES);
```

💡 Penjelasan:

Semua event diperlakukan dalam satu kelompok, tanpa memisahkan berdasarkan item.

Kesimpulan (Key Takeaways):

✅ SESSION Window Digunakan Untuk:

Mengelompokkan event berdasarkan aktivitas (tanpa start & end time tetap).

Menutup window hanya ketika ada gap inactivity yang lebih dari waktu tertentu.

✅ Perbedaan dengan TUMBLING & HOPPING Window:

- TUMBLING: Ukuran tetap, tidak overlap.
- HOPPING: Ukuran tetap, bisa overlap.
- SESSION: Ukuran dinamis, berdasarkan aktivitas (tidak overlap).

✅ Fleksibilitas Aggregation:

Bisa dipartisi berdasarkan key tertentu (PARTITION BY item).

Bisa melakukan agregasi (SUM, COUNT, AVG) dalam setiap session.

Cocok untuk analisis data streaming seperti menghitung agregasi dalam periode tertentu secara real-time.

----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

✅ Window Offset Bisa Dipakai di Mana Saja?

Offset ini bukan hanya untuk Tumbling Window, tapi juga bisa digunakan di Hopping Window dan Cumulative Window di Flink SQL.

Namun, tidak berlaku untuk Session Window karena window ini bergantung pada gap antar event, bukan interval tetap.

1️⃣ Tumbling Window (Window Tetap, Tidak Overlapping)

Offset menggeser batas waktu mulai dan akhir dari setiap window.

Example:

```
SELECT * FROM TUMBLE(
    DATA => TABLE Bid, 
    TIMECOL => DESCRIPTOR(bidtime), 
    SIZE => INTERVAL '10' MINUTES, 
    `OFFSET` => INTERVAL '2' MINUTES
);
```
Tanpa offset:

```
[00:00 - 00:10), [00:10 - 00:20), [00:20 - 00:30)...
```

Dengan OFFSET '2' MINUTES:

```
[00:02 - 00:12), [00:12 - 00:22), [00:22 - 00:32)...
```

👉 Jadi, kalau ada record di 00:05, window yang dipilih akan bergeser 2 menit lebih lambat dari normal.

2️⃣ Hopping Window (Window Tetap, Overlapping)

Offset menggeser waktu mulai semua window, tapi overlap tetap ada.

Example:

```
SELECT * FROM HOP(
    DATA => TABLE Bid, 
    TIMECOL => DESCRIPTOR(bidtime), 
    SLIDE => INTERVAL '5' MINUTES, 
    SIZE => INTERVAL '10' MINUTES, 
    `OFFSET` => INTERVAL '3' MINUTES
);
```

Tanpa offset (SLIDE 5 MIN, SIZE 10 MIN):

```
[00:00 - 00:10), [00:05 - 00:15), [00:10 - 00:20)...
```

Dengan OFFSET '3' MINUTES:
```
[00:03 - 00:13), [00:08 - 00:18), [00:13 - 00:23)...
```

👉 Jadi, semua window dimulai 3 menit lebih lambat dari normal, tapi tetap overlap setiap 5 menit.

3️⃣ Cumulative Window (Window Bertumbuh)

Offset menggeser titik awal dari window yang bertambah besar.

Example:

```
SELECT * FROM CUMULATE(
    DATA => TABLE Bid, 
    TIMECOL => DESCRIPTOR(bidtime), 
    STEP => INTERVAL '5' MINUTES, 
    SIZE => INTERVAL '20' MINUTES, 
    `OFFSET` => INTERVAL '4' MINUTES
);
```

Tanpa offset:

```
[00:00 - 00:05), [00:00 - 00:10), [00:00 - 00:20)...
```

Dengan OFFSET '4' MINUTES:

```
[00:04 - 00:09), [00:04 - 00:14), [00:04 - 00:24)...
```

👉 Jadi, window mulai dari jam 00:04 (bukan 00:00), lalu bertambah sesuai step.

4️⃣ Session Window (❌ Tidak Support Offset)

Session Window tidak bisa pakai offset karena window ini dibuat berdasarkan gap antar event, bukan interval tetap.

🎯 Kesimpulan

Window Type	Support Offset?	Berikut Penjelasannya:

| Window Type | Offset | Penjelasan |
|---------------|-------|-------------|
Tumbling Window	| ✅ Yes	| Offset menggeser batas window fix.
Hopping Window	| ✅ Yes	| Offset menggeser semua start window, overlap tetap ada.
Cumulative Window	| ✅ Yes	| Offset menggeser titik awal window bertumbuh.
Session Window	| ❌ No	| Tidak bisa pakai offset karena tergantung gap antar event.


