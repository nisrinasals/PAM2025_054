# SleepMix

Aplikasi Android untuk menciptakan suasana tidur yang nyaman melalui penggabungan suara alam (*ambient sound mixing*).

---

## Deskripsi Proyek

**SleepMix** adalah aplikasi Android yang memungkinkan pengguna untuk membuat dan memainkan campuran (*mix*) suara alam secara bersamaan dengan kontrol volume independen untuk setiap suara. Aplikasi ini dirancang untuk membantu pengguna menciptakan lingkungan suara yang kondusif bagi relaksasi dan tidur.

Aplikasi ini dibangun menggunakan **Jetpack Compose** sebagai antarmuka pengguna, **Room** sebagai basis data lokal, dan **Media3 / ExoPlayer** sebagai mesin pemutaran audio dengan layanan latar belakang (*foreground service*).

---

## Fitur Utama

### Manajemen Pengguna
- Registrasi akun baru dengan validasi email dan password (minimal 6 karakter)
- Login dengan verifikasi password menggunakan algoritma *hash* SHA-256 + Salt
- Manajemen sesi menggunakan **DataStore Preferences**
- Logout dengan konfirmasi

### Perpustakaan Suara
- Menampilkan 8 suara alam bawaan dalam tampilan grid
- Pencarian suara berdasarkan nama
- Pratinjau (*preview*) suara individual dengan kontrol volume
- Kategori suara: Nature, Weather, Ambience

### Suara Bawaan
| Nama | Kategori |
|------|----------|
| Bird | Nature |
| Cricket | Nature |
| Rain | Weather |
| Sea Waves | Nature |
| Thunderstorm | Weather |
| Firewood | Ambience |
| Frog | Nature |
| River | Nature |

### Manajemen Mix
- Membuat mix baru dengan nama (maksimal 30 karakter)
- Menambahkan 2–5 suara per mix
- Mencegah penambahan suara yang sama lebih dari satu kali
- Mengatur volume independen per suara (0–100%)
- Mengedit nama dan komposisi suara dalam mix yang sudah ada
- Menghapus mix atau suara individual dari mix
- Menampilkan daftar mix yang dibuat oleh pengguna

### Pemutaran Audio
- Pemutaran mix dengan semua suara secara bersamaan
- Efek *fade-in* dan *fade-out* (durasi 400ms) saat menambah/menghapus layer suara
- Kontrol volume secara *real-time* per suara
- Pemutaran berjalan sebagai **Foreground Service** sehingga tetap aktif saat aplikasi berada di latar belakang

---

## Arsitektur Aplikasi

Proyek ini mengikuti pola arsitektur **MVVM (Model-View-ViewModel)** dengan pemisahan lapisan sebagai berikut:

```
UI Layer (Jetpack Compose)
    │
    ▼
ViewModel Layer (StateFlow + Coroutines)
    │
    ▼
Repository Layer (Interface + Implementasi Offline)
    │
    ▼
Data Layer (Room DAO + DataStore)
```

### Komponen Utama

| Komponen | Deskripsi |
|----------|-----------|
| `MainActivity` | Titik masuk aplikasi, menginisialisasi konten Compose dan melakukan pengecekan basis data |
| `AplikasiSleepMix` | Kelas `Application`, menginisialisasi `ContainerDataApp` sebagai *dependency container* |
| `ContainerDataApp` | Menyediakan instansi *repository* dan DAO secara *lazy* |
| `MixPlaybackService` | `MediaSessionService` (Media3) yang mengelola pemutaran audio di latar belakang |
| `AudioController` | Mengatur `MediaPlayer` per suara, termasuk efek *fade-in/fade-out* |
| `SessionManager` | Menyimpan dan membaca sesi pengguna menggunakan DataStore |
| `PasswordHasher` | Mengimplementasikan enkripsi password dengan SHA-256 + Salt |

---

## Teknologi dan Dependensi

### Bahasa & Platform
- **Kotlin** 2.2.0
- **Android SDK** — minSdk 27, targetSdk 36, compileSdk 36
- **Java Version** — 11

### UI
- **Jetpack Compose BOM** 2024.09.00
- **Material3** (Material Design 3)
- **Material Icons Extended**
- **Navigation Compose** 2.9.6

### Data & Penyimpanan
- **Room** 2.8.4 — basis data SQLite lokal (ORM)
- **DataStore Preferences** 1.2.0 — manajemen sesi pengguna
- **KSP** 2.2.10-2.0.2 — *code generation* untuk Room

### Audio
- **Media3 ExoPlayer** 1.8.0 — pemutaran media
- **Media3 Session** 1.8.0 — integrasi sesi media (notifikasi)
- **ExoPlayer** 2.19.1

### ViewModel & Lifecycle
- **Lifecycle ViewModel Compose** 2.10.0
- **Lifecycle Runtime Compose** 2.10.0

### Build
- **AGP** (Android Gradle Plugin) 8.13.2
- **Gradle** 8.13

---

## Struktur Proyek

