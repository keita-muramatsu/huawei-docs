# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## このリポジトリの性質

ソースコードではなく、**Huawei の HedEx オフライン製品ドキュメントライブラリ（`.hdx` バンドル）を丸ごと git 管理しているリポジトリ**。ビルド・テスト・lint は存在しない。作業のほぼすべては「膨大な HTML から目的のトピックを探して読む」こと。

- 対象製品: NetEngine AR5700 / AR6700 / AR8000（README.md の取扱機器は `AR5710-H4T4N2X`）
- 製品バージョン: `V600R025C00` / ライブラリ `AEP1009U` v02 / 発行日 2026-03-05（`*.hdx/profile.xml`）
- 規模: トピック 20,986 件、HTML 約 21,000、画像 約 3,500、追跡ファイル 約 24,000

HTML は DITA から生成されたベンダー成果物。**手で編集しない。** コミットはライブラリのリリース単位で、メッセージは `V600R025C00:2026-03-05` 形式（`<製品バージョン>:<発行日>`）。編集対象になるのは基本的に `README.md` とこの `CLAUDE.md` だけ。

## ディレクトリ構成

すべて `NetEngine AR_V600R025C00_02_en_AEP1009U.hdx/` 配下。以降のパスは `resources/` からの相対で記す（HTML 内のリンクや `navi.xml` の `url` もこの基準）。

| パス | 内容 |
| --- | --- |
| `profile.xml` | ライブラリのメタ情報（バージョン、トピック数、検索ラベル定義、featureType 一覧） |
| `resources/navi.xml` | **全 20,986 トピックの目次ツリー**（`txt`=タイトル, `url`=パス, `id`=トピック ID）。最初に引くべきファイル |
| `resources/md5.xml` | トピック ID ↔ url の全件対応表 |
| `resources/*.html` | 設定ガイド本体（トップレベル直置き。約 5,000 件） |
| `resources/v6r25c00/cli/` | CLI コマンドリファレンス（7,686） |
| `resources/v6r25c00/mib/` | MIB リファレンス（3,697） |
| `resources/v6r25c00/log/` | ログメッセージリファレンス（2,153） |
| `resources/v6r25c00/alarm/` | アラーム（トラップ）リファレンス（1,222） |
| `resources/spec/`, `resources/spec1/` | 機能ごとのライセンス要件・ハードウェア要件・仕様/制限（`*_limitation_all.html`） |
| `resources/vrp/`, `resources/dc/`, `resources/cbb_ar/`, `resources/galaxy/` | 追加の設定/安全/開発トピック群 |
| `resources/toctopics/` | 目次ノード用の中間ページ |
| `resources/figure/`, `resources/images/`, `resources/*/figure/` | 図版 |
| `resources/public_sys-resources/` | 共通 CSS/JS（表示用。内容とは無関係） |
| `resources/lib_index/` | HedEx 全文検索用の Lucene インデックス（バイナリ、grep 不可） |

トップレベル HTML のファイル名プレフィックスがそのまま分類になっている: `vrp_<機能>_cfg_*`（BGP/ISIS/OSPF/SRv6/VRRP など）、`galaxy_*`（WLAN/AP/QoS/NAC/AAA/telemetry/ZTP）、`security_*` / `sec_hardening_*`、`ar_*`（製品説明・ハードウェア・インストール・ライセンス）、`feature-specific_*`。

## 目的のトピックを探す

### 1. まず `navi.xml` をタイトルで引く（最速）

23,258 行の単一ファイルなので、全文 grep より桁違いに速い。

```bash
cd "NetEngine AR_V600R025C00_02_en_AEP1009U.hdx/resources"
grep -o 'txt="[^"]*ospf abr[^"]*" url="[^"]*"' navi.xml
# => txt="display ospf abr-asbr" url="v6r25c00/CLI/DISPLAY-OSPF-ABR-ASBR(OSPFCOMMOM).html"
```

**重要: `navi.xml` / `md5.xml` の `url` は大文字を含むが、ディスク上のファイル名はすべて小文字。** Windows(NTFS) では両方開けるが、`git ls-files` の結果と突き合わせたり大小文字を区別するツールを使うときは小文字化すること（`tr 'A-Z' 'a-z'`）。

