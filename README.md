# LAPORAN PRAKTIKUM TUGAS 2 STSI4303
## PEMROGRAMAN BERBASIS PIRANTI BERGERAK (PBPB) / MSIM4401 MODUL 04

---

### IDENTITAS MAHASISWA
- **Nama Mahasiswa** : Muhamad Pasha Albara
- **NIM** : 054596122
- **Program Studi** : Sistem Informasi
- **Mata Kuliah** : STSI4303 - Pemrograman Berbasis Piranti Bergerak
- **Institusi** : Universitas Terbuka (UT)

---

### 1. DESKRIPSI TUGAS
Membuat proyek aplikasi mobile/web berbasis Ionic & Vue untuk menampilkan data cuaca dari endpoint RESTful API online:
- **Alamat API**: `https://api.open-meteo.com/v1/forecast?latitude=-6.2&longitude=106.8&hourly=temperature_2m`
- **Lokasi Prakiraan**: Jakarta, Indonesia (Latitude: -6.2°, Longitude: 106.8°)
- **Field Data yang Ditampilkan**:
  1. `time` (Waktu pengukuran prakiraan cuaca per jam)
  2. `temperature_2m` (Suhu udara permukaan per jam dalam satuan °C)
- **Ketentuan Desain (Standar Antislop UI)**:
  - Antarmuka berorientasi data riil, bebas dari ornamen klise AI (tanpa emoji dekoratif, tanpa gradien gelap artifisial, dan tanpa shadow mengambang).
  - Menggunakan palet netral terarah (Slate & Off-white) dengan kontras teks tinggi.
  - Sesuai kaidah arsitektur Modul 4 Aktivitas 8 UT: memisahkan pemanggilan data ke dalam class service `EndPointAccess` menggunakan library Axios.
  - Default menampilkan 10 data pertama (sesuai catatan soal), dengan opsi interaktif untuk melihat 24 jam atau seluruh data, serta pilihan mode Tabel dan Kartu.
  - Format angka dan waktu menggunakan `font-variant-numeric: tabular-nums` dan font monospace untuk presisi perbandingan data per jam.

---

### 2. STRUKTUR DIREKTORI PROYEK
Struktur direktori proyek mengikuti standar Modul 4 Aktivitas 8:
```text
C:\PraktikumUT\STSI4303\Tugas2
│
├── index.html                           # Entry point HTML aplikasi
├── package.json                         # Dependensi (Ionic Vue, Vue Router, Axios, TypeScript)
├── tsconfig.json / tsconfig.app.json    # Konfigurasi TypeScript & path alias (@)
├── vite.config.ts                       # Konfigurasi build Vite & resolve alias (@ -> src)
│
└── src/
    ├── App.vue                          # Komponen root aplikasi (IonApp & IonRouterOutlet)
    ├── main.ts                          # Inisialisasi IonicVue, CSS Ionic, dan router
    ├── router/
    │   └── index.ts                     # Definisi rute rute /home menggunakan Ionic Router
    ├── services/
    │   └── EndPointAccess/
    │       └── index.ts                 # Class service Axios (EndPointAccess)
    ├── theme/
    │   └── variables.css                # Variabel styling & palet warna Ionic
    └── views/
        └── Home.vue                     # Tampilan utama: header, tombol fetch, tabel cuaca
```

---

### 3. PENJELASAN KODE

#### A. Service Akses API (`src/services/EndPointAccess/index.ts`)
Sesuai modul praktikum UT, pemanggilan API dienkapsulasi dalam class `EndPointAccess` agar kode bersifat *reusable* dan *scalable*:
```typescript
import axios from 'axios';

export default class EndPointAccess {
  theUrl: string;

  constructor(url: string) {
    this.theUrl = url;
  }

  async getRes() {
    const response = await axios.get(this.theUrl);
    return response;
  }
}
```

#### B. Router Aplikasi (`src/router/index.ts`)
Mengatur navigasi rute utama mengarah ke halaman `Home.vue`:
```typescript
import { createRouter, createWebHistory } from '@ionic/vue-router';
import type { RouteRecordRaw } from 'vue-router';
import Home from '../views/Home.vue';

const routes: Array<RouteRecordRaw> = [
  { path: '/', redirect: '/home' },
  { path: '/home', name: 'Home', component: Home }
];

const router = createRouter({
  history: createWebHistory(import.meta.env.BASE_URL),
  routes
});

export default router;
```

#### C. Halaman Tampilan (`src/views/Home.vue`)
- Mengambil data dari endpoint Open-Meteo saat tombol **"Ambil Data Cuaca"** diklik.
- Mengubah struktur data dari array paralel (`hourly.time` dan `hourly.temperature_2m`) menjadi objek array `{ time, temperature_2m }`.
- Menampilkan data dalam tabel dengan styling modern, badge status suhu (Sejuk, Normal/Hangat, Panas), serta ringkasan metrik (Rata-rata, Min, Max).

---

### 4. CARA MENJALANKAN APLIKASI
1. Buka terminal PowerShell pada direktori proyek:
   ```powershell
   cd C:\PraktikumUT\STSI4303\Tugas2
   ```
2. Jalankan server lokal:
   ```powershell
   npm run dev
   ```
3. Buka browser pada alamat:
   ```
   http://localhost:5173/
   ```
4. Klik tombol **"Ambil Data Cuaca"** untuk melihat data tabel cuaca Jakarta.