```
app/src/main/java/com/example/sleepmix/
├── MainActivity.kt                     # Titik masuk aplikasi
│
├── media/
│   ├── AudioController.kt              # Manajemen MediaPlayer & efek fade
│   └── MixPlaybackService.kt           # Foreground Service pemutaran mix
│
├── repositori/
│   ├── ContainerApp.kt                 # Dependency container & Application class
│   ├── RepositoriMix.kt                # Interface & implementasi MixRepository
│   ├── RepositoriSound.kt              # Interface & implementasi SoundRepository
│   └── RepositoriUser.kt               # Interface & implementasi UserRepository
│
├── room/
│   ├── User.kt                         # Entity tabel pengguna
│   ├── Sound.kt                        # Entity tabel suara
│   ├── Mix.kt                          # Entity tabel mix
│   ├── MixSound.kt                     # Entity tabel relasi mix-suara
│   ├── MixWithSound.kt                 # Data class relasi (Embedded + Relation)
│   ├── SleepMixDatabase.kt             # Konfigurasi Room Database & SoundSeeds
│   └── dao/
│       ├── UserDao.kt
│       ├── SoundDao.kt
│       ├── MixDao.kt
│       └── MixSoundDao.kt
│
├── util/
│   ├── PasswordHasher.kt               # Enkripsi SHA-256 + Salt
│   └── SessionManager.kt              # DataStore untuk sesi pengguna
│
├── ui/
│   ├── navigation/
│   │   └── Navigation.kt              # NavHost & definisi semua rute
│   ├── components/
│   │   └── AvailableSoundItem.kt
│   ├── screens/
│   │   ├── LoginScreen.kt             # Halaman login
│   │   ├── RegisterScreen.kt          # Halaman registrasi
│   │   ├── HomeScreen.kt              # Perpustakaan suara
│   │   ├── SoundDetailScreen.kt       # Detail & pratinjau suara
│   │   ├── MyMixListScreen.kt         # Daftar mix pengguna
│   │   ├── CreateMixScreen.kt         # Buat mix baru
│   │   ├── MixDetailScreen.kt         # Detail & pemutaran mix
│   │   ├── EditMixScreen.kt           # Edit mix yang sudah ada
│   │   ├── EditVolumeScreen.kt        # Atur volume suara dalam mix
│   │   ├── SelectSoundScreen.kt       # Pilih suara untuk ditambahkan
│   │   ├── SetVolumeScreen.kt         # Atur volume awal suara baru
│   │   └── BrowseSoundScreen.kt       # Browse suara
│   └── theme/
│       ├── Color.kt
│       ├── Theme.kt
│       └── Type.kt
│
└── viewmodel/
    ├── LoginViewModel.kt
    ├── RegistrasiViewModel.kt
    ├── HomeViewModel.kt
    ├── BrowseSoundViewModel.kt
    ├── SoundDetailViewModel.kt
    ├── MyMixViewModel.kt
    ├── CreateMixViewModel.kt
    ├── EditMixViewModel.kt
    ├── EditVolumeViewModel.kt
    ├── MixDetailViewModel.kt
    ├── SelectSoundViewModel.kt
    └── provider/
        └── ViewModelFactory.kt        # Factory untuk semua ViewModel
```

---

## Basis Data

Aplikasi menggunakan **Room Database** versi 4 dengan skema sebagai berikut:

### Tabel `tblUser`
| Kolom | Tipe | Keterangan |
|-------|------|------------|
| `userId` | Int (PK, autoGenerate) | ID pengguna |
| `nama` | String | Nama lengkap |
| `email` | String | Alamat email (unik) |
| `passwordHash` | String | Password terenkripsi (salt:hash) |
| `isLoggedIn` | Boolean | Status login aktif |

### Tabel `tblSound`
| Kolom | Tipe | Keterangan |
|-------|------|------------|
| `soundId` | Int (PK, autoGenerate) | ID suara |
| `name` | String | Nama suara |
| `filePath` | String | Path resource audio |
| `iconRes` | Int | Resource ID ikon drawable |
| `category` | String | Kategori (Nature/Weather/Ambience) |
| `duration` | Int | Durasi dalam detik |

### Tabel `tblMix`
| Kolom | Tipe | Keterangan |
|-------|------|------------|
| `mixId` | Int (PK, autoGenerate) | ID mix |
| `userId` | Int (FK → tblUser) | Pemilik mix |
| `mixName` | String | Nama mix (maks. 30 karakter) |
| `creationDate` | Long | Tanggal pembuatan (epoch ms) |
| `lastModified` | Long | Tanggal modifikasi terakhir |

### Tabel `tblMixSound`
| Kolom | Tipe | Keterangan |
|-------|------|------------|
| `mixSoundId` | Int (PK, autoGenerate) | ID entri |
| `mixId` | Int (FK → tblMix, CASCADE) | Referensi mix |
| `soundId` | Int (FK → tblSound, CASCADE) | Referensi suara |
| `volumeLevel` | Float | Level volume (0.0 – 1.0) |

