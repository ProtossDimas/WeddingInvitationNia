# Undangan Pernikahan — Nia & Dimas

Website undangan pernikahan digital untuk acara **1 Agustus 2026, Bandung**. Dibangun dengan HTML/CSS/JS statis dan Cloudflare Pages Functions sebagai backend ringan.

Fitur utama:

- Halaman undangan statis (cover, profil mempelai, galeri foto/video, cerita, lokasi, hadiah)
- Nama tamu personal otomatis tampil di cover undangan lewat parameter URL (`?to=...`)
- RSVP & ucapan tamu, tersimpan ke **Google Sheets** (bukan database terpisah — cukup buka spreadsheet untuk lihat semua respon)
- Notifikasi real-time setiap ada RSVP/ucapan baru, dikirim lewat **Fonnte (WhatsApp)** dan **Telegram Bot** sebagai jalur cadangan
- Galeri foto/video dengan lightbox (swipe & keyboard navigation, pinch-to-zoom untuk foto)
- Animasi scroll (fly-in effect) dan musik latar dengan autoplay

---

## 1. Struktur folder

```
WeddingInvitation/
├── index.html          ← halaman undangan (statis)
├── _headers             ← konfigurasi header Cloudflare Pages (caching, dsb.)
├── functions/            ← Cloudflare Pages Functions (backend: API RSVP/ucapan, integrasi Google Sheets, Fonnte, Telegram)
├── photos/               ← foto mempelai & galeri
├── videos/               ← video galeri
├── music/                ← musik latar undangan
└── README.md
```

---

## 2. Konfigurasi (Environment Variables)

Fitur backend membutuhkan beberapa environment variable / secret yang diset di **Cloudflare Pages → Settings → Environment variables**:

| Variable | Keterangan |
| --- | --- |
| Kredensial Google Service Account | Untuk akses Google Sheets API (baca/tulis data RSVP & daftar tamu) |
| ID Google Spreadsheet | Spreadsheet tujuan penyimpanan RSVP & ucapan |
| Token Fonnte | Untuk kirim notifikasi WhatsApp saat ada RSVP/ucapan baru |
| Token Bot Telegram & Chat ID | Untuk kirim notifikasi cadangan lewat Telegram |

> Catatan: saat membuat/menyalin token bot Telegram, cek ulang karakternya — huruf `I` (i besar) dan `l` (L kecil) sering terlihat mirip dan menyebabkan token gagal terautentikasi.

Setelah environment variable diset, lakukan re-deploy (misalnya lewat "Retry deployment" atau push commit baru) agar terbaca oleh Functions.

---

## 3. Deploy ke Cloudflare Pages

1. Push seluruh folder project ini ke repository GitHub.
2. Di Cloudflare dashboard → **Pages → Create a project → Connect to Git** → pilih repo ini.
3. Build settings:
   - **Framework preset:** None
   - **Build command:** (kosongkan, atau `npm install` jika ada `package.json`)
   - **Build output directory:** `/` (root)
4. Klik **Save and Deploy**.
5. Set semua environment variable pada bagian 2 di atas, lalu re-deploy.

---

## 4. Kustomisasi konten

Sebagian besar teks dan konfigurasi ada langsung di `index.html`. Bagian yang umum diubah:

- Nama mempelai, orang tua, dan tanggal/jam acara (termasuk target countdown)
- Alamat lokasi acara + link Google Maps
- Nomor rekening/hadiah digital
- Cerita pasangan di bagian cerita
- Font pada bagian cover (saat ini memakai kombinasi **Cormorant Garamond** dan **Allura**)

### Foto & video galeri

- Foto profil mempelai: `photos/nia.jpg` dan `photos/dimas.jpg` (fallback ke inisial jika belum diupload)
- Foto background cover: `photos/cover.jpg` (fallback ke gradient jika belum ada)
- Galeri foto/video: upload berurutan sesuai penamaan yang dikenali `index.html` (`foto1.jpg`, `foto2.jpg`, ..., `video1.mp4`, ...)
- Musik latar: taruh file di folder `music/`

---

## 5. Cara kerja teknis (ringkas)

- Semua request RSVP/ucapan diproses oleh Pages Functions di folder `functions/`.
- Data RSVP & ucapan ditulis ke Google Sheets lewat Google Sheets API (autentikasi service account).
- Setiap entri baru memicu notifikasi ke WhatsApp (Fonnte) dan Telegram sebagai jalur cadangan bila salah satu gagal.
- Header caching (`_headers`) diatur agar aset statis (foto/video) tetap bisa ter-update setiap ada deployment baru — hindari directive `immutable` pada aset yang masih sering diganti.

---

## 6. Troubleshooting

| Masalah | Kemungkinan sebab & solusi |
| --- | --- |
| RSVP/ucapan tidak masuk ke Google Sheets | Cek kredensial service account & ID spreadsheet di environment variables, lalu lihat log Functions di dashboard Cloudflare |
| Notifikasi WhatsApp/Telegram tidak terkirim | Cek token Fonnte/Telegram masih valid; untuk Telegram, pastikan tidak salah ketik karakter `I`/`l` pada token |
| Update foto tidak muncul di situs live | Cek konfigurasi `_headers`, pastikan tidak ada directive `immutable` yang membuat browser/CDN cache foto lama |
| Musik tidak autoplay di sebagian browser | Gunakan pola muted-autoplay lalu unmute setelah interaksi pertama dari pengguna |

---

## Desain

Mobile-first, lebar maksimal 520px (di-tengah di layar besar) — cocok dibagikan lewat WhatsApp.
