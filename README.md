# SmartKASIR — Deploy ke GitHub Pages + Google Apps Script

Aplikasi ini terdiri dari 2 bagian:

1. **Frontend** (`index.html`) → di-hosting di **GitHub Pages** (HTTPS), supaya kamera untuk scan barcode bisa diakses langsung dari browser.
2. **Backend + Database** (`code.gs`) → tetap di **Google Apps Script**, terhubung ke Google Spreadsheet seperti sebelumnya. Sekarang `code.gs` hanya berfungsi sebagai **API JSON**, bukan lagi yang menampilkan halaman.

Sebelumnya frontend dan backend jadi satu (dijalankan dari dalam editor Apps Script memakai `google.script.run`). Karena `google.script.run` **hanya berfungsi jika HTML dibuka dari Apps Script**, dan tidak tersedia saat `index.html` di-hosting di GitHub Pages, file `index.html` sudah ditambahkan "jembatan" yang meniru `google.script.run` tapi lewat `fetch()` ke Web App Apps Script Anda. Jadi seluruh logika aplikasi (POS, inventory, hutang, laporan, dll) **tidak perlu diubah** dan tetap sama persis.

## Langkah Setup

### 1. Siapkan Backend (Google Apps Script)
1. Buka Google Spreadsheet yang ingin dijadikan database (atau buat baru).
2. Menu **Extensions → Apps Script**.
3. Hapus isi `Code.gs` default, lalu tempel seluruh isi file **`code.gs`** dari paket ini.
4. Di toolbar Apps Script, pilih fungsi `setupDatabase` lalu klik **Run** (sekali saja). Ini akan membuat sheet `Users`, `Produk`, `Pelanggan`, `Transaksi`, `Hutang`, `Riwayat_Bayar_Hutang` beserta data contoh & akun login default (`admin`/`admin123`, `kasir`/`kasir123`).
5. Klik **Deploy → New deployment**.
   - Type: **Web app**
   - Execute as: **Me**
   - Who has access: **Anyone**
6. Klik **Deploy**, lalu **salin URL** yang diakhiri `/exec`. Ini adalah `API_URL` Anda.

> Setiap kali Anda mengubah isi `code.gs` di kemudian hari, buat deployment baru (atau **Manage deployments → Edit → New version**) agar perubahan aktif.

### 2. Siapkan Frontend (GitHub Pages)
1. Buka file **`index.html`** dari paket ini.
2. Cari baris:
   ```js
   const API_URL = ""; // <-- TEMPEL URL WEB APP GOOGLE APPS SCRIPT ANDA DI SINI (diakhiri /exec)
   ```
   Isi dengan URL `/exec` dari langkah sebelumnya, contoh:
   ```js
   const API_URL = "https://script.google.com/macros/s/AKfycb.../exec";
   ```
3. Upload `index.html` ke repository GitHub Anda (bisa langsung lewat web GitHub: **Add file → Upload files**).
4. Di repo, buka **Settings → Pages**, pilih branch (misal `main`) dan folder `/root`, lalu **Save**.
5. Tunggu beberapa menit, GitHub akan memberi URL seperti `https://namauser.github.io/nama-repo/index.html`. Buka URL tersebut — aplikasi akan berjalan dan tersambung ke Spreadsheet Anda, dengan HTTPS penuh sehingga scan barcode via kamera bisa langsung dipakai (browser akan minta izin akses kamera).

### 3. Mode Simulasi (opsional, untuk uji tampilan)
Jika `API_URL` dikosongkan, aplikasi otomatis berjalan dengan data contoh lokal (tanpa terhubung ke Spreadsheet) — berguna untuk sekadar melihat tampilan sebelum backend selesai di-deploy.

## Catatan Teknis
- Komunikasi frontend ↔ backend memakai `fetch()` dengan `Content-Type: text/plain` (bukan `application/json`) — ini sengaja, agar browser **tidak** mengirim request *preflight* (`OPTIONS`) yang tidak didukung oleh Web App Apps Script. Apps Script tetap membaca isi body sebagai JSON di sisi server.
- Semua fungsi backend yang boleh dipanggil dari frontend didaftarkan lewat *whitelist* di `code.gs` (`getApiWhitelist_`) — ini mencegah fungsi lain di luar itu dipanggil dari luar.
- Fungsi backend baru yang ditambahkan (sebelumnya dipanggil dari frontend tapi belum ada di `code.gs` lama): `deleteProduct`, `deleteCustomer`, `getDebtHistory`.
- Kamera scan barcode (`html5-qrcode`) memerlukan konteks HTTPS aman — inilah alasan utama migrasi ke GitHub Pages, karena Apps Script menyajikan halaman lewat iframe `googleusercontent.com` yang membatasi akses kamera pada sebagian perangkat/browser.
