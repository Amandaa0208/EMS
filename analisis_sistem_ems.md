# Hasil Analisis Sistem: User Requirements, Use Case, dan User Flow (EMS Enterprise)

Dokumen ini menggabungkan hasil analisis kebutuhan pengguna, diagram use case, dan aliran pengguna (user flow) untuk memudahkan penyusunan laporan proyek atau bab skripsi Anda.

---

## BAB 1: ANALISIS KEBUTUHAN PENGGUNA (USER REQUIREMENTS)

Tahap ini memetakan perbedaan kebutuhan informasi dan fungsi sistem berdasarkan peran pengguna (*user personas*) di lingkungan operasional gedung.

### 1.1 Matriks Kebutuhan Peran Pengguna
Pengguna sistem dibagi menjadi tiga aktor dengan fokus dan tingkat kedetailan informasi yang berbeda:

| Dimensi Perbandingan | Pengelola Gedung (Teknisi / Operator Fasilitas) | Manajemen Gedung (Manajer Energi / Direksi / Keuangan) | Super Admin (IT / System Administrator) |
| :--- | :--- | :--- | :--- |
| **Fokus Utama** | Operasional harian di lapangan, kestabilan jaringan listrik, dan respon cepat terhadap gangguan/alarm. | Efisiensi biaya, pencapaian target efisiensi energi (KPI), audit finansial, dan perencanaan anggaran (budgeting). | Konfigurasi sistem dasar, keamanan data, integrasi perangkat Modbus, dan pemeliharaan database. |
| **Kebutuhan Informasi** | - Nilai tegangan (V) & arus (A) per phase.<br>- Status koneksi gateway Modbus.<br>- Peringatan *overshoot* beban secara real-time. | - Total konsumsi akumulatif (kWh).<br>- Estimasi biaya tagihan (Rupiah).<br>- Laporan kepatuhan audit (BPK/Internal).<br>- Tren proyeksi beban (Forecasting). | - Konfigurasi tarif PLN.<br>- Pemetaan alamat IP/Port Modbus.<br>- Manajemen database dan log audit sistem. |
| **Tingkat Detail Data** | Sangat detail, bersifat teknis, dan berorientasi pada waktu nyata (*seconds/minutes*). | Agregat (ringkasan), berorientasi finansial, dan jangka panjang (*daily/monthly/yearly*). | Parameter konfigurasi sistem dan integritas infrastruktur data. |

### 1.2 Rekomendasi Pemisahan Hak Akses (Role-Based Access Control)
Untuk pengembangan sistem lebih lanjut, diusulkan pembagian hak akses menu antarmuka sebagai berikut:

```mermaid
graph TD
    %% Styling
    classDef roles fill:#d8e2ff,stroke:#001a42,stroke-width:2px,color:#001a42,font-weight:bold;
    classDef menuStyle fill:#ffffff,stroke:#0058be,stroke-width:1.5px,color:#0b1c30;
    
    UserRole["Role Akses"]:::roles
    Pengelola["Pengelola / Teknisi"]:::roles
    Manajemen["Manajemen / Finansial"]:::roles
    AdminIT["Super Admin / IT"]:::roles
    
    MenuRealtime["Dashboard & Detail Telemetri 3-Phase"]:::menuStyle
    MenuAnalisa["Analisa Energi & Forecasting ML"]:::menuStyle
    MenuReports["Reports & Audit Trail"]:::menuStyle
    MenuSettings["Admin Settings (Tarif, DB, Gateway)"]:::menuStyle

    UserRole --> Pengelola
    UserRole --> Manajemen
    UserRole --> AdminIT

    %% Pemetaan Izin
    Pengelola -->|Akses Penuh| MenuRealtime
    Pengelola -->|Akses Terbatas| MenuAnalisa
    
    Manajemen -->|Akses Penuh| MenuAnalisa
    Manajemen -->|Akses Penuh| MenuReports
    
    AdminIT -->|Akses Penuh| MenuSettings
    AdminIT -->|Akses Penuh| MenuReports
```

---

## BAB 2: SPESIFIKASI USE CASE (USE CASE SPECIFICATIONS)

Use Case memetakan interaksi aktor (manusia maupun sistem eksternal) terhadap fungsionalitas yang disediakan oleh dashboard EMS Enterprise.

