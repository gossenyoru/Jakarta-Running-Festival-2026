# JRF Race Card · versi dua file (v30)

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

## Menyalakan GA4 nanti

GA4 sudah disiapkan tetapi **belum aktif**. Di bagian awal `index.html`, cari `GA4 CONFIG`, lalu isi dua nilai:

```js
measurementId: 'G-XXXXXXXXXX',
productionHost: 'nama-proyek.vercel.app',
enabled: true
```

Gunakan Measurement ID asli dari stream Web GA4 dan hostname produksi yang benar. `productionHost` boleh hostname saja atau URL HTTPS; kode menormalisasinya. Cocokkan host persis dengan domain yang dipakai pengunjung, termasuk `www` jika ada. Setelah mengubah HTML, commit ke GitHub dan tunggu deployment baru.

Versi statis dua file ini menyimpan konfigurasi GA4 langsung di HTML. Environment variable `GA_MEASUREMENT_ID` Vercel tidak dibaca karena tidak ada proses build. Tracking hanya berjalan ketika ID valid, hostname cocok, dan halaman menggunakan HTTP/HTTPS. Preview dengan hostname berbeda, localhost dan file lokal tidak mengirim analytics. ID dan hostname boleh kosong selama preview.

Di GA4, gunakan **Admin → Data streams → stream Web** untuk melihat Measurement ID. Matikan **Enhanced measurement** untuk stream instalasi ini agar pageview/history/form/download otomatis tidak menggandakan event yang dikirim kode. Google tag sudah ada di file ini; cukup isi konfigurasi, tidak perlu menambah snippet atau GTM lain untuk ID yang sama.

Untuk tes, buka URL produksi `/?ga_debug=1#builder`, kemudian **Admin → Data display → DebugView** dan **Reports → Realtime**. Mode debug hanya aktif pada halaman yang memakai `ga_debug=1`; hapus parameter untuk penggunaan biasa. Jangan pasang `debug_mode: false` untuk mematikannya.

| Event | Makna |
| --- | --- |
| `page_view` | Tampilan landing atau builder; router mengatur pageview sesuai hash. |
| `race_card_create_start` | Builder pertama kali dibuka pada entri riwayat browser. |
| `race_card_step_view` | Step berubah; kembali ke step sebelumnya menghitung tampilan baru. |
| `race_card_ready` | Step review pertama kali tercapai pada entri tersebut. |
| `race_card_download` | PNG siap dan browser diminta memulai unduhan kartu. |
| `race_card_share` | Native share selesai tanpa dibatalkan. |
| `race_card_template_download` | Unduhan PNG transparan dimulai. |

Pada **Admin → Data display → Custom definitions**, buat custom dimensions dengan Scope **Event** untuk `race_category`, `step_name`, `export_method`, `page_type` dan `has_photo`. Untuk laporan halaman, gunakan `page_type` karena landing dan builder berada di file `index.html` yang sama. Kategori dikirim sebagai `5K`, `10K`, `HM`, `FM`, `63K`.

Event ekspor yang relevan dapat ditandai sebagai key events di GA4. Preview, kegagalan ekspor, klik ganda saat ekspor diproses, serta share yang dibatalkan tidak dicatat sebagai hasil ekspor. Selesainya native share tidak membuktikan posting Instagram telah terbit, dan browser tidak mengonfirmasi file sudah tersimpan ke disk.

Nama pelari, nama file, foto, waktu finish, pace, detak jantung, klub, serta username Instagram tidak dikirim sebagai parameter custom event. Hanya parameter yang diizinkan kode dan query UTM kampanye yang diteruskan. Jangan memakai UTM berisi data pribadi.

[Setup GA4](https://support.google.com/analytics/answer/9304153?hl=en) · [Pageview manual](https://developers.google.com/analytics/devguides/collection/ga4/views) · [DebugView](https://support.google.com/analytics/answer/7201382?hl=en) · [Custom dimensions](https://support.google.com/analytics/answer/14239696?hl=en).

## Fitur versi terakhir

5K, 10K, HM, FM dan 63.3K · JRF tetap tersedia. Header statistik 10K/FM merah; 5K/HM biru. Kategori 63.3K memakai dua baris FM/HM dengan input waktu dan pace terpisah serta medali otomatis tanpa toggle PB. Seluruh aset dan layout tetap berasal dari versi terakhir. Kartu dan template diekspor sebagai PNG 1080 × 1920.

Penerimaan event ke property Google belum bisa dikonfirmasi sebelum ID dan domain produksi diisi dan diuji sesudah deploy.
