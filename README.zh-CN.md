# RetroBoxDB MSX

[English](README.md) | 中文

Microsoft MSX的单文件 SQLite 保存库。公开的 Catalog 只含元数据（校验值、DAT 与来源记录、头部字段、打包配方和程序），不含 ROM 数据，不能独立恢复文件；完整库保留在本地。

| 项目 | 数值 |
| --- | --- |
| 原始大小 | 源 ZIP 1,070 个，24.4 MiB（No-Intro 959 个，RetroAchievements 集合 111 个）；解压后 ROM 1,070 个，50.3 MiB |
| 入库后大小 | 完整库 22.2 MiB；公开 Catalog 8.0 MiB（不含 ROM 数据） |
| 比例 | 完整库为原 ZIP 的 91.3%，为解压后 ROM 总量的 44.2% |
| 使用的技术 | 存储 v4：64 KiB 块按 SHA256 去重，按 No-Intro 游戏族顺序装入最大 64 MiB 的 LZMA2 实体组（字典 64 MiB）；逐块 SHA256、逐对象 CRC32／MD5／SHA1／SHA256 校验；源 ZIP 由 TorrentZip 配方逐字节重建 |
| 导出性能 | Intel(R) Core(TM) i7-8650U CPU @ 1.90GHz，空闲负载，Python 3.14.4，含全部校验。按最新 DAT 整套导出（`export_set.py`，952 个文件，逐个按 DAT 哈希校验）：27.0 MiB/s，平均 2 毫秒／个；单个文件冷缓存（每次清空缓存，需解压所在组的前段）：ROM 平均 0.311 秒，TorrentZip 平均 0.326 秒 |

## 下载与说明

| 文件／文档 | 内容 |
| --- | --- |
| [RetroBoxDB.MSX.Catalog.sqlite](https://github.com/rshi0212/RetroBoxDB-MSX/releases/latest/download/RetroBoxDB.MSX.Catalog.sqlite) | 公开 Catalog（Release 附件，附 `SHA256SUMS`） |
| [存储 v4 说明](RetroBoxDB.Storage-v4.zh-CN.md)／[English](RetroBoxDB.Storage-v4.en.md) | 各平台的存储评估、内容、RA、中文名与维护 |
| [Technical design](RetroBoxDB.Storage-v4.Technical-Design.en.md) | 存储格式、平台适配、增量更新、校验 |
| [RA 清单](reports/ra-msx1-games.csv)／[汇总](reports/ra-msx1.json)、[构建报告](reports/msx1-build-report.json)、[审计处理](reports/audit-resolution-20261004.md) | 逐项数据 |

## 本平台的存储选择与特殊情况

全部本地收藏实测 14 种块／组组合（`assessment/data/storage-experiment-msx1.json`）：最小为 512 KiB / 64 MiB 12.47 MiB；按规则（最小值 0.5% 以内选块最小、再选组最小）采用 64 KiB / 64 MiB 12.50 MiB。ZIP 24.38 MiB，逐文件 LZMA 20.15 MiB。

- 卡带、软盘和磁带：`msx_hardware.media` 记录介质；卡带在 0x0000 或 0x4000 有 `AB` 头（INIT、STATEMENT、DEVICE、TEXT 地址），软盘按大小和引导扇区识别，磁带按 CAS 块标记识别。机型不在数据中；mapper 需要外部证据。
- RetroAchievements 的 MSX 目录（主机 29）混有 MSX 和 MSX2 游戏，按“平台优先、介质其次”分库（`tools/msx_route.py`）：每个 ZIP 只进一个库，依据依次为成员 SHA1 在 MSX2／MSX 的 DAT 中、文件名标 `(MSX2` 或有 `.mx2` 成员、复核表 `data/msx-routing.csv`（参考名单、MSX 软件数据库和开发比赛页面等公开资料、人工判断）、只出现在一个平台 DAT 中的标题、与某一平台 DAT ROM 共享 8 KiB 块达 10%（Hack、翻译版）。该目录 200 个 ZIP 中 111 个在本库，89 个在 [RetroBoxDB-MSX2](https://github.com/rshi0212/RetroBoxDB-MSX2)，没有未确认的文件。每个文件的依据见 `rom_annotations`（`kind='platform'`）和 `v_msx_headers.platform_evidence`。
- RetroAchievements 主机 29 由两个 MSX 库共用；RA 报告只统计与本库有关的游戏。

## 内容

| 项目 | 数值 |
| --- | --- |
| ROM 记录／游戏组／发行版本 | 1,008／622／952 |
| 各版 DAT 覆盖 | 20260618-055428：952/952 |
| 不在任何 DAT 的本地 ROM | 56 |
| RetroAchievements 集合中的 ROM 文件 | DAT 中有 55，仅 RA 收录 56，哈希不在最新 RA 快照 0（[清单](reports/ra-msx1-collection-unknown.csv)）；仍缺本地 ROM 的 RA 游戏见 [缺口清单](reports/ra-msx1-missing.csv) |
| No-Intro DB Export＋Dump Log 20260618-055428 | 952 个档案、961 个文件身份、11 条有文档的硬件声明；Dump Log Verified 0 |
| RetroAchievements（console 29） | 有成就的游戏 92 个：本地有 ROM 92（115 个 ROM），ROM 在兄弟库中 0，仅 DAT 有 0，仅 DB 文件 0，无 No-Intro 对应 0 |
| 中文名 | 944 条记录中 907 条有中文（577 个唯一名）；本地 ROM 907 个有中文名 |
| 完整库审计 | 1,011 个对象、2 个组、1,023 个 ZIP 配方，全部通过 |

源 ZIP 均可由 TorrentZip 配方逐字节重建（`v_file_checksums.exported_bytes_equal_source`）。

## 使用

```bash
# 用 Catalog 内嵌引擎做只读审计（stats、checksums FILE_ID、help 同理）
python3 -B -c 'import sqlite3,sys; c=sqlite3.connect(sys.argv[1]); s=c.execute("SELECT content FROM resources WHERE name=?",("engine.py",)).fetchone()[0]; c.close(); exec(compile(s,"RetroBoxDB:engine.py","exec"))' ./RetroBoxDB.MSX.Catalog.sqlite audit
# 完整库：按 DAT 版本、1G1R、RA 成就、TorrentZip／裸 ROM 组合导出
python3 -B tools/export_set.py RetroBoxDB.MSX.sqlite OUT --set 1g1r --ra achievements --container torrentzip --layout ra-category
# 增量加入新 DAT、DB Export／Dump Log、ROM 与 RA 快照
python3 -B tools/update_db.py RetroBoxDB.MSX.sqlite --discover --ra --catalog RetroBoxDB.MSX.Catalog.sqlite
```

只需 Python 3.10+ 标准库。`resources` 中的 `engine.py` 等是可执行代码，只应从自己构建或 SHA256 已核对的 Release 附件中执行。发布由 `.github/workflows/publish-catalog.yml` 完成：工作流从 `release/catalog-release.json` 固定的基础 Catalog 出发，注入本仓库提交中的引擎与文档，核对全部数据表摘要、运行测试与审计后发布。