### 2.1 Identifikasi Aktor (Actors)
1. **Admin / Super Admin (Aktor Utama):** Pengguna yang memantau konsumsi energi, melakukan analisis, menggunakan fitur forecasting, mengekspor laporan, serta mengubah pengaturan dasar sistem.
2. **Modbus TCP Gateway (Aktor Pendukung):** Sistem/perangkat eksternal yang mengirimkan data telemetri listrik (*Active Power*, *Energy*, *Voltage*, *Current*, *Frequency*) dari masing-masing gedung ke database secara periodik.

### 2.2 Daftar Use Case
Berikut adalah daftar fungsionalitas sistem yang dipetakan sebagai Use Case:

* **UC-01: Otentikasi / Login:** Mengakses sistem menggunakan username, password, dan memilih role akses yang sesuai.
* **UC-02: Keluar Sistem (Logout):** Mengakhiri sesi aktif pengguna dan mengunci kembali aplikasi.
* **UC-03: Melihat Dashboard Utama:** Memantau ringkasan real-time total konsumsi (kWh), estimasi biaya (Rp), status baseline (Normal/Warning), grafik konsumsi harian, distribusi beban, dan log aktivitas Modbus.
* **UC-04: Memfilter Log Gedung:** Menyaring tabel aktivitas gedung di dashboard berdasarkan nama gedung atau nomor port Modbus.
* **UC-05: Pengecekan Batas Baseline:** Sistem secara otomatis mendeteksi apakah beban melebihi target batas baseline harian yang diatur.
* **UC-06: Menganalisa Profil Energi:** Membandingkan pemakaian energi antar periode waktu dengan benchmark pembanding (minggu lalu, tahun lalu, atau target baseline).
* **UC-07: Mengekspor Analisis ke CSV:** Mengunduh tabel perbandingan analisis profil energi ke file format `.csv`.
* **UC-08: Investigasi Detail Selisih:** Menampilkan overlay dialog analisis wawasan anomali energi berdasarkan log HVAC dan telemetri.
* **UC-09: Memantau Detail Telemetri Gedung:** Melihat status koneksi port Modbus, akumulasi kWh, beban kW saat ini, serta grafik tegangan dan arus listrik 3-phase per phase (R/S/T).
* **UC-10: Melakukan Forecasting Energi:** Membuat prediksi beban listrik ke depan menggunakan model regresi *Random Forest*.
* **UC-11: Melatih Model Machine Learning:** Proses pelatihan algoritma regresi dengan membagi data latih/uji (80:20) secara otomatis sebelum membuat hasil prediksi.
* **UC-12: Membuat Laporan & Log Audit Trail:** Menampilkan laporan keuangan BPK, laporan efisiensi bulanan, rekapitulasi konsumsi gedung, serta log aktivitas audit trail.
* **UC-13: Mengekspor Laporan & Audit Trail:** Mengunduh dokumen laporan atau log audit trail dalam bentuk file CSV.
* **UC-14: Mengelola Parameter Sistem:** Mengakses menu administrasi parameter.
* **UC-15: Mengubah Tarif PLN:** Memperbarui tarif listrik dasar (Rupiah/kWh).
* **UC-16: Mengelola Gedung:** Menambah gedung baru (mengisi nama & port Modbus) atau menghapus gedung yang ada.
* **UC-17: Inisialisasi Tabel Database:** Pembuatan skema tabel telemetry baru di SQLite ketika gedung baru didaftarkan.
* **UC-18: Ingestion Data Telemetri:** Proses perekaman data telemetri berkala ke SQLite oleh perangkat Modbus eksternal.

### 2.3 Diagram Use Case (Mermaid)

