<template>
  <ion-page>
    <ion-header class="ion-no-border">
      <ion-toolbar class="custom-navbar">
        <div class="navbar-content">
          <div class="brand-block">
            <span class="institution-title">Universitas Terbuka</span>
            <h1 class="page-title">Prakiraan Cuaca Jakarta (STSI4303)</h1>
          </div>
          <div class="user-meta">
            <span class="user-name">Muhamad Pasha Albara</span>
            <span class="user-id">NIM: 054596122</span>
          </div>
        </div>
      </ion-toolbar>
    </ion-header>

    <ion-content :fullscreen="true" class="page-content">
      <main class="content-wrapper">
        <!-- Panel Konteks & Parameter API -->
        <section class="context-panel" aria-label="Informasi Sumber Data">
          <div class="context-details">
            <div class="context-item">
              <span class="context-label">Lokasi Pengukuran</span>
              <span class="context-value">Jakarta, Indonesia (Lat: -6.2, Lon: 106.8)</span>
            </div>
            <div class="context-item">
              <span class="context-label">Parameter Diambil</span>
              <span class="context-value">time, temperature_2m (°C)</span>
            </div>
            <div class="context-item full-width">
              <span class="context-label">REST API Endpoint</span>
              <code class="endpoint-code">https://api.open-meteo.com/v1/forecast?latitude=-6.2&longitude=106.8&hourly=temperature_2m</code>
            </div>
          </div>

          <div class="action-bar">
            <button 
              type="button" 
              class="primary-btn" 
              :disabled="isLoading" 
              @click="ambilData"
            >
              <span v-if="!isLoading">Ambil Data Cuaca</span>
              <span v-else>Mengambil Data...</span>
            </button>
          </div>
        </section>

        <!-- Ringkasan Metrik Data Cuaca -->
        <section v-if="weatherList.length > 0" class="metrics-row" aria-label="Ringkasan Statistik Cuaca">
          <div class="metric-card">
            <span class="metric-label">Rata-rata Suhu</span>
            <span class="metric-value">{{ avgTemp }} <span class="metric-unit">°C</span></span>
          </div>
          <div class="metric-card">
            <span class="metric-label">Suhu Terendah</span>
            <span class="metric-value">{{ minTemp }} <span class="metric-unit">°C</span></span>
          </div>
          <div class="metric-card">
            <span class="metric-label">Suhu Tertinggi</span>
            <span class="metric-value">{{ maxTemp }} <span class="metric-unit">°C</span></span>
          </div>
          <div class="metric-card">
            <span class="metric-label">Jumlah Data Ditampilkan</span>
            <span class="metric-value">{{ displayedData.length }} <span class="metric-unit">Jam</span></span>
          </div>
        </section>

        <!-- Panel Kontrol Tampilan & Filter -->
        <section v-if="weatherList.length > 0" class="filter-toolbar" aria-label="Kontrol Tampilan Data">
          <div class="filter-group">
            <span class="filter-label">Batas Tampilan:</span>
            <div class="segmented-control" role="group" aria-label="Pilihan Jumlah Baris">
              <button 
                type="button" 
                :class="['segment-btn', { active: showLimit === 10 }]" 
                @click="showLimit = 10"
              >
                10 Data Pertama
              </button>
              <button 
                type="button" 
                :class="['segment-btn', { active: showLimit === 24 }]" 
                @click="showLimit = 24"
              >
                24 Jam
              </button>
              <button 
                type="button" 
                :class="['segment-btn', { active: showLimit === 0 }]" 
                @click="showLimit = 0"
              >
                Semua Data ({{ weatherList.length }})
              </button>
            </div>
          </div>

          <div class="filter-group">
            <span class="filter-label">Tata Letak:</span>
            <div class="segmented-control" role="group" aria-label="Pilihan Mode Tampilan">
              <button 
                type="button" 
                :class="['segment-btn', { active: viewMode === 'table' }]" 
                @click="viewMode = 'table'"
              >
                Tabel
              </button>
              <button 
                type="button" 
                :class="['segment-btn', { active: viewMode === 'card' }]" 
                @click="viewMode = 'card'"
              >
                Kartu
              </button>
            </div>
          </div>
        </section>

        <!-- State: Sedang Memuat Data -->
        <div v-if="isLoading" class="state-container" role="status">
          <div class="loading-spinner" aria-hidden="true"></div>
          <p class="state-title">Memuat data prakiraan cuaca...</p>
          <p class="state-description">Menghubungi endpoint Open-Meteo untuk data koordinat Jakarta.</p>
        </div>

        <!-- State: Belum Ada Data -->
        <div v-else-if="weatherList.length === 0 && !errorMessage" class="state-container">
          <h2 class="state-title">Data Cuaca Belum Dimuat</h2>
          <p class="state-description">
            Silakan klik tombol "Ambil Data Cuaca" di atas untuk meminta data JSON dari API dan menampilkannya pada tabel.
          </p>
        </div>

        <!-- State: Terjadi Error -->
        <div v-else-if="errorMessage" class="state-container error-state" role="alert">
          <h2 class="state-title">Gagal Mengambil Data</h2>
          <p class="state-description">{{ errorMessage }}</p>
          <button type="button" class="retry-btn" @click="ambilData">Coba Lagi</button>
        </div>

        <!-- Tampilan Mode Tabel -->
        <section v-else-if="viewMode === 'table'" class="table-card" aria-label="Tabel Data Cuaca Per Jam">
          <div class="table-meta-header">
            <span class="table-caption">Data Prakiraan Suhu Per Jam (Open-Meteo)</span>
            <span class="row-counter">Menampilkan {{ displayedData.length }} dari {{ weatherList.length }} data</span>
          </div>

          <div class="table-scroll">
            <table class="data-table">
              <thead>
                <tr>
                  <th scope="col" class="col-num">No</th>
                  <th scope="col">Waktu Pengukuran (time)</th>
                  <th scope="col">Format Tanggal & Waktu</th>
                  <th scope="col" class="col-temp">Suhu (temperature_2m)</th>
                  <th scope="col">Kategori Suhu</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="(item, index) in displayedData" :key="item.time">
                  <td class="col-num font-mono">{{ index + 1 }}</td>
                  <td class="font-mono">{{ item.time }}</td>
                  <td>{{ formatDateTime(item.time) }}</td>
                  <td class="col-temp font-mono">{{ item.temperature_2m.toFixed(1) }} °C</td>
                  <td>
                    <span :class="['status-tag', getTempTagClass(item.temperature_2m)]">
                      {{ getTempCategory(item.temperature_2m) }}
                    </span>
                  </td>
                </tr>
              </tbody>
            </table>
          </div>
        </section>

        <!-- Tampilan Mode Kartu -->
        <section v-else class="cards-layout" aria-label="Daftar Kartu Cuaca Per Jam">
          <article 
            v-for="(item, index) in displayedData" 
            :key="item.time" 
            class="weather-card"
          >
            <div class="card-head">
              <span class="card-index font-mono">#{{ index + 1 }}</span>
              <span :class="['status-tag', getTempTagClass(item.temperature_2m)]">
                {{ getTempCategory(item.temperature_2m) }}
              </span>
            </div>
            <div class="card-body">
              <div class="temp-figure">
                <span class="temp-num font-mono">{{ item.temperature_2m.toFixed(1) }}</span>
                <span class="temp-deg">°C</span>
              </div>
            </div>
            <div class="card-foot">
              <span class="foot-formatted">{{ formatDateTime(item.time) }}</span>
              <code class="foot-iso font-mono">{{ item.time }}</code>
            </div>
          </article>
        </section>
      </main>
    </ion-content>
  </ion-page>
