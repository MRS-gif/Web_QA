# QA Workspace — Klinik Pratama Mandiri

Workspace Quality Assurance untuk aplikasi web **Klinik Pratama Mandiri** (Laravel + MySQL):
antrean daring, Rekam Medis Elektronik (RME), dan stok obat. Pengujian memakai
**Playwright (Chromium)** dengan skill Claude Code *Application Discovery* dan *Website QA Automation*.

> Semua angka dan berkas bukti di dokumen ini berasal dari eksekusi di komputer penyusun.
> Bagian bertanda `[ISI]` diisi dari `reports/RINGKASAN_HASIL.md` setelah `npm test`.


---

## 1. Ruang lingkup
| Item | Isi |
|---|---|
| Aplikasi target | Klinik Pratama Mandiri (repo web klinik: [tautan repo web]) |
| Modul diuji | M02 Antrean, M04 Resep & Stok Obat (+ M01 Autentikasi sebagai pendukung) |
| Alur bisnis | Booking → panggil dan periksa → resep → stok berkurang → serahkan obat |
| Peran | Admin, Dokter (2 akun), Apoteker, Pasien |
| Dokumen acuan | `klinik/SPESIFIKASI_KLINIK.md` (REQ-01 s.d. REQ-14) |
| Lingkungan | Lokal: Laragon, PHP, MySQL, `php artisan serve` |

## 2. Panduan menjalankan

### 2.1 Prasyarat
Node.js ≥ 20.6, Git, Python (untuk pembuat dokumen), Laragon (PHP dan MySQL aktif), dan Claude Code.

### 2.2 Jalankan web klinik (terminal 1, Terminal Laragon)
```
cd C:\laragon\www\klinik
php artisan migrate:fresh --seed
php artisan serve
```
Aplikasi tersedia di `http://127.0.0.1:8000`. `migrate:fresh` hanya untuk database pengembangan,
karena menghapus seluruh tabel.

### 2.3 Siapkan workspace QA (terminal 2)
```
cd C:\laragon\www\klinik\QA
npm run setup
copy .env.example .env
```
Isi `.env` (jangan di-commit). Nama variabel mengikuti `.env.example`; nilai akun diambil dari akun demo:
```
BASE_URL=http://127.0.0.1:8000
APP_NAME=Klinik Pratama Mandiri
TEST_ENV=local
ADMIN_USER=admin@klinik.test
ADMIN_PASS=<lihat SPESIFIKASI_KLINIK.md>
DOKTER_USER=dokter1@klinik.test
DOKTER_PASS=<...>
DOKTER2_USER=dokter2@klinik.test
DOKTER2_PASS=<...>
APOTEKER_USER=apoteker@klinik.test
APOTEKER_PASS=<...>
PASIEN_USER=pasien@klinik.test
PASIEN_PASS=<...>
```
Pasang skill bila belum ada: salin `.claude/skills/*` (lihat bagian Troubleshooting di modul).

### 2.4 Validasi environment
```
npm run test:env
```
Hasil yang benar: **TC000 lulus** dan muncul `screenshots/TC000_..._PASS.png`.

## 3. Eksekusi pengujian
| Tujuan | Perintah |
|---|---|
| Reset data uji | `php artisan migrate:fresh --seed` (di root proyek klinik) |
| Semua test (headless) | `npm test` |
| Lihat browser bergerak (untuk video) | `npm run test:headed` |
| Satu berkas test | `npx playwright test tests/<nama-berkas>.spec.js` |
| Laporan HTML Playwright | `npm run report:html` |
| Bersihkan hasil run | `npm run clean` |

Simpan sesi login (login manual di browser, kata sandi diketik sendiri):
```
npm run auth:save -- admin
npm run auth:save -- dokter
npm run auth:save -- apoteker
npm run auth:save -- pasien
```
Aturan etika: test yang gagal adalah **temuan**, bukan alasan mengubah expected result.
Tidak ada brute force, eksploitasi, atau load test.

## 4. Laporan
| Berkas | Isi | Cara membuat |
|---|---|---|
| `reports/RINGKASAN_HASIL.md` | Angka PASS/FAIL/SKIP dan daftar bukti per test case, dari hasil nyata | `node scripts/ringkasan.js` |
| `reports/Test_Case_Catalogue.xlsx` | Katalog test case dan traceability | Dibuat Claude Code pada Praktikum 2 |
| `reports/Website_QA_Testing_Report.docx` | Laporan QA lengkap | Praktikum 5 |
| `reports/Website_User_Tutorial.docx` | Tutorial pengguna berbasis screenshot nyata | Praktikum 5 |
| `bugs/bugs.json` | Daftar temuan dan statusnya | Praktikum 4 dan 6 |
| `results/html/` | Laporan HTML Playwright | `npm run report:html` |

Agar `node scripts/ringkasan.js` bisa dipanggil lewat npm, tambahkan satu baris di bagian `scripts` pada `package.json`:
```
"summary": "node scripts/ringkasan.js"
```
lalu jalankan `npm run summary`.

## 5. Bukti pembuktian (evidence)
| Bukti | Lokasi | Keterangan |
|---|---|---|
| Screenshot tiap langkah | `screenshots/` | Pola nama: `TC{NO}_{PERAN}_{FITUR}_STEP{NN}_{STATUS}.png` |
| Screenshot saat gagal | `screenshots/` | Kondisi halaman terakhir ketika test gagal |
| Metadata bukti | `results/evidence.jsonl` | TC, fitur, peran, halaman, waktu, hasil, berkas |
| Log console dan network | `results/logs.jsonl` | Per test case |
| Hasil mesin | `results/test-results.json` | Dasar angka di ringkasan |
| Trace Playwright | `results/artifacts/.../trace.zip` | `npx playwright show-trace <trace.zip>` |
| Arsip run lama | `archive/<waktu>/` | Dibuat otomatis sebelum tiap eksekusi |
| Video demo | [tautan video] | `npm run test:headed`, maks. 5 menit, plus penjelasan satu temuan |

