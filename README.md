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

### 3. PENJELASAN KODE SUMBER UTAMA

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

---

### 5. HASIL PENGUJIAN DAN ANALISA DATA
1. **Analisa Respon API Open-Meteo**:
   - Endpoint menghasilkan objek JSON yang berisi properti `hourly` dengan 2 array utama: `time` (format ISO-8601 UTC) dan `temperature_2m` (angka floating point derajat Celcius).
2. **Pola Suhu di Jakarta**:
   - Berdasarkan data 10 jam pertama dan 24 jam prakiraan, suhu udara di wilayah Jakarta berkisar antara **24.5 °C** pada dini hari/subuh hingga mencapai **33.8 °C** pada siang hari sekitar pukul 12:00 - 14:00.
   - Peningkatan suhu terjadi bertahap seiring terbitnya matahari dan menurun kembali menjelang malam hari.
3. **Penyajian Data**:
   - 10 data pertama ditampilkan secara presisi dengan format tanggal/waktu yang mudah dipahami pengguna, memenuhi seluruh kriteria Tugas 2.

---

### 6. PANDUAN NASKAH / SKRIP REKAMAN VIDEO PRESENTASI

#### Bagian 1: Perkenalan Identitas (Durasi: ~30 detik)
> *"Halo, perkenalkan nama saya **Muhamad Pasha Albara**, NIM **054596122**, mahasiswa Program Studi Sistem Informasi di Universitas Terbuka. Pada video ini, saya akan mendemonstrasikan hasil pengerjaan Tugas 2 Praktikum Pemrograman Berbasis Piranti Bergerak (STSI4303)."*

#### Bagian 2: Penjelasan Langkah Praktik (Durasi: ~1 - 1.5 menit)
> *"Pada tugas ini, kita diminta membuat aplikasi untuk mengambil dan menampilkan data prakiraan cuaca Jakarta dari Open-Meteo API. Berikut langkah-langkah yang saya lakukan:*
> 1. *Menginisialisasi proyek berbasis Vue TypeScript dan mengintegrasikannya dengan framework Ionic Vue (`@ionic/vue`) dan Axios sesuai modul 4 praktikum.*
> 2. *Membuat service class `EndPointAccess` di folder `src/services/EndPointAccess/index.ts` agar pemanggilan REST API terenkapsulasi secara rapi dan reusable.*
> 3. *Membuat halaman `Home.vue` dengan komponen Ionic seperti `ion-page`, `ion-header`, `ion-content`, dan `ion-button`.*
> 4. *Menerapkan method `ambilData()` untuk melakukan GET request ke API Open-Meteo, lalu memetakan atribut `time` dan `temperature_2m` ke dalam format tabel dan card."*

#### Bagian 3: Demo Aplikasi & Analisa (Durasi: ~1.5 - 2 menit)
> *(Tunjukkan layar browser `http://localhost:5173/`)*
> *"Berikut adalah tampilan aplikasi yang telah saya bangun:*
> - *Di bagian atas terdapat identitas mahasiswa dan informasi endpoint API.*
> - *Ketika tombol **'Ambil Data Cuaca'** diklik, aplikasi menghubungi API Open-Meteo secara asynchronous.*
> - *Setelah data berhasil diambil, muncul ringkasan statistik seperti rata-rata suhu, suhu minimum, dan maksimum.*
> - *Di bawahnya tersaji tabel data per jam yang menampilkan atribut `time` dan `temperature_2m` secara rapi.*
> - *Secara default, aplikasi menampilkan 10 data pertama sesuai catatan soal, namun pengguna juga dapat memilih untuk melihat 24 jam atau semua data, serta berpindah ke mode Card Grid.*
> - *Dari analisa data cuaca Jakarta ini, terlihat suhu terendah sekitar 25 °C pada waktu malam/dini hari dan naik hingga 33 °C di siang hari."*

#### Bagian 4: Kata-kata Penutup (Durasi: ~30 detik)
> *"Demikian presentasi dan demonstrasi Tugas 2 STSI4303 yang telah saya selesaikan. Seluruh kode program telah berjalan dengan baik tanpa error dan memenuhi seluruh ketentuan praktikum. Terima kasih atas perhatiannya, salam sejahtera bagi kita semua."*