```mermaid
graph LR
    %% Pengaturan Style Node
    classDef actorStyle fill:#d8e2ff,stroke:#001a42,stroke-width:2px,color:#001a42,font-weight:bold;
    classDef usecaseStyle fill:#ffffff,stroke:#0058be,stroke-width:1.5px,color:#0b1c30,font-size:13px;
    classDef boundaryStyle fill:#f8f9ff,stroke:#cbd5e1,stroke-width:2px;

    %% Deklarasi Aktor
    Admin["Admin / Super Admin"]:::actorStyle
    ModbusGateway["Modbus TCP Gateway"]:::actorStyle

    %% Batasan Sistem (System Boundary)
    subgraph SystemBoundary["Sistem EMS Enterprise"]
        direction TB

        %% 1. Autentikasi
        UC_Login(["UC-01: Otentikasi / Login"]):::usecaseStyle
        UC_Logout(["UC-02: Keluar Sistem (Logout)"]):::usecaseStyle

        %% 2. Dashboard
        UC_Dashboard(["UC-03: Melihat Dashboard Utama"]):::usecaseStyle
        UC_FilterDashboard(["UC-04: Memfilter Log Gedung"]):::usecaseStyle
        UC_CheckBaseline(["UC-05: Pengecekan Batas Baseline"]):::usecaseStyle

        %% 3. Analisa & Detail
        UC_Analisa(["UC-06: Menganalisa Profil Energi"]):::usecaseStyle
        UC_ExportAnalisa(["UC-07: Mengekspor Analisis ke CSV"]):::usecaseStyle
        UC_Investigasi(["UC-08: Investigasi Detail Selisih"]):::usecaseStyle
        UC_DetailGedung(["UC-09: Memantau Detail Telemetri Gedung"]):::usecaseStyle

        %% 4. Forecasting
        UC_Forecast(["UC-10: Melakukan Forecasting Energi"]):::usecaseStyle
        UC_TrainModel(["UC-11: Melatih Model Machine Learning"]):::usecaseStyle

        %% 5. Reports & Audit
        UC_Reports(["UC-12: Membuat Laporan & Log Audit Trail"]):::usecaseStyle
        UC_ExportReports(["UC-13: Mengekspor Laporan & Audit Trail"]):::usecaseStyle

        %% 6. Admin Settings
        UC_Settings(["UC-14: Mengelola Parameter Sistem"]):::usecaseStyle
        UC_UpdateTarif(["UC-15: Mengubah Tarif PLN"]):::usecaseStyle
        UC_ManageGedung(["UC-16: Mengelola Gedung"]):::usecaseStyle
        UC_InitTable(["UC-17: Inisialisasi Tabel Database"]):::usecaseStyle

        %% 7. Data Ingestion (System Actor)
        UC_IngestData(["UC-18: Ingestion Data Telemetri"]):::usecaseStyle
    end

    %% Hubungan Aktor Utama
    Admin --> UC_Login
    Admin --> UC_Logout
    Admin --> UC_Dashboard
    Admin --> UC_Analisa
    Admin --> UC_DetailGedung
    Admin --> UC_Forecast
    Admin --> UC_Reports
    Admin --> UC_Settings

    %% Hubungan Aktor Pendukung
    ModbusGateway --> UC_IngestData

    %% Relasi Antar Use Case (Include/Extend)
    UC_FilterDashboard -.->|"<<extend>>"| UC_Dashboard
    UC_Dashboard -.->|"<<include>>"| UC_CheckBaseline
    
    UC_ExportAnalisa -.->|"<<extend>>"| UC_Analisa
    UC_Investigasi -.->|"<<extend>>"| UC_Analisa
    
    UC_Forecast -.->|"<<include>>"| UC_TrainModel
    
    UC_ExportReports -.->|"<<extend>>"| UC_Reports
    
    UC_Settings -.->|"<<include>>"| UC_UpdateTarif
    UC_Settings -.->|"<<include>>"| UC_ManageGedung
    UC_ManageGedung -.->|"<<include>>"| UC_InitTable
    
    UC_IngestData -.->|"<<include>>"| UC_InitTable
```

---

## BAB 3: ALIRAN PENGGUNA (USER FLOWS)

User Flow menggambarkan langkah interaksi logis pengguna saat bernavigasi dari satu fungsi ke fungsi lainnya.

### 3.1 Alur Navigasi Utama (Global User Flow)

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

### 3.2 Alur Detail Per Fitur (Detail Flows)

#### A. Alur Otentikasi & Login (Authentication Flow)
1. Pengguna membuka antarmuka aplikasi Streamlit.
2. Pengguna memasukkan username, password, dan memilih role akses (Admin / Super Admin).
3. Pengguna menekan tombol "Masuk ke Dashboard".
4. Sistem memeriksa validitas data:
   * Jika valid: Mengeset variabel sesi login, memuat profil pengguna, dan menampilkan halaman utama (Dashboard).
   * Jika salah: Menampilkan pesan kesalahan dan menolak akses masuk.

