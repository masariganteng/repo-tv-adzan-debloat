# Panduan Pemula: Android TV Makeover (dari repo asli, langkah demi langkah)

Panduan ini untuk **Windows**. Ikuti urut dari Langkah 1. Setiap langkah ada tiga bagian:
**Lakukan**, **Hasil yang benar**, dan **Kalau gagal**. Jangan lompat langkah.

Hasil akhirnya: layar Home TV Anda diganti launcher bersih ala Apple TV, bisa dikontrol dari HP,
dan (opsional) ada widget **jam shalat + adzan**.

---

## Baca dulu (2 menit)

| Pertanyaan | Jawaban |
|---|---|
| Berapa lama? | sekitar 45-60 menit (kebanyakan menunggu unduhan) |
| Butuh ruang di PC? | sekitar **3 GB** kosong di drive C: (atau drive lain, lihat Langkah 3) |
| TV apa yang cocok? | Android TV / Google TV, **Android 9 ke atas**. **Tidak** untuk Samsung Tizen, LG webOS, Fire TV. |
| Aman? | Tidak ada root. Semua bisa dibatalkan (Langkah 14). Tetap **backup dulu** (Langkah 7). |
| Apa yang saya butuhkan? | PC Windows, TV dan PC di **Wi-Fi yang sama**, remote TV, internet. |

> Peringatan: jangan nonaktifkan paket sistem sembarangan di Langkah 8. Langkah itu **boleh dilewati**.
> Launcher tetap jalan tanpa debloat.

---

## Langkah 1. Aktifkan "ADB" di TV

ADB adalah pintu yang membuat PC bisa mengirim aplikasi ke TV lewat Wi-Fi.

**Lakukan** (pakai remote TV):
1. Buka **Pengaturan → Preferensi Perangkat → Tentang**.
2. Tekan **OK** pada tulisan **Build** sebanyak **7 kali** sampai muncul "Anda sekarang developer".
3. Tekan Back. Masuk **Opsi Pengembang**.
4. Nyalakan **USB debugging**. Kalau ada **Network debugging** / **ADB debugging**, nyalakan juga.
5. Catat **alamat IP TV**: Pengaturan → Jaringan → (nama Wi-Fi Anda). Contoh: `192.168.18.73`.

**Hasil yang benar:** Opsi Pengembang muncul, dan Anda punya alamat IP seperti `192.168.x.x`.

**Kalau gagal:** menu "Opsi Pengembang" tidak ada → ulangi tekan Build 7 kali, pastikan menekan **OK** di
remote, bukan sekadar menggeser.

> Tips: di router, **reservasi IP tetap** untuk TV supaya alamatnya tidak berubah-ubah.
> Di panduan ini IP TV ditulis `192.168.18.73`. **Ganti dengan IP TV Anda.**

---

## Langkah 2. Cek PC bisa menjangkau TV

**Lakukan:** buka **Command Prompt** (tekan tombol Windows, ketik `cmd`, Enter), lalu:
```
ping 192.168.18.73
```

**Hasil yang benar:** muncul balasan `Reply from 192.168.18.73 ... time=...ms`.

**Kalau gagal** (`Request timed out`):
- TV dan PC harus di Wi-Fi yang sama (bukan Wi-Fi tamu).
- TV sedang mati/standby → nyalakan, lalu coba lagi.
- IP salah → cek lagi di Pengaturan TV.

---

## Langkah 3. Pasang alat di PC (sekali saja)

Buka **PowerShell** (tombol Windows, ketik `powershell`, Enter) dan jalankan satu per satu.
Setelah tiap pemasangan, **tutup lalu buka lagi** PowerShell.

**3a. Git dan Python**
```
winget install --id Git.Git -e
winget install --id Python.Python.3.11 -e
```

**3b. Java (JDK 17)**
```
winget install --id Microsoft.OpenJDK.17 -e
```

**3c. Android SDK (alat build)**
1. Buka https://developer.android.com/studio#command-line-tools-only
2. Unduh **Command line tools** untuk Windows (file ZIP, ±150 MB).
3. Buat folder `C:\Android\sdk\cmdline-tools\latest`.
4. Ekstrak isi folder `cmdline-tools` dari ZIP ke dalam `latest`, sehingga ada
   `C:\Android\sdk\cmdline-tools\latest\bin\sdkmanager.bat`.

