# Join di Flink

1. INNER JOIN (Join Biasa)

Definisi: Menggabungkan dua tabel berdasarkan kondisi tertentu. Hanya baris yang match di kedua tabel yang akan muncul.

Masalah di Streaming: Karena data terus masuk, Flink harus menyimpan semua data lama agar bisa mencocokkan data baru.

Contoh:
```
SELECT * 
FROM Orders 
INNER JOIN Product 
ON Orders.productId = Product.id;
```
Kalau Orders berisi:


order_id | productId
---------|-----------
1        | 101
2        | 102

Dan Product berisi:

id  | name
----|------
101 | Laptop
102 | Phone

Hasilnya:

order_id | productId | name
---------|-----------|------
1        | 101       | Laptop
2        | 102       | Phone

2. OUTER JOIN (LEFT, RIGHT, FULL)
   
Definisi: Sama seperti INNER JOIN, tapi kalau tidak ada pasangan yang cocok, masih bisa tampil.

Jenis-jenisnya:

LEFT JOIN → Semua data dari tabel kiri tetap ada, walaupun tidak ada pasangan di tabel kanan.

RIGHT JOIN → Semua data dari tabel kanan tetap ada, walaupun tidak ada pasangan di tabel kiri.

FULL OUTER JOIN → Semua data dari kedua tabel tetap ada, walaupun tidak ada pasangan yang cocok.

Contoh:

```
SELECT * 
FROM Orders 
LEFT JOIN Product 
ON Orders.product_id = Product.id;
```
Kalau Orders ada tambahan:

order_id | product_id
---------|-----------
1        | 101
2        | 102
3        | 103  <-- Tidak ada di Product

Maka hasilnya:

order_id | product_id | name
---------|-----------|------
1        | 101       | Laptop
2        | 102       | Phone
3        | 103       | NULL  <-- Tidak ada match

3. INTERVAL JOIN (Join dengan Waktu)
   
Definisi: Join dua tabel dengan syarat waktu.

Kondisi:
Harus ada satu kondisi kecocokan (ON Orders.id = Shipments.order_id).

Harus ada batas waktu (contoh: order harus dikirim dalam 4 jam).

Contoh:

```
SELECT * 
FROM Orders o, Shipments s 
WHERE o.id = s.order_id 
AND o.order_time BETWEEN s.ship_time - INTERVAL '4' HOUR AND s.ship_time;
```

Ini berarti order hanya cocok jika dikirim dalam waktu 4 jam setelah dibuat.

4. TEMPORAL JOIN (Join dengan Data Historis)
   
Definisi: Join tabel dengan versi historisnya berdasarkan waktu.

Kegunaan: Untuk menambahkan informasi dari tabel yang berubah-ubah (contoh: nilai tukar mata uang dari waktu ke waktu).

Contoh:

```
SELECT 
   order_id, price, orders.currency, conversion_rate, order_time
FROM orders
LEFT JOIN currency_rates FOR SYSTEM_TIME AS OF orders.order_time
ON orders.currency = currency_rates.currency;
```

Artinya, setiap orders akan dicocokkan dengan nilai tukar mata uang yang berlaku pada saat order dilakukan.

5. PROCESSING TIME TEMPORAL JOIN (Join dengan Waktu Proses)
   
Definisi: Sama seperti Temporal Join, tapi pakai processing time (waktu saat Flink menerima data).

Kunci Utama: Selalu pakai data terbaru yang tersedia.

Contoh:
```
SELECT 
  o.amount, r.rate, o.amount * r.rate 
FROM Orders o,
     LATERAL TABLE (Rates(o.proctime)) r
WHERE r.currency = o.currency;
```

Di sini, Orders digabung dengan nilai tukar mata uang terbaru dari Rates.

6. LOOKUP JOIN (Join dengan Database Eksternal)
   
Definisi: Join tabel dengan data dari database lain (misalnya MySQL).

Contoh:

```
SELECT o.order_id, o.total, c.country, c.zip
FROM Orders AS o
JOIN Customers FOR SYSTEM_TIME AS OF o.proc_time AS c
ON o.customer_id = c.id;
```

Orders akan dicocokkan dengan detail customer dari MySQL.

7. ARRAY EXPANSION (CROSS JOIN UNNEST)
   
Definisi: Kalau tabel punya kolom array, bisa dipecah jadi beberapa baris.

Contoh:

```
SELECT order_id, tag
FROM Orders 
CROSS JOIN UNNEST(tags) AS t (tag);
```

Kalau Orders ada:

order_id | tags
---------|-----------------
1        | ['tech', 'sale']
Maka hasilnya:

order_id | tag
---------|------
1        | tech
1        | sale

8. TABLE FUNCTION JOIN (LATERAL TABLE)
   
Definisi: Join dengan fungsi yang menghasilkan tabel baru.

Contoh:

```
SELECT order_id, res
FROM Orders,
LATERAL TABLE(table_func(order_id)) t(res);
```

Setiap baris di Orders akan diproses oleh fungsi table_func(order_id) yang mengembalikan beberapa baris hasil.

Ringkasan JOIN di Flink SQL

Join Type	                    | Fungsi
------------------------------|----------------------------------------------------------------
INNER JOIN	                  | Join biasa, hanya data yang cocok di kedua tabel yang diambil.
OUTER JOIN	                  | Tetap menyimpan baris dari satu atau kedua tabel walaupun tidak ada match.
INTERVAL JOIN	                | Join dengan batasan waktu (misalnya "match order dalam 4 jam").
TEMPORAL JOIN	                | Join dengan versi historis tabel (misalnya nilai tukar mata uang pada saat tertentu).
PROCESSING TIME TEMPORAL JOIN |	Mirip Temporal Join tapi pakai data terbaru.
LOOKUP JOIN	                  | Join dengan database eksternal seperti MySQL.
ARRAY EXPANSION	              | Pecah array menjadi beberapa baris.
TABLE FUNCTION JOIN	          | Join dengan hasil dari fungsi yang mengembalikan tabel.

Kapan Pakai Join yang Mana?

- Pakai INNER JOIN kalau dua tabel harus cocok.
- Pakai LEFT JOIN kalau mau simpan semua data dari tabel kiri, walaupun tidak ada pasangan.
- Pakai INTERVAL JOIN kalau butuh batasan waktu (contoh: order dikirim dalam 4 jam).
- Pakai TEMPORAL JOIN kalau ingin pakai data historis.
- Pakai LOOKUP JOIN kalau perlu ambil data dari MySQL atau database eksternal.
- Pakai ARRAY EXPANSION kalau ada list dalam kolom yang perlu dipecah.
