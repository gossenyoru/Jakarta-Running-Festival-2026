# JRF 2026 Race Card · v37

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

## Perbaikan upload foto · v37

- Area upload sekarang menggunakan input file native yang langsung menerima tap, tetap bisa difokuskan melalui keyboard, dan memakai filter galeri `image/*`.
- JPG/PNG dari galeri atau penyedia file Android tidak ditolak hanya karena tipe MIME kosong atau generik. File tetap harus berhasil dibaca sebagai gambar.
- Pembacaan mencoba decoder gambar native terlebih dahulu, kemudian ImageBitmap, lalu FileReader bila diperlukan. Gagal pada satu jalur tidak langsung menghentikan upload.
- Hasil foto diperkecil secara proporsional sampai maksimum 2560 piksel pada sisi terpanjang dan empat megapiksel. Foto pengganti, proses yang dibatalkan, serta canvas ekspor sementara dibersihkan setelah dipakai.
- Indikator proses muncul saat foto dibaca. Tombol lanjut dan ekspor menunggu foto siap. Decoder yang tidak merespons memiliki timeout dan tombol upload kembali bisa dipakai. Pemilihan file yang sama bisa diulang.
- Foto lama tetap tersedia bila penggantinya gagal. File kosong, format HEIC yang tidak didukung, dan pembacaan terlalu lama mendapat pesan yang sesuai.

Foto diproses di browser perangkat; foto dan nama file tidak dikirim ke analytics.

