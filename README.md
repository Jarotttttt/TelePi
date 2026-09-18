# TelePi

**Kendalikan Pi coding agent Anda langsung dari Telegram: prompt suara, screenshot, handoff sesi, dan terminal handback.**

TelePi adalah jembatan (bridge) Telegram untuk [Pi coding agent](https://github.com/badlogic/pi-mono). Berjalan secara lokal di komputer Anda (mendukung Windows, Linux, dan macOS), membuka sesi Pi nyata di repositori Anda, memungkinkan Anda melanjutkan coding dari ponsel, dan mengembalikan sesi persis ke terminal saat Anda kembali.

**Untuk siapa TelePi dibuat:** Pengembang yang menggunakan Pi dan menginginkan remote control mobile yang aman: balas dari jalan, kirim screenshot/foto bug, dikte prompt dengan voice memo, pantau progres pemanggilan tool secara real-time, lalu lanjutkan di terminal CLI tanpa kehilangan konteks riwayat obrolan.

---

## ⚡ Mulai Cepat dalam 5 Menit

### Prasyarat
- **Node.js 22.19+** (atau Node.js 24+)
- Token bot Telegram dari [@BotFather](https://t.me/BotFather)
- User ID numerik Telegram Anda (dapatkan dari [@userinfobot](https://t.me/userinfobot))
- Pi CLI telah terpasang dan terautentikasi (`~/.pi/agent/auth.json` atau `%USERPROFILE%\.pi\agent\auth.json`)

### Langkah Instalasi & Menjalankan

1. **Clone repository ini:**
   ```bash
   git clone https://github.com/Jarotttttt/TelePi.git
   cd TelePi
   ```

2. **Pasang dependensi:**
   ```bash
   npm install
   ```

3. **Konfigurasi file `.env`:**
   Salin template contoh:
   ```bash
   cp .env.example .env
   ```
   *(Di Windows PowerShell: `Copy-Item .env.example .env`)*

   Buka `.env` dan isi minimal variabel wajib berikut:
   ```dotenv
   TELEGRAM_BOT_TOKEN=123456789:AAFf_token_dari_botfather
   TELEGRAM_ALLOWED_USER_IDS=123456789
   TELEPI_WORKSPACE=D:\Path\Ke\Proyek\Utama\Anda
   ```

4. **Build proyek:**
   ```bash
   npm run build
   ```

5. **Jalankan bot:**
   * **Mode pengembangan (auto-reload via tsx):**
     ```bash
     npm run dev
     ```
   * **Mode produksi (menjalankan hasil build):**
     ```bash
     npm start
     ```

6. **Verifikasi:**
   Buka Telegram, cari bot Anda, lalu kirimkan perintah `/start`. Bot akan membalas dengan status workspace, sesi aktif, dan backend audio.

---

## 🔄 Alur Sesi: Handoff & Handback

### 1. Terminal CLI ➔ Telegram (`/handoff`)
Saat Anda sedang bekerja di Pi CLI di laptop dan ingin melanjutkan dari ponsel:
1. Di terminal Pi CLI, ketik `/handoff`.
2. Ekstensi akan menyimpan sesi Anda dan meluncurkan TelePi di latar belakang (*background process*).
3. Buka Telegram: percakapan sebelumnya sudah tersambung utuh. Lanjutkan mengetik atau kirim voice note!

### 2. Telegram ➔ Terminal CLI (`/handback`)
Saat Anda kembali ke meja kerja dan ingin kembali ke terminal:
1. Di chat Telegram, ketik `/handback`.
2. TelePi akan menutup sesi aktif dan mengirimkan perintah resume, contoh:
   ```bash
   cd "D:\Path\Ke\Proyek" && pi --session "C:\Users\User\.pi\agent\sessions\...\session.jsonl"
   ```
   atau cara cepat:
   ```bash
   cd "D:\Path\Ke\Proyek" && pi -c
   ```
3. **Auto-Clipboard:** Perintah di atas **otomatis disalin ke clipboard komputer Anda** (mendukung `clip.exe` di Windows, `pbcopy` di macOS, serta `wl-copy`/`xclip`/`xsel` di Linux).
4. Buka terminal Anda, tekan `Ctrl+V` (atau `Cmd+V`) dan tekan Enter. Anda kembali ke Pi CLI dengan seluruh pesan dari Telegram tersimpan lengkap!

---

## 📱 Daftar Perintah Telegram

| Perintah | Deskripsi |
| :--- | :--- |
| `/start` | Pesan selamat datang, info sesi aktif, dan status backend transkripsi suara. |
| `/help` | Panduan cepat penggunaan dan daftar perintah. |
| `/commands` | Membuka command picker interaktif (filter: All, TelePi, dan perintah Pi). |
| `/new` | Membuat sesi baru (menampilkan pemilih workspace jika ada beberapa folder proyek). |
| `/retry` | Mengirim ulang prompt terakhir jika terjadi gangguan jaringan atau output terputus. |
| `/handback` | Mengembalikan sesi ke terminal dan menyalin perintah resume ke clipboard lokal. |
| `/abort` | Membatalkan paksa eksekusi prompt/tool Pi yang sedang berjalan. |
| `/session` | Menampilkan detail sesi aktif (ID sesi, file JSONL, workspace, model AI). |
| `/sessions` | Menampilkan daftar seluruh sesi dari semua workspace dengan tombol switch. |
| `/sessions <id\|path>` | Berpindah sesi secara instan menggunakan ID/prefix atau path file. |
| `/model` | Memilih model AI yang berbeda menggunakan *inline keyboard*. |
| `/tree` | Menampilkan struktur pohon riwayat percakapan (tree diagram). |
| `/branch <id>` | Berpindah kembali ke turn/node tertentu (dengan opsi summary cabang lama). |
| `/label [nama]` | Menambahkan atau menghapus label penanda pada node sesi tertentu. |

*Catatan:* Sesi, tombol interaktif, dan status `/retry` terisolasi per chat / per forum topic, sehingga Anda dapat menggunakan beberapa topic forum Telegram secara independen tanpa bentrok.

---

## 🎙️ Pesan Suara & Gambar (Multi-Modal)

### 1. Pesan Suara (Voice Memo & Audio)
Kirim pesan suara atau rekaman audio apa pun ke bot Telegram. TelePi akan mentranskripsikannya ke teks dan langsung menjalankannya sebagai prompt coding ke agen.

Backend transkripsi yang didukung:
* **Cloud Transcription (Direkomendasikan di Windows):**
  Tambahkan API Key di `.env`:
  ```dotenv
  OPENAI_API_KEY=sk-...
  ```
  Atau gunakan endpoint penyedia kustom yang kompatibel dengan format Whisper:
  ```dotenv
  TELEPI_TRANSCRIPTION_URL=https://api.provider.com/v1/audio/transcriptions
  TELEPI_TRANSCRIPTION_MODEL=whisper-1
  TELEPI_TRANSCRIPTION_API_KEY=key-anda
  ```
* **Offline On-Device (Sherpa-ONNX):**
  Mendukung model Parakeet offline berbasis CPU/Intel:
  ```dotenv
  SHERPA_ONNX_MODEL_DIR=D:\Path\Ke\Model\sherpa-onnx-nemo-parakeet
  ```

### 2. Kirim Screenshot / Foto
Kirim foto atau dokumen gambar (PNG/JPEG) ke bot Telegram:
* Gambar akan otomatis diumpankan sebagai `ImageContent` ke model vision Pi.
* Jika menyertakan *caption*, teks tersebut akan menjadi prompt instruksi.
* Jika tanpa *caption*, agen otomatis diminta menganalisis gambar yang dikirimkan.

---

## 🤖 Interactive Extension Dialogs

Jika Anda memiliki script ekstensi Pi yang meminta interaksi pengguna (*human-in-the-loop*), TelePi mengonversinya secara otomatis ke komponen native Telegram:
* `ctx.ui.select(title, options)` ➔ Tombol pilihan *inline keyboard*.
* `ctx.ui.confirm(title, message)` ➔ Tombol *Yes / No*.
* `ctx.ui.input(title, placeholder)` ➔ Menunggu balasan teks dari pengguna.
* `ctx.ui.notify(message, type)` ➔ Notifikasi info / warning / error di chat.

---

## 📬 External Prompt Inbox (Otomasi Otomatis)

TelePi mendukung eksekusi prompt otomatis dari script eksternal, cron job, atau webhook tanpa perlu membuka chat Telegram:
1. Aktifkan konfigurasi di `.env`:
   ```dotenv
   TELEPI_PROMPT_INBOX_DIR=D:\Path\Ke\Folder\Inbox
   TELEPI_PROMPT_INBOX_INTERVAL_MS=60000  # Default 60 detik (minimal 1000ms)
   ```
2. Skrip eksternal (cron, CI/CD, backup watcher) cukup meletakkan file berakhiran `.txt` ke folder tersebut.
3. TelePi akan membaca file tersebut, mengeksekusi instruksinya di sesi Pi, mengirimkan hasilnya ke akun Telegram Anda, dan menghapus file `.txt` setelah diproses.

---

## 🛡️ Keamanan & Pembatasan Akses

* **Strict User Allowlist:** Hanya ID Telegram yang terdaftar di `TELEGRAM_ALLOWED_USER_IDS` yang dapat menggunakan bot. Siapa pun yang menemukan bot Anda akan langsung ditolak jika tidak terdaftar.
* **Workspace Scoped:** Tool coding (`read`, `write`, `edit`, `bash`) otomatis di-scope ke direktori proyek aktif.
* **Bash Safe Guards:**
  * Timeout default **120 detik** untuk mencegah terminal hang jika perintah interaktif dijalankan.
  * Perlindungan anti-rekursif untuk mencegah bot mematikan dirinya sendiri secara tidak sengaja.

---

## 🏗️ Perintah Pengujian & Pengembangan

```bash
# Menjalankan bot dalam mode development (auto-reload tsx)
npm run dev

# Kompilasi TypeScript
npm run build

# Menjalankan seluruh pengujian (Vitest)
npm test

# Menjalankan pengujian dengan laporan cakupan kode (coverage report)
npm run test:coverage

# Memeriksa status konfigurasi lokal
node dist/cli.js status
```

---

## 📄 Lisensi

Proyek ini dilisensikan di bawah lisensi **MIT License** — lihat berkas [LICENSE](LICENSE) untuk detail lengkap.
TelePi dibangun di atas ekosistem luar biasa dari [Pi Coding Agent](https://github.com/badlogic/pi-mono).