</template>

<script lang="ts">
import { 
  IonPage, 
  IonHeader, 
  IonToolbar, 
  IonContent 
} from '@ionic/vue';
import { defineComponent } from 'vue';
import EndPointAccess from '@/services/EndPointAccess';

interface WeatherItem {
  time: string;
  temperature_2m: number;
}

let resData: EndPointAccess;

export default defineComponent({
  name: 'Home',
  components: {
    IonPage,
    IonHeader,
    IonToolbar,
    IonContent
  },
  data() {
    return {
      weatherList: [] as WeatherItem[],
      isLoading: false,
      errorMessage: '',
      showLimit: 10,
      viewMode: 'table' as 'table' | 'card'
    };
  },
  computed: {
    displayedData(): WeatherItem[] {
      if (this.showLimit === 0) {
        return this.weatherList;
      }
      return this.weatherList.slice(0, this.showLimit);
    },
    avgTemp(): string {
      if (this.displayedData.length === 0) return '0.0';
      const sum = this.displayedData.reduce((acc, curr) => acc + curr.temperature_2m, 0);
      return (sum / this.displayedData.length).toFixed(1);
    },
    minTemp(): string {
      if (this.displayedData.length === 0) return '0.0';
      const min = Math.min(...this.displayedData.map(d => d.temperature_2m));
      return min.toFixed(1);
    },
    maxTemp(): string {
      if (this.displayedData.length === 0) return '0.0';
      const max = Math.max(...this.displayedData.map(d => d.temperature_2m));
      return max.toFixed(1);
    }
  },
  methods: {
    async ambilData() {
      this.isLoading = true;
      this.errorMessage = '';
      try {
        const url = 'https://api.open-meteo.com/v1/forecast?latitude=-6.2&longitude=106.8&hourly=temperature_2m';
        resData = new EndPointAccess(url);
        const response = await resData.getRes();

        if (response && response.data && response.data.hourly) {
          const times: string[] = response.data.hourly.time;
          const temps: number[] = response.data.hourly.temperature_2m;
          
          this.weatherList = times.map((t, idx) => ({
            time: t,
            temperature_2m: temps[idx]
          }));
        } else {
          this.errorMessage = 'Format data dari respons API tidak valid.';
        }
      } catch (err: any) {
        console.error('Koneksi API gagal:', err);
        this.errorMessage = err.message || 'Gagal terhubung ke layanan Open-Meteo.';
      } finally {
        this.isLoading = false;
      }
    },
    formatDateTime(isoString: string): string {
      if (!isoString) return '-';
      const date = new Date(isoString);
      return date.toLocaleString('id-ID', {
        day: '2-digit',
        month: 'short',
        year: 'numeric',
        hour: '2-digit',
        minute: '2-digit'
      });
    },
      getTempCategory(temp: number): string {
        if (temp < 25) return 'Sejuk';
        if (temp <= 30) return 'Normal';
        return 'Hangat';
      },
    getTempTagClass(temp: number): string {
      if (temp < 25) return 'tag-cool';
      if (temp <= 30) return 'tag-normal';
      return 'tag-warm';
    }
  }
});
</script>

