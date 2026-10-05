# SiAbs — Sistem Absensi Siswa

Sistem absensi siswa berbasis kartu (tap) untuk SMK Nusantara 1 Kota Tangerang.
Frontend berupa halaman web statis (HTML, CSS, JavaScript) yang di-host di GitHub Pages.

**Demo:** https://bsbe4727-wq.github.io/Siabs/
**Halaman admin:** https://bsbe4727-wq.github.io/Siabs/?page=admin

## Status

| Bagian | Status |
|---|---|
| Frontend halaman absensi | Selesai (masih data dummy / `localStorage`) |
| Frontend dashboard admin | Selesai (masih data statis) |
| Backend / API | Belum tersambung |
| Reader kartu (hardware) | Belum tersambung |

## Struktur

```
Siabs/
├── index.html   # Halaman absensi + dashboard admin
└── README.md
```

## Untuk Tim Backend

Frontend akan memanggil API di bawah ini. Semua endpoint di bawah adalah **usulan**, silakan diskusikan dan ubah bila perlu, lalu update dokumen ini supaya frontend dan backend tetap sinkron.

### Base URL

```
http://localhost:3000/api      # development
https://<domain-backend>/api   # production
```

Frontend menyimpan base URL di satu konstanta (`API`) sehingga mudah diganti.

### Ketentuan umum

- Format request dan response: **JSON** (`Content-Type: application/json`).
- Zona waktu: **WIB (Asia/Jakarta)**.
- Format tanggal: `DD/MM/YYYY`, format jam: `HH:mm:ss`.
- Endpoint admin wajib mengirim header `Authorization: Bearer <token>`.
- **CORS:** backend harus mengizinkan origin `https://bsbe4727-wq.github.io` (dan `http://localhost:*` untuk development).
- Format error seragam:

```json
{ "message": "Pesan error yang bisa ditampilkan ke pengguna" }
```

### Autentikasi

| Method | Endpoint | Keterangan |
|---|---|---|
| POST | `/login` | Login admin, mengembalikan token |
| POST | `/logout` | Mengakhiri sesi |

Request `POST /login`:

```json
{ "username": "admin", "password": "******" }
```

Response `200`:

```json
{ "token": "eyJhbGciOi...", "nama": "Admin" }
```

Response `401`: username atau password salah.

### Absensi (tap kartu)

| Method | Endpoint | Auth | Keterangan |
|---|---|---|---|
| POST | `/absensi` | Tidak | Catat kehadiran dari tap kartu / input NIS |
| GET | `/absensi` | Admin | Riwayat absensi |
| DELETE | `/absensi` | Admin | Hapus riwayat (opsional) |

Request `POST /absensi`:

```json
{ "nis": "1001" }
```

Response `200`:

```json
{
  "nama": "Muhammad Naufal",
  "nis": "1001",
  "kelas": "X PPLG",
  "status": "Hadir",
  "tgl": "05/10/2026",
  "jam": "08:15:20"
}
```

| Status code | Arti | Perilaku frontend |
|---|---|---|
| 404 | Kartu / NIS tidak terdaftar | Tampilkan pesan error |
| 429 | Absen berulang (masih cooldown) | Indikator reader jadi merah |
| 500 | Server error | Tampilkan pesan gagal |

Untuk 429, sebaiknya sertakan lama tunggu agar frontend sinkron:

```json
{ "message": "Tunggu sebentar", "retry_after": 20 }
```

> **Cooldown harus ditegakkan di backend.** Cooldown 20 detik di frontend hanya efek tampilan dan bisa dilewati dengan refresh halaman.

Query untuk `GET /absensi`:

| Parameter | Contoh | Keterangan |
|---|---|---|
| `limit` | `10` | Jumlah data |
| `q` | `naufal` | Cari nama / NIS |
| `dari`, `sampai` | `2026-10-01` | Rentang tanggal |
| `kelas` | `X PPLG` | Filter kelas |
| `status` | `Hadir` | Filter status |

### Dashboard admin

| Method | Endpoint | Keterangan |
|---|---|---|
| GET | `/admin/statistik` | Angka di kartu dashboard |
| GET | `/status` | Status reader, database, sinkronisasi |

Response `GET /admin/statistik`:

```json
{
  "total_siswa": 128,
  "hadir_hari_ini": 4,
  "terlambat": 7,
  "tidak_hadir": 5
}
```

Response `GET /status`:

```json
{ "reader": "online", "database": "aktif", "sinkronisasi": "siap" }
```

### Data siswa (menu Kelola Data Siswa)

| Method | Endpoint | Keterangan |
|---|---|---|
| GET | `/siswa` | Daftar siswa (pagination + `?q=`) |
| POST | `/siswa` | Tambah siswa (termasuk UID kartu) |
| PUT | `/siswa/:id` | Ubah data siswa |
| DELETE | `/siswa/:id` | Hapus siswa |

Contoh body `POST /siswa`:

```json
{ "nama": "Muhammad Naufal", "nis": "1001", "kelas": "X PPLG", "uid_kartu": "04A1B2C3" }
```

### Rekap dan export

| Method | Endpoint | Keterangan |
|---|---|---|
| GET | `/rekap` | Rekap presensi (filter seperti `GET /absensi`) |
| GET | `/rekap/export?format=xlsx` | Unduh file rekap (`xlsx` atau `csv`) |

## Pertanyaan yang Perlu Disepakati

- [ ] Jam batas **terlambat** dan kriteria **tidak hadir**, serta siapa yang menghitung (backend).
- [ ] Reader kartu: USB (mode keyboard), ESP32/Arduino kirim langsung ke backend, atau Web NFC?
- [ ] Jika reader mengirim langsung ke backend, frontend perlu notifikasi real-time (polling, WebSocket, atau SSE).
- [ ] Apakah ada role selain admin (guru, wali kelas)?
- [ ] Masa berlaku token dan perilaku saat token habis (frontend akan logout saat menerima 401).

## Menjalankan Frontend Secara Lokal

Cukup buka `index.html` di browser. Untuk pengujian dengan backend lokal, pastikan CORS sudah diizinkan.

## Catatan Keamanan

- Jangan menyimpan password, token, atau data asli siswa di repo ini (repo bersifat publik).
- Login admin harus divalidasi oleh backend. Pengecekan di JavaScript saja tidak aman karena kodenya bisa dibaca siapa pun.
