# JRF Race Card · versi dua file (v32)

Repository ini hanya membutuhkan:

```text
index.html
README.md
```

Seluruh CSS, JavaScript, font dan PNG sudah tertanam di `index.html`. Tidak ada folder assets, package.json, npm, build, atau vercel.json yang perlu di-upload. File HTML sekitar 23 MB karena seluruh aset disertakan.

## Upload ke GitHub

1. Ekstrak ZIP ini terlebih dahulu.
2. Upload **index.html dan README.md hasil ekstrak** ke root repository GitHub. ZIP ini berisi tepat dua file langsung pada level teratas.
3. Pastikan `index.html` benar-benar file HTML, bukan ZIP yang diganti namanya. Jangan upload file ZIP sebagai halaman website.
4. Untuk beralih dari paket versi 29, repository tujuan cukup berisi dua file di atas. Konfigurasi build lama seperti `package.json` dan `vercel.json` tidak dipakai pada versi ini.

## Deploy ke Vercel tanpa build

Import repository GitHub tersebut. Gunakan:

| Pengaturan | Nilai |
| --- | --- |
| Framework Preset | Other |
| Root Directory | `./` / root repository |
| Build Command | Kosong; aktifkan Override lalu hapus command lama |
| Output Directory | `.` (root repository), bukan `dist` |
| Install Command | Kosong; hapus override install lama jika ada |

Klik Deploy atau Redeploy. Jika proyek Vercel sebelumnya memakai `npm run build` dan `dist`, ubah pengaturan tersebut sebelum Redeploy. Preview dari GitHub harus menampilkan website, bukan teks berawalan `PK`.

