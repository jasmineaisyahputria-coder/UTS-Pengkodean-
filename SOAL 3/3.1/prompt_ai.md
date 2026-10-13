# Log Prompt AI — Gemini Canvas

## Tujuan
Membuat form entri jurnal akuntansi (Voucher Entry) berbasis web 
dengan validasi otomatis zero-discrepancy (Total Debit = Total Kredit).

## Prompt yang Digunakan

"Buatkan saya form entri jurnal akuntansi (Voucher Entry) 
menggunakan HTML + Tailwind CSS dalam satu file HTML.

Fitur yang WAJIB ada:
1. Header form: nomor_voucher, tanggal (date picker), deskripsi
2. Tabel baris jurnal dengan kolom:
   - Akun (dropdown: 101 Kas, 102 Piutang, 401 Pendapatan, 501 Beban Gaji)
   - Debit (input number)
   - Kredit (input number)
   - Tombol Hapus per baris
3. Tombol '+ Tambah Baris' untuk menambah baris jurnal
4. Di bawah tabel, tampilkan:
   - Total Debit (live update)
   - Total Kredit (live update)
   - Selisih (Total Debit - Total Kredit)
   - Indikator warna: HIJAU kalau balance, MERAH kalau tidak
5. Tombol 'SUBMIT' hanya aktif jika:
   - Total Debit == Total Kredit
   - Total Debit > 0
   - Minimal 2 baris jurnal
6. Kalau tidak balance, tampilkan pesan merah 'TIDAK BALANCE'
7. Tampilan clean, profesional, dengan Tailwind CSS

Berikan kode lengkap dalam 1 file HTML siap pakai."

## Hasil dari Gemini

Gemini menghasilkan file `index.html` dengan fitur:
- Form header (Voucher, Tanggal, Deskripsi)
- Tabel dinamis baris jurnal dengan dropdown akun
- Live calculation total debit/kredit
- Indikator warna otomatis (hijau/merah)
- Tombol submit dengan validasi otomatis
- Desain Tailwind CSS yang clean

## Verifikasi Manual

Saya sudah uji form dengan 2 skenario:

### Skenario 1 — Kondisi BALANCE ✅
- VCH-001, tanggal 12/10/2026
- Baris 1: 101 Kas, Debit 500000, Kredit 0
- Baris 2: 401 Pendapatan, Debit 0, Kredit 500000
- Hasil: Total Debit = Total Kredit = Rp 500.000
- Indikator: HIJAU "BALANCE - Jurnal siap disubmit"
- Tombol Submit: AKTIF ✅

### Skenario 2 — Kondisi TIDAK BALANCE ❌
- Kredit baris 2 diubah jadi 300000
- Hasil: Total Debit 500000 ≠ Total Kredit 300000 (selisih 200000)
- Indikator: MERAH "TIDAK BALANCE - Periksa kembali"
- Tombol Submit: NONAKTIF (disabled) ❌

## Kesimpulan

Form berhasil mengimplementasikan asas zero-discrepancy: 
user TIDAK BISA submit jurnal yang tidak balance. Ini mencegah 
kesalahan pencatatan di level frontend sebelum data masuk ke database.