**HEIC:** dukungan tergantung browser. Safari 17 memperkenalkan dukungan HEIC; paket ini tidak menyertakan konverter HEIC untuk seluruh browser. Bila gagal, gunakan JPG/PNG atau screenshot foto. [Catatan WebKit tentang Safari 17](https://webkit.org/blog/14205/news-from-wwdc23-webkit-features-in-safari-17-beta/).

**Jika galeri tetap tidak terbuka di browser dalam aplikasi:** buka URL yang sama di Safari pada iPhone atau Chrome pada Android. Website tidak bisa memperbaiki pemilih file yang dibatasi oleh aplikasi pembukanya.

### Pengujian dan batas verifikasi

Lulus 31 pengujian otomatis pada logic upload dan empat tracker, termasuk simulasi tipe file kosong, fallback decoder, HEIC native/unsupported, timeout, pembatalan, penggantian foto, pelepasan memori dan pemilihan ulang. Render sembilan kombinasi kategori/Virgin tetap identik dengan v36. Pembacaan JPG dan PNG aktual, pengecilan foto, serta ekspor PNG 1080 × 1920 juga diperiksa menggunakan renderer canvas native.

Simulasi tersebut bukan pengujian langsung perangkat Android/iPhone. Browser otomatis tidak dapat dijalankan di lingkungan ini, sehingga sentuhan galeri, izin akses, perilaku penyedia cloud dan batas memori setiap HP masih perlu dicek pada perangkat setelah deployment.

### Cek singkat setelah deploy

1. Tunggu Vercel **Ready** lalu buka website baru di Chrome Android dan Safari iPhone. Tutup tab lama atau refresh sebelum tes.
2. Masuk ke langkah foto. Tap area upload; pilih JPG/PNG dari penyimpanan lokal. Coba juga foto kamera berukuran besar dan posisi portrait.
3. Batalkan pemilihan lalu coba lagi. Ganti foto, pilih file yang sama lagi, Clear, lalu tambahkan foto baru. Tombol harus bisa dipakai setelah berhasil maupun gagal.
4. Lanjutkan ke Preview & Share, unduh PNG, lalu cek orientasi dan isi fotonya. Uji juga link dari aplikasi tempat pengunjung biasanya membuka website.
5. Untuk HEIC, gunakan browser yang mendukungnya atau pilih salinan JPG/PNG. Bila sebuah foto di cloud belum tersedia lokal, simpan ke perangkat dahulu lalu pilih ulang.

[Input file dan penggunaan opacity untuk kontrol native](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/input/file) · [Tipe MIME file dapat kosong](https://developer.mozilla.org/en-US/docs/Web/API/Blob/type).

## Konfigurasi analytics: stream baru terpasang

```js
measurementId: 'G-PJ64F058FN',
productionHost: 'jakartarunningfestival2026.vercel.app',
enabled: true
```

Measurement ID stream baru `G-PJ64F058FN` dan hostname produksi `jakartarunningfestival2026.vercel.app` sudah terisi. Replace index.html di GitHub, commit, lalu tunggu deployment Vercel selesai. Tidak perlu menambah snippet Google tag lain atau environment variable. Google tag hanya dipasang sekali di bagian head. `gtag('config', ...)` mengatur pageview awal secara otomatis; tidak ada pageview manual kedua atau Google Tag Manager tambahan.

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

Kode tidak mengubah pengaturan akun GA4. Setelah membuat stream Web baru:

1. Buka stream Web baru di **Admin → Data collection and modification → Data streams**. Pastikan Measurement ID-nya `G-PJ64F058FN` dan URL website `https://jakartarunningfestival2026.vercel.app`. ID dan hostname tersebut sudah diisi di index.html. Upload ulang file dan tunggu deployment selesai.
2. Untuk laporan yang berfokus pada empat aksi ini, **matikan Enhanced measurement** pada stream. Ini menghentikan pengukuran tambahan seperti scroll/form otomatis. Custom event di atas tetap dikirim oleh kode website. Event bawaan Google seperti `page_view`, `session_start`, `first_visit`, atau `user_engagement` masih dapat muncul; empat event yang dibatasi di sini adalah custom event aplikasi.
3. Gunakan hanya satu Google tag untuk ID stream baru di file ini; tidak perlu menambah tag kedua atau matching condition di **Custom configurations**. Empat event sudah dikirim langsung dari website ketika konfigurasi aktif. Panel Custom configurations dapat tetap kosong.
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
3. Di tab browser lain, buka [URL debug JRF](https://jakartarunningfestival2026.vercel.app/?ga_debug=1#builder). Saat builder dibuka, cari `race_card_create` di DebugView pada property yang memiliki stream baru `G-PJ64F058FN`.
4. Lengkapi data dan unduh template atau kartu. Cari `template_download` atau `race_card_download`. Uji share pada perangkat yang mendukungnya; pembatalan tidak menghasilkan event share sukses.
5. Klik nama event di DebugView untuk melihat parameter. Jika lebih dari satu perangkat terdeteksi, pilih perangkat yang digunakan melalui **DEBUG DEVICE**.
6. Untuk tes biasa, buka URL tanpa `ga_debug=1`, lalu lihat **Reports → Realtime → Event count by Event name**. Laporan Events/Explore mengikuti waktu pemrosesan GA4; gunakan DebugView untuk pengecekan langsung.

Jika DebugView kosong, periksa versi deployment, ekstensi pemblokir analytics, Measurement ID, hostname yang sedang dibuka, dan pilihan property. Status `window.JRFAnalytics.status.tagLoaded` hanya menandakan file Google tag termuat, bukan bukti penerimaan event oleh GA4.

[DebugView Google](https://support.google.com/analytics/answer/7201382?hl=en) · [Pengecekan implementasi](https://developers.google.com/analytics/devguides/collection/ga4/troubleshoot).

## Tampilan dan kategori

Versi ini melanjutkan v36 dengan perbaikan upload foto dan empat tracker yang sama. GA4 baru `G-PJ64F058FN` sudah terpasang dan hostname produksi dikembalikan ke `jakartarunningfestival2026.vercel.app`. Panel kategori 5K/HM tetap biru #013ECA termasuk varian Virgin; 10K/FM merah; 63.3K hitam/silver. Header statistik 5K/HM biru, 10K/FM merah. Kategori 63.3K tetap memakai dua baris statistik FM merah di atas dan HM biru di bawah, medali otomatis, serta input waktu dan pace terpisah.

Seluruh aset, warna kategori dan layout kartu dipertahankan. Canvas sementara dilepas setelah digunakan agar penggunaan memori lebih rendah. Preview lokal dapat dibuka langsung dari index.html; analytics sengaja tidak berjalan pada file lokal.

Analytics dikonfigurasi untuk mengirim ke stream baru saat website dibuka di hostname produksi. Kode dan pengujian lokal dapat memastikan pemanggilan event dan perilaku ekspor. Penerimaan event di property GA4 perlu diverifikasi setelah deployment; paket ini tidak mengklaim bahwa pengaturan akun GA4 sudah diubah atau data live sudah diterima.
