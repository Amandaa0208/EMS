# Dokumentasi User Flow - EMS Enterprise

Dokumen ini menjelaskan alur perjalanan pengguna (User Flow) saat menggunakan sistem **EMS Enterprise (Sistem Pemantauan Energi Gedung)**. Alur ini memandu bagaimana pengguna berinteraksi dengan antarmuka Streamlit, mengalirkan data, dan mengoperasikan fitur-fitur analisis dari halaman awal hingga pengaturan administrasi.

---

## 1. Peta Navigasi Global (Global User Flow)

Berikut adalah bagan alir navigasi utama yang menggambarkan pintu masuk pengguna ke dalam aplikasi dan opsi menu yang tersedia setelah berhasil login.

```mermaid
graph TD
    %% Styling
    classDef startStyle fill:#0058be,stroke:#001a42,stroke-width:2px,color:#ffffff,font-weight:bold;
    classDef stepStyle fill:#ffffff,stroke:#0058be,stroke-width:1.5px,color:#0b1c30;
    classDef checkStyle fill:#ff9f1c,stroke:#b36b00,stroke-width:2px,color:#ffffff,font-weight:bold;
    classDef endStyle fill:#ba1a1a,stroke:#410002,stroke-width:2px,color:#ffffff,font-weight:bold;
    
    StartNode(["Mulai: Buka Aplikasi"]):::startStyle
    LoginForm["Form Login: Input Kredensial & Role"]:::stepStyle
    AuthCheck{"Validasi Sukses?"}:::checkStyle
    ShowError["Tampilkan Pesan Error"]:::stepStyle
    
    %% Alur Operasional Linear (Top-to-Bottom)
    LoadDashboard["Dashboard: Memantau Kondisi Energi & Status Baseline"]:::stepStyle
    GoToAnalisa["Analisa Energi: Membandingkan Konsumsi & Unduh CSV"]:::stepStyle
    GoToProfile["Profile Gedung: Monitoring Live Telemetri 3-Phase"]:::stepStyle
    GoToForecast["Forecasting: Memicu Prediksi Beban Listrik (ML)"]:::stepStyle
    GoToReports["Reports & Audit: Mengunduh Laporan Kepatuhan"]:::stepStyle
    GoToSettings["Admin Settings: Update Tarif PLN & Kelola Gedung"]:::stepStyle
    
    Logout["Klik Logout & Hapus Sesi"]:::stepStyle
    EndNode(["Selesai: Sesi Berakhir"]):::endStyle

    %% Hubungan Aliran
    StartNode --> LoginForm
    LoginForm --> AuthCheck
    AuthCheck -- Tidak --> ShowError
    ShowError --> LoginForm
    
    AuthCheck -- Ya --> LoadDashboard
    LoadDashboard --> GoToAnalisa
    GoToAnalisa --> GoToProfile
    GoToProfile --> GoToForecast
    GoToForecast --> GoToReports
    GoToReports --> GoToSettings
    GoToSettings --> Logout
    Logout --> EndNode
```

---

## 2. Alur Detail Per Fitur (Detail Flows)

Berikut adalah rincian langkah demi langkah dari setiap fungsi utama aplikasi:

### A. Alur Otentikasi & Login (Authentication Flow)
Alur ini menjamin bahwa hanya pengguna terdaftar (*Admin* dengan username `admin` atau *Super Admin* dengan username `superadmin`) yang dapat masuk ke panel dashboard.

```mermaid
graph TD
    classDef step fill:#ffffff,stroke:#0058be,stroke-width:1.5px,color:#0b1c30;
    classDef check fill:#ff9f1c,stroke:#b36b00,stroke-width:2px,color:#ffffff,font-weight:bold;

    A[Input Username & Password]:::step --> B[Pilih Role Akses: Admin / Super Admin]:::step
    B --> C[Klik 'Masuk ke Dashboard']:::step
    C --> D{"Apakah Kredensial Valid?"}:::check
    D -- Ya --> E[Set st.session_state['logged_in'] = True]:::step
    E --> F[Inisialisasi Role & Nama Pengguna]:::step
    F --> G[Rerun halaman ke Dashboard Utama]:::step
    D -- Tidak --> H[Tampilkan st.error]:::step
    H --> A
```