> Ruang C: sempit? Ganti `C:\Android\sdk` di panduan ini dengan drive lain, misalnya `D:\Android\sdk`,
> secara konsisten di semua langkah.

Lalu buka **Git Bash** (klik kanan di desktop → *Git Bash Here*) dan jalankan:
```
yes | /c/Android/sdk/cmdline-tools/latest/bin/sdkmanager.bat --sdk_root="C:\\Android\\sdk" --licenses
/c/Android/sdk/cmdline-tools/latest/bin/sdkmanager.bat --sdk_root="C:\\Android\\sdk" "platforms;android-34" "build-tools;34.0.0" "platform-tools"
```

**Hasil yang benar:** setelah beberapa menit ada folder `C:\Android\sdk\platforms\android-34` dan
`C:\Android\sdk\build-tools\34.0.0`.

**Kalau gagal:**
- `java` tidak dikenali → tutup dan buka lagi Git Bash.
- Unduhan terhenti → jalankan perintah yang sama sekali lagi.

> Mulai langkah ini seterusnya, **pakai Git Bash**, bukan PowerShell.

---

## Langkah 4. Ambil repo asli

**Pilihan A (mudah, tanpa Git):**
1. Buka halaman repo asli di GitHub: `<ISI-ALAMAT-REPO-ASLI>`.
2. Klik tombol hijau **Code → Download ZIP**.
3. Ekstrak ke folder yang namanya **tanpa spasi**, misalnya `C:\Users\NAMA\tv-makeover`.

**Pilihan B (dengan Git):**
```
cd /c/Users/NAMA
git clone <ISI-ALAMAT-REPO-ASLI> tv-makeover
```

Lalu masuk ke foldernya (ganti `NAMA`):
```
cd /c/Users/NAMA/tv-makeover
ls
```

**Hasil yang benar:** terlihat `launcher`, `tools`, `apps`, `backup.sh`, `README.md`.

**Kalau gagal:** `No such file` → periksa nama folder. Kalau hasil ekstrak ZIP punya folder di
dalam folder (`tv-makeover/tv-makeover/...`), masuk ke folder yang lebih dalam sampai `ls` benar.

---

## Langkah 5. Siapkan alat penghubung ke TV

Repo asli menyiapkan ini lewat `setup.sh` yang ditulis untuk Mac. Di Windows lakukan manual.

**Lakukan** (di folder repo, Git Bash):
```
python -m venv .venv
.venv/Scripts/python -m pip install -r requirements.txt
echo "192.168.18.73" > .tv-ip
```
(ganti `192.168.18.73` dengan IP TV Anda).

**Dua perbaikan wajib untuk Windows**

**5a.** Ganti isi file `tools/tv` dengan ini (supaya tahu lokasi Python di Windows):
```
#!/bin/sh
D="$(cd "$(dirname "$0")/.." && pwd)"
PY="$D/.venv/bin/python"
[ -x "$PY" ] || PY="$D/.venv/Scripts/python"
exec "$PY" "$D/tools/tv.py" "$@"
```

**5b.** Buka `tools/tv.py`, cari baris `step = 60000` dan ubah jadi:
```
step = 8000
```
(Potongan 60.000 gagal diam-diam di beberapa TV: muncul error `md5 mismatch`.)

**Hasil yang benar:** `.venv/Scripts/python -c "import adb_shell; print('ok')"` mencetak `ok`.

**Kalau gagal:** `python` tidak dikenali → tutup dan buka lagi Git Bash; pastikan Langkah 3a sudah dijalankan.

---

## Langkah 6. Sambungkan ke TV

**Lakukan:**
```
sh tools/tv shell "getprop ro.product.model; getprop ro.build.version.sdk"
```

**Penting:** lihat layar **TV**. Akan muncul **"Allow USB debugging?"**.
Centang **"Always allow"**, lalu pilih **OK** dengan remote. Anda hanya punya sekitar 1 menit.

**Hasil yang benar:** tercetak nama model TV dan angka SDK. **Angka SDK harus 28 atau lebih.**

