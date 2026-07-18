# Dokumentasi User Flow — EMS Enterprise (Per Role)

Dokumen ini menjelaskan alur perjalanan pengguna (User Flow) saat menggunakan sistem **EMS Enterprise (Sistem Pemantauan Energi Gedung)**. Alur ini dibagi berdasarkan **dua role** yang memiliki hak akses berbeda.

---

## 1. Definisi Role & Hak Akses

| Role | Jabatan | Hak Akses Sidebar |
| :--- | :--- | :--- |
| **Super Admin** | Manajemen Gedung | Dashboard, Analisa Energi, Profile Gedung, **Forecasting**, **Reports & Audit**, **Admin Settings** |
| **Admin** | Pengelola Gedung | Dashboard, Analisa Energi, Profile Gedung |

> **Catatan:** Admin (Pengelola Gedung) tidak melihat menu Forecasting, Reports & Audit, dan Admin Settings di sidebar. Pembatasan ini bersifat *structural* — menu tidak ditampilkan, bukan hanya di-disable.

---

## 2. Peta Navigasi Global (Per Role)

### 2A. Super Admin — Manajemen Gedung (Akses Penuh)

```mermaid
graph TD
    classDef startStyle fill:#0058be,stroke:#001a42,stroke-width:2px,color:#ffffff,font-weight:bold;
    classDef stepStyle fill:#ffffff,stroke:#0058be,stroke-width:1.5px,color:#0b1c30;
    classDef checkStyle fill:#ff9f1c,stroke:#b36b00,stroke-width:2px,color:#ffffff,font-weight:bold;
    classDef endStyle fill:#ba1a1a,stroke:#410002,stroke-width:2px,color:#ffffff,font-weight:bold;
    classDef superStyle fill:#d8e2ff,stroke:#001a42,stroke-width:2px,color:#001a42,font-weight:bold;

    StartNode(["Mulai: Buka Aplikasi"]):::startStyle
    LoginForm["Form Login: Super Admin"]:::stepStyle
    AuthCheck{"Validasi Sukses?"}:::checkStyle
    ShowError["Tampilkan Pesan Error"]:::stepStyle

    LoadDashboard["1. Dashboard: Memantau Kondisi Energi & Status Baseline"]:::stepStyle
    GoToAnalisa["2. Analisa Energi: Membandingkan Konsumsi & Unduh CSV"]:::stepStyle
    GoToProfile["3. Profile Gedung: Monitoring Live Telemetri 3-Phase"]:::stepStyle
    GoToForecast["4. Forecasting: Prediksi Beban Listrik via AI/ML"]:::superStyle
    GoToReports["5. Reports & Audit: Mengunduh Laporan Kepatuhan"]:::superStyle
    GoToSettings["6. Admin Settings: Update Tarif PLN & Kelola Gedung"]:::superStyle

    Logout["Klik Logout & Hapus Sesi"]:::stepStyle
    EndNode(["Selesai: Sesi Berakhir"]):::endStyle

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

### 2B. Admin — Pengelola Gedung (Akses Terbatas)

```mermaid
graph TD
    classDef startStyle fill:#0058be,stroke:#001a42,stroke-width:2px,color:#ffffff,font-weight:bold;
    classDef stepStyle fill:#ffffff,stroke:#0058be,stroke-width:1.5px,color:#0b1c30;
    classDef checkStyle fill:#ff9f1c,stroke:#b36b00,stroke-width:2px,color:#ffffff,font-weight:bold;
    classDef endStyle fill:#ba1a1a,stroke:#410002,stroke-width:2px,color:#ffffff,font-weight:bold;
    classDef blockedStyle fill:#fee2e2,stroke:#ba1a1a,stroke-width:2px,stroke-dasharray: 5 5,color:#ba1a1a;

    StartNode(["Mulai: Buka Aplikasi"]):::startStyle
    LoginForm["Form Login: Admin"]:::stepStyle
    AuthCheck{"Validasi Sukses?"}:::checkStyle
    ShowError["Tampilkan Pesan Error"]:::stepStyle

    LoadDashboard["1. Dashboard: Memantau Kondisi Energi & Status Baseline"]:::stepStyle
    GoToAnalisa["2. Analisa Energi: Membandingkan Konsumsi & Unduh CSV"]:::stepStyle
    GoToProfile["3. Profile Gedung: Monitoring Live Telemetri 3-Phase"]:::stepStyle

    BlockedForecast["❌ Forecasting — Tidak Tersedia"]:::blockedStyle
    BlockedReports["❌ Reports & Audit — Tidak Tersedia"]:::blockedStyle
    BlockedSettings["❌ Admin Settings — Tidak Tersedia"]:::blockedStyle

    Logout["Klik Logout & Hapus Sesi"]:::stepStyle
    EndNode(["Selesai: Sesi Berakhir"]):::endStyle

    StartNode --> LoginForm
    LoginForm --> AuthCheck
    AuthCheck -- Tidak --> ShowError
    ShowError --> LoginForm

    AuthCheck -- Ya --> LoadDashboard
    LoadDashboard --> GoToAnalisa
    GoToAnalisa --> GoToProfile
    GoToProfile --> Logout
    Logout --> EndNode

    GoToProfile -.->|Akses Ditolak| BlockedForecast
    GoToProfile -.->|Akses Ditolak| BlockedReports
    GoToProfile -.->|Akses Ditolak| BlockedSettings