<style scoped>
/* Page Layout */
.page-content {
  --background: #f8fafc;
}

.custom-navbar {
  --background: #ffffff;
  border-bottom: 1px solid #e2e8f0;
  padding: 0.25rem 0;
}

.navbar-content {
  display: flex;
  justify-content: space-between;
  align-items: center;
  max-width: 1040px;
  margin: 0 auto;
  padding: 0.75rem 1rem;
}

.institution-title {
  display: block;
  font-size: 0.75rem;
  color: #64748b;
  font-weight: 600;
  letter-spacing: 0.02em;
  text-transform: uppercase;
  margin-bottom: 2px;
}

.page-title {
  font-size: 1.125rem;
  font-weight: 700;
  color: #0f172a;
  margin: 0;
  line-height: 1.3;
}

.user-meta {
  text-align: right;
  display: flex;
  flex-direction: column;
}

.user-name {
  font-size: 0.875rem;
  font-weight: 600;
  color: #0f172a;
}

.user-id {
  font-size: 0.75rem;
  color: #64748b;
  font-family: ui-monospace, SFMono-Regular, Menlo, monospace;
}

.content-wrapper {
  max-width: 1040px;
  margin: 0 auto;
  padding: 1.5rem 1rem 3rem 1rem;
  display: flex;
  flex-direction: column;
  gap: 1.25rem;
}

/* Context Panel */
.context-panel {
  background: #ffffff;
  border: 1px solid #e2e8f0;
  border-radius: 8px;
  padding: 1.25rem;
  display: flex;
  justify-content: space-between;
  align-items: flex-end;
  gap: 1.25rem;
  flex-wrap: wrap;
}