**Kalau gagal:**
- `TimeoutError` → ulangi sekali lagi (TV kadang lambat). Pastikan layar izin diterima.
- Angka di bawah 28 → TV terlalu tua, launcher ini tidak bisa dipasang.

---

## Langkah 7. Backup dulu

**Lakukan:**
```
sh backup.sh
```
Tunggu 1-2 menit.

**Hasil yang benar:** tertulis `Backup saved to backup/...`. Di dalamnya ada daftar paket, setelan,
dan screenshot TV.

**Kalau gagal:** jalankan sekali lagi. Jangan lanjut kalau backup belum jadi.

> Folder `backup/` berisi data TV Anda. **Jangan dibagikan** ke orang lain.

---

## Langkah 8. Debloat (opsional, boleh dilewati)

Debloat = menonaktifkan aplikasi bawaan pabrikan yang membuat TV lambat (iklan, telemetri).
Aplikasinya **dinonaktifkan, tidak dihapus**, jadi bisa dihidupkan lagi.

> **Jangan** langsung memakai `profiles/xiaomi-mitv-axso0.txt`. File itu untuk satu model Xiaomi saja.
> Nama paket tiap merek berbeda.

**Lakukan:**
1. Buka `backup/<tanggal>/packages_all.txt`. Isinya seperti `package:/system/...=nama.paket`.
2. Untuk tiap nama paket yang mau dinonaktifkan, cek dulu apakah ada di TV Anda:
   ```
   grep "=nama.paket$" backup/<tanggal>/packages_all.txt
   ```
   Kalau tidak ada hasil, lewati paket itu.
3. Nonaktifkan **satu per satu**:
   ```
   sh tools/tv shell "pm disable-user --user 0 nama.paket"
   ```

**Hasil yang benar:** `Package nama.paket new state: disabled-user`.

**Jangan pernah nonaktifkan:** layanan input TV/HDMI, remote/Bluetooth, keyboard, Google Play Store,
Google Play Services.

**Kalau layar TV jadi aneh:** aktifkan lagi dengan `pm enable nama.paket` (Langkah 14).

---

## Langkah 9. Atur launcher

Buka file ini dengan Notepad atau VS Code:
`launcher/app/src/main/res/values/config.xml`

Ubah yang perlu:

| Setelan | Artinya | Contoh |
|---|---|---|
| `dock_packages` | 5 aplikasi di dock, kiri ke kanan | Netflix, YouTube, Play Store |
| `youtube_channel`, `instagram_user`, `facebook_page` | **kosongkan** (isi bawaan milik pembuat repo) | (dikosongkan) |
| `greeting_name` | nama di salam "Good evening, Nama" | `Ahmad` |
| `columns` | jumlah app per baris (3-6) | `5` |

Cari nama paket sebuah aplikasi di TV:
```
sh tools/tv shell "pm list packages | grep -i netflix"
```
Hasil `package:com.netflix.ninja` → tulis `com.netflix.ninja` di `dock_packages`.

**Hasil yang benar:** file tersimpan tanpa error tanda `<` atau `>` yang hilang.

---

## Langkah 10. Build (membuat file aplikasi)

**Lakukan:** set lokasi alat. Jalankan ini **setiap membuka Git Bash baru**:
```
export JAVA_HOME=$(ls -d "/c/Program Files/Microsoft/jdk-17"* | head -1)
export PATH="$JAVA_HOME/bin:$PATH"
export ANDROID_HOME=/c/Android/sdk
```
Beri tahu Gradle di mana SDK berada (sekali saja, pakai **slash biasa**):
```
echo "sdk.dir=C:/Android/sdk" > launcher/local.properties
```
Build:
```
cd launcher
./gradlew -q clean assembleDebug --no-watch-fs
cd ..
ls -la launcher/app/build/outputs/apk/debug/app-debug.apk
```
Build pertama sekitar **3-5 menit** (mengunduh Gradle dan pustaka). Tidak ada tulisan = normal.

**Hasil yang benar:** ada file `app-debug.apk` berukuran sekitar 4-5 MB.

**Kalau gagal:**
- `filename, directory name... syntax is incorrect` → isi `local.properties` salah. Harus `sdk.dir=C:/Android/sdk`.
- `SDK location not found` → Langkah 3c belum selesai, atau `local.properties` tidak ada.
- **Selalu pakai `clean`.** Tanpa itu Gradle bisa memakai hasil lama.

