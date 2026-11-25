# The Last of Us Part I — Linux/Proton: catatan perbaikan

Tanggal: 2026-11-26. Game: ElAmigos repack, dipasang sebagai shortcut non-Steam di Steam.

## Gejala

- Steam crash reporter: "Something went wrong with this game" / dialog Naughty Dog.
- Atau koin berputar beberapa detik lalu crash.
- Atau dialog: "Steam SDK Failed to initialize. The game will now exit."

## Keadaan akhir yang BERHASIL

Game diluncurkan **langsung lewat Proton**, bukan lewat Steam Library:

```bash
"/mnt/transcend/Games/The Last of Us Part I/play-tlou.sh"
```

Jangan klik Play di Steam Library selama `steam_appid.txt` ada — Steam akan mengira ini game asli AppID 1888930 dan menolak dengan **"No licenses"**.

## Isi `play-tlou.sh`

```sh
#!/bin/sh
cd "/mnt/transcend/Games/The Last of Us Part I"
export STEAM_COMPAT_DATA_PATH=/home/vm/.local/share/Steam/steamapps/compatdata/2822776626
export STEAM_COMPAT_CLIENT_INSTALL_PATH=/home/vm/.local/share/Steam
export STEAM_COMPAT_INSTALL_PATH="/mnt/transcend/Games/The Last of Us Part I"
export WINEDLLOVERRIDES="amd_ags_x64.dll=b"
export PROTONFIXES_DISABLE_UPSCALERS=1
export UMU_ID=1888930
export UMU_USE_STEAM=0
export PROTON_LOG=1
export DXVK_FILTER_DEVICE_NAME="AMD Radeon RX 6600"
exec /home/vm/.local/share/Steam/compatibilitytools.d/GE-Proton11-14-x86_64/proton waitforexitandrun "/mnt/transcend/Games/The Last of Us Part I/tlou-i.exe"
```

Arti bagian penting:

- `UMU_ID` + `UMU_USE_STEAM=0`: membuat Proton menjalankan `umu.exe` dan menonaktifkan `lsteamclient` — ini yang menghilangkan error **Steam SDK Failed to initialize** (injeksi Steam API ke cracked game).
- `PROTONFIXES_DISABLE_UPSCALERS=1`: mencegah GE-Proton mengunduh manifest upscaler dari internet (timeout lama).
- `WINEDLLOVERRIDES="amd_ags_x64.dll=b"`: override DLL AMD ke built-in.

## Akar masalah yang ditemukan (urutan kronologis)

1. **GE-Proton tidak terdetect** di dropdown Compatibility. Fix: pasang di
   `~/.local/share/Steam/compatibilitytools.d/GE-Proton11-14-x86_64/`, restart Steam penuh.
   Log pembuktian: `~/.local/share/Steam/logs/compat_log.txt` mencatat AppID `2822776626` di-mapping ke `GE-Proton11-14-x86_64`.
2. **Steam SDK Failed to initialize** — Steam meng-inject API ke game non-Steam. Overlay sudah dimatikan tapi tidak cukup. Fix: launcher langsung dengan `UMU_ID`/`UMU_USE_STEAM=0` di atas.
3. **`steam_appid.txt` = 1888930** diperlukan launcher langsung (game punya `steam_api64.dll` asli yang mencari appID). Yang menyebabkan "No licenses" adalah menjalankan lewat Steam Library, bukan file-nya.
4. **.NET SDK 3.1.426** dipasang ke prefix (sebelum perbaikan terakhir):
   ```
   ~/.local/share/Steam/steamapps/compatdata/2822776626/pfx/drive_c/Program Files/dotnet/sdk/3.1.426
   ```
5. **Penyebab utama crash koin: `mfc140u.dll` tidak ada di prefix**, padahal dibutuhkan `CChromaEditorLibrary64.dll`. Fix:
   ```bash
   STEAM_COMPAT_DATA_PATH=/home/vm/.local/share/Steam/steamapps/compatdata/2822776626 \
   STEAM_COMPAT_CLIENT_INSTALL_PATH=/home/vm/.local/share/Steam \
   PROTONFIXES_DISABLE_UPSCALERS=1 \
   ~/.local/share/Steam/compatibilitytools.d/GE-Proton11-14-x86_64/proton runinprefix \
   "/mnt/transcend/Games/The Last of Us Part I/_CommonRedist/vcredist/2019/VC_redist.x64.exe" \
   /install /quiet /norestart
   ```
   Setelah ini error `ERROR: Unable to load g_textDbNet` di `log.txt` hilang dan game masuk.
6. `psolibs/main/cache` dipindahkan ke `cache.bak-20261126` (file `.tbl`/`.blb` 0 byte). Game membuat ulang folder cache. **Bukan akar masalah**, tapi tidak merugikan.
7. SAM/Resizable BAR **tidak aktif** (BAR Region 0 = 256M), jadi bukan penyebab.
8. GPU terdeteksi benar: AMD RX 6600 (RADV), feature level 12_2, dipakai game. Intel UHD 630 tidak dipakai.

## File logging / diagnosa

- Log game:
  `~/.local/share/Steam/steamapps/compatdata/2822776626/pfx/drive_c/users/steamuser/Saved Games/The Last of Us Part I/log.txt`
- Status crash terakhir: `user-data.txt` di folder yang sama (field `active-jobs`, `time-since-game-start-seconds`).
- Log Steam compatibility: `~/.local/share/Steam/logs/compat_log.txt`.
- Log Proton: stdout dari launcher (PROTON_LOG=1), atau `/tmp/opencode/tlou-run*.log` dari sesi diagnosa.

## Checklist kalau di-install ulang / prefix baru

1. Pastikan GE-Proton ada di `compatibilitytools.d` dan Steam di-restart penuh.
2. Buat ulang `play-tlou.sh` persis seperti di atas.
3. Buat `steam_appid.txt` berisi `1888930` di root game.
4. Jalankan installer .NET SDK 3.1.426 (`Downloads/Programs/dotnet-sdk-3.1.426-win-x64.exe`) dengan `proton runinprefix ... /install /quiet /norestart`.
5. Jalankan VC_redist.x64.exe dari `_CommonRedist/vcredist/2019` dengan `proton runinprefix ... /install /quiet /norestart`. **Ini yang paling penting — jangan dilewat.**
6. Verifikasi:
   ```bash
   ls .../pfx/drive_c/windows/system32/mfc140u.dll
   ls .../pfx/drive_c/Program\ Files/dotnet/sdk/3.1.426
   ```
7. Launch dari terminal, jangan dari Steam Library. Biarkan shader compile selesai; jangan matikan paksa (memunculkan dialog "did not quit properly / safe mode").