---

### B. Alur Analisa Profil Energi (Energy Analysis Flow)
Pada modul **Analisa Energi**, pengguna dapat menyaring data histori penggunaan energi berdasarkan rentang waktu, kategori beban, dan pembanding.

```mermaid
graph TD
    classDef step fill:#ffffff,stroke:#0058be,stroke-width:1.5px,color:#0b1c30;
    classDef check fill:#ff9f1c,stroke:#b36b00,stroke-width:2px,color:#ffffff,font-weight:bold;

    A[Masuk Halaman 'Analisa Energi']:::step --> B[Pilih Periode: 7 Hari / 30 Hari / Bulan Ini / Kustom]:::step
    B --> C{"Periode == Kustom?"}:::check
    C -- Ya --> D[Tampilkan Date Input: Start & End Date]:::step
    C -- Tidak --> E[Pilih Kategori Beban & Benchmark Pembanding]:::step
    D --> E
    E --> F[Klik Tombol 'Terapkan']:::step
    F --> G[Simpan ke st.session_state filter yang diterapkan]:::step
    G --> H[Hitung ulang data kWh & Rupiah sesuai filter]:::step
    H --> I[Perbarui Grafik Garis Plotly & Tabel Breakdown]:::step
    I --> J{Pilihan Tindakan Pengguna}:::check
    J -->|Opsi 1| K[Klik 'Ekspor Data ke CSV']:::step
    J -->|Opsi 2| L[Klik 'Investigasi Detail Selisih']:::step
    K --> K1[Download file .csv hasil kalkulasi]:::step
    L --> L1[Buka overlay modal Wawasan AI / Anomali]:::step
```

---

### C. Alur Pemantauan Detail Telemetri (Building Profile Flow)
Alur ini berjalan secara real-time. Data diperbarui secara dinamis setiap 2 detik menggunakan fitur `st.fragment` dari Streamlit.

```mermaid
graph TD
    classDef step fill:#ffffff,stroke:#0058be,stroke-width:1.5px,color:#0b1c30;
    classDef loop fill:#eff4ff,stroke:#0058be,stroke-width:2px,stroke-dasharray: 5 5,color:#0058be;

    A[Masuk Halaman 'Profile Gedung']:::step --> B[Pilih Gedung dari Dropdown]:::step
    B --> C[Muat data dari tabel SQLite yang relevan]:::step
    
    subgraph RealTimeLoop["Fragment Real-time Loop (Setiap 2 Detik)"]
        direction TB
        D[Ambil baris telemetri terakhir dari DB]:::step
        D --> E[Hitung Tegangan/Arus Phase R, S, T]:::step
        E --> F[Update metrik card: Port, KWH, kW, Frekuensi]:::step
        F --> G[Gambar ulang Grafik Tegangan & Grafik Arus]:::step
        G --> H[Update tabel telemetri log terbaru]:::step
    end
    
    C --> RealTimeLoop
```

---

### D. Alur Forecasting Beban Energi (Machine Learning Flow)
Di modul **Forecasting**, pengguna melatih model *Random Forest* untuk memproyeksikan konsumsi daya listrik gedung di masa mendatang.