---

## Langkah 11. Pasang ke TV

Kirim APK dalam potongan kecil dan cek keutuhannya (md5). Jalankan seluruh blok ini sekaligus:
```
.venv/Scripts/python - <<'EOF'
import sys; sys.path.insert(0,"tools")
import tv, base64, hashlib
d=tv.connect()
data=open("launcher/app/build/outputs/apk/debug/app-debug.apk","rb").read()
b=base64.b64encode(data).decode()
r="/data/local/tmp/atvhome.apk"; t=r+".b64"
tv.sh(d,f"rm -f {t} {r}")
for i in range(0,len(b),8000): tv.sh(d,f"echo -n '{b[i:i+8000]}' >> {t}")
ok = tv.sh(d,f"base64 -d {t} > {r}; rm -f {t}; md5sum {r}").split()[0]==hashlib.md5(data).hexdigest()
print("md5 cocok:", ok)
EOF
```

**Hasil yang benar:** tercetak `md5 cocok: True`. **Hanya lanjut kalau `True`.**

Lalu:
```
sh tools/tv shell "pm install -r -t /data/local/tmp/atvhome.apk"
sh tools/tv shell "rm /data/local/tmp/atvhome.apk"
sh tools/tv shell "appops set com.kulwant.atvhome SYSTEM_ALERT_WINDOW allow"
sh tools/tv shell "am start -n com.kulwant.atvhome/.HomeActivity"
```

**Hasil yang benar:** `Success`, lalu layar TV menampilkan launcher baru: jam besar, tombol Inputs/Wi-Fi/Settings,
dan dock aplikasi di bawah.

**Kalau gagal:** `False` → ulangi blok pengiriman. `Unknown package` → file belum terkirim utuh, ulangi.

---

## Langkah 12. Jadikan layar Home (tombol Home di remote)

Sampai sini launcher baru sudah tampil, tapi tombol **Home** di remote mungkin masih membuka
launcher lama. Di banyak TV Android 10 satu-satunya cara andal adalah menonaktifkan launcher bawaan.

**Lakukan hanya setelah** Langkah 11 menunjukkan launcher baru tampil dan Anda bisa menggerakkannya
dengan remote.

Cek nama launcher bawaan:
```
sh tools/tv shell "cmd package resolve-activity --brief -a android.intent.action.MAIN -c android.intent.category.HOME"
```
Contoh hasil: `com.google.android.tvlauncher/.MainActivity`. Nonaktifkan nama paketnya:
```
sh tools/tv shell "pm disable-user --user 0 com.google.android.tvlauncher"
sh tools/tv shell "pm disable-user --user 0 com.google.android.tvrecommendations"
```

**Hasil yang benar:** tekan **Home** di remote → yang terbuka launcher baru.

**Kalau gagal / layar kosong:** langsung ke Langkah 14 dan aktifkan lagi launcher lama.

---

## Langkah 13. Kontrol dari HP (opsional)

1. Di HP (Wi-Fi yang sama), buka browser ke `http://192.168.18.73:8080` (pakai IP TV Anda).
2. Tap **Pair with TV**.
3. Di TV muncul "Allow this phone to control the TV?". Tekan **Kanan**, lalu **OK** di remote.
4. Pertama kali memakai tab Remote, TV menanyakan "Allow USB debugging?" lagi. Centang **Always allow**.

**Hasil yang benar:** trackpad, volume, dan daftar aplikasi muncul di HP.

---

## Langkah 14. Cara membatalkan

Semuanya bisa dikembalikan:
```
sh tools/tv shell "pm enable com.google.android.tvlauncher"
sh tools/tv shell "pm enable com.google.android.tvrecommendations"
sh tools/tv shell "pm uninstall com.kulwant.atvhome"
sh tools/tv shell "pm enable nama.paket"
```
Ulangi baris terakhir untuk **setiap paket** yang tadi Anda nonaktifkan di Langkah 8.

---

## Tambahan: widget jam shalat + adzan (opsional)

