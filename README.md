# Dusun Dawung Kulon — Supabase + Hosting

## 1. Koneksi ke Supabase

1. Buat project Supabase baru.
2. Catat dua nilai ini dari `Project Settings → API`:
   - `SUPABASE_URL`
   - `SUPABASE_ANON_KEY`
3. Buka `script.js` dan ganti nilai:
   ```js
   const SUPABASE_URL = 'https://...supabase.co';
   const SUPABASE_ANON_KEY = 'sb_publishable_...';
   ```
4. Di Supabase, buat tabel berikut di SQL Editor:
   ```sql
   create table acara (
     id uuid primary key default gen_random_uuid(),
     judul text not null,
     tanggal text not null,
     deskripsi text not null,
     foto text,
     created_at timestamptz default now()
   );

   create table hasil_bumi (
     id uuid primary key default gen_random_uuid(),
     judul text not null,
     pemilik text,
     deskripsi text not null,
     foto text,
     created_at timestamptz default now()
   );
   ```
5. Aktifkan Row Level Security (RLS) pada kedua tabel dan tambahkan policy:
   ```sql
   alter table acara enable row level security;
   alter table hasil_bumi enable row level security;

   create policy "Publik boleh baca acara" on acara for select using (true);
   create policy "Login boleh kelola acara" on acara for all
     using (auth.role() = 'authenticated') with check (auth.role() = 'authenticated');

   create policy "Publik boleh baca hasil_bumi" on hasil_bumi for select using (true);
   create policy "Login boleh kelola hasil_bumi" on hasil_bumi for all
     using (auth.role() = 'authenticated') with check (auth.role() = 'authenticated');
   ```
6. Buat bucket Storage bernama `foto-dusun` dan buat policy:
   - Bucket diatur `Public`
   - Policy:
     ```sql
     create policy "Publik boleh lihat foto" on storage.objects for select
       using (bucket_id = 'foto-dusun');

     create policy "Login boleh unggah foto" on storage.objects for insert
       with check (bucket_id = 'foto-dusun' and auth.role() = 'authenticated');

     create policy "Login boleh hapus foto" on storage.objects for delete
       using (bucket_id = 'foto-dusun' and auth.role() = 'authenticated');
     ```
7. Buat user admin di `Authentication → Users` dengan email dan password.

## 2. Form login dan otentikasi

- `index.html` sudah memiliki modal login dengan `id="login-email"` dan `id="login-password"`.
- `script.js` mengirim email/password ke Supabase lewat:
  ```js
  supabaseClient.auth.signInWithPassword({ email, password })
  ```
- Jadi email akan dikenali langsung oleh Supabase Auth.
- Jika login gagal, kemungkinan besar:
  - email/password salah,
  - user belum dibuat di Authentication,
  - `SUPABASE_URL` / `SUPABASE_ANON_KEY` salah,
  - atau aplikasi live masih pakai versi lama.

## 3. Siap di hosting (static site)

File ini adalah website statis, cukup hosting folder `WDK`:

- `index.html`
- `script.js`
- `style.css`

Tidak perlu backend tambahan karena koneksi ke Supabase dilakukan langsung dari browser.

### Pilihan hosting mudah

1. Netlify
   - Buat akun di https://app.netlify.com
   - Drag & drop folder `WDK` ke Netlify Drop
   - Atau hubungkan Git repo kalau pakai GitHub.

2. Vercel
   - Buat akun di https://vercel.com
   - Upload project sebagai Static Site
   - Pastikan `index.html` berada di root.

3. Cloudflare Pages
   - Buat akun di https://pages.cloudflare.com
   - Upload folder statis atau sambungkan Git

4. GitHub Pages
   - Push repo ke GitHub
   - Aktifkan GitHub Pages pada branch `main` atau `gh-pages`

## 4. Validasi final

- Pastikan `script.js` yang dipakai browser benar-benar berisi konfigurasi Supabase baru.
- Jika masih muncul pesan error spesifik, buka DevTools → Network → `script.js` atau Sources dan pastikan file aktif cocok.
- Jika ingin, salin URL live atau isi `script.js` yang dimuat browser, saya bisa bantu bandingkan versi lokal vs versi live.
