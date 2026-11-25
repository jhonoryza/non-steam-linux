# Resident Evil 4 Remake (non-Steam + Proton) — catatan instalasi & troubleshooting

Tujuan: kalau game harus di-install ulang / crash lagi, agent lain bisa langsung mengikuti ini tanpa mengulang diagnosis.

Status terakhir: **BERHASIL jalan** (26 Nov 2026).

## Lokasi & identitas

| Item | Nilai |
|---|---|
| Folder game | `/home/vm/hdd/programs/Resident Evil 4 2023` |
| Executable | `re4.exe` |
| Sumber game | repack ElAmigos (diunduh dari ovagames.com), crack EMPRESS |
| Steam shortcut AppID | `3642743466` (non-Steam game) |
| AppID game asli | `2050650` |
| Steam user ID | `104758288` |
| Shortcut config | `/home/vm/.local/share/Steam/userdata/104758288/config/shortcuts.vdf` |
| Proton mapping | `/home/vm/.local/share/Steam/config/config.vdf` → `CompatToolMapping["3642743466"]` |
| Proton yang dipakai | `Proton 8.0` (`/mnt/transcend/SteamLibrary/steamapps/common/Proton 8.0`) |
| Proton lain terpasang | Proton 10.0, Proton - Experimental, GE-Proton11-14-x86_64 |

## Konfigurasi final yang bekerja

**Launch Options** (Steam → klik kanan shortcut → Properties → Launch Options):

```
WINEDLLOVERRIDES="steam_api64=n,b;winmm=n,b;amd_ags_x64=n,b" PROTON_LOG=1 %command%
```

**File di folder game:**

- `steam_appid.txt` → isi `2050650`
- `steam_api64.ini` → dari crackfix EMPRESS (`AppId=2050650`)
- `EMP.dll` → dari crackfix EMPRESS (13.772.800 byte, md5 `6a0ab7d49ddba5bb6fc6ef06b466afbd`)
- Backup file lama: `EMP.dll.bak`, `steam_api64.ini.bak`

`PROTON_LOG=1` bisa dicopot setelah stabil; log-nya di `~/steam-2050650.log`.

## Kronologi masalah & akar penyebab

1. **"Unable to initialize SteamAPI. Please make sure Steam is running and you are logged in to an account entitled to the game."**
   - Muncul karena game dijalankan sebagai *non-Steam shortcut* lewat Proton. Steam menyuntik DLL Steam/Proton yang menimpa DLL crack EMPRESS.
   - Fix: `WINEDLLOVERRIDES` + `steam_appid.txt` + turunkan Proton ke 8.0. Proton 10 / Experimental bikin error ini muncul lagi.
   - Setting lewat GUI Steam tidak perlu restart Steam; edit `shortcuts.vdf` manual saat Steam jalan **perlu** restart.

2. **Crash Report Tool — Error Code `0x2000000`**
   - Setelah SteamAPI beres, game masih crash di inisialisasi renderer. Log Proton menunjukkan `re4.exe` melempar `0xc06d007f` lalu `CrashReport.exe` jalan.
   - Akar: `EMP.dll` dari repack bertanggal 14 Mei 2023 = **sebelum** EMPRESS merilis crackfix Linux (16 Mei 2023). Crackfix release note: *"Fixed crashing under Linux via wine/proton."*
   - Fix: ganti `EMP.dll` + `steam_api64.ini` dengan versi crackfix.

## Crackfix

- Arsip tersimpan: `/home/vm/hdd/download/Resident.Evil.4.Crackfix-EMPRESS.7z` (246 MB)
- Magnet infohash: `40bb37324fc6b3639704c571adea3712ec6bd95`
- Sumber: rilis `Resident.Evil.4.Crackfix-EMPRESS` (DODI / 1337x / kanal EMPRESS)
- Cara baca/extract di mesin ini: `bsdtar` (tidak perlu 7z)

```bash
A='/home/vm/hdd/download/Resident.Evil.4.Crackfix-EMPRESS.7z'
G='/home/vm/hdd/programs/Resident Evil 4 2023'

bsdtar -tf "$A"                    # lihat isi
bsdtar -xOf "$A" 'Crackfix/EMP.dll' > "$G/EMP.dll"
bsdtar -xOf "$A" 'Crackfix/steam_api64.ini' > "$G/steam_api64.ini"
```

**Penting:** dari seluruh isi `Crackfix/`, hanya `EMP.dll` dan `steam_api64.ini` yang beda dengan file lama. `re4.bin`, `steam_api64.dll`, `steam_api64.emp`, `amd_ags_x64.dll` sudah identik (hash sama) — jangan ditimpa. Folder `dlc/` juga jangan ditimpa (bisa bikin DLC hilang).

## Langkah reinstall ringkas

1. Install/pastikan folder game ada di path di atas, `re4.exe` tersedia.
2. Tambahkan `re4.exe` ke Steam sebagai **Non-Steam Game**.
3. Properties → Compatibility → pilih **Proton 8.0** (jangan Experimental / Proton 10).
4. Isi Launch Options seperti di atas.
5. Taruh `steam_appid.txt` berisi `2050650` di folder game.
6. Ekstrak `EMP.dll` + `steam_api64.ini` dari arsip crackfix (backup dulu file lama).
7. Klik Play. Tidak perlu restart Steam jika semua setting lewat GUI.

## Verifikasi berhasil

- Steam log (`~/steam-2050650.log`) tidak lagi mengeluarkan `SteamAPI_Init() failed` dari `re4.exe`.
- Folder ini terbuat: `~/.local/share/Steam/steamapps/compatdata/3642743466/pfx/drive_c/users/Public/Documents/EMPRESS/2050650/` (tanda EMPRESS initializing).
- Tidak muncul Crash Report Tool / `0x2000000`.
- Game masuk ke menu utama.

## Catatan tambahan

- Selama diagnosis, RDR2 (AppID `4275203992`) juga jalan dengan Proton 10 + `-vulkan` dan sedang compile shader. Jangan ganti Proton RDR2 saat RE4 dites.
- Kalau setelah ini muncul error baru:
  - "This game requires Windows 10 or later" → terlalu sering ganti Proton; tambahkan `re4.exe` ulang ke Steam atau paksa Windows 10 via Protontricks di prefix `3642743466`.
  - Masih crash `0x2000000` → coba GE-Proton 8.27 (bukan 7.x) + RE Framework versi EMPRESS-compatible, dengan `WINEDLLOVERRIDES="dinput8=n,b" %command%`.
  - No audio / DLC tidak muncul → efek samping crackfix yang dilaporkan sebagian user; jangan timpa folder `dlc/` secara membabi buta.