> **Ini bukan bagian dari repo asli.** Fiturnya dibuat terpisah dan sudah ada di folder proyek yang
> sedang Anda pakai ini. Kalau Anda memakai repo asli saja, salin 4 hal berikut dari folder ini ke repo Anda,
> menimpa file yang sama, lalu ulangi **Langkah 10 dan 11**:
>
> 1. `launcher/app/src/main/java/com/kulwant/atvhome/PrayerTimes.java` (file baru)
> 2. `launcher/app/src/main/java/com/kulwant/atvhome/HomeActivity.java` (menimpa)
> 3. `launcher/app/src/main/res/values/config.xml` (menimpa, atau salin bagian `prayer_*` dan `adzan_sound`)
> 4. folder `launcher/app/src/main/res/raw/` berisi `adzan.mp3` dan `adzan_subuh.mp3`

Setelannya di `config.xml`:

| Setelan | Isi |
|---|---|
| `prayer_city` | nama kota, contoh `Gresik` |
| `prayer_lat`, `prayer_lng` | koordinat desimal kota Anda (selatan dan barat bernilai negatif) |
| `prayer_fajr_angle`, `prayer_isha_angle`, `prayer_ihtiyath_min` | Kemenag RI: `20`, `18`, `2` |
| `adzan_sound` | `false` untuk mematikan suara |
| `prayer_enabled` | `false` untuk mematikan widget |

Cara kerja:
- Jam shalat dihitung langsung di TV, **tanpa internet**.
- Diam 30 detik di Home → layar hitam menampilkan jam, hitung mundur, dan jam shalat.
- Saat waktu shalat tiba (dan Home sedang tampil) → banner "Adzan ..." dan suara adzan. Tombol remote apa pun menghentikannya.
- Zona waktu mengikuti TV. Pastikan sudah benar (WIB/WITA/WIT).

Uji (pelankan volume dulu):
```
sh tools/tv shell "am start -n com.kulwant.atvhome/.HomeActivity --ez test_adzan true"
```

Audio adzan: ganti file di folder `res/raw/` dengan nama yang sama (huruf kecil, tanpa spasi), lalu build
dan pasang ulang. Pastikan lisensi rekaman Anda memperbolehkan dipakai atau dibagikan.

---

## Masalah umum

| Gejala | Penyebab / solusi |
|---|---|
| `TimeoutError` saat menyambung | Layar "Allow USB debugging?" belum diterima, atau TV sibuk. Ulangi. |
| `md5 mismatch` atau `md5sum:` | Potongan pengiriman terlalu besar. Pastikan Langkah 5b sudah dilakukan. |
| APK di TV tidak berubah setelah build | Build tanpa `clean`. Ulangi `./gradlew -q clean assembleDebug --no-watch-fs`. |
| Tombol Home membuka launcher lama | Langkah 12 belum dilakukan. |
| Layar TV hitam | Cek **input TV** (HDMI vs Android). Kalau input benar, Langkah 14 untuk kembali ke launcher lama. |
| HP tidak bisa membuka halaman remote | HP harus di Wi-Fi yang sama, dan alamatnya harus **IP TV** (bukan nama). |
| Adzan tidak berbunyi | Home harus sedang tampil saat waktunya. Cek volume TV dan output audio. |
| `java` atau `python` tidak dikenali | Tutup dan buka lagi Git Bash setelah memasang. |
| TV Android 11+ hanya punya "Wireless debugging" | Menu itu memakai port acak dan kode pairing, tidak cocok dengan alat ini. Sambungkan TV ke PC lewat USB sekali dan jalankan `adb tcpip 5555` (belum teruji di panduan ini). |

---

## Daftar periksa akhir

- [ ] Opsi Pengembang aktif, IP TV dicatat
- [ ] `ping` berhasil
- [ ] Git, Python, JDK 17, Android SDK terpasang
- [ ] Repo diambil, `tools/tv` dan `tools/tv.py` diperbaiki untuk Windows
- [ ] Sambungan ADB berhasil dan "Always allow" dicentang
- [ ] **Backup dibuat**
- [ ] `config.xml` diatur (dock, counter sosial dikosongkan)
- [ ] Build berhasil, `md5 cocok: True`, terpasang `Success`
- [ ] Launcher tampil di TV, lalu baru nonaktifkan launcher lama

Selesai. Kalau macet, catat **nomor langkah** dan **pesan error-nya**; itu yang paling membantu saat meminta bantuan.
