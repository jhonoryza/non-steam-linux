# Planet of Lana (non-Steam)

## Identitas

- Shortcut Steam: **Planet of Lana** (appid shortcut `4025338952`)
- EXE: `/home/vm/hdd/programs/Planet of Lana/Planet of Lana.exe`
- Prefix Proton: `/home/vm/.local/share/Steam/steamapps/compatdata/4025338952`
- AppID asli game: `1608230` (`steam_appid.txt`)
- Edisi: repack ElAmigos, Unity 2020.3.41f1 (IL2CPP), Steamworks asli di
  `Planet of Lana_Data/Plugins/x86_64/`

## Gejala

Logo FMOD tampil, lalu aplikasi menutup diri sendiri. Lewat Steam: mati ~8 detik setelah Play.
Tidak ada dialog error.

## Penyebab

`SteamAPI_Init()` gagal karena Proton memakai `lsteamclient` dan koneksi ke Steam global user
tidak tersedia:

```
[S_API FAIL] SteamAPI_Init() failed; connect to global user failed.
```

Game ini tidak punya emulator Steam; ia memakai `steam_api64.dll` + `steamclient64.dll` asli.
`flt.ini` (configElAmigos) berisi `Offline=0`, jadi game tetap menuntut Steam.
Mengubahnya ke `Offline=1` justru membuat game mati lebih cepat (~14 detik), jadi dikembalikan
ke `Offline=0`.

Log `Player.log` hanya berisi `LoadFail` untuk settings/slot save (normal untuk install baru)
dan `Curl error 42` (telemetri, tidak fatal). Tidak ada crash exception yang menjelaskan exit.

## Perbaikan

LaunchOptions shortcut:

```
UMU_ID=1608230 UMU_USE_STEAM=0 %command%
```

Ini membuat GE-Proton menambahkan `WINEDLLOVERRIDES=lsteamclient=d`, sehingga `SteamAPI_Init()`
memuat `steamclient64.dll` milik game sendiri (tidak ada lagi `S_API FAIL`).

## Verifikasi

- Mode default: `SteamAPI_Init()` gagal, proses keluar ~8 detik saat diluncurkan Steam.
- Mode UMU: Steamworks plugin game ter-load, proses bertahan ~58 detik dalam uji langsung.
- `flt.ini`: sudah dikembalikan ke `Offline=0`.

## Catatan

- Steam saat meluncurkan shortcut ini memakai Proton 10.0; opsi UMU diuji dengan GE-Proton11-14.
- Steam menyimpan `shortcuts.vdf` dari memori setiap kali game diluncurkan. Restart Steam
  setelah mengubah LaunchOptions agar tidak tertimpa.
- status: kandidat perbaikan, belum terverifikasi sampai masuk menu utama.
