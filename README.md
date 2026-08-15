# Budget Restock Gayanara Terbatas: Produk Terlaris vs Dead Stock

Tim buying Gayanara menghadapi keterbatasan budget untuk melakukan restock persediaan produk. Saya menganalisis data transaksi (tanpa order cancelled / returned) menggunakan SQL untuk mengidentifikasi 10 produk terlaris, brand andalan, dan produk dead stock. Dari analisis, ditemukan bahwa salah satu produk terlaris (PRD029 (Shirt Slim Fit Tropika Style)) telah menghasilkan pendapatan tertinggi tetapi stoknya tersisa 26 unit, brand Riang Apparel, NusaBrand, dan Cendana Co telah memberikan kontribusi pendapatan terbesar, sedangkan produk PRD137 (Jaket Varsity SandangIndo) yang stoknya tinggi tetapi penjualan rendah. Rekomendasi saya adalah segera melakukan pemesanan ulang untuk produk PRD029, prioritaskan budget pemesanan ulang untuk 3 brand andalan, dan memberikan diskon untuk produk dead stock PRD137.

**1. Masalah:**

Tim buying Gayanara menghadapi keterbatasan budget untuk melakukan restock (pemesanan ulang) persediaan produk. Produk mana yang paling laku dan brand mana yang menjadi andalan, supaya  tidak salah alokasi budget (membeli produk yang kurang laku)?
- Identifikasi 10 produk terlaris berdasarkan unit terjual (membuang transaksi dengan status cancelled / returned)
- Hitung kontribusi pendapatan (revenue) per brand untuk menentukan brand andalan
- Deteksi produk dead stock (memiliki stok tinggi namun penjualan sangat rendah) untuk rekomendasi diskon

**2. Proses:**

_Top 10 Best-seller_
```sql
SELECT
	p.product_id,
	p.name AS nama_produk,
	p.brand, p.stock AS sisa_stok, 
	SUM(oi.quantity) AS total_unit_terjual, 
	SUM(oi.subtotal_idr) AS total_pendapatan
FROM order_items oi
JOIN orders o ON o.order_id = oi.order_id 
JOIN products p ON p.product_id = oi.product_id 
WHERE o.order_status NOT IN ('cancelled', 'returned')
GROUP BY p.product_id, p.name, p.brand, p.stock 
ORDER BY total_unit_terjual DESC
LIMIT 10;
```
|product_id|nama_produk                    |brand        |sisa_stok|total_unit_terjual|total_pendapatan|
|:---:|:---|:---|:---:|:---:|:---:|
|PRD299    |T-Shirt Graphic BajuKita       |BajuKita     |      168|                33|        13167000|
|PRD213    |Topi Baseball SandangIndo      |SandangIndo  |       46|                32|         6368000|
|PRD221    |Jacket Coach NusaBrand         |NusaBrand    |       34|                32|         3808000|
|PRD280    |Kemeja Oxford Riang Apparel    |Riang Apparel|      150|                32|         7328000|
|PRD006    |Celana Kulot Tropika Style     |Tropika Style|      193|                31|         3689000|
|PRD043    |Kaos Oversize Tropika Style    |Tropika Style|       97|                31|         4619000|
|PRD178    |Dompet Kulit Pesona Indo       |Pesona Indo  |      110|                31|         7719000|
|PRD029    |Shirt Slim Fit Tropika Style   |Tropika Style|       26|                30|        14970000|
|PRD081    |Celana Jogger Kanvas Lokal     |Kanvas Lokal |       48|                30|         6870000|
|PRD189    |Ikat Pinggang Kulit SandangIndo|SandangIndo  |       88|                30|        11970000|

_Pendapatan per Brand_ - https://datastudio.google.com/reporting/e9960068-8e87-4eb6-afc7-6be1b0b381cb

```sql
SELECT
	p.brand,
	SUM(oi.quantity) AS total_unit_terjual, 
	SUM(oi.subtotal_idr) AS total_pendapatan
FROM order_items oi
JOIN orders o ON o.order_id = oi.order_id 
JOIN products p ON p.product_id = oi.product_id 
WHERE o.order_status NOT IN ('cancelled', 'returned')
GROUP BY p.brand
ORDER BY total_pendapatan DESC;
```
|brand        |total_unit_terjual|total_pendapatan|
|:---|:---:|:---:|
Riang Apparel|               615|       160435000|
NusaBrand    |               508|       157492000|
Cendana Co   |               647|       149033000|
SandangIndo  |               537|       134123000|
Tropika Style|               543|       132297000|
Senja Wear   |               526|       126624000|
BajuKita     |               474|       119316000|
Pesona Indo  |               506|       113594000|
Kanvas Lokal |               585|       107055000|
Ratu Mode    |               501|        96799000|

_Produk Dead Stock_

```sql
SELECT
	p.product_id,
	p.name AS nama_produk,
	p.brand,
	p.stock AS sisa_stok, 
	SUM(oi.quantity) AS total_unit_terjual
FROM order_items oi
JOIN orders o ON o.order_id = oi.order_id 
JOIN products p ON p.product_id = oi.product_id 
WHERE o.order_status NOT IN ('cancelled', 'returned')
GROUP BY p.product_id, p.name, p.brand, p.stock 
HAVING sisa_stok > 200 AND total_unit_terjual <= 11
ORDER BY sisa_stok DESC, total_unit_terjual ASC;
```

product_id|nama_produk                  |brand      |sisa_stok|total_unit_terjual|
|:---|:---|:---|:---:|:---:|
PRD137    |Jaket Varsity SandangIndo    |SandangIndo|      407|                 6|
PRD151    |Celana Jeans Slim Senja Wear |Senja Wear |      379|                11|
PRD278    |Shirt Slim Fit Senja Wear    |Senja Wear |      271|                 9|
PRD128    |Dress Mini Casual Pesona Indo|Pesona Indo|      253|                11|
PRD142    |Celana Jogger Ratu Mode      |Ratu Mode  |      239|                 5|
PRD002    |Kaos Oversize BajuKita       |BajuKita   |      239|                11|

**3. Insight**

- Dari tabel Top 10 Best-seller, terlihat PRD029 (Shirt Slim Fit Tropika Style) adalah produk dengan pendapatan tertinggi, yaitu Rp 14.970.000, namun stoknya tersisa 26 unit. Juga produk PRD221 (Jacket Coach NusaBrand) yang sisa stoknya 34 unit.
- Dari tabel pendapatan per brand, terlihat brand Riang Apparel, NusaBrand, dan Cendana Co telah memberikan konstribusi pendapatan yang terbesar.
- Dari tabel produk dead stock, terlihat PRD137 (Jaket Varsity SandangIndo) adalah produk yang memiliki stok tinggi namun penjualan rendah.

**4. Rekomendasi:**

- Segera pemesanan ulang  untuk produk PRD029 (Shirt Slim Fit Tropika Style) yang stoknya tersisa 26 unit.
- Prioritaskan budget pemesanan ulang untuk 3 brand andalan: Riang Apparel, NusaBrand, dan Cendana Co.
- Perlu diberikan diskon untuk produk PRD137 (Jaket Varsity SandangIndo) yang memiliki stok tinggi tetapi penjualan rendah.
