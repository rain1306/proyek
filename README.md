# RuangBelajar — LMS Sekolah

LMS web mobile-first untuk satu sekolah, berbasis Next.js App Router, TypeScript, Better Auth, Drizzle ORM, PostgreSQL Neon, Vercel Blob dan PWA. Nama sementara **RuangBelajar** dapat diganti. Warna merek: `#4F7D32` dan `#A82020`. Logo asli serta ikon 192/512/maskable disertakan.

## Yang tersedia

- Admin: buat akun guru/murid/admin, reset password dan pencabutan sesi, buat kelas dengan guru pengampu.
- Guru: buat kelas, materi teks + tautan HTTPS, pengumuman, diskusi, tugas dengan rubrik dan penilaian manual.
- Murid: gabung kelas dengan kode, baca materi, tandai selesai, unduh teks, balas diskusi, kumpulkan tugas teks/PDF/gambar.
- Kuis pilihan ganda: pembuat soal visual, indikator per soal, jadwal mulai/akhir, batas percobaan, pengacakan soal dan opsi, waktu server, lanjutkan percobaan, autosave perangkat/server, nilai otomatis.
- Monitoring: rata-rata, hasil per indikator, progres materi, hasil pretest/posttest, riwayat percobaan dan ekspor CSV yang dapat dibuka di Excel.
- Pembatasan akses kelas di server, query SQL berparameter, snapshot soal per percobaan dan audit perubahan nilai/reset password.
- PWA installable, ikon maskable, halaman offline, draf kuis lokal dan unduh materi teks.

## Persiapan lokal

Butuh Node.js 22+, npm, database Neon/PostgreSQL, dan editor kode.

1. Ekstrak ZIP, buka terminal di folder `lms-sekolah`.
2. `npm ci`
3. Salin `.env.example` menjadi `.env.local`. Isi `DATABASE_URL` dengan connection string Neon berakhiran `-pooler` dan `sslmode=require`.
4. Buat secret acak: `node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"`. Salin hasil ke `BETTER_AUTH_SECRET`. Jangan bagikan atau commit secret.
5. Isi `BETTER_AUTH_URL=http://localhost:3000`, `BOOTSTRAP_EMAIL`, dan `BOOTSTRAP_PASSWORD` minimal 12 karakter.
6. `npm run db:setup` untuk tabel autentikasi dan LMS. File migrasi awal sudah ada.
7. `npm run admin:create` sekali untuk membuat admin pertama.
8. `npm run dev`, buka http://localhost:3000, masuk dengan akun admin tadi.

Buat guru dan murid melalui menu Pengguna; buat kelas dan pilih guru pengampu. Guru dapat masuk, menambahkan materi, kuis, dan tugas. Murid masuk dengan akun sekolah kemudian bergabung menggunakan kode kelas.

Pengguna dapat mengganti password melalui tombol Akun. Reset password dilakukan admin. Versi ini tidak menggunakan email reset otomatis; admin menyampaikan password baru melalui saluran sekolah yang sesuai. Pengguna harus memperlakukan password awal sebagai rahasia.

## Unggah tugas

Hubungkan Vercel Blob, isi `BLOB_READ_WRITE_TOKEN` dan `BLOB_PUBLIC_BASE_URL` sesuai alamat store, termasuk slash akhir, misalnya `https://xxxx.public.blob.vercel-storage.com/`.

Lampiran opsional dibatasi 10 MB dan MIME PDF/JPEG/PNG/WebP. Tanpa Blob, tugas teks tetap dapat dikumpulkan. Lampiran memakai URL publik acak: siapa pun yang memperoleh URL dapat membuka file. Untuk data sensitif, ubah ke private Blob dengan endpoint unduh yang memeriksa keanggotaan kelas sebelum penggunaan produksi. MIME bukan pemindaian malware; versi ini belum menyediakan antivirus/karantina.

## GitHub → Vercel → Neon

1. Buat repositori GitHub, lalu upload isi folder proyek (bukan `node_modules`, `.next`, `.env.local`). `.gitignore` disertakan.
2. Di Vercel pilih **Add New Project → Import Git Repository**. Framework: Next.js. Root directory adalah folder yang berisi `package.json`.
3. Tambahkan `DATABASE_URL`, `BETTER_AUTH_SECRET`, `BETTER_AUTH_URL` (URL domain produksi HTTPS), dan konfigurasi Blob jika dipakai.
4. Migrasi dan bootstrap dijalankan dari komputer terhadap database produksi sebelum login. Jangan masukkan bootstrap password ke repositori. Hapus variabel bootstrap setelah akun dibuat.
5. Deploy. Jika domain berubah, perbarui `BETTER_AUTH_URL` lalu redeploy.
6. Gunakan database berbeda untuk preview/development. Jangan jalankan percobaan pada data murid produksi.
7. Jalankan checklist di `docs/VALIDASI.md` sebelum dipakai sekolah.

Pilih region fungsi Vercel dekat region database Neon. Cadangan dan pemulihan perlu disiapkan sesuai paket Neon yang dipakai. Jumlah pengguna serentak harus diuji terhadap paket aktual; proyek ini tidak mengklaim kapasitas tertentu.

## Perintah

- `npm run typecheck` — pemeriksaan TypeScript
- `npm test` — pengujian penilaian dan kerahasiaan kunci
- `npm run build` — build produksi
- `npm run db:generate` — menghasilkan migrasi autentikasi setelah skema berubah
- `npm run db:auth` — menerapkan migrasi autentikasi
- `npm run db:migrate` — menerapkan SQL LMS awal

Jangan menjalankan `db:generate` pada setiap deploy tanpa perubahan skema. Untuk perubahan LMS berikutnya, tambahkan migrasi berurutan; jangan menghapus tabel produksi.

## Batas versi 1

Kuis otomatis mendukung pilihan ganda. Esai dinilai melalui tugas/rubrik. Bank soal berada di setiap kuis; belum menjadi katalog soal lintas kelas yang dapat digunakan ulang. Diskusi mendukung topik/balasan, dengan pin/lock oleh guru, tetapi belum lampiran, moderasi lengkap atau chat realtime. Pengumuman tersedia di aplikasi; push notification/email pengingat belum disediakan. Materi dapat diunduh sebagai teks, tetapi belum browser materi offline penuh. Draf lokal tersedia saat tab kuis masih terbuka; muat ulang tanpa koneksi menampilkan halaman offline, bukan halaman kuis. Sinkronisasi dilakukan saat halaman aktif, bukan background sync.

Grafik ditampilkan sebagai bar indikator/progress native untuk menjaga halaman ringan. Tampilan menggunakan CSS responsif sendiri, bukan Tailwind/shadcn. Form menggunakan validasi HTML + Zod di server, bukan React Hook Form. Ini mengurangi dependensi tanpa mengubah alur inti. Recharts disertakan sebagai dependensi untuk pengembangan grafik lanjutan.

Fitur lanjutan seperti orang tua, sertifikat, analitik psikometrik, pembelajaran adaptif, AI, impor akun massal dan kuis offline belum dibuat. Tidak ada klaim bahwa aktivitas login mengukur kompetensi. Pretest/posttest harus memiliki indikator dan instrumen yang sebanding.

Belum ada kredensial Neon/Vercel milik pengguna, sehingga integrasi database produksi, unggah Blob dan alur login nyata harus diverifikasi setelah konfigurasi. Catatan pemeriksaan aktual ada di `docs/VALIDASI.md`.