[Dokumentasi Vercel: website statis dan Skip Build Step](https://vercel.com/docs/builds/configure-a-build).

## Preview lokal

Buka `index.html` di browser. Tombol Create Race Card berpindah ke `#builder` dalam file yang sama. Gambar dan font tidak memerlukan file terpisah. Untuk pengujian native share gunakan URL HTTPS dari Vercel; dukungannya mengikuti browser/perangkat.

## Analytics: pola yang sama dengan Pocari

GA4 sudah terisi dan aktif untuk domain produksi:

```js
measurementId: 'G-W6291G9928',
productionHost: 'jakartarunningfestival2026.vercel.app',
enabled: true
```

Versi 32 memakai pola Google tag langsung dari implementasi Pocari: tag di bagian head, pageview otomatis dari `gtag('config', ...)`, serta lima nama custom event yang sama. Tidak ada pageview manual atau event funnel versi 31. ID, hostname, tampilan, input, aset dan rendering race card tetap dipertahankan.

| Event | Dikirim ketika |
| --- | --- |
| `page_view` | Halaman website dimuat; otomatis oleh Google tag. |
| `race_card_create` | Masuk ke builder dari landing, atau membuka `#builder` langsung. Masuk kembali dari landing menghitung aksi baru; pemanggilan router berulang pada tampilan yang sama tidak menambah event. |
| `race_card_download` | PNG race card berhasil dibuat dan browser diminta memulai unduhan. |
| `template_download` | PNG template transparan berhasil dibuat dan browser diminta memulai unduhan. |
| `race_card_share` | Native share selesai tanpa dibatalkan. |
| `race_card_save` | Browser tidak mendukung share file, lalu kartu diunduh sebagai fallback tombol Share. |

Seperti Pocari, semua custom event menggunakan `race_format` dan `race_category`. JRF mengirim `race_format: 'on-ground'`; kategori dikirim sebagai `5K`, `10K`, `HM`, `FM`, atau `63K` (label di website tetap 63.3K). Tidak ada event ganda `race_card_create_start` atau `race_card_template_download` di versi ini. Data historis dari nama event lama tetap mengikuti nama lamanya di GA4.

### Pengaturan GA4 setelah replace

1. Buka **Admin → Data collection and modification → Data streams → stream Web JRF**. Pastikan Measurement ID-nya `G-W6291G9928`.
2. Aktifkan **Enhanced measurement** untuk pengukuran otomatis seperti `scroll`, sama seperti laporan Pocari. Bila sebelumnya dimatikan mengikuti README versi 31, ubah sekarang.
3. Klik roda gigi Enhanced measurement. Pada **Page views → Show advanced settings**, matikan **Page changes based on browser history events** agar pindah `#builder` tidak dihitung sebagai pageview tambahan. Pageview saat halaman dimuat tetap otomatis. Pengukuran `scroll` dapat tetap aktif; `Form interactions` bersifat opsional dan hanya berlaku bila GA mendeteksi elemen form.
4. Pada **Admin → Data display → Custom definitions**, tambahkan custom dimensions dengan scope **Event** untuk `race_format` dan `race_category` jika ingin membuat laporan berdasarkan format/kategori. Pendaftaran dimension tidak diperlukan hanya untuk menerima dan menghitung nama event.
5. Opsional: tandai `race_card_download` dan `race_card_share` sebagai key events pada **Admin → Data display → Events → Recent events** setelah namanya muncul.

Kode website tidak bisa mengubah kartu Home **Suggested for you** secara langsung; GA4 memilih kartu itu berdasarkan aktivitas penggunaan Analytics. Daftar custom event dan penerimaan datanya tetap perlu diperiksa melalui Realtime/DebugView.

### Verifikasi setelah deployment

1. Replace `index.html` dan `README.md` di root repository GitHub, commit, lalu tunggu deployment produksi Vercel berstatus **Ready**. Tidak diperlukan environment variable, build step, atau tag Google kedua.
2. Buka [URL debug JRF](https://jakartarunningfestival2026.vercel.app/?ga_debug=1#builder) di browser. Di GA4 property JRF 2026, buka **Admin → Data display → DebugView**.
3. Masuk builder, lengkapi data, lalu unduh template atau kartu. Cari `race_card_create`, `template_download`, atau `race_card_download`. Uji share jika perangkat mendukungnya.
4. Untuk pengunjung biasa, gunakan URL tanpa `ga_debug=1`. Periksa **Reports → Realtime → Event count by Event name**. Daftar **Recent events** dan laporan biasa mengikuti waktu pemrosesan GA4.

Tracking hanya berjalan di hostname produksi tersebut melalui HTTP/HTTPS. File lokal, localhost, dan deployment preview dengan hostname lain tetap tidak mengirim event. Jika domain produksi berubah, sesuaikan `productionHost`. ID dan hostname ini tersimpan langsung dalam HTML, bukan environment variable Vercel.

Preview kartu, ekspor yang gagal, klik berulang saat ekspor sedang diproses, dan native share yang dibatalkan tidak menghasilkan custom event sukses. Status `window.JRFAnalytics.status.tagLoaded` hanya menandakan Google tag termuat, bukan bukti bahwa GA4 telah menerima event.

Custom event tidak mengirim nama pelari, nama file, foto, waktu finish, pace, detak jantung, klub, atau username Instagram. Parameter selain format/kategori tidak diteruskan. URL halaman dan referrer dibersihkan; hanya parameter UTM kampanye yang diizinkan dalam URL halaman.

[Google tag dan custom events](https://developers.google.com/analytics/devguides/collection/ga4/events) · [Pageview otomatis](https://developers.google.com/analytics/devguides/collection/ga4/views) · [Enhanced measurement](https://support.google.com/analytics/answer/9216061?hl=en) · [DebugView](https://support.google.com/analytics/answer/7201382?hl=en).

## Fitur versi terakhir

5K, 10K, HM, FM dan 63.3K · JRF tetap tersedia. Header statistik 10K/FM merah; 5K/HM biru. Kategori 63.3K memakai dua baris FM/HM dengan input waktu dan pace terpisah serta medali otomatis tanpa toggle PB. Seluruh aset dan layout tetap berasal dari versi terakhir. Kartu dan template diekspor sebagai PNG 1080 × 1920.

ID dan hostname sudah terpasang. Penerimaan event ke property GA4 tetap perlu diperiksa di Realtime/DebugView setelah file ini dideploy; pengujian lokal tidak mengirim event ke Google.
