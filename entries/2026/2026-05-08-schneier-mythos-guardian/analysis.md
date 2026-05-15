# analysis: Bruce Schneier "How dangerous is Anthropic's Mythos AI?" (The Guardian, 2026-05-08)

## TL;DR

**Verdict: `mixed`** — Schneier の §1 (Mythos uniqueness 割引け) と §2 (AI 脆弱性発見能力は本物) は **AISI 公式評価 + AISLE reproduction + Stenberg curl 評価** の 3 独立 source で empirically 支持。 §3 short-term リスク論も具体例で支持。 一方 §3 long-term **「endgame は AI-enhanced defender 優位」** は jagged frontier と AI tool の novel category 弱さから **明確に弱い**。 §4-6 (規則系 hacking 波及論) は記事の本論で、 Schneier の 5 年越し thesis (= *The Coming AI Hackers* 2021 + *A Hacker's Mind* 2023) の robustness を引き継ぐ。 記事の最大の盲点は **AI 生成コード自体の新規攻撃面** (= prompt injection / supply chain / 訓練データ汚染) と **Project Glasswing の antitrust / access 排除問題** の未論。

## 記事の章別評価

| 章 | 主張 | 評価 | 根拠 |
|---|---|---|---|
| §1 | Anthropic uniqueness を割引け、 GPT-5.5 / smaller model で同等 | **supported** | UK AISI 公式数値 (GPT-5.5 = 71.4% vs Mythos = 68.6%、 margin of error 内 tie); AISLE が 8 model で flagship FreeBSD exploit を 8/8 検出; Stenberg「marketing が hype 本体」 |
| §2 | 現代 AI は脆弱性発見が本当に上手い | **supported** | Mozilla Firefox 271 件 (Mythos attribute、 Opus 4.6 比で 12 倍); OpenBSD 27 年もの logic flaw; FreeBSD NFS RCE; curl も多数発見 |
| §3 short | 短期はより危険、 patch 不能 system 多数 | **supported** | curl 8.21.0 patch まで 1.5 ヶ月; 「finding/exploiting は finding/fixing より易しい」 という古典的非対称 |
| §3 long | endgame は AI-enhanced defender 優位 | **weak** | Stenberg「AI tool は確立済 error category は得意だが novel vulnerability type は不得手」 (= defender 側は type catalog 依存); AISLE jagged frontier (= 安価 model fleet で broad coverage = attacker 側に有利な scaling); Schneier の wish thinking |
| §4-5 | 規則系 (税法 / 環境 / 食品安全) への波及論 | **supported** (本論) | Schneier の 5 年越し thesis、 *Hacking the Tax Code* (2023) で具体展開済; 政治プロセスによる patch 困難性は carried interest loophole 等の歴史的実例で robust |
| §6 | AI 革命 = 認知の体外 scale 化、 産業革命と同型 | analogy として有用 | 評価対象でなく frame、 robustness 判定不要 |

## 主要 claim と検証 status

### 1. Mythos の uniqueness 主張 → **disputed**

Anthropic は「too dangerous to release」 と framing したが、 3 つの独立 source が uniqueness 否定:

- **AISI** ([source](https://www.aisi.gov.uk/blog/our-evaluation-of-claude-mythos-previews-cyber-capabilities)): Expert task で GPT-5.5 = 71.4% (±8.0%) vs Mythos = 68.6% (±8.7%)、 statistical tie
- **AISLE** ([source](https://aisle.com/blog/ai-cybersecurity-after-mythos-the-jagged-frontier)): 8 model で flagship FreeBSD exploit を 8/8 検出 (= 含 GPT-OSS-20b、 active 3.6B param、 $0.11/M tokens)
- **Stenberg** ([source](https://daniel.haxx.se/blog/2026/05/11/mythos-finds-a-curl-vulnerability/)): 「the big hype around this model so far was primarily marketing. I see no evidence that this setup finds issues to any particular higher or more advanced degree than the other tools have done before Mythos」

→ Schneier の §1 主張は empirically 支持。

### 2. Mozilla 271 件 vs CVE 3 件 gap → **partial**

Schneier は「Mozilla used Mythos to find **271 vulnerabilities**」 と書くが:

- **271 件は事実** (Mozilla 公式 [hacks.mozilla.org](https://hacks.mozilla.org/2026/05/behind-the-scenes-hardening-firefox/)): Firefox 150 で Mythos attribute、 sec-high 180 / sec-moderate 80 / sec-low 11、 直前 Opus 4.6 で 22 件 → 12 倍
- **ただし正式 advisory で Claude credited な CVE は 3 件のみ** (CVE-2026-6746/6757/6758、 [SecurityWeek](https://www.securityweek.com/claude-mythos-finds-271-firefox-vulnerabilities/) 報): 271 = 「discrete code defects」、 大半は 3 CVE bundle の variant
- Mozilla 自身は「vulnerabilities」 ではなく **「security bugs」** と表現

→ Schneier は word choice (「vulnerabilities」) で metric を inflate。 partial verdict。

### 3. OpenBSD 27 年もの脆弱性 → **verified** (技術詳細)

- **TCP SACK 実装**: 2 flaw の連鎖 (lower bound check 欠如 + 32-bit signed seq num の sign bit overflow)
- 攻撃: 2 packet で任意 OpenBSD ホスト crash
- コスト: campaign $20K、 surface した specific run **$50 未満**
- 27 年潜伏 (= OpenBSD 2.0 = 1996 以来)
- AISLE 検証: GPT-OSS-120b (5.1B active) が single call で recover、 Kimi K2 も成功、 **Qwen3 32B は「robust」 と confidently 誤断言**

→ 技術的に novel discovery、 ただし Schneier 記事は OpenBSD vuln を引用していない (= 「Mozilla 271 件」 のみ)。 別 source で verify した独立 fact。

### 4. FreeBSD NFS RCE (CVE-2026-4747) → **verified、 ただし contamination 疑念**

- 17 年もの、 `svc_rpc_gss_validate` の stack buffer overflow
- Mythos の creative engineering = 15 RPC request 分割で 32 bytes × 15 = full ROP chain を kernel BSS に build
- AISLE で 8/8 model 検出 (含 GPT-OSS-20b)
- **rival.security 指摘**: 「Mythos 'Discovered' a CVE Already in Its Training Data」 ([source](https://rival.security/posts/mythos-discovered-a-cve-already-in-its-training-data---and-thats-still-worrying)) → 17 年もの bug は public patch / advisory が long tail で training data に入っている可能性、 「発見」 vs 「memorization の retrieval」 の切り分けは未解決

### 5. 「endgame は defender 優位」 → **weak (wish thinking)**

- セキュリティ経済学の伝統的非対称: attacker 1 件見つければ勝ち、 defender は全て塞ぐ必要
- AI が両者を等しく加速するなら **比率は不変**
- Stenberg 観察: 「**AI tool は確立済 error category は得意だが novel vulnerability type は不得手**」 → defender 側は type catalog に依存、 novel category は attacker が先行する構造
- AISLE jagged frontier: 「safe で cheap な model fleet を broad に展開する」 戦略は attacker 側に optimal、 defender はそれを全部塞ぐ必要

### 6. 「投資銀行は既に密かに税法 AI hacking」 → **partial / 興味深い ironic 反転**

- Schneier は「**密かに**」 と書いたが、 検索で **JPMorganChase が Project Glasswing に正規参加** ([ProMarket](https://www.promarket.org/2026/04/22/the-antitrust-risks-of-anthropics-project-glasswing-and-the-ai-avengers/)) 判明
- 公式 access purpose は security 用途、 ただし内部で税法 analysis 流用すれば Glasswing 条件違反になる構造
- 主要 Big4 (PwC / Deloitte 等) は機械学習を tax planning に過去 5-10 年使用済 (公開資料あり)、 LLM 固有の多 jurisdiction 合成 loophole 発見能力は **public evidence 段階で未確認**
- Schneier の「**密かに**」 rhetoric は事実を弱める方向に外している

### 7. Double Dutch Irish Sandwich の例 → **outdated**

- Ireland が 2020 年に制度改正で phase-out 済、 grandfathering も 2020 終了
- 「過去に存在した複雑な合成 loophole」 例としてはまだ機能
- 「現在 active な脅威」 文脈で読まれる懸念は残る

## Schneier 記事の盲点 (Schneier が論じていない重要 dimension)

### A. **AI 生成コード自体の新規攻撃面**

Schneier は「AI = 脆弱性発見器 + 修正器」 の二面のみ論じ、 **AI = 新規脆弱性の generator** という第三面を完全に無視。 含意:

- LLM 生成コードへの prompt injection vector
- RAG / agentic tooling 経由の supply chain compromise
- 訓練データ汚染による回帰的脆弱パターン
- AI agent への過剰権限委譲による logic flaw

「endgame は defender 優位」 主張は、 この第三面が attacker 側に新しい attack surface を生むことで根本的に揺らぐ。

### B. **Training data contamination**

17 年もの CVE は public patch / advisory / debug log が long tail で存在。 「AI が発見した」 vs 「AI が memorize した」 の切り分けが記事中に無い。 これは Mythos 評価にとって critical な切り分け (= novel reasoning capability vs retrieval capability)。

### C. **Project Glasswing の antitrust / access 排除問題**

- 40+ 企業 (Apple/Google/Microsoft/Amazon/Cisco/CrowdStrike/NVIDIA/**JPMorganChase**) の concerted action
- ProMarket (Madhavi Singh): illegal cartel / restraint of trade の懸念
- Anthropic 反論: $100M usage credits + $4M OSS security 寄付 + 40+ org への access 拡大
- curl 作者 Stenberg ですら数週間の access 遅延 → 「access broadening」 主張に現場乖離
- 「AI vendor / 大企業 / OSS maintainer の 3 階層 power gap」 が現場の本質、 attacker-defender 二項対立では捉えられない

### D. **fuzzer / static analyzer との比較 baseline 不在**

- VentureBeat 等は「static analysis tools, fuzzers, and auditors had all missed this logic flaw」 と書くが、 「missed for 27 years」 = 結果論的 absence
- modern fuzzer (KLEE / angr / OSS-Fuzz) が今再走したら何分で見つけるかは別問題
- 「AI でしか見つけられない」 主張は conventional tool との empirical comparison が不在
- AISLE 自身が OpenSSL に 15 CVEs / curl に 5 CVEs を production system で出しているが、 fuzzer baseline との比較は無い

## Schneier の知的系譜

- **2021**: 学術論文 [*The Coming AI Hackers*](https://www.schneier.com/academic/archives/2021/04/the-coming-ai-hackers.html) (Belfer Center) で thesis 確立
- **2023**: 書籍 *A Hacker's Mind* (W. W. Norton、 publication Feb 2023; 著者の book announcement は 2022-11) — 「あらゆる rule system は hack 可能、 hack 能力は power に偏在」 という general thesis
- **2023**: 個別 essay [*Hacking the Tax Code*](https://www.schneier.com/blog/archives/2023/02/hacking-the-tax-code.html)
- **2026-04**: Schneier 自身が事前ブログで「[Anthropic Mythos = very much a PR play](https://www.schneier.com/blog/archives/2026/04/on-anthropics-mythos-preview-and-project-glasswing.html)」 と評価
- **2026-05-08**: 本 Guardian 記事 = thesis の Mythos event への適用版

→ Guardian 記事は新規 thesis ではなく、 5 年越し議論の「現実が thesis に追いついた」 タイミングの event reporting。 §4-6 (規則系波及) が著者にとっての本論、 §1-3 は導入。

## Stenberg curl 評価の詳細 (= 単一 maintainer 現場一次 evidence)

最も殺傷力の高い独立評価:

- 178K LOC scan、 報告 5 件中 **実 vulnerability は 1 件** (= signal/noise 1/5)
  - 3 件: false positive (documented API behavior の誤判定)
  - 1 件: "just a bug" (security 影響無し)
  - 1 件: **severity-low CVE** (curl 8.21.0 / 2026-06 末 release)
- 既存 AI tool (AISLE / Zeropath / OpenAI Codex) は curl で **200-300 件** の bugfix 触発、 Mythos < 既存
- Stenberg 一般評価: 「AI powered code analyzers are significantly better at finding security flaws and mistakes in source code than any traditional code analyzers did in the past」 / 「Not using AI code analyzers ... leave opportunity to attackers」
- AI tool の固有強み: code/comment mismatch 検出、 unsupported platform analysis、 third-party API 理解、 flaw summarize
- **limitation**: 「**確立済 error category**」 は得意、 「**novel vulnerability type**」 は不得手 (= jagged frontier の bright side が OpenBSD SACK、 commodity 領域が FreeBSD NFS、 という 2 領域の存在を裏付け)

## 読者にとっての takeaway

### 即時 action

- **curl 8.21.0 監視** (2026-06 末): 自プロジェクトで curl 使用箇所があれば 6 月末 release で upgrade
- **自リポの AI scan hygiene**: 公開 repo に対し GitHub CodeQL / Semgrep / dependabot 等の自動 AI scan を有効化しているか sweep。 Stenberg の baseline 認識 = 「**AI tool 未使用は attacker への opportunity 譲渡**」

### 思考の reflex

- **大手 AI announcement の sanity check**: 1 次 source (= 評価対象 system の maintainer 本人 = Stenberg / de Raadt / Torvalds 類) の意見を必ず探す。 PR と現場乖離の reflex check
- **「AI が pass = secure」 を信じない**: Qwen3 32B が OpenBSD SACK で「robust」 と confidently 誤断言。 単一 model の pass は jagged frontier で破綻、 重複 scan 必須
- **「AI で発見」 vs 「training data memorize」 の切り分け**: 17 年もの bug が AI で見つかった時、 「**novel reasoning による発見**」 と「**public archive の retrieval**」 を判別する reflex

### 規則系 hacking 倫理線

Schneier の predict (= 大企業による税法 AI hacking) は技術的に時間問題。 academia / public sector 側でも「規程の loophole を AI に探させる」 等の道具は available だが、 use の正当性は別線。 道具の available 性 ≠ 使用の正当性、 という分離は重要 — 攻撃方向に reflex を作らず、 規則系 system に対しても防御方向の reflex のみ持つ。

## 検証作業の途中で起きた私 (Claude) の誤りと修正

1. **Mythos 実在性疑義 (誤)**: 私の knowledge cutoff (2026-01) 後に Anthropic が発表した model を「公式 family に存在しない」 と推測した。 **完全に間違い**、 取り下げ
2. **271 件 = OSS-Fuzz baseline 上乗せ説 (誤)**: Mozilla 公式は Mythos attribute の initial evaluation pass 単独で 271 件、 baseline 上乗せではない。 ただし 「code defects vs CVE」 gap の指摘は別経路で正当
3. **uniqueness 主張を「assertion 段階」 と評価 (過小評価)**: AISI 公式数値 + AISLE 実験 + Stenberg 現場 の 3 独立 source で empirically 裏付け済だった

これらは「**cutoff 後の event を推測で埋めず、 必ず一次 source で verify**」 という規律の重要性を示す事例。
