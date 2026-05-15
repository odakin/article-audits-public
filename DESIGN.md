# DESIGN.md — article-audits-public

## Generated mirror として運用、 直接編集禁止

`~/Claude/article-audits/` (private) が source of truth、 ここはその派生 public mirror。

**Why**: (a) private 側の commit history に sensitive な内容 (= draft / private visibility entry / 原文 archive) が含まれ、 同一 repo で公開すると history 漏洩のリスクがある。 別 repo で history を独立させれば、 mirror の commit log は公開可能 entry のみを含む clean state にできる。 (b) sync 時に whitelist で field strip するため、 private 側の `internal_notes` 等が漏れない。 (c) mirror が generated 派生物だと明示することで、 直接編集 → 上書き事故を予防。
**代替案**: (a) 同 repo で branch 分割 → branch 切り替えコスト + 公開可否の管理が branch operation 依存になる、 (b) 同 repo に `public/` subdir → GitHub repo-level visibility と齟齬、 (c) Phase 2 deferred (= mirror を作らず meta field のみ) → 公開機構は最終的に必要なので migration を回避できる初期から立てる方が cleaner。
**How to apply**:
- mirror 側の `entries/` + `INDEX.md` は `sync-public.py` が完全自動生成 → 直接編集禁止
- mirror 側の `README.md` / `LICENSE` / `CLAUDE.md` / `SESSION.md` / `DESIGN.md` は手動 maintenance 対象
- mirror 側の commit / push は **odakin が手動** で実施 (= 私 / Claude は push しない、 sync 直後の diff を eye-balling してから)

## 多層 defense (= 4 段 leak ガード)

公開 mirror への private 情報流出を、 互いに独立した **4 層** で防ぐ:

| 層 | mechanism | 責務 |
|---|---|---|
| 1 | `PUBLIC_META_FIELDS` 白 list (= sync-public.py) | meta.yaml top-level field の whitelist。 `internal_notes` / `visibility` 等 internal field を strip |
| 2 | `COPY_FILES` 白 list (= sync-public.py) | mirror に copy する file 名の whitelist。 `article.md` (= 原文 verbatim、 copyright risk) を意図的に除外 |
| 3 | `.leakage-denylist.txt` content scan (= sync-public.py) | analysis.md / sources.yaml / 出力 meta.yaml の **内容** を denylist marker でスキャン、 hit なら sync を abort |
| 4 | `hooks/mirror-pre-push` (= git hook) | push 対象 commit 全 diff を denylist marker でスキャン、 hit なら push を abort (= 層 1-3 を bypass された場合の最終 catch) |

このうち **層 3-4 は 2026-05-15 の初期 audit で発生した leak 事故** (= analysis.md narrative に private repo 名を書いた → git history に残存) を契機に追加。 詳細は private 側 DESIGN.md。

**mirror 側で守ること**:
- mirror は generated 派生物、 直接編集禁止 (層 1-2 を bypass する経路)
- mirror の git hook は private 側 `~/Claude/article-audits/hooks/mirror-pre-push` への symlink、 内容を mirror で書き換えない
- mirror の denylist は private 側 `~/Claude/article-audits/.leakage-denylist.txt` を参照、 mirror に denylist を含めない (= denylist 自体が「これらは private」 という情報を公開してしまう)

**push 時のフロー**:
1. private 側で `python scripts/sync-public.py` 実行 (= 層 1-3 通過)
2. mirror 側で diff を eye-balling
3. mirror 側で `git add -A && git commit ...`
4. mirror 側で `git push` (= 層 4 が起動、 denylist スキャン → clean なら push)

## meta.yaml は field whitelist で strip (= 層 1)

mirror 側の `meta.yaml` は private 側の subset のみ。

**Why**: `visibility` / `status` / `internal_notes` は管理用 internal field で、 公開する意味がない。 また `internal_notes` には private な観測が入る可能性があるため誤公開リスクを構造的に消す。
**How to apply**:
- `sync-public.py` の `PUBLIC_META_FIELDS` whitelist に列挙された field のみ mirror に書く
- 新規 field を private meta.yaml に追加する時は、 公開可否を判断して whitelist 更新も検討

## ライセンス: CC BY-SA 4.0

公開 analysis を CC BY-SA 4.0 で release。

**Why**: (a) fact-check / audit 結果は公共財として再利用される価値がある、 (b) Share-Alike で「商業的に rebrand して閉じ込める」 を防ぐ、 (c) Attribution 要求で odakin の検証作業への credit を保持。
**代替案**: (a) public domain (CC0) → 商業的再 packaging を許す、 (b) CC BY (Share-Alike なし) → 同上、 (c) CC BY-NC → academia / journalism が non-commercial 判定で揉める。
**How to apply**:
- `LICENSE` ファイルに CC BY-SA 4.0 の正本 text
- `README.md` で license を明示
- 各 entry の analysis.md は header / footer で license refer する必要なし (= リポ全体に適用)

## INDEX.md は audit_date 降順

`INDEX.md` は `sync-public.py` が verdict + audit_date sort key で再生成。

**Why**: 最新の audit が top に来る方が訪問者の興味と一致する。
**How to apply**:
- sort: `audit_date` descending
- 各 entry 1 行: title (リンク) + author + publication + audit_date + verdict
- 将来 100 件超えたら year / tag filter を追加検討
