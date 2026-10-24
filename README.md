# BengkelKu

Platform booking servis, riwayat perawatan, dan pengingat kendaraan.
Proyek 1 Pengembangan Aplikasi Berbasis Web (PABWE 2026).

| Lapisan | Teknologi |
|---|---|
| Frontend | Nuxt 4 (SPA, TypeScript), Pinia, Tailwind CSS v4, lucide-vue-next, SweetAlert2, Vitest |
| Backend | Express.js 5, SQLite (better-sqlite3), JWT, bcrypt, Zod, Swagger UI, Jest + Supertest |
| CI/CD | Jenkins (`Jenkinsfile`), Docker Compose |

## Fitur

**Pelanggan:** daftar dan login, kelola kendaraan, booking servis dengan pilihan jam yang tersedia, batalkan booking, riwayat servis, cetak invoice, pengingat servis berikutnya (berdasarkan tanggal atau kilometer).

**Admin bengkel:** dashboard (booking per bulan, pendapatan, layanan terpopuler), kelola layanan dan harga, konfirmasi dan kerjakan booking, catat hasil servis (otomatis membuat riwayat, menaikkan odometer, dan menjadwalkan servis berikutnya), daftar pelanggan.

## Struktur repository

```
.
├── backend/                 API Express + SQLite
│   ├── src/
│   │   ├── config/          env dan konstanta
│   │   ├── db/              koneksi, skema, seed
│   │   ├── routes/          pemetaan URL ke controller
│   │   ├── controllers/     lapisan HTTP (baca request, kirim response)
│   │   ├── services/        logika bisnis
│   │   ├── repositories/    seluruh query SQL
│   │   ├── validators/      skema Zod
│   │   ├── middlewares/     auth, role, validasi, error handler
│   │   ├── utils/           AppError, tanggal, paginasi, response
│   │   └── docs/            spesifikasi OpenAPI
│   └── tests/               Jest + Supertest
├── frontend/                Nuxt 4 (srcDir: src/)
│   └── src/
│       ├── features/        satu folder per fitur
│       │   └── <fitur>/     api/ states/ pages/ components/ modals/ layouts/
│       ├── helpers/         apiHelper, toolsHelper, actionHelper
│       ├── hooks/           useInput
│       ├── middleware/      penjaga rute global
│       ├── routes.ts        deklarasi rute
│       └── router.options.ts
├── docs/ERD.md              rancangan basis data dan aturan bisnis
├── docker-compose.yml
└── Jenkinsfile
```

Alur data: `route → controller → service → repository → SQLite` (backend) dan `page → store (Pinia) → api → apiHelper → REST API` (frontend).

## Menjalankan di komputer lokal

Prasyarat: Node.js 20 atau lebih baru (atau Bun untuk frontend).

```bash
git clone https://github.com/<username>/ifs24030-pabwe2026-p1.git
cd ifs24030-pabwe2026-p1

# 1. Backend  (http://localhost:8000, dokumentasi http://localhost:8000/api/docs)
cd backend
cp .env.example .env
npm install
npm run dev

# 2. Frontend (terminal baru, http://localhost:3000)
cd frontend
cp .env.example .env
npm install        # atau: bun install
npm run dev        # atau: bun run dev
```

Database SQLite dibuat otomatis di `backend/data/` dan diisi data contoh saat pertama kali server berjalan.

**Akun demo**

| Role | Email | Kata sandi |
|---|---|---|
| Admin | admin@bengkelku.id | Admin123! |
| Pelanggan | budi@example.com | Customer123 |

Reset data contoh: `npm run seed` di folder `backend`.

## Pengujian

```bash
cd backend  && npm run lint && npm run test:coverage
cd frontend && npm run test:coverage     # lapisan logika wajib 100%
```

## Menjalankan dengan Docker

```bash
docker compose up -d --build
# Frontend http://localhost:3000, API http://localhost:8000/api
```

Ganti `JWT_SECRET` dan `ADMIN_PASSWORD` lewat environment sebelum dipakai di server publik.

## Deployment online

- **Backend:** host di layanan yang mendukung *persistent volume* (Railway, Fly.io, atau VPS), karena SQLite menyimpan data di file. Set `NODE_ENV=production`, `JWT_SECRET`, `CORS_ORIGINS` (URL frontend), dan `DB_PATH` ke lokasi volume.
- **Frontend:** SPA statis. Build dengan `VITE_API_BASEURL=https://<url-backend>/api npx nuxt generate`, lalu publikasikan `.output/public` ke Netlify, Vercel, atau Nginx. `public/_redirects` dan `vercel.json` sudah menangani fallback rute SPA.

## Jenkins

`Jenkinsfile` menjalankan: checkout, lint dan test backend, test dan build frontend, lalu deploy dengan `docker compose`. Laporan coverage diarsipkan sebagai artefak build. Buat job tipe *Pipeline from SCM* yang menunjuk ke repository ini.

## Ringkasan endpoint

| Grup | Endpoint |
|---|---|
| Auth | `POST /api/auth/register`, `POST /api/auth/login` |
| Pengguna | `GET/PUT /api/users/me`, `PUT /api/users/me/password`, `GET /api/users` (admin) |
| Kendaraan | `GET/POST /api/vehicles`, `GET/PUT/DELETE /api/vehicles/:id`, `GET /api/vehicles/:id/history` |
| Layanan | `GET /api/services`, `POST/PUT/DELETE /api/services/:id` (admin) |
| Booking | `GET /api/bookings/availability`, `GET/POST /api/bookings`, `GET /api/bookings/:id`, `PATCH /api/bookings/:id/cancel`, `PATCH /api/bookings/:id/status` (admin), `POST /api/bookings/:id/complete` (admin) |
| Riwayat | `GET /api/history`, `GET /api/history/:id` (data invoice) |
| Lainnya | `GET /api/reminders`, `GET /api/stats/dashboard`, `GET /api/health` |

Dokumentasi lengkap dan dapat dicoba langsung ada di `/api/docs`.