#### B. Alur Analisa Profil Energi (Energy Analysis Flow)
1. Pengguna berpindah ke halaman **Analisa Energi**.
2. Pengguna menyesuaikan parameter penyaringan (Periode Analisa, Kategori Beban, dan Pembanding).
3. Jika periode yang dipilih adalah "Kustom", sistem menampilkan komponen input tanggal mulai dan tanggal selesai secara dinamis.
4. Pengguna menekan tombol "Terapkan".
5. Sistem menarik data historis dari database SQLite, melakukan kalkulasi statistik konsumsi (kWh) dan biaya (Rupiah), kemudian menggambar ulang grafik garis Plotly serta memperbarui tabel breakdown di layar.
6. Pengguna memiliki opsi untuk:
   * Mengekspor data ke CSV (mengunduh file).
   * Membuka modal investigasi detail guna melihat wawasan AI mengenai anomali daya.

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

#### C. Alur Pemantauan Detail Telemetri (Building Profile Flow)
1. Pengguna memilih menu **Profile Gedung**.
2. Pengguna memilih gedung spesifik dari menu pilihan (*dropdown*).
3. Sistem secara otomatis menjalankan loop perulangan setiap 2 detik (*fragment rendering*).
4. Di setiap perulangan, sistem memuat data baris telemetri terbaru dari tabel SQLite gedung terkait, memperbarui angka-angka metrik utama, dan merender ulang grafik visualisasi tegangan serta arus listrik 3-phase di antarmuka pengguna.

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

#### D. Alur Forecasting Energi (Machine Learning Flow)
1. Pengguna masuk ke halaman **Forecasting**.
2. Pengguna memilih gedung sasaran dan durasi prediksi (24 Jam atau 7 Hari ke depan).
3. Sistem menampilkan indikator pemrosesan (*spinner*) selama proses komputasi berlangsung.
4. Algoritma Random Forest memotong data histori (80:20), melakukan pelatihan model regresi, menghitung metrik evaluasi akurasi, dan menghasilkan ramalan nilai kW di waktu mendatang.
5. Sistem menampilkan skor metrik ($R^2$ & RMSE), memplot grafik perbandingan histori vs prediksi masa depan, serta memaparkan rekomendasi AI mengenai jam beban puncak.

#### E. Alur Pengelolaan Parameter & Gedung (Admin Settings Flow)
1. Pengguna (Admin) masuk ke menu **Admin Settings**.
2. Admin dapat mengubah Tarif PLN (mengisi angka Rp/kWh lalu menekan Simpan) untuk memperbarui variabel kalkulasi biaya tagihan secara global.
3. Admin dapat mengatur ulang target baseline energi tahunan/bulanan/harian.
4. Admin dapat menambahkan gedung operasional baru dengan mengisi Nama dan Port Modbus:
   * Sistem terhubung ke database SQLite.
   * Sistem menjalankan perintah DDL untuk meng-generate tabel pembacaan telemetri Modbus baru khusus untuk port gedung tersebut (`device_port_XXX_readings`).
   * Sistem menyisipkan baris telemetri inisiasi awal agar antarmuka tidak kosong.
   * Sistem memperbarui daftar gedung di sesi aktif (*session state*).

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

## BAB 4: KODE SCRIPT PLANTUML (UNTUK STARUML / PLANTTEXT)

Berikut adalah kode script PlantUML yang dapat Anda gunakan di **StarUML**, **PlantText**, atau editor PlantUML lainnya untuk me-render diagram use case secara otomatis:

