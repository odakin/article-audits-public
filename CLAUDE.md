# CLAUDE.md — article-audits-public

⚠️ **このリポは generated mirror。 直接編集禁止。**

Source of truth: `~/Claude/article-audits/` (private)

公開可否は private 側の `entries/**/meta.yaml: visibility: public` で driven。 mirror の同期は private 側の `scripts/sync-public.py` が実行する。 mirror への直接 commit は次回 sync で上書きされる。

## このリポを触る session で守ること

- **記事 audit の作業は private 側で行う**: 検証 / 編集 / claim 評価は `~/Claude/article-audits/entries/...` で
- **mirror 側のファイル変更は禁止**: README.md / LICENSE / CLAUDE.md / SESSION.md / DESIGN.md は手動 maintenance 対象、 それ以外 (= INDEX.md + entries/) は sync script が生成
- **commit は sync 後**: `python ~/Claude/article-audits/scripts/sync-public.py` 実行後、 diff を眺めて適切なら mirror 側で commit / push (= 私の手で push しない、 odakin が手動)

## ライセンス

公開 analysis は CC BY-SA 4.0 (= [LICENSE](LICENSE))。 引用範囲は fair use の short quote のみ。

## layer 配置

mirror = 公開観客向け = layer 1 相当 (= 観客広い)。 ただし claude-config と異なり「Claude 全 user 向け規約」 ではなく「odakin の audit 結果公開」 が目的。