> **Catatan:** Database menggunakan `fallbackToDestructiveMigration()` untuk menangani pembaruan skema. Data suara bawaan (*seed*) diinisialisasi secara otomatis melalui `SleepMixDatabaseCallback` saat database pertama kali dibuat.

---

## Alur Navigasi

```
Login ──────────────────► Home (Perpustakaan Suara)
  │                           │
Register                      ├─► Sound Detail (Pratinjau)
                               │
                               └─► My Mix List
                                       │
                                       ├─► Create Mix
                                       │       │
                                       │       └─► Select Sound ──► Set Volume ──► (kembali ke Create Mix)
                                       │
                                       └─► Mix Detail (Putar Mix)
                                               │
                                               └─► Edit Mix
                                                       │
                                                       ├─► Edit Volume (per suara)
                                                       └─► Select Sound ──► Set Volume ──► (kembali ke Edit Mix)
```

### Daftar Layar (Screen)

| Halaman | Rute | Keterangan |
|---------|------|------------|
| Login | `login` | Autentikasi pengguna |
| Register | `register` | Pendaftaran akun baru |
| Home | `home/{userId}` | Perpustakaan suara & navigasi utama |
| Sound Detail | `sound_detail/{soundId}` | Pratinjau suara individual |
| My Mix List | `my_mix/{userId}` | Daftar mix milik pengguna |
| Create Mix | `create_mix/{userId}` | Pembuatan mix baru |
| Mix Detail | `mix_detail/{mixId}/{userId}` | Detail & tombol putar mix |
| Edit Mix | `edit_mix/{mixId}/{userId}` | Pengeditan mix yang ada |
| Edit Volume | `edit_volume/{mixId}/{soundId}` | Ubah volume suara dalam mix |
| Select Sound | `select_sound/{userId}/{fromEdit}/{mixId}` | Pilih suara dari perpustakaan |
| Set Volume | `set_volume/{soundId}/{userId}/{fromEdit}/{mixId}` | Atur volume awal suara baru |

---

## Cara Menjalankan

### Prasyarat
- **Android Studio** Hedgehog (2023.1.1) atau yang lebih baru
- **JDK 21**
- **Android Emulator** atau perangkat fisik dengan Android 8.1 (API 27) ke atas

### Langkah-langkah

1. **Clone repositori**
   ```bash
   git clone <url-repositori>
   cd SleepMix
   ```

2. **Buka di Android Studio**
   - Pilih *File → Open* dan arahkan ke direktori proyek

3. **Sync Gradle**
   - Android Studio akan melakukan sinkronisasi dependensi secara otomatis
   - Pastikan koneksi internet tersedia untuk mengunduh dependensi

4. **Tambahkan file audio**
   - Letakkan file audio di `app/src/main/res/raw/` dengan nama:
     `bird`, `cricket`, `rain`, `sea`, `thunder`, `firewood`, `frog`, `river`
   - Format yang didukung: `.mp3`, `.ogg`, `.wav`

5. **Tambahkan file ikon suara**
   - Letakkan drawable icon di `app/src/main/res/drawable/` dengan nama yang sesuai:
     `bird`, `cricket`, `rain`, `sea`, `thunder`, `firewood`, `frog`, `river`

6. **Build dan jalankan**
   - Pilih target perangkat (emulator atau fisik)
   - Klik *Run → Run 'app'* atau tekan `Shift+F10`

### Perizinan Aplikasi
Perizinan berikut dideklarasikan dalam `AndroidManifest.xml`:
- `FOREGROUND_SERVICE` — untuk pemutaran audio di latar belakang
- `FOREGROUND_SERVICE_MEDIA_PLAYBACK` — tipe layanan media
- `WAKE_LOCK` — mencegah perangkat tertidur saat audio diputar

---

## Persyaratan Sistem

| Aspek | Spesifikasi |
|-------|-------------|
| Sistem Operasi | Android 8.1 (API Level 27) ke atas |
| Target SDK | Android 16 (API Level 36) |
| Arsitektur | arm64-v8a, x86_64 |
| RAM | Minimal 2 GB (disarankan 3 GB) |
| Penyimpanan | Minimal 50 MB ruang kosong |

---

## Catatan Pengembangan

- Aplikasi menggunakan **`fallbackToDestructiveMigration()`** pada Room Database, yang berarti seluruh data akan dihapus saat versi skema diperbarui. Untuk lingkungan produksi, disarankan mengimplementasikan **migrasi manual**.
- Password pengguna disimpan dalam format `salt:hash` menggunakan **SHA-256**. Ini merupakan metode enkripsi satu arah yang aman untuk penyimpanan lokal.
- Sesi pengguna disimpan menggunakan **DataStore Preferences** dan bertahan meskipun aplikasi ditutup. Sesi hanya dihapus saat pengguna melakukan logout secara eksplisit.
- `AudioController` menggunakan `Handler` pada *main thread* untuk animasi volume, sehingga tidak memblokir UI.
