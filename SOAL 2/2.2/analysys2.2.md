# SOAL 2.2 — Penanganan Concurrency (Race Condition)

## A. Identifikasi Masalah: Overselling

**Skenario:**
Dua kasir (A dan B) memproses penjualan untuk item yang sama 
pada detik yang bersamaan. Stok di gudang hanya tersisa 1 unit.

**Alur bermasalah (tanpa proteksi):**

| Waktu | Kasir A | Kasir B | Stok |
|-------|---------|---------|------|
| t=0 | Cek stok → 1 unit ✅ | - | 1 |
| t=1 | - | Cek stok → 1 unit ✅ | 1 |
| t=2 | INSERT SALE 1 | - | 0 |
| t=3 | - | INSERT SALE 1 | **-1** ❌ |

**Kesimpulan:** Terjadi **phantom stock / overselling** — stok tercatat 
negatif padahal secara fisik barang sudah habis.

---

## B. Solusi Utama: SELECT ... FOR UPDATE

Teknik `SELECT ... FOR UPDATE` akan **MENGUNCI baris** item saat 
transaksi dimulai. Kasir kedua **HARUS MENUNGGU** sampai kasir pertama 
menyelesaikan transaksinya (COMMIT/ROLLBACK).

### Cuplikan Kode (Kasir A):

```sql
BEGIN;

-- KUNCI baris item ITM001
-- Kasir B akan ANTRI di statement ini sampai kasir A COMMIT
SELECT stock_qty 
FROM inventory_items 
WHERE item_id = 'ITM001' 
FOR UPDATE;

-- Cek stok di aplikasi, jika cukup → insert
INSERT INTO inventory_transactions (item_id, trx_type, qty)
VALUES ('ITM001', 'SALE', 1);

COMMIT;
-- Setelah COMMIT, kunci lepas → kasir B baru bisa lanjut