.context-details {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(220px, 1fr));
  gap: 1rem;
  flex: 1;
}

.context-details .full-width {
  grid-column: 1 / -1;
}

.context-item {
  display: flex;
  flex-direction: column;
  gap: 0.25rem;
}

.context-label {
  font-size: 0.75rem;
  font-weight: 600;
  color: #64748b;
  text-transform: uppercase;
  letter-spacing: 0.04em;
}

.context-value {
  font-size: 0.875rem;
  color: #0f172a;
  font-weight: 500;
}

.endpoint-code {
  font-size: 0.8rem;
  background: #f1f5f9;
  color: #0f172a;
  padding: 0.4rem 0.6rem;
  border-radius: 4px;
  border: 1px solid #e2e8f0;
  word-break: break-all;
  font-family: ui-monospace, SFMono-Regular, Menlo, monospace;
}

.action-bar {
  display: flex;
  align-items: center;
}

.primary-btn {
  background: #0f172a;
  color: #ffffff;
  border: 1px solid #0f172a;
  border-radius: 6px;
  padding: 0.625rem 1.25rem;
  font-size: 0.875rem;
  font-weight: 600;
  cursor: pointer;
  transition: background-color 0.15s ease;
  white-space: nowrap;
}

.primary-btn:hover:not(:disabled) {
  background: #1e293b;
}

.primary-btn:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

/* Metrics Row */
.metrics-row {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(180px, 1fr));
  gap: 1rem;
}

.metric-card {
  background: #ffffff;
  border: 1px solid #e2e8f0;
  border-radius: 8px;
  padding: 1rem;
  display: flex;
  flex-direction: column;
  gap: 0.35rem;
}

.metric-label {
  font-size: 0.75rem;
  font-weight: 600;
  color: #64748b;
  text-transform: uppercase;
  letter-spacing: 0.04em;
}

.metric-value {
  font-size: 1.5rem;
  font-weight: 700;
  color: #0f172a;
  font-variant-numeric: tabular-nums;
}

.metric-unit {
  font-size: 0.875rem;
  font-weight: 500;
  color: #64748b;
}

/* Filter Toolbar */
.filter-toolbar {
  background: #ffffff;
  border: 1px solid #e2e8f0;
  border-radius: 8px;
  padding: 0.75rem 1rem;
  display: flex;
  justify-content: space-between;
  align-items: center;
  flex-wrap: wrap;
  gap: 1rem;
}

.filter-group {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  flex-wrap: wrap;
}

.filter-label {
  font-size: 0.8125rem;
  font-weight: 600;
  color: #475569;
}

.segmented-control {
  display: inline-flex;
  background: #f1f5f9;
  border-radius: 6px;
  padding: 2px;
  border: 1px solid #e2e8f0;
}

.segment-btn {
  background: transparent;
  border: none;
  padding: 0.35rem 0.75rem;
  font-size: 0.8125rem;
  font-weight: 500;
  color: #475569;
  border-radius: 4px;
  cursor: pointer;
  transition: all 0.15s ease;
}

.segment-btn:hover {
  color: #0f172a;
}

.segment-btn.active {
  background: #ffffff;
  color: #0f172a;
  font-weight: 600;
  box-shadow: 0 1px 2px rgba(0, 0, 0, 0.05);
}

/* States */
.state-container {
  background: #ffffff;
  border: 1px solid #e2e8f0;
  border-radius: 8px;
  padding: 3rem 1.5rem;
  text-align: center;
}

.state-title {
  font-size: 1.125rem;
  font-weight: 700;
  color: #0f172a;
  margin: 0 0 0.5rem 0;
}

.state-description {
  font-size: 0.875rem;
  color: #64748b;
  max-width: 500px;
  margin: 0 auto;
  line-height: 1.5;
}

.loading-spinner {
  width: 32px;
  height: 32px;
  border: 3px solid #e2e8f0;
  border-top-color: #0f172a;
  border-radius: 50%;
  animation: spin 0.75s linear infinite;
  margin: 0 auto 1rem auto;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}

