# JRF 2026 Race Card · v34

Website statis dengan dua file:

```text
index.html
README.md
```

Seluruh CSS, JavaScript, font dan gambar sudah tertanam di `index.html`. Tidak diperlukan folder assets, package.json, build, atau environment variable. PNG kartu dan template berukuran 1080 × 1920.

## Replace di GitHub dan deploy

1. Ekstrak ZIP. Upload `index.html` dan `README.md` hasil ekstrak ke **root repository** untuk mengganti file lama dengan nama yang sama. File ZIP berisi tepat dua file itu, langsung pada level teratas.
2. Commit ke branch produksi yang terhubung ke Vercel. Tunggu deployment berstatus **Ready**.
3. Buka URL produksi dan lakukan hard refresh agar memakai versi terbaru.

Gunakan pengaturan Vercel berikut jika belum sesuai:

| Pengaturan | Nilai |
| --- | --- |
| Framework Preset | Other |
| Root Directory | Root repository / `./` |
| Build Command | Kosong; hapus override `npm run build` bila masih ada |
| Output Directory | `.`; hapus pengaturan `dist` bila masih ada |
| Install Command | Kosong |

Tidak perlu upload ZIP sebagai halaman website atau mengganti namanya menjadi index.html. [Pengaturan build website statis di Vercel](https://vercel.com/docs/builds/configure-a-build).

## Konfigurasi analytics yang dipakai

```js
measurementId: 'G-W6291G9928',
productionHost: 'jakartarunningfestival2026.vercel.app',
enabled: true
```

GA4 Measurement ID dan hostname dipertahankan. Google tag hanya dipasang sekali di bagian head. `gtag('config', ...)` mengatur pageview awal secara otomatis; tidak ada pageview manual kedua atau Google Tag Manager tambahan.

Tracking aktif ketika halaman memakai HTTP/HTTPS dan hostname produksi cocok. File lokal, localhost dan preview deployment dengan hostname berbeda tidak mengirim analytics. Jika suatu hari memakai domain lain, sesuaikan `productionHost` di bagian awal HTML.

## Tepat empat custom event

| Tujuan | Nama event | Kapan dihitung |
| --- | --- | --- |
| Mulai membuat race card | `race_card_create` | Masuk builder dari landing, atau membuka `#builder` langsung. |
| Share card | `race_card_share` | Native share selesai tanpa dibatalkan. |
| Download card | `race_card_download` | PNG berhasil dibuat dan browser diminta memulai unduhan. Termasuk fallback tombol Share pada perangkat tanpa dukungan share file. |
| Download template only | `template_download` | PNG template transparan berhasil dibuat dan browser diminta memulai unduhan. |

Tidak ada custom event untuk langkah wizard, review, preview, atau save terpisah. Masuk kembali ke builder dari landing menghitung aksi create baru. Pemanggilan router berulang saat masih berada di builder tidak menghitung create tambahan. Klik ganda ketika file sedang dibuat dicegah oleh pengunci ekspor.

Share yang dibatalkan, kegagalan pembuatan PNG, dan preview tidak dihitung sebagai hasil share/download. Share yang berubah menjadi unduhan hanya menghasilkan `race_card_download`, tanpa `race_card_share`. Browser memberi konfirmasi selesainya native share; hasil itu tidak membuktikan posting sudah terbit di Instagram. Unduhan dicatat ketika browser diminta mengunduh, karena website tidak bisa memeriksa file sudah tersimpan ke disk.

Custom parameter yang diizinkan:

| Parameter | Nilai |
| --- | --- |
| `race_format` | `on-ground` |
| `race_category` | `5K`, `10K`, `HM`, `FM`, `63K` |
| `export_method` | `download`, `native_share`, `share_fallback`; hanya untuk aksi ekspor |

Kategori 63.3K di website dikirim sebagai `63K` untuk konsistensi data. Nama pelari, nama file, foto, finish time, pace, detak jantung, klub, dan username tidak dikirim sebagai custom parameter. URL halaman dan referrer dibersihkan; hanya query UTM kampanye yang diizinkan dalam URL halaman.

## Pengaturan yang dilakukan di GA4

Kode tidak mengubah pengaturan akun GA4. Gunakan property **JRF 2026** yang sudah ada, lalu:

1. Buka **Admin → Data collection and modification → Data streams → stream Web JRF**. Pastikan Measurement ID adalah `G-W6291G9928` dan URL website adalah `https://jakartarunningfestival2026.vercel.app`.
2. Untuk laporan yang berfokus pada empat aksi ini, **matikan Enhanced measurement** pada stream. Ini menghentikan pengukuran tambahan seperti scroll/form otomatis. Custom event di atas tetap dikirim oleh kode website. Event bawaan Google seperti `page_view`, `session_start`, `first_visit`, atau `user_engagement` masih dapat muncul; empat event yang dibatasi di sini adalah custom event aplikasi.
3. Tidak perlu membuat stream baru, ID baru, Google tag kedua, atau matching condition di **Custom configurations**. Event sudah dikirim langsung dari website. Panel Custom configurations dapat tetap kosong.
4. Opsional: sesudah datanya diterima, tandai empat nama tersebut sebagai **key events** melalui **Admin → Data display → Events → Recent events**. Penandaan ini tidak diperlukan untuk menghitung Total users per event.

[Jenis event dan pemasangan Google tag](https://developers.google.com/analytics/devguides/collection/ga4/events) · [Enhanced measurement](https://support.google.com/analytics/answer/9216061?hl=en).

## Laporan jumlah pengguna untuk empat aksi

**Total users** menunjukkan jumlah pengguna unik menurut identifikasi GA4 pada rentang tanggal yang dipilih. **Event count** menunjukkan jumlah aksi. Contoh: satu pengguna mengunduh dua kali menghasilkan 1 Total user dan 2 event untuk download. Identifikasi pengguna mengikuti browser/perangkat dan cookie; tanpa sistem login, satu orang di perangkat berbeda dapat terhitung terpisah.

Untuk membuat satu tabel yang hanya memuat empat tracker:

1. Buka **Explore → Free form** di property JRF 2026. Beri nama **JRF · 4 aksi race card**.
2. Pada **Dimensions**, klik **+**, pilih **Event name**, lalu **Import**.
3. Pada **Metrics**, klik **+**, pilih **Total users** dan **Event count**, lalu **Import**.
4. Pada **Tab settings**, isi **Rows → Event name** dan **Values → Total users, Event count**.
5. Pada **Filters**, pilih **Event name → matches regex** dan gunakan pola:

```text
^(race_card_create|race_card_share|race_card_download|template_download)$
```

6. Pilih rentang tanggal yang mencakup waktu deployment dan pengujian. Tabel akan memuat nama event yang sudah diterima dalam rentang itu; event tanpa data dapat belum tampil sebagai baris.

Pengguna yang sama dapat melakukan beberapa jenis aksi. Jangan menjumlahkan empat baris Total users sebagai total pengunjung unik website. Data historis dari event versi lama tetap tersimpan; filter di atas membatasi tabel ke empat nama yang digunakan sekarang.

Custom dimensions tidak perlu dibuat hanya untuk menghitung pengguna per nama event. Jika nanti ingin breakdown kategori, tambahkan dimension scope **Event** dengan event parameter `race_category`; `export_method` dapat ditambahkan untuk memisahkan unduhan biasa dan fallback Share.

[Events report dan definisi metrik](https://support.google.com/analytics/answer/12926615?hl=en) · [Free-form exploration](https://support.google.com/analytics/answer/9327972?hl=en).

## Memastikan GA4 menerima event

1. Tunggu deployment Vercel terbaru **Ready**.
2. Di GA4, buka **Admin → Data display → DebugView**.
3. Di tab browser lain, buka [URL debug JRF](https://jakartarunningfestival2026.vercel.app/?ga_debug=1#builder). Saat builder dibuka, cari `race_card_create` di DebugView.
4. Lengkapi data dan unduh template atau kartu. Cari `template_download` atau `race_card_download`. Uji share pada perangkat yang mendukungnya; pembatalan tidak menghasilkan event share sukses.
5. Klik nama event di DebugView untuk melihat parameter. Jika lebih dari satu perangkat terdeteksi, pilih perangkat yang digunakan melalui **DEBUG DEVICE**.
6. Untuk tes biasa, buka URL tanpa `ga_debug=1`, lalu lihat **Reports → Realtime → Event count by Event name**. Laporan Events/Explore mengikuti waktu pemrosesan GA4; gunakan DebugView untuk pengecekan langsung.

Jika DebugView kosong, periksa versi deployment, ekstensi pemblokir analytics, Measurement ID, hostname yang sedang dibuka, dan pilihan property. Status `window.JRFAnalytics.status.tagLoaded` hanya menandakan file Google tag termuat, bukan bukti penerimaan event oleh GA4.

[DebugView Google](https://support.google.com/analytics/answer/7201382?hl=en) · [Pengecekan implementasi](https://developers.google.com/analytics/devguides/collection/ga4/troubleshoot).

## Tampilan dan kategori

Versi ini melanjutkan v33. Panel kategori 5K/HM tetap biru #013ECA termasuk varian Virgin; 10K/FM merah; 63.3K hitam/silver. Header statistik 5K/HM biru, 10K/FM merah. Kategori 63.3K tetap memakai dua baris statistik FM merah di atas dan HM biru di bawah, medali otomatis, serta input waktu dan pace terpisah.

Seluruh aset dan logika render kartu dipertahankan. Preview lokal dapat dibuka langsung dari index.html; analytics sengaja tidak berjalan pada file lokal.

Kode dan pengujian lokal dapat memastikan pemanggilan event dan perilaku ekspor. Penerimaan event di property GA4 perlu diverifikasi setelah deployment; paket ini tidak mengklaim bahwa pengaturan akun GA4 sudah diubah atau data live sudah diterima.
