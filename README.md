# Jadwal TV – layar jadwal perkuliahan

Aplikasi web untuk menampilkan status ruang kuliah secara langsung di layar TV. Data diisi di Google Sheets, disajikan oleh Google Apps Script, dan halaman tampilannya di-host di GitHub Pages. Tidak butuh server sendiri.

```
Google Sheets  →  Apps Script (Web App, JSON)  →  index.html di GitHub Pages  →  browser TV
   (admin)          cache 60 detik                   ambil ulang tiap 5 menit
```

## Isi folder

| Berkas | Fungsi |
|---|---|
| `index.html` | Halaman tampilan TV (semua CSS dan JS di satu berkas) |
| `sw.js` | Service worker agar halaman tetap terbuka saat internet putus |
| `Code.gs` | Backend Apps Script yang membaca Google Sheets |

## Pemasangan

### 1. Siapkan Google Sheets dan Apps Script

1. Buat Google Sheets baru, misalnya "Jadwal TV Gedung F4".
2. Buka **Ekstensi → Apps Script**, hapus isi bawaan, tempel seluruh isi `Code.gs`, lalu simpan.
3. Pilih fungsi `siapkan` di toolbar lalu klik **Jalankan**. Izinkan akses saat diminta.
   Lembar Pengaturan, Ruang, Jeda, Kegiatan, Perubahan, dan Pengumuman akan terbentuk beserta data contoh hari Kamis.
4. Klik **Terapkan → Deployment baru → jenis Aplikasi web**.
   - Jalankan sebagai: **Saya**
   - Yang memiliki akses: **Siapa saja**
5. Salin URL yang berakhiran `/exec`. Coba buka di browser: harus muncul teks JSON.

### 2. Hubungkan halaman tampilan

Buka `index.html`, cari bagian `CONFIG`, lalu isi:

```js
SUMBER_DATA: "https://script.google.com/macros/s/XXXXXXXX/exec",
```

### 3. Host di GitHub Pages

1. Buat repositori, unggah `index.html` dan `sw.js`.
2. **Settings → Pages → Deploy from a branch**, pilih `main` dan folder `/ (root)`.
3. Halaman tersedia di `https://<akun>.github.io/<repo>/`.

### 4. Pasang di TV

Buka alamat berikut di browser TV atau perangkat yang tersambung ke TV:

```
https://<akun>.github.io/<repo>/?lokasi=F4
```

Sarannya gunakan mode kiosk, misalnya Chrome dengan `--kiosk` di mini PC atau Raspberry Pi, atau aplikasi *kiosk browser* di Android TV box. Atur juga jam perangkat ke sinkron otomatis dengan zona WIB.

## Parameter URL

| Parameter | Contoh | Fungsi |
|---|---|---|
| `lokasi` | `?lokasi=F6` | Gedung tempat TV dipasang. Ruang gedung ini tampil paling atas dengan tanda "Anda di sini". |
| `gedung` | `?gedung=F4,F6` | Hanya menampilkan gedung tertentu. |
| `rotasi` | `?rotasi=0` | Mematikan pergantian otomatis ke linimasa. |
| `tema` | `?tema=gelap` / `?tema=terang` | Memaksa tema warna. |
| `hari` | `?hari=Kamis` | Menampilkan jadwal hari tertentu (untuk uji coba). |
| `jam` | `?jam=09:30` | Mensimulasikan jam (untuk uji coba). |
| `demo` | `?demo=1` | Menampilkan panel simulasi. Tombol **D** juga menampilkan atau menyembunyikannya. |

Satu sheet bisa dipakai beberapa TV sekaligus. Misalnya TV di Gedung F6 cukup dibuka dengan `?lokasi=F6`.

## Cara mengisi data

Setiap judul kolom punya catatan. Arahkan kursor ke sel judul untuk membacanya.

**Ruang.** Satu baris per ruang. Urutan baris menentukan urutan tampil. Isi `aktif` dengan *Tidak* untuk menyembunyikan ruang tanpa menghapusnya.

**Jeda.** Gunakan `hari = Semua` untuk jeda harian. Jika satu hari punya jadwal istirahat berbeda, tulis baris lengkap khusus hari itu. Contohnya Jumat dengan istirahat salat Jumat. Pada hari tersebut, hanya baris miliknya yang dipakai. Kolom `besar = Ya` menandai istirahat: kuliah dipotong otomatis dan banner berwarna biru.

**Kegiatan.** Satu baris per pertemuan mingguan. Tulis blok utuh, tidak perlu per slot 50 menit.
- `13:20` sampai `16:20` otomatis dipotong istirahat Ashar.
- Baris per slot juga boleh. Baris dengan ruang, kode, kelas, mata kuliah, dan dosen yang sama akan digabung.
- Jarak lebih dari 60 menit dianggap pertemuan terpisah.
- `matkul_tv` (opsional) dipakai untuk nama mata kuliah yang terlalu panjang.
- `warna`: oranye, biru, emas, iup, magenta, ungu, hijau, toska, merah, coklat, abu, atau kode hex.

**Perubahan.** Hanya berlaku pada tanggal yang diisi.

| Jenis | Yang diisi | Hasil di layar |
|---|---|---|
| Batal | tanggal, kode, dan bila perlu kelas, ruang, mulai, keterangan | Baris berstatus merah "Dibatalkan", judul dicoret, dan diumumkan di teks berjalan |
| Pengganti | tanggal dan isian lengkap seperti Kegiatan | Kelas muncul dengan label "Pengganti" dan diumumkan |
| Libur | tanggal, keterangan, ruang dikosongkan | Seluruh layar menampilkan libur |
| Libur | tanggal, ruang, keterangan | Hanya ruang itu ditutup |

Untuk **pindah ruang**, buat dua baris: *Batal* di ruang lama dan *Pengganti* di ruang baru.

**Pengumuman.** Teks berjalan di bawah layar. Kolom `mulai` dan `sampai` opsional untuk membatasi periode tampil.

**Pengaturan.** Mengatur lokasi layar, judul, jam tampil, warna gedung (`warna_gedung_F4`, `warna_gedung_F6`, dan seterusnya), serta pesan bawaan.

## Kapan perubahan muncul di TV?

Setiap edit di sheet langsung menghapus cache Apps Script. TV mengambil data tiap 5 menit, jadi perubahan tampil paling lambat sekitar 5 menit. Menu **Jadwal TV → Terapkan perubahan sekarang** tersedia jika cache perlu dihapus manual.

Mengubah isi sheet **tidak** memerlukan deploy ulang. Deploy ulang hanya diperlukan jika `Code.gs` diubah: **Terapkan → Kelola deployment → edit → Versi baru**. Dengan cara ini URL tetap sama.

## Perilaku otomatis

- Status per ruang dihitung dari jam perangkat: berlangsung, segera dimulai (15 menit sebelum), jeda, kosong, selesai, dibatalkan, atau libur.
- Papan status tampil 40 detik, linimasa 15 detik. Lebih dari 12 ruang otomatis dibagi menjadi beberapa halaman.
- Data terakhir disimpan di browser. Jika internet putus, layar tetap menampilkan jadwal dan di header muncul keterangan "Offline".
- Jadwal berganti otomatis saat tengah malam. Halaman dimuat ulang tiap pukul 03.00.
- Posisi tampilan bergeser beberapa piksel tiap 2 menit untuk mencegah *burn-in*.
- Di luar jam tampil, layar hanya menampilkan jam.