### 2. 全文検索

`resources/` 全体の `grep -r` は約 10 秒で完走する。実用範囲だが、可能なら対象を絞る。

```bash
grep -rl "AR5710-H4T4N2X" --include='*.html' resources/        # 約10秒
grep -rl "sub-interface" resources/v6r25c00/cli/               # ディレクトリを絞れば数秒
```

### 3. トピック ID から辿る

HTML の `<meta name="DC.Identifier">` にある `EN-US_CLIREF_...` / `EN-US_TOPIC_...` から、

```bash
grep -o 'id="EN-US_CLIREF_0000002148641644" url="[^"]*"' resources/md5.xml
```

## ファイル名の規約

| 種別 | 規約 | 例 |
| --- | --- | --- |
| CLI | `<コマンド名をハイフン/アンダースコア化>(<モジュール>).html`。同名コマンドはモジュール接尾辞で区別 | `display-ospf-brief(ospfcommom).html`, `aaa(esap_aaa).html` |
| ログ | `<ログ名>_<重大度>.html`（モジュール索引は `dc_ar_log_<モジュール>.html`） | `rt_ovr_lmt_4.html` → ログ ID `BGP/4/RT_OVR_LMT` |
| アラーム | `dc_ar_alarm_<モジュール>_<OID>_<トラップ名>.html` | `dc_ar_alarm_isis_1.3.6.1.4.1.2011.5.25.24.2.4.49_isissystemidcfgconflict.html` |
| MIB | `dc_ar_<mib名>_{singlenode,alarmnode}_<OID>.html` / `..._{mibtablecatalog,alarmnodecatalog}.html` | `dc_ar_huawei-ntp-mib_singlenode_1.3.6.1.4.1.2011.6.4.2.14.1.html` |
| 仕様/制限 | `spec/<機能>_limitation_all.html` | `spec/vrrp_limitation_all.html` |

OID が分かっていればアラーム/MIB はファイル名 glob で直接引ける: `ls resources/v6r25c00/alarm/ | grep '1.3.6.1.4.1.2011.5.25.24'`

## HTML の読み方

生成物なのでタグが冗長。中身だけ取り出すときは `<title>` と `DC.*` メタを先に見る。

- タイトル: `<title>` / `<h1 class="topicTitle-h1">`
- 分類メタ: `DC.Identifier`（トピック ID）、`featurename`、`featuretype`、`DC.Audience.Job`（Reference / Configuration / Installation）
- CLI ページのセクション: `class="clifunc"`（Function）、`cliformat`（Format）、`cliparam`（Parameters）、`cliview`（Views）、`clidefaultlevel`（Default Level）、`clidesc`（Usage Guidelines）、`cliexample`（Example）。各セクションの本文は `<class>body`
- 関連トピックは `<meta name="DC.Relation">` に列挙される

## 巨大ツリーの扱い（重要）

`resources/` は数万エントリあり、**再帰的な `ls -R` / `du -s` / `find` は数分かかってタイムアウトする**。ファイル一覧が欲しいときは git のインデックスを使う。

```bash
git ls-files 'NetEngine*/resources/v6r25c00/cli/*' | wc -l
git ls-files | awk -F/ '{print $1"/"$2"/"$3}' | sort | uniq -c | sort -rn
```

単一ディレクトリの `ls` は速いので、`ls resources/spec/` のように深さを固定して使う。

## 目次の最上位

`navi.xml` の第 1 階層 = ドキュメントセットの構成。

`Documentation Guide` / `More Documents, Videos, and Tools` / `What's New` / `Security Declaration` / `Version Mapping` / `Regulatory Compliance Statement` / `Get to Know the Product` / `Installation` / **`Configuration`** / `Fault Management` / **`References`** / `Development`

- `Configuration` → CLI 設定ガイド、Web 設定ガイド、Webmaster 設定ガイド、セキュリティハードニング、CLI/Web 設定例、相互接続ガイド
- `References` → Command Reference / Log Reference / MIB Reference（= `v6r25c00/` 配下）
- `Fault Management` → 保守・トラブルシューティング / アラーム処理