```mermaid
graph TD
    classDef step fill:#ffffff,stroke:#0058be,stroke-width:1.5px,color:#0b1c30;
    classDef model fill:#d8e2ff,stroke:#001a42,stroke-width:2px,color:#001a42;

    A[Masuk Halaman 'Forecasting']:::step --> B[Pilih Gedung & Rentang Waktu Prediksi: 24 Jam / 7 Hari]:::step
    B --> C[Muat histori data dari SQLite]:::step
    C --> D[Picu st.spinner 'Melatih Model...']:::step
    
    subgraph ML_Process["Proses Machine Learning"]
        E[Ekstraksi Fitur Waktu: Jam, Hari, Hari dalam Minggu]:::model
        E --> F[Split data ke Train & Test: 80% & 20%]:::model
        F --> G[Latih Random Forest Regressor]:::model
        G --> H[Prediksi Data Test & Evaluasi Skor R2 & RMSE]:::model
        H --> I[Latih ulang model dengan seluruh dataset]:::model
        I --> J[Prediksi beban listrik untuk horizon ke depan]:::model
    end
    
    D --> ML_Process
    ML_Process --> K[Tampilkan metrik evaluasi akurasi R2 & RMSE]:::step
    K --> L[Gambarkan grafik garis perbandingan historis vs prediksi]:::step
    L --> M[Tampilkan Rekomendasi AI Peak-Shaving]:::step
```

---

### E. Alur Pengelolaan Parameter & Gedung (Admin Settings Flow)
Pengaturan administrasi memungkinkan modifikasi parameter tarif, batas baseline, serta struktur kategori dan gedung.

```mermaid
graph TD
    classDef step fill:#ffffff,stroke:#0058be,stroke-width:1.5px,color:#0b1c30;
    classDef db fill:#f1f5f9,stroke:#94a3b8,stroke-width:2px,color:#334155;
    classDef check fill:#ff9f1c,stroke:#b36b00,stroke-width:2px,color:#ffffff,font-weight:bold;

    A[Masuk Halaman 'Admin Settings']:::step --> B{Pilih Blok Konfigurasi}:::check
    
    %% Jalur Tarif
    B -->|1. Tarif PLN| C[Masukkan Nilai Tarif per kWh]:::step
    C --> D[Klik 'Simpan Perubahan Tarif PLN']:::step
    D --> E[Perbarui st.session_state['tarif_pln'] secara global]:::step
    
    %% Jalur Target Baseline
    B -->|2. Target Baseline| F[Pilih Tipe Rentang & Batas kWh]:::step
    F --> G[Klik 'Simpan Target']:::step
    G --> H[Perbarui st.session_state['batas_angka']]:::step
    
    %% Jalur Gedung Baru
    B -->|3. Tambah Gedung| I[Input Nama Gedung & Nomor Port Modbus]:::step
    I --> J[Klik 'Tambah Gedung']:::step
    J --> K[Hubungkan ke ems.db]:::db
    K --> L[Jalankan CREATE TABLE device_port_XXX_readings]:::db
    L --> M[Masukkan baris telemetri inisiasi pertama]:::db
    M --> N[Tambahkan ke st.session_state['gedung_list']]:::step
    N --> O[Picu st.rerun untuk memuat gedung baru]:::step
```

---

## 3. Kode Script PlantUML untuk User Flow (StarUML / PlantText)

Berikut adalah kode script PlantUML (Activity Diagram) yang dapat Anda gunakan di **StarUML**, **PlantText**, atau editor PlantUML lainnya untuk me-render User Flow secara otomatis dengan garis yang lurus dan rapi:

### A. Global User Flow (Alur Navigasi Utama)
```plantuml
@startuml
skinparam roundcorner 10
skinparam ActivityBackgroundColor White
skinparam ActivityBorderColor #0058be
skinparam ArrowColor #0b1c30

start
:Buka Aplikasi;
:Form Login: Input Kredensial & Role;
if (Validasi Sukses?) then (tidak)
  :Tampilkan Pesan Error;
  detach
else (ya)
  :Dashboard: Memantau Kondisi Energi & Status Baseline;
  :Analisa Energi: Membandingkan Konsumsi & Unduh CSV;
  :Profile Gedung: Monitoring Live Telemetri 3-Phase;
  :Forecasting: Memicu Prediksi Beban Listrik (ML);
  :Reports & Audit: Mengunduh Laporan Kepatuhan;
  :Admin Settings: Update Tarif PLN & Kelola Gedung;
  :Klik Logout & Hapus Sesi;
  stop
endif
@enduml
```

