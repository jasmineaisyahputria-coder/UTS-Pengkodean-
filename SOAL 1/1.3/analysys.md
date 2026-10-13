# SOAL 1.3 — Analisis Audit & Keamanan Finansial

## Mengapa Model Header-Line Lebih Andal dari Flat Table?

### 1. Dari Sudut Pandang Audit Trail

Model Header-Line memisahkan informasi transaksi menjadi dua tingkat:
- **Header (journal_headers)**: menyimpan informasi global satu transaksi 
  (tanggal, nomor referensi, deskripsi, status posting).
- **Line (journal_lines)**: menyimpan detail akun-akun yang terlibat 
  beserta nilai debit/kredit.

**Keunggulan untuk audit:**
- Auditor dapat melacak satu transaksi utuh dengan cukup melihat 1 baris 
  di journal_headers, lalu seluruh rinciannya di journal_lines.
- Menghindari duplikasi informasi header di setiap baris, sehingga 
  mengurangi risiko inkonsistensi data (misalnya: 1 transaksi punya 
  2 tanggal berbeda karena typo).
- Memudahkan pengecekan double-entry: cukup query 
  `SUM(debit) = SUM(credit)` per header_id.

### 2. Dari Sudut Pandang 3NF (Third Normal Form)

Aturan 3NF: setiap atribut non-key harus bergantung penuh pada 
primary key tabelnya, bukan pada sebagian key atau atribut lain.

**Pada model Flat Table (satu tabel gabungan):**
- Kolom `tanggal_transaksi` hanya bergantung pada `header_id`, 
  BUKAN pada `line_id`. 
- Ini melanggar 2NF (partial dependency) dan otomatis melanggar 3NF 
  (transitive dependency) karena ada atribut yang bergantung pada 
  non-key (header_id → tanggal).

**Pada model Header-Line:**
- Semua atribut di `journal_headers` bergantung penuh pada `header_id`. ✅
- Semua atribut di `journal_lines` bergantung penuh pada `line_id`. ✅
- Relasi antar tabel dijaga oleh Foreign Key. ✅
- **Memenuhi 3NF dengan sempurna.**

### 3. Implikasi Keamanan Finansial

- **Mencegah anomali saldo tidak seimbang (unbalanced):** 
  Dengan struktur Header-Line, kita dapat menambahkan constraint/trigger 
  yang memastikan setiap header memiliki total debit = total credit 
  sebelum boleh di-posting.
- **Audit trail yang jelas:** Perubahan status (DRAFT → POSTED → REVERSED) 
  hanya perlu di-update di 1 baris header, tidak menyentuh ratusan baris line.
- **Isolasi risiko:** Jika satu baris line rusak/korup, header dan line lain 
  tetap utuh, sehingga pemulihan data lebih mudah.

## Kesimpulan

Model Header-Line adalah bentuk normalisasi yang tidak hanya memenuhi 
kaidah akademis (3NF) tetapi juga praktis untuk audit trail, verifikasi 
double-entry, dan keamanan finansial. Sebaliknya, flat table rentan 
terhadap anomali update, duplikasi data, dan sulit dilacak per transaksinya.