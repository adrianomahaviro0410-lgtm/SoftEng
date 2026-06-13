# 📚 SmartStudy

> Aplikasi manajemen tugas & jadwal kuliah dengan algoritma prioritas SKS otomatis — berbasis web, terintegrasi Supabase cloud database.

<div align="center">

![Version](https://img.shields.io/badge/versi-v8.0-7c6cfa?style=for-the-badge)
![Platform](https://img.shields.io/badge/platform-Web%20Browser-60a5fa?style=for-the-badge)
![Database](https://img.shields.io/badge/database-Supabase-2dd4bf?style=for-the-badge)
![License](https://img.shields.io/badge/lisensi-PKM--KC%202026-fbbf24?style=for-the-badge)

</div>

---

## Tentang SmartStudy

Mahasiswa rata-rata mengambil **18–24 SKS per semester** dengan mata kuliah yang masing-masing memiliki bobot berbeda. Tanpa sistem prioritas yang tepat, tugas sering dikerjakan berdasarkan yang paling mudah — bukan yang paling penting.

**SmartStudy** hadir sebagai solusi dengan algoritma prioritas otomatis yang mempertimbangkan:
- ⚖️ **Bobot SKS** mata kuliah
- ⏰ **Jarak waktu** deadline
- 🔥 **Tingkat kesulitan** tugas

Hasilnya: setiap tugas mendapat skor prioritas yang bersifat dinamis, sehingga kamu selalu tahu **mana yang harus dikerjakan duluan**.

---

## ✨ Fitur Utama

| Tab | Fitur |
|-----|-------|
| 🏠 **Dashboard** | Statistik tugas aktif, kartu prioritas tertinggi, banner deadline merah, daftar semua tugas |
| 📖 **Matkul** | Tambah/edit/hapus mata kuliah dengan SKS, jadwal, ruangan, dan warna |
| ✅ **Tugas** | Daftar tugas diurutkan otomatis berdasarkan skor prioritas (0–99) |
| 📅 **Jadwal** | Jadwal kuliah mingguan terbentuk otomatis dari data matkul |
| ➕ **Tambah** | Form tambah tugas dengan preview skor prioritas real-time |
| 📊 **Tracker** | Progress belajar per matkul, streak harian, catatan refleksi |
| ⚙️ **Setelan** | Toggle notifikasi, mode gelap/terang, reset data |

### Fitur Lainnya
- 🌙 **Dark / Light Mode** — toggle tema tampilan
- 🔔 **Notifikasi deadline** — banner & badge otomatis untuk tugas ≤ 3 hari
- ☁️ **Cloud database** — data tersimpan di Supabase, bisa diakses dari device manapun
- 👥 **Multi-user** — satu project Supabase bisa dipakai banyak pengguna dengan data terpisah
- 📱 **Responsive** — tampilan mobile-friendly

---

## Algoritma Prioritas SKS

```
Skor = (SKS × 10) + (urgency × 3) + (kesulitan × 5)
```

| Komponen | Nilai | Penjelasan |
|----------|-------|------------|
| `SKS` | 1–6 | Jumlah SKS mata kuliah |
| `urgency` | `max(0, 10 - hari_tersisa)` | Semakin dekat deadline, semakin tinggi |
| `kesulitan` | 1=Mudah, 2=Sedang, 3=Sulit | Tugas sulit butuh waktu lebih awal |

### Contoh Kalkulasi

| Tugas | SKS | Deadline | Kesulitan | Skor | Level |
|-------|-----|----------|-----------|------|-------|
| UTS Software Engineering | 4 | Besok | Sulit (3) | **82** | 🔴 Tinggi |
| Laporan PKM-KC | 4 | 2 hari | Sulit (3) | **77** | 🔴 Tinggi |
| Quiz Database | 3 | 4 hari | Sedang (2) | **58** | 🟡 Sedang |
| Presentasi Analisis | 3 | 6 hari | Sedang (2) | **48** | 🟡 Sedang |
| Essay Pancasila | 2 | 7 hari | Mudah (1) | **30** | 🟢 Rendah |

> **UTS SE:** `(4×10) + (max(0,10−1)×3) + (3×5)` = `40 + 27 + 15` = **82**

---

## Tech Stack

```
Frontend  : HTML5 + CSS3 + Vanilla JavaScript (ES2022)
Database  : Supabase (PostgreSQL) — cloud, gratis
DB Client : @supabase/supabase-js v2.49.4
Icons     : Tabler Icons v3.19.0
Font      : DM Sans + DM Mono (Google Fonts)
```

 **Tidak perlu install apapun** — cukup browser dan koneksi internet.

---

## Cara Menjalankan

### 1. Setup Database Supabase

1. Daftar di [supabase.com](https://supabase.com) (gratis)
2. Buat project baru
3. Buka **SQL Editor** → paste SQL berikut → klik **Run**

```sql
create extension if not exists "uuid-ossp";

create table if not exists mata_kuliah (
  id         uuid primary key default uuid_generate_v4(),
  user_id    text not null default 'default',
  name       text not null,
  sks        integer not null check (sks between 1 and 6),
  day        text not null,
  start_time text not null,
  end_time   text not null,
  room       text,
  color      text not null default '#7c6cfa',
  progress   integer not null default 0 check (progress between 0 and 100),
  created_at timestamptz default now()
);

create table if not exists tugas (
  id           uuid primary key default uuid_generate_v4(),
  user_id      text not null default 'default',
  matkul_id    uuid references mata_kuliah(id) on delete set null,
  matkul_name  text,
  matkul_color text,
  name         text not null,
  sks          integer not null,
  type         text not null default 'Tugas',
  difficulty   integer not null default 2,
  deadline     date not null,
  score        integer not null default 0,
  done         boolean not null default false,
  created_at   timestamptz default now()
);

create table if not exists catatan (
  id         uuid primary key default uuid_generate_v4(),
  user_id    text not null default 'default',
  content    text not null,
  created_at timestamptz default now()
);
```

4. Aktifkan RLS dan buat policy:

```sql
alter table mata_kuliah enable row level security;
alter table tugas        enable row level security;
alter table catatan      enable row level security;

create policy "public_access" on mata_kuliah for all using (true) with check (true);
create policy "public_access" on tugas        for all using (true) with check (true);
create policy "public_access" on catatan      for all using (true) with check (true);
```

### 2. Ambil API Credentials

Di Supabase → **Settings** → **API** → copy:
- **Project URL** — `https://xxxx.supabase.co`
- **Publishable Key** — `sb_publishable_...` *(jangan pakai Service Role Key)*

### 3. Konfigurasi & Jalankan

Buka `smartstudy_v8.html` dengan Notepad / VS Code, cari bagian ini:

```javascript
// KONFIGURASI — EDIT BAGIAN INI
const SUPABASE_URL = 'GANTI_URL';   // ← paste Project URL
const SUPABASE_KEY = 'GANTI_KEY';   // ← paste Publishable Key
const DEFAULT_UID  = 'user1';       // ← ganti dengan nama/ID kamu
```

Simpan file → **double-click** untuk buka di browser → selesai! 🎉

 Koneksi berhasil jika muncul titik hijau **● Cloud** di pojok kanan atas.

---

## Struktur Database

```
mata_kuliah          tugas                    catatan
───────────          ─────                    ───────
id (uuid PK)         id (uuid PK)             id (uuid PK)
user_id              user_id                  user_id
name                 matkul_id (FK)           content
sks                  matkul_name              created_at
day                  name
start_time           sks
end_time             type
room                 difficulty
color                deadline
progress             score
created_at           done
                     created_at
```

---

## Struktur File

```
smartstudy/
├── smartstudy_v8.html      ← App utama (buka ini di browser)
├── smartstudy_schema.sql   ← SQL untuk setup database Supabase
├── PANDUAN_SETUP.html      ← Panduan visual step-by-step
└── README.md               ← Dokumentasi ini
```

---

## Multi-User

Satu project Supabase bisa dipakai seluruh anggota tim. Setiap orang cukup set `DEFAULT_UID` yang berbeda:

```javascript
// Jeremiah
const DEFAULT_UID = 'jeremiah2026';

// Adriano
const DEFAULT_UID = 'adriano2026';

// Aurellio
const DEFAULT_UID = 'aurellio2026';
```

Data masing-masing pengguna **100% terpisah** — tidak bisa saling melihat.

---

## Troubleshooting

| Error | Penyebab | Solusi |
|-------|----------|--------|
| `relation does not exist` | SQL schema belum dijalankan | Jalankan SQL di SQL Editor Supabase |
| `row-level security policy violation` | RLS aktif tapi policy belum dibuat | Jalankan SQL policy `public_access` |
| `Invalid API key` | Key salah atau format lama | Gunakan Publishable Key (bukan Service Role) |
| Titik merah di header | Koneksi gagal | Cek internet dan kredensial |
| Data tidak muncul | User ID berbeda | Pastikan `DEFAULT_UID` konsisten |
| Jadwal kosong | Belum ada matkul | Tambah matkul di tab Matkul dulu |

---

## Tim Pengembang

| Nama | NIM | 
|------|-----|
| Jeremiah Mark *(Ketua)* | 2802471846 |
| Adriano Mahaviro | 2802458434 |
| Aurellio Martha Yoga Ranendra | 2802443092 |
| Fransillia Tanciady | 2802458421 |
| Jocelyn Patricia Kong | 2802478404 |

**Dosen Pembimbing:** Muhammad Alfhi Saputra, S.Kom., M.Kom.  
**Institusi:** Universitas Bina Nusantara (BINUS), Jakarta  
**Program:** PKM-KC (Program Kreativitas Mahasiswa — Karsa Cipta) 2026

---

<div align="center">
  <sub>Made with love for PKM-KC 2026 — Universitas Bina Nusantara</sub>
</div>
