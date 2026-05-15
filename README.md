# article-audits-public

odakin による記事 audit の公開 mirror。 一次 source まで遡って検証した記事評価を順次公開する。

## このリポについて

- **Source of truth は別の private リポ**。 ここはその派生 mirror で、 `visibility: public` がついた entry のみが自動同期される
- **直接編集禁止**: このリポへの commit は upstream の sync script から自動生成される。 直接の commit は次回 sync で上書きされる
- **原文 archive は含めない**: 著作権 respect のため、 audit 対象の記事原文 (= verbatim copy) は private 側に留まる。 公開側は分析・source 一覧・evaluation summary のみ

## 構造

```
entries/<year>/<date>-<slug>/
├── analysis.md       # 検証ログ + 最終評価
├── sources.yaml      # 一次 source 一覧 (= type / authority / supports_claims 付き)
└── meta.yaml         # 公開可能 frontmatter (= title / verdict / key_claims 等)
```

各 entry の最新 index は [`INDEX.md`](INDEX.md) を参照。

## ライセンス

公開 entry の **analysis.md** および **INDEX.md** は odakin の著作物として **[CC BY-SA 4.0](LICENSE)** で公開。 引用・改変・共有自由、 同等ライセンス下で再公開可能。

audit 対象の原文 (= 引用部分) は元著者・媒体の著作権に従う。 引用は fair use 範囲、 short quote に止めている。 `sources.yaml` の URL は単なる source 参照 list で公開に問題なし。

## verdict 語彙

各 audit には記事全体の最終評価 = `verdict` がつく:

- `supported`: 主要 claim 大半が verified、 thesis 堅牢
- `mixed`: verified と weak / falsified が混在、 thesis 部分的に有効
- `weak`: 主要 claim に empirical 裏付け乏しい、 wish thinking 含む
- `debunked`: 主要 claim が falsified、 thesis 不成立
- `inconclusive`: source 不足で全体判定不能

個別 claim 単位では `verified` / `disputed` / `partial` / `unverified` / `falsified` / `inconclusive` / `outdated`。

## 再 audit と link rot

audit は時点での評価。 主要 claim の前提が変化したり、 元 source が link rot した場合は同 entry 内の `analysis.md` に **再 audit section** を追記する運用 (= 同 entry 内で時系列に積み上げる、 別 entry を切らない)。 `audit_date` は最終 audit 時点を表す。
