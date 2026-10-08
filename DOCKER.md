# Install AGC Tutorial DE di Nusapod (Docker)

Proyek ini **terpisah** dari AGC Drachin: repo sendiri, image sendiri, container sendiri, database sendiri.
Situs drama Anda tidak terpengaruh apa pun.

Alur: **file script → repo GitHub baru → image Docker otomatis → container baru di Nusapod**.

---

## Langkah 1 — Buat repo GitHub baru

1. Di GitHub klik **New repository** → nama: `agc-tutorial-de` → **Create repository**.
   (Jangan upload ke repo `agc-drachin`.)
2. Upload **seluruh isi** folder `agc-tutorial-de`, termasuk folder tersembunyi **`.github`** dan file **`.dockerignore`**.
   - Di Windows: aktifkan *View → Hidden items* agar terlihat.
   - Di Mac: tekan `Cmd + Shift + .` di Finder.
3. **Commit changes**.

## Langkah 2 — Image Docker otomatis

1. Tab **Actions** → workflow **Build Docker image** berjalan (±3 menit) dan harus hijau ✅.
   Workflow ini juga menguji image (versi Jerman, Inggris, root, port kustom) sebelum dinyatakan lulus.
2. Image Anda: `ghcr.io/USERNAME-GITHUB/agc-tutorial-de:latest`
3. Profil GitHub → **Packages** → `agc-tutorial-de` → **Package settings** → **Change visibility** → **Public**.

## Langkah 3 — Container baru di Nusapod

| Pengaturan | Isi |
|---|---|
| Image | `ghcr.io/USERNAME-GITHUB/agc-tutorial-de:latest` |
| Port container | `8080` |
| **Persistent volume** | **baru**, mount ke `/var/www/html/storage` (jangan pakai volume situs drama) |
| RAM | 512 MB disarankan |
| Domain | domain Jerman Anda + SSL |

### Environment variables

Nilai default sudah diatur untuk situs tutorial software Jerman. **Yang wajib diisi hanya 5:**

```
SITE_URL=https://domain-anda.de
YOUTUBE_API_KEY=...            (boleh sama dengan situs drama; kuota dibagi)
CONTACT_EMAIL=...
LEGAL_NAME=Nama lengkap / nama usaha
LEGAL_ADDRESS=Jalan ... | Kota | Negara
```

Opsional (default sudah terisi):

| Nama | Default | Keterangan |
|---|---|---|
| `SITE_NAME` | `Software Tutorials` | ganti sesuai nama brand/domain |
| `SITE_NICHE` | `Business-Software-Tutorials` | topik situs (muncul di judul, meta, pencarian) |
| `SITE_ITEM` | `Tutorial` | sebutan untuk satu video |
| `YOUTUBE_CHANNELS` | `@Lexware,UCjioq2QuGj7tR0zcV6eTjZQ,@DATEV,@IONOSDeutschland,@MicrosoftDE` | channel resmi, dipisah koma |
| `YOUTUBE_EXCLUDE` | `werbung,werbespot,trailer,teaser,#shorts` | buang video yang judulnya memuat kata ini |
| `YOUTUBE_INCLUDE` | kosong | hanya ambil judul yang memuat salah satu kata ini |
| `MIN_DURATION` | `90` | detik; video lebih pendek disembunyikan |
| `AI_ENABLED`, `AI_PROVIDER`, `AI_API_KEY`, `AI_MODEL` | mati | deskripsi unik berbahasa Jerman |
| `ADS_HEAD`, `ADS_TOP`, `ADS_BELOW_PLAYER`, `ADS_IN_LIST`, `ADS_SIDEBAR`, `ADS_FOOTER` | kosong | kode iklan |
| `PORT` | `8080` | isi hanya jika hosting mewajibkan port lain |

## Langkah 4 — Cek hasil

- **Logs** container: sekitar 20 detik setelah start muncul `Channel diperbarui: Lexware …` lalu `Selesai. Unit API terpakai …`.
- Buka domain → tutorial, playlist, Impressum, Datenschutz sudah tampil.
- Bila ada console: `php /var/www/html/cron/status.php` untuk ringkasan.

## Wajib sebelum daftar AdSense (situs Jerman/EU)

1. **Impressum** berisi nama & alamat asli (`LEGAL_NAME`, `LEGAL_ADDRESS`).
2. Teks **Datenschutz** bawaan hanya template, bukan nasihat hukum. Cek ulang dengan generator Datenschutz Jerman.
3. Di AdSense, aktifkan pesan persetujuan cookie (CMP) di menu **Privasi & pesan** untuk pengunjung EU.
4. Domain sendiri (`.de` atau `.com`), lalu daftarkan sitemap di Google Search Console.

## Update nanti

Ubah file di repo `agc-tutorial-de` → Actions membuat & menguji image baru → **Redeploy** container tutorial di Nusapod.

## Masalah umum

| Gejala | Solusi |
|---|---|
| Nusapod gagal pull image | Visibility package belum **Public**, atau nama image tidak huruf kecil semua |
| Situs kosong | Lihat logs; biasanya `YOUTUBE_API_KEY` salah / API belum di-enable |
| Isi hilang setelah redeploy | Volume belum di-mount ke `/var/www/html/storage` |
| Banyak video iklan ikut masuk | Tambah kata di `YOUTUBE_EXCLUDE` |
| 502 / tidak bisa diakses | Port container di panel harus `8080` (atau sama dengan `PORT`) |
