# Verifikasi PSAK 2 dengan NotebookLM

## Sumber yang Digunakan
PDF PSAK 207 tentang Laporan Arus Kas + Riset Web NotebookLM

## Ringkasan Hasil Riset PSAK 2

### 1. Definisi Kas dan Setara Kas (PSAK 2 / PSAK 207)

**Kas (Cash):**
Meliputi saldo uang tunai di tangan (cash on hand) dan simpanan 
rekening giro di bank (demand deposits) yang dapat digunakan secara 
bebas tanpa pembatasan.

**Setara Kas (Cash Equivalents):**
Investasi jangka pendek yang sangat likuid, mudah dikonversi menjadi 
kas dalam jumlah yang dapat ditentukan, dan memiliki risiko perubahan 
nilai yang tidak signifikan. Biasanya berjangka waktu jatuh tempo 
3 bulan atau kurang sejak tanggal perolehannya (seperti deposito 
singkat atau deposito berjangka ≤ 90 hari).

### 2. Pengakuan Kas (Cash Recognition)

Kas diakui dalam laporan posisi keuangan (neraca) saat hak kepemilikan 
atau penguasaan atas kas secara legal berpindah ke entitas, yaitu ketika:
- Terjadi alir masuk/keluar kas secara aktual
- Dana simpanan di bank secara efektif siap digunakan/ditarik 
  tanpa hambatan

### 3. Persyaratan Dokumentasi / Bukti Transaksi Kas

**Bukti Utama:**
- Bukti Kas Keluar (BKK) / Voucher Pengeluaran Kas untuk transaksi pengeluaran
- Bukti Kas Masuk (BKM) untuk penerimaan

**Dokumen Pendukung Sah:**
- Faktur / Invoice
- Nota / Kuitansi
- Surat Permintaan Pembayaran (Purchase Requisition / PO)
- Laporan Penerimaan Barang (Receiving Report)
- Bukti Cek / Bilyet Giro atau Resi Transfer Bank

### 4. Prinsip Pengendalian Internal untuk Transaksi Kas Keluar

**a. Pemisahan Fungsi (Segregation of Duties):**
Fungsi otorisasi (manajemen/atasan), fungsi penyimpanan/kasir (custody), 
dan fungsi pencatatan akuntansi harus dipegang oleh personil yang berbeda.

**b. Otorisasi Berlapis:**
Setiap pengeluaran kas wajib diotorisasi oleh pejabat berwenang 
sesuai limitasi nominal.

**c. Sistem Voucher & Pembayaran Non-Tunai:**
Pengeluaran kas jumlah besar dilakukan melalui Cek/Bilyet Giro/Transfer 
Bank, sedangkan transaksi kecil menggunakan dana kas kecil (petty cash) 
dengan sistem dana tetap (imprest system).

**d. Dokumen Pre-numbered & Rekonsiliasi Periodik:**
Penggunaan formulir Bukti Kas Keluar bernomor urut tercetak 
(pre-numbered) serta pelaksanaan rekonsiliasi bank dan penghitungan 
kas fisik (cash opname) secara berkala.

## Kepatuhan Kode Form Entri Jurnal Saya

Kode form entri jurnal saya SUDAH SESUAI PSAK 2 karena:

| Persyaratan PSAK 2 | Implementasi di Form |
|---|---|
| Nomor voucher sebagai bukti | Field "No. Voucher" |
| Tanggal transaksi | Field "Tanggal" |
| Deskripsi transaksi | Field "Deskripsi/Keterangan" |
| Validasi keseimbangan | Total Debit = Total Kredit |
| Mencegah inkonsistensi | Tombol Submit hanya aktif jika balance |
| Asas double-entry | Setiap transaksi ada debit & kredit |
| Bukti kas keluar (BKK) | Nomor voucher unik per transaksi |

## Rekomendasi Tambahan untuk Sistem Produksi

Untuk kepatuhan PSAK 2 secara penuh, form perlu ditambahkan:
- Field `created_by` untuk audit trail (siapa input)
- Field `approved_by` untuk otorisasi kas keluar
- Log waktu submit (`created_at`)
- Hash checksum untuk verifikasi integritas data
- Upload bukti dokumen (invoice/kuitansi) sebagai attachment

## Kesimpulan

Form entri jurnal berbasis web saya sudah menerapkan prinsip dasar 
pengendalian internal kas sesuai PSAK 2. Validasi zero-discrepancy 
(Total Debit = Total Kredit) mencegah kesalahan pencatatan sejak awal, 
sebelum data masuk ke database. Ini sejalan dengan prinsip pengendalian 
internal: pencegahan (preventive control) lebih baik daripada deteksi 
setelah kejadian.