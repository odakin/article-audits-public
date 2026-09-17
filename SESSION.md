# SESSION.md — article-audits-public

## 2026-09-18 — root の入口 (AGENTS.md) 追加 + 「Claude が push しない」 規則の対象を絞った

- Codex は `AGENTS.md` しか自動発見しないので、 規約どおり root に薄い入口を置いた (中身は `CLAUDE.md` / `SESSION.md` を読めと言うだけ、 契約 = claude-config `CONVENTIONS.md#agent-instruction-entrypoints`)。
- `CLAUDE.md` の「commit は sync 後 … 私の手で push しない」 の**対象を sync が書く file (INDEX.md + entries/) に限定**した = 公開される中身を人が見るための規則で、 root の維持 file (規約・入口・ライセンス) は対象外。 `scripts/sync-public.py` は entries/ と INDEX.md しか書かず削除もしないので、 root の file は同期で消えない。

## 2026-05-15 第 1 セッション = mirror 初期化 + 第 1 entry 同期 + history wipe

Source of truth `~/Claude/article-audits/` の初期化と同時に mirror を作成。 `scripts/sync-public.py` で第 1 entry (= Schneier Mythos Guardian audit) を同期。

- 第 1 entry: `entries/2026/2026-05-08-schneier-mythos-guardian/`
- verdict: `mixed` (= 20 source で triangulation した結果、 Schneier 記事は §1 / §2 が支持、 §3-long が弱、 §4-6 が thesis robust)

### 初期 history (旧 commit) の wipe — 2026-05-15

第 1 commit (旧 hash 785b007) の analysis.md diff に private 文脈が混入していたことを deep safety sweep で検出。 push 前 (= remote 未設定) なので **mirror の `.git` を完全 wipe + reinit** で history を clean 化。

加えて upstream 側に **4 層 defense (= field whitelist + file whitelist + content denylist scan + git pre-push hook)** を実装。 mirror 側の `.git/hooks/pre-push` は upstream の `hooks/mirror-pre-push` への symlink。 push 時に commit + 全 diff + commit message を denylist scan、 hit すれば push を abort。

現在の mirror は 1 clean commit `0989257` のみ、 全 37 denylist markers で 0 hits。

直接編集は禁止 — 次セッションで mirror 側に触る必要が出たら、 まず upstream 側で対応するか CLAUDE.md / README.md / LICENSE 等の手動 maintenance 対象かを切り分ける。 hook 更新は upstream `~/Claude/article-audits/hooks/` を編集 (symlink 経由で mirror に即時反映)。