```plantuml
@startuml
left to right direction
skinparam packageStyle rectangle
skinparam roundcorner 10
skinparam actorStyle awesome

skinparam usecase {
    BackgroundColor White
    BorderColor #0058be
    ArrowColor #0b1c30
}

actor User as "Admin / Super Admin"
actor Modbus as "Modbus TCP Gateway"

rectangle "Sistem EMS Enterprise" {
    
    package "Autentikasi" {
        usecase UC01 as "UC-01: Otentikasi / Login"
        usecase UC02 as "UC-02: Keluar Sistem (Logout)"
    }
    
    package "Dashboard & Telemetri" {
        usecase UC03 as "UC-03: Melihat Dashboard Utama"
        usecase UC04 as "UC-04: Memfilter Log Gedung"
        usecase UC05 as "UC-05: Pengecekan Batas Baseline"
        usecase UC09 as "UC-09: Memantau Detail Telemetri Gedung"
    }
    
    package "Analisa & Forecasting" {
        usecase UC06 as "UC-06: Menganalisa Profil Energi"
        usecase UC07 as "UC-07: Mengekspor Analisis ke CSV"
        usecase UC08 as "UC-08: Investigasi Detail Selisih"
        usecase UC10 as "UC-10: Melakukan Forecasting Energi"
        usecase UC11 as "UC-11: Melatih Model Machine Learning"
    }
    
    package "Laporan & Audit" {
        usecase UC12 as "UC-12: Membuat Laporan & Log Audit Trail"
        usecase UC13 as "UC-13: Mengekspor Laporan & Audit Trail"
    }
    
    package "Pengaturan (Settings)" {
        usecase UC14 as "UC-14: Mengelola Parameter Sistem"
        usecase UC15 as "UC-15: Mengubah Tarif PLN"
        usecase UC16 as "UC-16: Mengelola Gedung"
        usecase UC17 as "UC-17: Inisialisasi Tabel Database"
        usecase UC18 as "UC-18: Ingestion Data Telemetri"
    }
}

' Links from User (Left side)
User ---> UC01
User ---> UC02
User ---> UC03
User ---> UC06
User ---> UC09
User ---> UC10
User ---> UC12
User ---> UC14

' Links for Include/Extend (Top to down/left to right flow)
UC04 .right.> UC03 : <<extend>>
UC03 .down.> UC05 : <<include>>

UC07 .right.> UC06 : <<extend>>
UC08 .down.> UC06 : <<extend>>

UC10 .down.> UC11 : <<include>>

UC13 .right.> UC12 : <<extend>>

UC14 .down.> UC15 : <<include>>
UC14 .right.> UC16 : <<include>>
UC16 .down.> UC17 : <<include>>

' Modbus connects from right
UC18 <--- Modbus
UC18 .down.> UC17 : <<include>>

@enduml
```

---

## BAB 5: PANDUAN MENGATUR GARIS LURUS & RAPI DI STARUML

Jika Anda menggambar atau mengimpor Use Case ini di **StarUML**, garis-garis hubungan seringkali meliuk-liuk secara tidak rapi. Ikuti langkah berikut agar semua garis lurus dan rapi:

### 5.1 Mengubah Tipe Garis Menjadi Lurus (90 Derajat atau Diagonal Langsung)
Secara default, StarUML menggunakan tipe garis melengkung/bebas. Anda bisa mengubahnya menjadi **Rectilinear** (siku-siku 90 derajat) atau **Oblique** (lurus langsung tanpa belokan):
* **Cara Cepat (Semua Garis):** 
  1. Tekan tombol `Ctrl + A` pada keyboard untuk menyeleksi seluruh diagram.
  2. Buka menu utama di bagian atas: **Format** -> **Line Style**.
  3. Pilih **Oblique** jika Anda ingin garis lurus diagonal langsung (point-to-point).
  4. Pilih **Rectilinear** jika Anda ingin garis siku-siku 90 derajat yang rapi (hanya bergerak horizontal dan vertikal).

### 5.2 Merapikan Posisi Elemen (Align & Distribute)
Agar garis tidak saling silang, posisikan aktor dan usecase secara simetris:
* **Tata Letak Standar:**
  * Tempatkan Aktor **Admin / Super Admin** di ujung paling **Kiri**.
  * Tempatkan Aktor **Modbus TCP Gateway** di ujung paling **Kanan**.
  * Tempatkan seluruh elips **Usecase** di bagian **Tengah** (di dalam kotak batas sistem).
* **Menggunakan Alat Perata Otomatis (Alignment):**
  1. Seleksi elips usecase yang ingin Anda sejajarkan (klik sambil tahan tombol `Shift`).
  2. Klik kanan pada area seleksi, lalu pilih **Alignment** -> **Align Center (Vertically)** untuk membuat mereka berbaris lurus vertikal dari atas ke bawah.
  3. Pilih **Alignment** -> **Distribute Vertically** agar jarak antar elips usecase dari atas ke bawah sama rata secara presisi.

---

## BAB 6: KODE SCRIPT PLANTUML UNTUK USER FLOW (ACTIVITY DIAGRAM)

Berikut adalah kode script PlantUML (Activity Diagram) yang dapat Anda gunakan di **StarUML**, **PlantText**, atau editor PlantUML lainnya untuk me-render User Flow secara otomatis dengan garis lurus yang rapi:

### 6.1 Global User Flow (Alur Navigasi Utama)
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

### 6.2 Alur Detail Login
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

### 6.3 Alur Detail Analisa Energi
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

### 6.4 Alur Detail Profile Gedung
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

### 6.5 Alur Detail Forecasting
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

### 6.6 Alur Detail Admin Settings
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