```

---

## 3. Alur Detail Per Fitur

### A. Alur Otentikasi & Login (Kedua Role)

Alur ini menjamin bahwa hanya pengguna terdaftar (*Admin* dengan username `admin` atau *Super Admin* dengan username `superadmin`) yang dapat masuk ke panel dashboard. Setelah login, sidebar menampilkan menu sesuai role.

```mermaid
graph TD
    classDef step fill:#ffffff,stroke:#0058be,stroke-width:1.5px,color:#0b1c30;
    classDef check fill:#ff9f1c,stroke:#b36b00,stroke-width:2px,color:#ffffff,font-weight:bold;
    classDef roleAdmin fill:#eff4ff,stroke:#3b82f6,stroke-width:2px,color:#0b1c30;
    classDef roleSuperAdmin fill:#d8e2ff,stroke:#001a42,stroke-width:2px,color:#001a42;

    A["Input Username & Password"]:::step --> B["Pilih Role Akses: Admin / Super Admin"]:::step
    B --> C["Klik 'Masuk ke Dashboard'"]:::step
    C --> D{"Apakah Kredensial Valid?"}:::check
    D -- Ya --> E["Set session_state logged_in = True"]:::step
    E --> F{"Role yang Dipilih?"}:::check
    F -- Admin --> G["Inisialisasi: Pengelola Gedung"]:::roleAdmin
    G --> G1["Sidebar: Dashboard, Analisa Energi, Profile Gedung"]:::roleAdmin
    F -- Super Admin --> H["Inisialisasi: Manajemen Gedung"]:::roleSuperAdmin
    H --> H1["Sidebar: Semua 6 Menu Lengkap"]:::roleSuperAdmin
    G1 --> I["Rerun ke Dashboard Utama"]:::step
    H1 --> I
    D -- Tidak --> J["Tampilkan Pesan Error"]:::step
    J --> A
```

---

### B. Alur Analisa Profil Energi (Super Admin & Admin)

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

### C. Alur Pemantauan Detail Telemetri (Super Admin & Admin)

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

### D. Alur Forecasting Beban Energi — ⚠️ Khusus Super Admin

Di modul **Forecasting**, pengguna melatih model *Random Forest* untuk memproyeksikan konsumsi daya listrik gedung di masa mendatang. **Fitur ini hanya tersedia untuk Super Admin (Manajemen Gedung).**

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

### E. Alur Reports & Audit — ⚠️ Khusus Super Admin

Modul pelaporan dan audit trail. **Hanya tersedia untuk Super Admin (Manajemen Gedung).**

```mermaid
graph TD
    classDef step fill:#ffffff,stroke:#0058be,stroke-width:1.5px,color:#0b1c30;
    classDef check fill:#ff9f1c,stroke:#b36b00,stroke-width:2px,color:#ffffff,font-weight:bold;

    A[Masuk Halaman 'Reports & Audit']:::step --> B{Pilih Jenis Laporan}:::check
    B -->|Audit BPK| C[Lihat Laporan Audit Keuangan Energi]:::step
    B -->|Efisiensi| D[Lihat Laporan Efisiensi Bulanan]:::step
    B -->|Rekap Gedung| E[Lihat Rekapitulasi Konsumsi per Gedung]:::step
    B -->|Audit Trail| F[Lihat Log Audit Trail Transaksi Energi]:::step
    C --> G{Ekspor Laporan?}:::check
    D --> G
    E --> G
    F --> G
    G -- Ya --> H[Unduh dalam Format CSV]:::step
    G -- Tidak --> I[Selesai]:::step