Kata sandi pada screenshot otomatis ditutup (masking). `.env` dan `auth/` tidak boleh di-commit.

## 6. Ringkasan hasil terbaru
Status per **25 November 2026**. Angka berasal dari keluaran nyata `npm run test:env`, `npm run discover`, dan `npm run auth:save`.
Setelah `npm test` selesai, perbarui bagian ini dari `reports/RINGKASAN_HASIL.md`.

### 6.1 Status tahapan
| Tahap | Status | Bukti |
|---|---|---|
| Validasi environment (TC000) | Selesai, lulus | `screenshots/TC000_..._PASS.png`, `results/` |
| Penyimpanan sesi login 4 peran (admin, dokter, apoteker, pasien) | Selesai | `auth/*.json` (rahasia, tidak di-commit) |
| Application Discovery: crawler | Selesai | `discovery/data/discovery-*.json`, `discovery/screenshots/` |
| Discovery Gate 1 sampai 5 (dokumen, handover) | [ISI: Selesai / Belum] | `discovery/documents/` |
| Test plan dan katalog test case | [ISI: Selesai / Belum] | `reports/Test_Case_Catalogue.xlsx` |
| Eksekusi `npm test` | [ISI: Selesai / Belum] | `results/test-results.json` |
| Bug report dan laporan DOCX | [ISI: Selesai / Belum] | `bugs/bugs.json`, `reports/` |

### 6.2 Hasil eksekusi test
| Parameter | Nilai | Keterangan |
|---|---|---|
| Lingkungan | `http://127.0.0.1:8000`, `local`, Chromium desktop | Laragon, `php artisan serve` |
| Total test case yang sudah dieksekusi | 1 | Hanya TC000 |
| Lulus (PASS) | 1 | TC000 Validasi environment: aplikasi dapat diakses |
| Gagal (FAIL) | 0 | |
| Dilewati (SKIP/BLOCKED) | 0 | |
| Durasi eksekusi | 9,1 detik | |
| Temuan bug | [ISI setelah `npm test`] | `bugs/bugs.json` |

> Angka di 6.2 baru mencakup validasi environment. Setelah test fungsional dijalankan,
> ganti seluruh tabel dengan isi `reports/RINGKASAN_HASIL.md`.

### 6.3 Hasil discovery (crawler read-only)
| Cakupan | Halaman ditemukan | Berkas |
|---|---|---|
| Pengunjung (tanpa login) | 3: `/` (dialihkan ke `/login`), `/daftar`, `/login` | `discovery-public.json` |
| Admin | 2: `/admin/antrean`, `/admin/walkin` | `discovery-admin.json` |
| Dokter | 2: `/dokter/antrean`, `/dokter/rekam-medis` | `discovery-dokter.json` |
| Apoteker | 2: `/apotek/resep`, `/apotek/obat` | `discovery-apoteker.json` |
| Pasien | 2: `/pasien`, `/pasien/booking` | `discovery-pasien.json` |

Crawler hanya mengikuti tautan `<a href>`. Halaman yang bergantung pada data atau aksi form
(misalnya halaman pemeriksaan dokter dan detail rekam medis) belum terlihat dan perlu diuji lewat test otomatis.

**Pengamatan nyata:** saat dijalankan tanpa sesi login, permintaan ke `/admin/antrean`, `/dokter/antrean`,
`/apotek/resep`, dan `/pasien` seluruhnya dialihkan ke `/login`. Ini mendukung REQ-03 (halaman terlindungi),
tetapi masih berstatus *Observed* sampai dibuktikan oleh test otomatis.

## 7. Matriks traceability

### 7.1 Matriks halaman dan peran (dari discovery)
| Page ID | URL | Peran yang menemukan | Status bukti |
|---|---|---|---|
| P-PUB-01 | `/login` | Pengunjung | Observed |
| P-PUB-02 | `/daftar` | Pengunjung | Observed |
| P-ADM-01 | `/admin/antrean` | Admin | Observed |
| P-ADM-02 | `/admin/walkin` | Admin | Observed |
| P-DOK-01 | `/dokter/antrean` | Dokter | Observed |
| P-DOK-02 | `/dokter/rekam-medis` | Dokter | Observed |
| P-APT-01 | `/apotek/resep` | Apoteker | Observed |
| P-APT-02 | `/apotek/obat` | Apoteker | Observed |
| P-PSN-01 | `/pasien` | Pasien | Observed |
| P-PSN-02 | `/pasien/booking` | Pasien | Observed |

ID halaman di atas adalah penomoran sementara untuk README ini. Samakan dengan Page ID di `discovery/documents/` bila berbeda.

## 8. Struktur folder
```
QA/
├── .claude/skills/   # skill Discovery dan QA Automation
├── klinik/           # SPESIFIKASI_KLINIK.md
├── discovery/        # data, screenshots, documents, diagrams, handover
├── tests/            # skrip Playwright
├── reports/          # laporan DOCX, XLSX, RINGKASAN_HASIL.md
├── screenshots/      # bukti visual
├── results/          # hasil runner, evidence.jsonl, logs
├── bugs/             # bugs.json
├── scripts/          # pembuat dokumen dan ringkasan.js
├── lib/              # fixtures, env loader, masking
├── .env              # TIDAK di-commit
└── playwright.config.js
```
