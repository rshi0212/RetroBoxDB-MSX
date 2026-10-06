MSX Catalog, storage v4 (64 KiB blocks, 2 solid LZMA2 groups of up to 64 MiB). Metadata only: **no ROM payloads are published**; `compression_groups`, `chunks` and `object_chunks` are empty.

- First release of this platform.
- The RetroAchievements MSX folder holds MSX and MSX2 games; each ZIP goes to one database, platform first, medium second (`tools/msx_route.py`, reviewed list `data/msx-routing.csv`); the basis is in `rom_annotations` / `v_msx_headers.platform_evidence`.
- One schema for all twenty-three platforms: the header tables of the eight new platforms (Game Gear, PC Engine, SuperGrafx, MSX, MSX2, Virtual Boy, Game & Watch, Super A'Can: `pce_hardware`, `msx_hardware`, `vb_hardware`) and `rom_annotations` exist in every Catalog; tables of other platforms and provider tables have no rows.
- RetroAchievements reports look up sibling databases (NES<->FDS, SNES<->Satellaview, WonderSwan<->WonderSwan Color, NeoGeo Pocket<->NeoGeo Pocket Color, PC Engine<->SuperGrafx, MSX<->MSX2): a game whose ROM is stored there is `local_other_platform`, not a gap.
- Source: 1,070 ZIPs (nointro 959, retroachievements 111), 24.4 MiB (1,070 ROM files, 50.3 MiB uncompressed). Populated database: 22.2 MiB (91.3% of the ZIPs). All source ZIPs are reproduced byte-for-byte.
- Contents: 1,008 ROM records, 622 games, 952 releases; DAT versions: 20260618-055428.
- RetroAchievements: 92 of 92 games with achievements have a local ROM.
- Export (Intel(R) Core(TM) i7-8650U CPU @ 1.90GHz, idle, all checks): whole newest-DAT set with export_set.py 27.0 MiB/s (952 files); single file with a cold cache 0.311 s (ROM) / 0.326 s (TorrentZip) on average.
- Full audit of the populated database: 1,011 objects, 2 groups, 1,023 archive plans, no errors.

The release workflow starts from the base Catalog pinned by SHA256 in `release/catalog-release.json`, injects the engine and documents of the tagged commit, checks every data-table digest, SQLite integrity and foreign keys, runs the Catalog audit and the repository tests. Verify the download with `SHA256SUMS`.

[中文说明](https://github.com/rshi0212/RetroBoxDB-MSX/blob/main/README.zh-CN.md)