```

---

### F. Alur Pengelolaan Parameter & Gedung — ⚠️ Khusus Super Admin

Pengaturan administrasi memungkinkan modifikasi parameter tarif, batas baseline, serta struktur kategori dan gedung. **Hanya tersedia untuk Super Admin (Manajemen Gedung).**

```mermaid
graph TD
    classDef step fill:#ffffff,stroke:#0058be,stroke-width:1.5px,color:#0b1c30;
    classDef db fill:#f1f5f9,stroke:#94a3b8,stroke-width:2px,color:#334155;
    classDef check fill:#ff9f1c,stroke:#b36b00,stroke-width:2px,color:#ffffff,font-weight:bold;

    A[Masuk Halaman 'Admin Settings']:::step --> B{Pilih Blok Konfigurasi}:::check
    
    %% Jalur Tarif
    B -->|1. Tarif PLN| C[Masukkan Nilai Tarif per kWh]:::step
    C --> D[Klik 'Simpan Perubahan Tarif PLN']:::step
    D --> E[Perbarui st.session_state tarif_pln secara global]:::step
    
    %% Jalur Target Baseline
    B -->|2. Target Baseline| F[Pilih Tipe Rentang & Batas kWh]:::step
    F --> G[Klik 'Simpan Target']:::step
    G --> H[Perbarui st.session_state batas_angka]:::step

    %% Jalur Kategori Beban
    B -->|3. Kategori Beban| I1[Lihat/Tambah/Hapus Kategori Beban]:::step
    
    %% Jalur Gedung Baru
    B -->|4. Tambah Gedung| I[Input Nama Gedung & Nomor Port Modbus]:::step
    I --> J[Klik 'Tambah Gedung']:::step
    J --> K[Hubungkan ke ems.db]:::db
    K --> L[Jalankan CREATE TABLE device_port_XXX_readings]:::db
    L --> M[Masukkan baris telemetri inisiasi pertama]:::db
    M --> N[Tambahkan ke st.session_state gedung_list]:::step
    N --> O[Picu st.rerun untuk memuat gedung baru]:::step
```

---

## 4. Kode Script PlantUML (Untuk StarUML / PlantText)

### A. User Flow Super Admin (Manajemen Gedung)
```plantuml
@startuml
skinparam roundcorner 10
skinparam ActivityBackgroundColor White
skinparam ActivityBorderColor #0058be
skinparam ArrowColor #0b1c30

title User Flow — Super Admin (Manajemen Gedung)

start
:Buka Aplikasi;
:Form Login: Input **superadmin** / **superadmin**;
:Pilih Role: **Super Admin**;
if (Validasi Sukses?) then (tidak)
  :Tampilkan Pesan Error;
  detach
else (ya)
  :Inisialisasi Sesi: **Manajemen Gedung**;
  :Sidebar menampilkan **6 menu lengkap**;
  :1. Dashboard: Memantau Kondisi Energi & Status Baseline;
  :2. Analisa Energi: Membandingkan Konsumsi & Unduh CSV;
  :3. Profile Gedung: Monitoring Live Telemetri 3-Phase;
  #LightBlue:4. Forecasting: Prediksi Beban Listrik via AI/ML;
  #LightBlue:5. Reports & Audit: Mengunduh Laporan Kepatuhan;
  #LightBlue:6. Admin Settings: Update Tarif PLN & Kelola Gedung;
  :Klik Logout & Hapus Sesi;
  stop
endif
@enduml
```

### B. User Flow Admin (Pengelola Gedung)
```plantuml
@startuml
skinparam roundcorner 10
skinparam ActivityBackgroundColor White
skinparam ActivityBorderColor #0058be
skinparam ArrowColor #0b1c30

title User Flow — Admin (Pengelola Gedung)

start
:Buka Aplikasi;
:Form Login: Input **admin** / **admin**;
:Pilih Role: **Admin**;
if (Validasi Sukses?) then (tidak)
  :Tampilkan Pesan Error;
  detach
else (ya)
  :Inisialisasi Sesi: **Pengelola Gedung**;
  :Sidebar menampilkan **3 menu monitoring**;
  :1. Dashboard: Memantau Kondisi Energi & Status Baseline;
  :2. Analisa Energi: Membandingkan Konsumsi & Unduh CSV;
  :3. Profile Gedung: Monitoring Live Telemetri 3-Phase;
  note right
    Menu berikut **tidak tersedia**:
    ❌ Forecasting
    ❌ Reports & Audit
    ❌ Admin Settings
  end note
  :Klik Logout & Hapus Sesi;
  stop
endif
@enduml
```

### C. Alur Detail Login (Kedua Role)
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
  if (Role?) then (Admin)
    :Inisialisasi: **Pengelola Gedung**;
    :Sidebar: 3 Menu Monitoring;
  else (Super Admin)
    :Inisialisasi: **Manajemen Gedung**;
    :Sidebar: 6 Menu Lengkap;
  endif
  :Rerun ke Dashboard Utama;
  stop
endif
@enduml
```

### D. Alur Detail Analisa Energi
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

### E. Alur Detail Profile Gedung
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

### F. Alur Detail Forecasting (Super Admin Only)
```plantuml
@startuml
skinparam roundcorner 10
skinparam ActivityBackgroundColor White
skinparam ActivityBorderColor #0058be
skinparam ArrowColor #0b1c30

start
:Masuk Halaman 'Forecasting';
note right: Hanya Super Admin
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

### G. Alur Detail Admin Settings (Super Admin Only)
```plantuml
@startuml
skinparam roundcorner 10
skinparam ActivityBackgroundColor White
skinparam ActivityBorderColor #0058be
skinparam ArrowColor #0b1c30

start
:Masuk Halaman 'Admin Settings';
note right: Hanya Super Admin
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
  :3. Kategori Beban;
  :Lihat/Tambah/Hapus Kategori;
split currents
  :4. Manajemen Gedung;
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


