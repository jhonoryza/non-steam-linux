# Horizon Zero Dawn (non-Steam)

## Identitas

- Shortcut Steam: **Horizon Zero Dawn** (appid shortcut `3614216959`)
- EXE: `/home/vm/hdd/programs/Horizon Zero Dawn/HorizonZeroDawn.exe`
- Prefix Proton: `/home/vm/.local/share/Steam/steamapps/compatdata/3614216959`
- Edisi: GOG Complete Edition (`goggame-1209025141`)
- Proton saat kejadian: Proton - Experimental; verifikasi juga dengan GE-Proton11-14

## Gejala

Tombol Play langsung berhenti lagi (~4 detik). Tidak ada jendela game.

## Penyebab

Prefix dibuat baru sehingga tidak memiliki `mfc140.dll` (MFC runtime untuk VC++ 2015-2022).

Log Wine:
```
err:module:import_dll Library mfc140.dll (which is needed by
  L"Z:\\home\\vm\\hdd\\programs\\Horizon Zero Dawn\\HorizonZeroDawn.exe") not found
err:module:loader_init Importing dlls for ... failed, status c0000135
```

`c0000135` = `STATUS_DLL_NOT_FOUND`, jadi ini bukan masalah GPU/Proton version.

## Perbaikan

Menyalin DLL dari prefix lain yang sehat (The Last of Us Part I, appid `2822776626`):

```sh
cp /home/vm/.local/share/Steam/steamapps/compatdata/2822776626/pfx/drive_c/windows/system32/mfc140.dll \
   /home/vm/.local/share/Steam/steamapps/compatdata/3614216959/pfx/drive_c/windows/system32/mfc140.dll
```

Sumber alternatif bila prefix itu hilang: `_CommonRedist/vcredist/2022/VC_redist.x64.exe`
atau `.../2019/VC_redist.x64.exe` di `~/hdd/programs/steams/steamapps/common/Steamworks Shared/`,
diinstal lewat prefix GameObject Proton.

## Verifikasi

Launch ulang dengan `STEAM_COMPAT_DATA_PATH=.../3614216959`, `SteamGameId=3614216959`,
`PROTON_LOG=1`. Hasil: tidak ada lagi `import_dll`/`loader_init failed`, proses bertahan >30 detik.

## Catatan

- Perubahan ada di dalam prefix, jadi tidak perlu mengubah shortcut atau LaunchOptions.
- Rollback: hapus `mfc140.dll` dari `system32` prefix tersebut.
- status: sudah diverifikasi sampai lewat error DLL; belum diverifikasi visual sampai menu utama.
