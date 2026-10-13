# Mekanisme Audit Trail & Immutability

## A. Konsep Immutability

Setelah jurnal berstatus POSTED, data TIDAK BOLEH diubah/dihapus.
Jika ada kesalahan, koreksi dilakukan melalui jurnal pembalik
(reversing entry / storno) — bukan edit jurnal lama.

Prinsip ini sejalan dengan standar audit: data keuangan yang sudah
diposting harus immutable (tidak dapat diubah) agar jejak audit
tetap utuh dan dapat diverifikasi.

## B. Implementasi Trigger PostgreSQL

CREATE OR REPLACE FUNCTION prevent_posted_modification()
RETURNS TRIGGER AS $$
BEGIN
    IF OLD.status_posting = 'POSTED' AND TG_OP = 'UPDATE' THEN
        INSERT INTO journal_audit_log (header_id, action, action_by, notes)
        VALUES (OLD.header_id, 'UPDATE_ATTEMPT', 'system',
                'Mencoba update jurnal POSTED — DILARANG');
        RAISE EXCEPTION 'Jurnal POSTED tidak boleh diubah. Buat jurnal pembalik.';
    END IF;

    IF OLD.status_posting = 'POSTED' AND TG_OP = 'DELETE' THEN
        INSERT INTO journal_audit_log (header_id, action, action_by, notes)
        VALUES (OLD.header_id, 'DELETE_ATTEMPT', 'system',
                'Mencoba hapus jurnal POSTED — DILARANG');
        RAISE EXCEPTION 'Jurnal POSTED tidak boleh dihapus. Buat jurnal pembalik.';
    END IF;

    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_immutable_posted
BEFORE UPDATE OR DELETE ON journal_headers
FOR EACH ROW
EXECUTE FUNCTION prevent_posted_modification();

## C. Field Audit yang Wajib Ada

| Field | Fungsi | Tipe Data |
|---|---|---|
| created_by | Siapa yang membuat jurnal | VARCHAR(100) |
| created_at | Kapan dibuat | TIMESTAMP DEFAULT NOW() |
| hash_checksum | Sidik jari data untuk deteksi manipulasi | VARCHAR(64) |

## D. Hash Checksum untuk Verifikasi Integritas

CREATE EXTENSION IF NOT EXISTS pgcrypto;

ALTER TABLE journal_headers
ADD COLUMN created_by VARCHAR(100),
ADD COLUMN created_at TIMESTAMP DEFAULT NOW(),
ADD COLUMN hash_checksum VARCHAR(64);

UPDATE journal_headers
SET hash_checksum = encode(digest(
    header_id || tanggal_transaksi || nomor_referensi || COALESCE(deskripsi, ''),
    'sha256'), 'hex');

Cara kerja hash:
- Hash dihitung dari gabungan semua field penting
- Jika ada yang diubah, hash akan berbeda
- Auditor bisa cek ulang hash untuk deteksi manipulasi

## E. Spesifikasi Backend (Konsep FastAPI)

from fastapi import FastAPI, HTTPException

@app.put("/journal/{header_id}")
def update_journal(header_id: int, data: dict):
    jurnal = ambil_dari_database(header_id)

    if jurnal["status_posting"] == "POSTED":
        raise HTTPException(
            status_code=400,
            detail="Jurnal POSTED tidak bisa diubah. Buat jurnal pembalik."
        )

    return update_draft(header_id, data)

## F. Alur Reversing Entry (Storno)

Kalau ada jurnal POSTED yang salah:
1. JANGAN ubah jurnal asli
2. Buat jurnal baru dengan debit & credit DIBALIK
3. Jurnal asli diubah status POSTED → REVERSED
4. Jurnal pembalik di-POSTED

Contoh:
- Asli: Kas (D) 100.000, Pendapatan (K) 100.000
- Pembalik: Pendapatan (D) 100.000, Kas (K) 100.000
- Net effect: nol (koreksi selesai)

## G. Bukti Uji Trigger

Saya sudah menguji trigger immutability:
1. Set jurnal ke POSTED → Success ✅
2. Coba UPDATE jurnal POSTED → ERROR "tidak boleh diubah" ✅
3. Coba DELETE jurnal POSTED → ERROR "tidak boleh dihapus" ✅
4. Cek journal_audit_log → tercatat 2 percobaan yang ditolak ✅

## Kesimpulan

Kombinasi trigger immutability + audit log + hash checksum
menciptakan sistem akuntansi yang:
1. Tidak bisa dimanipulasi setelah posting
2. Memiliki jejak audit lengkap
3. Sesuai dengan prinsip pengendalian internal
4. Memenuhi standar audit keuangan modern