.error-state {
  border-color: #fecdd3;
  background: #fff1f2;
}

.error-state .state-title {
  color: #be123c;
}

.error-state .state-description {
  color: #9f1239;
}

.retry-btn {
  margin-top: 1rem;
  background: #be123c;
  color: #ffffff;
  border: none;
  padding: 0.5rem 1rem;
  font-size: 0.8125rem;
  font-weight: 600;
  border-radius: 6px;
  cursor: pointer;
}

/* Table Card */
.table-card {
  background: #ffffff;
  border: 1px solid #e2e8f0;
  border-radius: 8px;
  overflow: hidden;
}

.table-meta-header {
  padding: 0.875rem 1.25rem;
  display: flex;
  justify-content: space-between;
  align-items: center;
  border-bottom: 1px solid #e2e8f0;
  background: #ffffff;
}

.table-caption {
  font-size: 0.875rem;
  font-weight: 600;
  color: #0f172a;
}

.row-counter {
  font-size: 0.75rem;
  color: #64748b;
}

.table-scroll {
  width: 100%;
  overflow-x: auto;
}

.data-table {
  width: 100%;
  border-collapse: collapse;
  text-align: left;
  font-size: 0.875rem;
}

.data-table th {
  background: #f8fafc;
  color: #475569;
  font-weight: 600;
  font-size: 0.75rem;
  text-transform: uppercase;
  letter-spacing: 0.04em;
  padding: 0.75rem 1rem;
  border-bottom: 1px solid #e2e8f0;
}

.data-table td {
  padding: 0.75rem 1rem;
  border-bottom: 1px solid #f1f5f9;
  color: #1e293b;
}

.data-table tbody tr:last-child td {
  border-bottom: none;
}

.data-table tbody tr:hover {
  background: #f8fafc;
}

.col-num {
  width: 56px;
  text-align: center;
  color: #64748b;
}

.col-temp {
  font-weight: 600;
  color: #0f172a;
}

.font-mono {
  font-family: ui-monospace, SFMono-Regular, Menlo, monospace;
  font-variant-numeric: tabular-nums;
}

/* Status Tags */
.status-tag {
  display: inline-block;
  padding: 0.2rem 0.5rem;
  font-size: 0.75rem;
  font-weight: 600;
  border-radius: 4px;
  border: 1px solid transparent;
}

.tag-cool {
  background: #f0f9ff;
  color: #0369a1;
  border-color: #bae6fd;
}

.tag-normal {
  background: #f8fafc;
  color: #334155;
  border-color: #cbd5e1;
}

.tag-warm {
  background: #fff7ed;
  color: #c2410c;
  border-color: #fed7aa;
}

/* Card Mode */
.cards-layout {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(210px, 1fr));
  gap: 1rem;
}

.weather-card {
  background: #ffffff;
  border: 1px solid #e2e8f0;
  border-radius: 8px;
  padding: 1rem;
  display: flex;
  flex-direction: column;
  gap: 0.75rem;
}

.card-head {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.card-index {
  font-size: 0.75rem;
  font-weight: 600;
  color: #64748b;
}

.card-body {
  padding: 0.25rem 0;
}

.temp-figure {
  display: flex;
  align-items: baseline;
  gap: 0.25rem;
}

.temp-num {
  font-size: 2rem;
  font-weight: 700;
  color: #0f172a;
  line-height: 1;
}

.temp-deg {
  font-size: 1rem;
  font-weight: 500;
  color: #64748b;
}

.card-foot {
  border-top: 1px solid #f1f5f9;
  padding-top: 0.5rem;
  display: flex;
  flex-direction: column;
  gap: 0.2rem;
}

.foot-formatted {
  font-size: 0.8125rem;
  font-weight: 500;
  color: #334155;
}

.foot-iso {
  font-size: 0.7rem;
  color: #64748b;
}

/* Responsive adjustment */
@media (max-width: 640px) {
  .navbar-content {
    flex-direction: column;
    align-items: flex-start;
    gap: 0.5rem;
  }
  .user-meta {
    text-align: left;
  }
  .context-panel {
    align-items: stretch;
  }
  .primary-btn {
    width: 100%;
  }
}
</style>