### B. Alur Detail Login
```plantuml
@startuml
skinparam roundcorner 10
skinparam ActivityBackgroundColor White
skinparam ActivityBorderColor #0058be
skinparam ArrowColor #0b1c30

start
:Input Username & Password;
:Pilih Role Akses (Admin / Super Admin);
:Klik 'Masuk ke Dashboard';
if (Kredensial Valid?) then (tidak)
  :Tampilkan Error;
  stop
else (ya)
  :Set session_state['logged_in'] = True;
  :Inisialisasi Role & Nama Pengguna;
  :Rerun ke Dashboard Utama;
  stop
endif
@enduml
```

### C. Alur Detail Analisa Energi
```plantuml
@startuml
skinparam roundcorner 10
skinparam ActivityBackgroundColor White
skinparam ActivityBorderColor #0058be
skinparam ArrowColor #0b1c30

start
:Masuk Halaman 'Analisa Energi';
:Pilih Periode;
if (Periode == Kustom?) then (ya)
  :Tampilkan Input Tanggal (Start & End);
else (tidak)
endif
:Pilih Kategori Beban & Benchmark;
:Klik Tombol 'Terapkan';
:Simpan Filter ke Session State;
:Hitung Ulang kWh & Rupiah;
:Perbarui Grafik Plotly & Tabel Breakdown;
split
  :Klik 'Ekspor Data ke CSV';
  :Unduh File CSV;
split currents
  :Klik 'Investigasi Detail Selisih';
  :Buka Modal Wawasan AI / Anomali;
end split
stop
@enduml
```

### D. Alur Detail Profile Gedung
```plantuml
@startuml
skinparam roundcorner 10
skinparam ActivityBackgroundColor White
skinparam ActivityBorderColor #0058be
skinparam ArrowColor #0b1c30

start
:Masuk Halaman 'Profile Gedung';
:Pilih Gedung dari Dropdown;
:Muat Data dari Tabel SQLite;
repeat
  :Ambil Baris Telemetri Terakhir dari DB;
  :Hitung Tegangan & Arus per Phase (R/S/T);
  :Update Metrik Card (KWH, kW, Frekuensi);
  :Gambar Ulang Grafik Tegangan & Arus;
  :Update Tabel Telemetri Log Terbaru;
  :Tunggu 2 Detik (Fragment Rerun);
repeat while (Halaman Aktif?) is (ya) not (tidak)
stop
@enduml
```

### E. Alur Detail Forecasting
```plantuml
@startuml
skinparam roundcorner 10
skinparam ActivityBackgroundColor White
skinparam ActivityBorderColor #0058be
skinparam ArrowColor #0b1c30

start
:Masuk Halaman 'Forecasting';
:Pilih Gedung & Horizon (24 Jam / 7 Hari);
:Muat Histori Data dari SQLite;
:Latih Random Forest Regressor;
:Split Data (80% Train, 20% Test);
:Evaluasi Skor R2 & RMSE;
:Latih Ulang Model dengan Semua Data;
:Prediksi Beban Listrik ke Depan;
:Tampilkan Metrik Evaluasi;
:Gambar Grafik Aktual vs Prediksi;
:Tampilkan AI Insights (Peak-Shaving);
stop
@enduml
```

### F. Alur Detail Admin Settings
```plantuml
@startuml
skinparam roundcorner 10
skinparam ActivityBackgroundColor White
skinparam ActivityBorderColor #0058be
skinparam ArrowColor #0b1c30

start
:Masuk Halaman 'Admin Settings';
split
  :1. Tarif PLN;
  :Masukkan Nilai Tarif per kWh;
  :Klik 'Simpan Perubahan Tarif';
  :Update tarif_pln Global;
split currents
  :2. Target Baseline;
  :Pilih Rentang & Batas kWh;
  :Klik 'Simpan Target';
  :Update batas_angka Global;
split currents
  :3. Tambah Gedung;
  :Input Nama & Port Modbus;
  :Klik 'Tambah Gedung';
  :CREATE TABLE device_port_XXX_readings;
  :Insert Baris Inisiasi DB;
  :Tambahkan ke gedung_list;
  :Picu st.rerun;
end split
stop
@enduml
```

