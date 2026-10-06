# RetroBoxDB MSX

English | [中文说明](README.zh-CN.md)

Single-file SQLite preservation database for Microsoft MSX. The public Catalog holds metadata only (checksums, DAT and provenance records, header fields, archive recipes and the processing code); it contains no ROM data and cannot restore files. The populated database stays local.

| Item | Value |
| --- | --- |
| Original size | 1,070 source ZIPs, 24.4 MiB (No-Intro 959, RetroAchievements sets 111); 1,070 ROM files, 50.3 MiB uncompressed |
| Stored size | populated database 22.2 MiB; public Catalog 8.0 MiB (no ROM data) |
| Ratio | 91.3% of the source ZIPs, 44.2% of the uncompressed ROM files |
| Technology | storage v4: SHA256-deduplicated 64 KiB blocks packed in No-Intro family order into solid LZMA2 groups of up to 64 MiB (64 MiB dictionary); per-block SHA256 and per-object CRC32/MD5/SHA1/SHA256 verification; source ZIPs reproduced byte-for-byte from TorrentZip plans |
| Export performance | Intel(R) Core(TM) i7-8650U CPU @ 1.90GHz, idle, Python 3.14.4, all checks included. whole newest-DAT set with `export_set.py` (952 files, each checked against the DAT hashes): 27.0 MiB/s, 2 ms per file on average; single file with a cold cache (the group is decoded up to the file): ROM 0.311 s, TorrentZip 0.326 s on average |

## Downloads and documents

| File / document | Content |
| --- | --- |
| [RetroBoxDB.MSX.Catalog.sqlite](https://github.com/rshi0212/RetroBoxDB-MSX/releases/latest/download/RetroBoxDB.MSX.Catalog.sqlite) | Public Catalog (Release asset with `SHA256SUMS`) |
| [Storage v4 guide](RetroBoxDB.Storage-v4.en.md) / [中文](RetroBoxDB.Storage-v4.zh-CN.md) | Storage evaluation, contents, RA, names and maintenance for every platform |
| [Technical design](RetroBoxDB.Storage-v4.Technical-Design.en.md) | Storage format, platform adapters, incremental updates, verification |
| [RA list](reports/ra-msx1-games.csv) / [summary](reports/ra-msx1.json), [build report](reports/msx1-build-report.json), [audit resolution](reports/audit-resolution-20261004.md) | Detailed data |

## Storage choice and platform specifics

14 block/group combinations measured on the whole local collection (`assessment/data/storage-experiment-msx1.json`): smallest 512 KiB / 64 MiB at 12.47 MiB; by the rule (within 0.5% of the smallest, the smallest block, then the smallest group) 64 KiB / 64 MiB at 12.50 MiB. ZIPs 24.38 MiB, per-file LZMA 20.15 MiB.

- Cartridges, floppy disks and tapes: `msx_hardware.media` records the medium; cartridges carry the `AB` header at 0x0000 or 0x4000 (INIT, STATEMENT, DEVICE, TEXT), disks are recognised by size and boot sector, tapes by the CAS block magic. The machine generation is not in the data; mappers need external evidence.
- The RetroAchievements MSX folder (console 29) mixes MSX and MSX2 games. It is split platform first, medium second (`tools/msx_route.py`): each ZIP goes to exactly one database by a member SHA1 in the MSX2 / MSX DATs, an `(MSX2` tag or `.mx2` member, the reviewed list `data/msx-routing.csv` (reference lists, public references such as MSX software databases and contest pages, manual knowledge), a title only one platform's DATs name, or at least 10% of 8 KiB blocks shared with one platform's DAT ROMs (hacks, translations). Of its 200 ZIPs, 111 are in this database and 89 in [RetroBoxDB-MSX2](https://github.com/rshi0212/RetroBoxDB-MSX2); none is unconfirmed. The basis of each file is in `rom_annotations` (`kind='platform'`) and `v_msx_headers.platform_evidence`.
- RetroAchievements console 29 is shared with the other MSX database; the RA report covers only games tied to this database.

## Contents

| Item | Value |
| --- | --- |
| ROM records / games / releases | 1,008 / 622 / 952 |
| DAT coverage per version | 20260618-055428: 952/952 |
| Local ROMs in no DAT | 56 |
| ROM files of the RetroAchievements set | in a No-Intro DAT 55, RA only 56, hash not in the latest RA snapshot 0 ([list](reports/ra-msx1-collection-unknown.csv)); RA games still without a local ROM: [gap list](reports/ra-msx1-missing.csv) |
| No-Intro DB Export + Dump Log 20260618-055428 | 952 archives, 961 file identities, 11 documented hardware assertions; Dump Log Verified 0 |
| RetroAchievements (console 29) | 92 games with achievements: 92 with a local ROM (115 ROMs), 0 with the ROM in a sibling database, 0 DAT only, 0 DB file only, 0 without a No-Intro counterpart |
| Chinese names | 907 of 944 rows translated (577 unique); 907 local ROMs have a Chinese name |
| Populated-database audit | 1,011 objects, 2 groups, 1,023 archive plans, all passed |

Every source ZIP is reproduced byte-for-byte from its TorrentZip plan (`v_file_checksums.exported_bytes_equal_source`).

## Usage

```bash
# Query-only audit with the Catalog's embedded engine (also: stats, checksums FILE_ID, help)
python3 -B -c 'import sqlite3,sys; c=sqlite3.connect(sys.argv[1]); s=c.execute("SELECT content FROM resources WHERE name=?",("engine.py",)).fetchone()[0]; c.close(); exec(compile(s,"RetroBoxDB:engine.py","exec"))' ./RetroBoxDB.MSX.Catalog.sqlite audit
# Populated database: export by DAT version, 1G1R, RA achievements, TorrentZip or plain ROMs
python3 -B tools/export_set.py RetroBoxDB.MSX.sqlite OUT --set 1g1r --ra achievements --container torrentzip --layout ra-category
# Add new DATs, DB Export / Dump Log snapshots, ROMs and an RA snapshot incrementally
python3 -B tools/update_db.py RetroBoxDB.MSX.sqlite --discover --ra --catalog RetroBoxDB.MSX.Catalog.sqlite
```

Python 3.10+ standard library only. `engine.py` and the other `resources` entries are executable code; run them only from a database you built or a Release asset whose SHA256 you verified. Releases are produced by `.github/workflows/publish-catalog.yml`: it starts from the base Catalog pinned in `release/catalog-release.json`, injects the engine and documents of this commit, checks every data-table digest, runs the tests and the audit, then publishes.
