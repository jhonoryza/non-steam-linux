# NieR: Automata (non-Steam)

## Identitas

- Shortcut Steam: **NieR Automata** (appid shortcut `3621726195`)
- EXE: `/home/vm/hdd/programs/NierAutomata/Nier Automata/NieRAutomata.exe`
- Prefix Proton: `/home/vm/.local/share/Steam/steamapps/compatdata/3621726195`
- Edisi: repack Repack-Games, crack CPY (`CPY.ini`, `steam_api64.dll`, `steamclient64.dll`)
- AppID asli game: `524220`

## Gejala

Play → jendela kecil berjudul **Error** muncul, isinya:

```
Unable to create interface ISteamUser
```

Setelah OK, game keluar (~8 detik).

## Penyebab

Crack CPY tidak bisa mengambil interface Steam karena Proton menyuntik `lsteamclient` (proxy Steam
bawaan Proton) ke dalam prefix. EXE, prefix, dan DLL tidak ada yang hilang.

## Perbaikan

LaunchOptions shortcut di
`~/.local/share/Steam/userdata/104758288/config/shortcuts.vdf`:

```
UMU_ID=524220 UMU_USE_STEAM=0 %command%
```

`UMU_USE_STEAM=0` membuat GE-Proton menambahkan `WINEDLLOVERRIDES=lsteamclient=d`,
sehingga crack CPY memakai `steamclient64.dll`-nya sendiri.

**Restart Steam setelah mengubah shortcuts.vdf** agar LaunchOptions terbaca.

## Verifikasi

Launch dengan env `UMU_ID=524220 UMU_USE_STEAM=0`: tidak ada dialog Error,
jendela `NieR:Automata` muncul dan tetap hidup sampai dihentikan manual.

## Catatan

- Efek samping: overlay Steam di dalam game tidak aktif.
- Steam menyimpan `shortcuts.vdf` dari memori setiap kali ada game diluncurkan (LastPlayTime).
  Edit file langsung akan hilang kalau game lain dijalankan sebelum Steam di-restart.
  Setelah restart Steam, LaunchOptions terbaca dan ikut tersimpan. Kalau hilang lagi, set lewat
  Steam → klik kanan shortcut → Properties → Launch Options.
- status: sudah diverifikasi lewat dialog Error; belum diverifikasi visual sampai menu utama.
