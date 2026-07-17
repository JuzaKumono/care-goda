---
title: "Claude × Obsidian × Codex 最強の共通セカンドブレイン完全構築ガイド"
author: "りゅう@Obsidianガチ勢"
handle: "@obsidianstudio9"
date: 2026-05-29
source: https://x.com/obsidianstudio9/status/2060161232360059022
---

# Claude × Obsidian × Codex 最強の共通セカンドブレイン完全構築ガイド

By **りゅう@Obsidianガチ勢** (@obsidianstudio9) | 午前9:48 · 2026年5月29日

---

[

![画像](https://pbs.twimg.com/media/HI53_V0bwAA0a_U?format=jpg&name=medium)







](/obsidianstudio9/article/2060161232360059022/media/2057714009827426304)

Claude × Obsidian × Codex 最強の共通セカンドブレイン完全構築ガイド

Obsidianに複数の脳を作ってる人は完全に時代遅れです。セカンドブレインを１つにして運用すると、コード生成・設計・改善の質が格段に上がります。今回は、海外で大バズしたの設計をもとに、Claude Code・Codex・Obsidianを1つの共通ファイル領域で連携させる方法を、ハマりどころ込みで解説します。Claude Code と Codex、両方使っている人は必見です

こんな悩み、ありませんか？

-   Claude Code と Codex のどっち使うか、毎回迷う
    
-   Obsidian の Vault が AI 連携で活かしきれていない
    
-   スキルが増えすぎて把握できない
    
-   MCP を入れすぎてコンテキストが重い
    
-   3ツールの設定がバラバラで二重メンテになる
    

これ、3ツールを「独立したツール」として扱っているから起きる現象です。

[

![画像](https://pbs.twimg.com/media/HJcprcCbcAAuNAG?format=jpg&name=medium)







](/obsidianstudio9/article/2060161232360059022/media/2060161180782784512)

OpenAI 創設メンバーの Andrej Karpathy 氏が、自分の知識ベースを LLM Wiki として運用する設計を 2026 年 4 月に公開しました。GitHub Gist で 41,000 人以上にブクマされていて、彼はこう書いています。

Obsidian is the IDE; the LLM is the programmer; the wiki is the codebase.

Obsidian は IDE、LLM はプログラマー、wiki はコードベース。

この比喩を3ツール統合に応用するとこう読めます。Obsidian は「ノートを置く場所」ではなく「Claude と Codex が共有する作業環境」。Claude Code と Codex は「別々のツール」ではなく「同じ共通ファイル領域を読み書きする2人のプログラマー」。

今回はこの設計を、ステップバイステップで日本語解説します👇

## 

なぜ今「共通ファイル」が必要なのか

ここ数ヶ月、Claude Code と Codex CLI を併用する開発者が一気に増えました。

ある統計だと、Codex CLI の npm 月間ダウンロード数は 2025 年 4 月の 82,000 件から、2026 年 3 月には 14,530,000 件まで伸びて います。

177倍。

公式統計では週間アクティブユーザーも 300 万人（年初比5倍）、月間で2億件超のリクエストを処理している。

一方で Claude Code 側も止まっていません。

JetBrains が 2026 年 4 月に発表した Developer Ecosystem Survey 2026 では、Claude Code の開発者満足度（CSAT）が 91%、推奨度（NPS）が +54 で業界最高水準。世界の開発者の 18%、米国・カナダだと 24% が業務で採用していて、過去 6 ヶ月で採用率が 6 倍に伸びています。

このトレンドの読み方を間違えると、二択思考に陥ります。

「Codex に乗り換えるべきか、Claude Code を継続するか」。

実はこの問いの立て方そのものが、3ツール時代の本質を見落としています。

正しい問いは「両方使う前提で、共通する設計はどこにあるか」のはず。

3ツールを別物として扱うと何が起きるか

Claude Code、Codex、Obsidian を「それぞれ独立したツール」として扱うと、3つの問題が同時進行で起こります。

1つ目は MCP の過剰投入によるコンテキスト消費爆発です。

ある実測ベンチマークでは、GitHub MCP 経由で開発タスクを回したケースで月額 $55.20 のコストがかかった一方、gh CLI 直叩きに置換したら $3.20 まで下がりました。

17倍のコスト差。

別の比較では Microsoft Graph MCP と mgc CLI でコンテキスト消費に 35倍の開き が出ています。Playwright MCP のスナップショット1回分が 56KB（トークン換算）を食う一方、CLI ならファイルパスの参照だけで完結する。

[

![画像](https://pbs.twimg.com/media/HJcprb2bkAAOT91?format=jpg&name=medium)







](/obsidianstudio9/article/2060161232360059022/media/2060161180732461056)

「便利だから」で MCP を入れ続けると、コンテキストウィンドウが MCP のメタデータで埋まって、肝心のコード文脈が押し出されていきます。

ある業界エンジニアが書いた「MCP is Dead. Long Live the CLI」という記事は、Hacker News で 400 ポイント超、300 件以上のコメントが付きました。「MCP は不要、CLI で十分」という主張が、一定の支持を集めるレベルまで来ている。

2つ目は Claude と Codex の切り替えコストの累積です。

設定ファイル、スキル、エージェント、ツール許可リスト、それぞれが別々のディレクトリで管理されると、片方で覚えた知見がもう片方で再利用できない。両方使う前提で設計されていないんです。

3つ目は Obsidian の Vault が AI 連携の中で活かしきれない問題。

ナレッジが Vault に蓄積されているのに、Claude Code は「セッション中の Read ツール」でしか触れず、Codex は「.agents/skills の中だけ」で完結し、両者が Vault を「中央ハブ」として扱う仕組みがありません。メモを資産化したくて Obsidian を導入したのに、AI 側からは「ただのフォルダ」扱い。

スケールするほど崩壊する

Gartner が 2026 年に出した予測では、2027 年までにエージェント AI プロジェクトの 40% 以上が失敗または中止される とされています。理由のトップは「透明性不足」で、79% のプロジェクトが「何をしているか可視化できていない」状態。

これは Claude vs Codex の話じゃないんです。「ルール・スキル・エージェント・MCP を増やすほど、自分のシステムを自分で理解できなくなる」という構造問題。

ある匿名の開発者が公開した事例では、40 体のエージェントを 1 ヶ月運用したら全部廃棄せざるを得なくなったと報告されています。context rot（コンテキスト腐敗）と compaction による情報喪失で、一定規模を超えると人間側もエージェント側も全体像を維持できなくなる。

スケールすると壊れる。これが3ツール統合で最初に押さえるべき制約です。

Karpathy の比喩で整理する

ここで Karpathy 氏が提唱した、LLM Wiki という比喩が効いてきます。

Obsidian is the IDE; the LLM is the programmer; the wiki is the codebase.

3者を別物として扱うのではなく、1つの開発環境 として組み立てる視点です。

これを Claude Code × Codex × Obsidian に置き換えるとこうなります。

-   Obsidian Vault = コードベース（知識・判断・履歴の物理的な置き場）
    
-   Claude Code = プログラマー1号（設計・判断・コンテキスト管理を担当）
    
-   Codex CLI = プログラマー2号（実装・並列実行・軽量タスクを担当）
    
-   \_shared-ai/ = 両プログラマーが共有するヘッダーファイル群
    

このメタファーに切り替えた瞬間、「個別最適」から「統合最適」への視点転換が起きます。

\_shared-ai/skills/ に置いた共通スキルは「Claude のスキル」でも「Codex のスキル」でもなく、両者が読む共有ヘッダーになる。Claude が書いた CLAUDE.md と Codex が読む AGENTS.md は、別ファイルではなく同じ思想の別実装に変わる。Vault は「メモを溜める場所」ではなく両プログラマーの中央リポジトリに格上げされる。

これがこの記事の出発点です。3ツールを統合する設計を、ステップバイステップで書いていきます。

## 

全体アーキテクチャ — 3層スキル構造の俯瞰

3ツール統合を実装する前に、設計の見取り図を持っておくと迷子になりません。

ここで提案する構造は、Anthropic と OpenAI それぞれが公式で出している「3層」の考え方を踏まえて統合したものです。ざっくりこんな配置になります。

L1: プラグイン固有スキル（teams / coordinator / obsidian-sync 等）

→ 各プラグインの責務に密着、特定ツール内でのみ動く

L2: \_shared-ai/skills/（共通スキル領域）

→ Claude Code と Codex の両方が読む、横断利用前提

L3: 個別プロジェクト固有スキル（00-Projects/<name>/.claude/skills/）

→ 単一プロジェクト内で閉じる、案件特化

この3層をなぜ採用するか、根拠を1つずつ書いていきます。

Anthropic Skills 2.0 の progressive disclosure

公式ドキュメントによると、Anthropic Skills 2.0 は Level 1 / 2 / 3 の3段階でロードする設計です。Level 1 はメタデータ（name と description）でシステムプロンプトに常時ロード、Level 2 が SKILL.md 本体の指示文でトリガー発動時にロード、Level 3 が scripts / references / templates で実行時に必要なら追加ロード。これは「1つのスキル内」の段階開示です。

エンジニアリングブログにはこう書かれています。

Claude initially sees only skill names and descriptions in its system prompt, then loads full details only when relevant to the current task.

要は「スキルを100個入れても、常時ロードはメタデータ部分だけ」。これが共通スキル群を大規模に増やせる根拠になっています。

OpenAI Codex 側の3層構造

OpenAI も似た構造で Codex Skills を出しています。公式の Codex Developers Docs によると Skills は3層に分かれていて、Codex に自動インストールされるコア機能の System、コミュニティ精選で手動インストールする Curated、開発中の Experimental という構成です。

そして Codex は .agents/skills を 現在の作業ディレクトリからリポジトリ root まで遡って自動スキャン する設計です。

Codex reads skills from repository, user, admin, and system locations, scanning .agents/skills in every directory from your current working directory up to the repository root.

つまり「どこに配置するかでスコープを制御する」仕組みになっています。

統合した3層配置

[

![画像](https://pbs.twimg.com/media/HJcprcBacAAromQ?format=jpg&name=medium)







](/obsidianstudio9/article/2060161232360059022/media/2060161180778524672)

Anthropic の「単一スキル内の3段階」と Codex の「3スコープ層」を統合すると、ユーザー側の運用設計として L1 / L2 / L3 という配置軸が見えてきます。

L1（プラグイン固有）は、Claude Code のプラグインキャッシュ配下に置くスキル群。例えば teams プラグインの article-factory や coordinator プラグインの orchestrate のように、特定プラグインの責務に密着していて、他のツールから直接呼び出すことを想定しないもの。

L2（共通スキル領域）は、\_shared-ai/skills/ 配下に置くスキル群。ここが今回の記事の主役で、Claude Code と Codex の両方が junction 経由で同じ実体を参照する。例えば「公式ベストプラクティス準拠チェック」「MCP 棚卸し」「AI×AI ファクトチェック」のように両ツールで共通に使うロジックを集約する場所です。

L3（プロジェクト固有）は、個別案件の .claude/skills/ 配下に置くスキル群。例えば「特定の事業案件向けの提案書テンプレ」「特定のクライアント向けの議事録フォーマット」のように、1つのプロジェクト内で閉じるスキルを置く場所です。

配置判断フロー

新規スキルを書くとき、どの層に置くかは1つの問いで決まります。

「これは何個のツール／プロジェクトから呼ばれるか」。

-   1ツール内・1プラグインで完結 → L1
    
-   2ツール（Claude + Codex）で共通に呼ぶ → L2
    
-   1プロジェクト内でしか意味がない → L3
    

迷ったら L1 が default です。

理由は、L1 は影響範囲が局所化されていて、後で L2 に昇格させる方が安全だから。逆方向（L2 から L1 へ降格）は、参照箇所を全部書き換える必要があって面倒です。

「最初は L1 で書いて、2ツールから呼ばれることが実証されたら L2 に昇格」のフローを default にすると、設計の手戻りが減ります。

このアーキテクチャを実装する具体手順を、Step 1 から書いていきます。

## 

Step 1: \_shared-ai/ ディレクトリ構造を作る

理論はここまで。手を動かしていきます。

[

![画像](https://pbs.twimg.com/media/HJcprb0aEAAF3__?format=jpg&name=medium)







](/obsidianstudio9/article/2060161232360059022/media/2060161180723974144)

最初にやるのは、共通ファイル領域の物理的な置き場所を決めることです。外付けクラウドドライブ（Google Drive や OneDrive の同期領域）配下に \_shared-ai/ という名前のディレクトリを切るのが基本構成。

理由は2つ。ローカルマシン依存にならず複数台から参照できること、Claude Code と Codex の両方が junction / symlink で同じ実体を読めること。

中の構造はこうなります。

\_shared-ai/

├── skills/ # L2共通スキル群

│ ├── bestpra-checker/

│ ├── cli-vs-mcp-audit/

│ ├── factcheck-ai-cross/

│ └── codex-handoff/

├── knowledge/ # 共通ノウハウ（公式ベスプラ・パターン集）

├── prompts/ # プロンプトテンプレート

├── scripts/ # 共通スクリプト

└── README.md # 運用ルール

各ディレクトリの責務はこう整理します。skills/ には Step 3 で組み立てる4本（と将来追加分）が SKILL.md + references/ + scripts/ の標準フォーマットで入る。knowledge/ には Claude Code 公式ベストプラクティス・Anthropic Skills 設計指針・OpenAI Codex 推奨パターンなど、スキル本体ではなくスキルを書く時に参照する判断材料を集約する。prompts/ にはリサーチ・執筆・Codex 引き継ぎなどの定型プロンプト雛形を、scripts/ には文字数チェック・Markdown整形・DOCX生成などの汎用ツールを置く。トップの README.md がこの共通ファイル領域の運用ルールを担います。

PARA 構造と並列で運用する

クラウドドライブ直下にこの \_shared-ai/ を置く時、隣に PARA 構造を並列配置するのを推奨します。

ClaudeCode/

├── 00-Projects/ # 進行中の案件

├── 01-Areas/ # 締切なし継続業務

├── 02-Resources/ # 参考素材・テンプレ

├── 03-Archive/ # 終了/凍結

├── \_inbox/ # 仮置き

├── \_shared-ai/ # ★ 共通ファイル領域

└── Obsidian-Vault/ # 中央ナレッジハブ

PARA は Tiago Forte が提唱した「プロジェクト・エリア・リソース・アーカイブ」の4分類です。ここで重要なのが、\_shared-ai/ を PARA と独立した第5階層 として扱うこと。

Projects 配下に共通スキルを置くと、プロジェクト終了時に Archive へ移動して他プロジェクトから参照不能になる。Resources 配下に置くと、素材ライブラリと共通スキルが混ざって判断軸がぼやける。

\_shared-ai/ は 時間軸でアーカイブされない・プロジェクト固有でない・素材ではなく実行可能資産 という3条件を満たすので、独立した階層が必要です。

README.md の運用ルール

トップに置く README.md には、運用ルールを明文化しておきます。

ある実装事例の note 記事に、こんな表現がありました。

思想ファイルとトップポスト集があるかどうかで、構築の速度も AI の理解度もまったく変わります。

これは仮想チームを CLAUDE.md ベースで運用する開発者の言葉ですが、\_shared-ai/ でも同じです。

README.md に書くのは「思想ファイル」相当の4項目。各サブディレクトリの責務と境界、新規スキル追加時の判断フロー（L1 / L2 / L3 のどこに置くか）、信頼ソースの基準（Anthropic セキュリティ警告「自分が書いたか公式由来か」）、棚卸し頻度（年1回 MCP / 月1回スキル / 四半期 ephemeral）。

これがあると、半年後の自分が「なぜこのスキルがここにあるのか」を再構築せずに済みます。

Polaris 型との対比

Obsidian × Claude Code 統合の海外実装事例として、Polaris（北極星）フォルダ構造というのがあります。Logs / Commonplace / Outputs / Utilities の5フォルダで Vault を切り、AI 生成物は Outputs に閉じ込めて「Vault 本体に混入させない」という設計思想です。

\_shared-ai/ はこの逆の哲学。AI 生成物を「閉じ込める」のではなく、「Vault と共通スキル領域の両方から相互参照する」前提で組みます。tier 評価でソース信頼度を担保すれば、混入リスクは制御可能。

どちらが正解という話ではなく、運用者がどちらの哲学を取るか最初に決めておく必要があります。

ここまでで共通ファイル領域の物理的な置き場所が決まりました。次は Claude Code と Codex を junction で繋ぎます。

## 

Step 2: Codex junction で2ツールを接続する

\_shared-ai/ を作ったら、次は Codex 側からこの実体を参照できるようにします。ここで使うのが junction（Windows）または symlink（Mac/Linux）です。

[

![画像](https://pbs.twimg.com/media/HJcprcPbQAAs2H_?format=jpg&name=medium)







](/obsidianstudio9/article/2060161232360059022/media/2060161180837298176)

Codex は ~/.codex/skills/ 配下を自動スキャンする設計なので、その配下にクラウドドライブの \_shared-ai/skills/ を「リンク」として置けば、Codex は普通のディレクトリとして読みに行きます。

Windows: junction で接続

管理者権限のコマンドプロンプトで mklink /J を1発叩くだけ。

mklink /J "%USERPROFILE%\\.codex\\skills\\shared" "<クラウドドライブパス>\\\_shared-ai\\skills"

junction なので Codex 側は ~/.codex/skills/shared/<skill-name>/ という普通のパスとして認識します。シンボリックリンクと違って権限要件が緩く、別ドライブ間でも貼れる。Windows ではこれが一番安定です。

Mac / Linux: symlink で接続

こちらはもっと素直。

ln -s "<クラウドドライブパス>/\_shared-ai/skills" "$HOME/.codex/skills/shared"

POSIX symlink なので Codex 側からは透過的にディレクトリとして見えます。

Claude Code 側の設定

Claude Code 側はもう一段工夫が必要です。

Claude Code はプラグインキャッシュ配下の ~/.claude/plugins/cache/.../skills/ を自動ロードする設計で、ホームディレクトリ配下の任意パスを自動スキャンする仕組みが default では入っていない。

そこで CLAUDE.md の中から 明示的に \`\_shared-ai/skills/<skill-name>/SKILL.md\` を Read する スタイルを取ります。

\## 共通スキル領域

\- 共通スキルは \`<クラウドドライブパス>/\_shared-ai/skills/\` に集約

\- 各スキルの SKILL.md は必要時に Read で読み込む

\- 詳細仕様は \`\_shared-ai/README.md\` 参照

Claude が「共通スキルが必要」と判断した瞬間、Read ツールで該当パスを読みに行く動線が作れます。Codex のような完全自動スキャンではないですが、progressive disclosure の Level 1 を CLAUDE.md で代替する形と読めます。

config.toml の設定（Codex 側）

Codex 側の ~/.codex/config.toml も整えます。

model = "gpt-5.5"

model\_reasoning\_effort = "medium"

\# Windows sandbox 設定（環境依存）

\[windows\]

sandbox = "elevated"

\# 共通スキル領域は junction 経由で自動認識

\# ~/.codex/skills/shared/ → \_shared-ai/skills/

OpenAI 公式ドキュメントによれば、Codex は repository / user / admin / system の4スコープから skills を読み込みます。junction で ~/.codex/skills/shared/ に貼った共通スキルは user スコープとして認識され、リポジトリ横断でアクセス可能になる。

接続パターンは5種類ある

ちなみに Claude Code × Obsidian × Codex の接続パターンは、海外実装事例だと5系統に整理されています。同じディレクトリを複数ツールから参照する symlink パターン（本記事推奨）、Vault 自体を Git リポジトリ化して同期する vault-as-repo パターン、WebSocket / REST で双方向通信する MCP bridge パターン、プロジェクトごとに .agents/skills を持つ per-repo パターン、Quarto Markdown で計算可能な Vault を作る QMD パターンの5種です。

5パターンの中で junction / symlink が一番シンプルで運用負荷が低い。MCP bridge は柔軟ですが、前述の 35倍コンテキスト消費問題に直結するので、まず junction で組んでから必要に応じて MCP を補助的に追加する順序が安全です。

接続後の動作確認

junction が貼れたら、両ツールから動作確認します。

[

![画像](https://pbs.twimg.com/media/HJcprcUbcAAQ3OD?format=jpg&name=medium)







](/obsidianstudio9/article/2060161232360059022/media/2060161180858281984)

\# Codex 側

ls ~/.codex/skills/shared/

\# → bestpra-checker / cli-vs-mcp-audit / ... が見える

\# Claude Code 側（CLAUDE.md 参照経由）

\# Claude にこう聞く

\# →「\_shared-ai/skills/bestpra-checker/SKILL.md を読んで」

両方で同じ実体にアクセスできれば、接続成功。物理的な接続はこれで完了です。あとは中身（共通スキル4本）を書くだけ。

次の Step 3 では、共通スキル4本の中身を組み立てていきます。

## 

Step 3: 共通スキル4本を組み立てる

ここがこの記事の中核です。

\_shared-ai/skills/ 配下にどんなスキルを置くか。これを決めると、Claude Code と Codex の両方が「同じ判断軸」で動くようになります。逆にこの設計を間違えると、2ツール間で挙動が割れる。

[

![画像](https://pbs.twimg.com/media/HJcprcbbwAALYPv?format=jpg&name=medium)







](/obsidianstudio9/article/2060161232360059022/media/2060161180887662592)

実運用で効く共通スキルを4本、設計思想と一緒に書いていきます。

スキルの最小単位を Anthropic から学ぶ

まず、Agent Skills 2.0 の公式仕様を1度押さえます。

Agent Skills are modular capabilities that extend Claude's functionality. Each Skill packages instructions, metadata, and optional resources (scripts, templates) that Claude uses automatically when relevant.

つまり1つのスキルは「指示文 + メタデータ + 必要なら補助リソース」の3点セット。

このうち最小必須は、SKILL.md ファイル先頭に置く YAML frontmatter です。

\---

name: skill-name-kebab-case

description: This skill should be used when the user asks to "phrase1", "phrase2", or discusses \[topic\].

version: 0.1.0

\---

name と description、この2つだけがクラウド起動時に常時ロードされる「Level 1 情報」になっています。

ここが設計の効くポイントです。

Anthropic は Skills 2.0 で progressive disclosure（段階的開示）という仕組みを導入していて、Level 1 のメタデータが常時ロード、Level 2 の SKILL.md 本体がトリガー発動時にロード、Level 3 の scripts / references / templates が必要時にロードされる3段階構成になっています。

Claude initially sees only skill names and descriptions in its system prompt, then loads full details only when relevant to the current task.

要は「スキルを何個入れてもコンテキストを圧迫しない」設計。100個追加してもメタデータの行数で済むので、共通スキル群を躊躇なく増やせるわけです。

ここを最初に押さえると、4本どころか20本でも投入する勇気が出てきます。

OpenAI Codex 側の Skills 機構

OpenAI 側も似た仕組みを提供しています。公式ドキュメントによると Codex は .agents/skills 階層を自動スキャンする設計で、Anthropic 仕様で書いた SKILL.md を junction 経由で Codex に渡したらそのまま読める状態が現実として成立しています。

この互換性こそが、\_shared-ai/skills/ を中央集約する根拠なんです。

スキル設計の鉄則

OpenAI Codex 公式ドキュメントには、スキル設計の原則がこう書かれています。

[

![画像](https://pbs.twimg.com/media/HJcprcsa0AAFIyU?format=jpg&name=medium)







](/obsidianstudio9/article/2060161232360059022/media/2060161180958904320)

Keep each skill focused on one job, prefer instructions over scripts unless you need deterministic behavior or external tooling, and write imperative steps with explicit inputs and outputs.

1スキル1責任、指示文優先、入出力を明示。この3点はそのまま \_shared-ai/skills/ の品質基準として採用していい。

加えて Anthropic 側にはセキュリティ警告もあります。

Use Skills only from trusted sources: those you created yourself or obtained from Anthropic.

スキルはコード実行権限を持つので、信頼できるソース以外から導入してはいけない。「共通スキルだから安全」ではなく「自分が書いたから安全」「公式のものだから安全」が判断基準になります。

共通スキル1: bestpra-checker（公式準拠チェック）

最初の1本は、自分が書いたスキルを「公式仕様に準拠しているか」自動チェックする bestpra-checker です。これがあると、新規スキル追加時の品質ブレが減る。

具体的なチェック項目は frontmatter の必須キー（name / description / version）が揃っているか、description が自然言語トリガー形式になっているか、本文長が 1,500〜3,000 字に収まっているか、I/O が冒頭で明示されているか、補助データが references/ や data/ に分離されているか、MCP 依存になっていないか（CLI 代替があれば優先）、1スキル1責任になっているか、セキュリティ警告（信頼ソース以外禁止）がカバーされているかの8項目。

スキルを書いた後にこの8項目を流す、というのが運用ルーチンです。実は、書く側の判断より、書いた後にチェックする方が再現性が高い。「動くスキル」と「正しく書かれたスキル」は別問題で、後者は機械チェックの方が確実です。

共通スキル2: cli-vs-mcp-audit（年1回棚卸し）

2本目は MCP 棚卸し用の cli-vs-mcp-audit。Claude Code と Codex の両方で、現在登録されている MCP サーバーを列挙して「CLI 代替が出来るか」を判定するスキルです。

[

![画像](https://pbs.twimg.com/media/HJcprc6bgAAqLwK?format=jpg&name=medium)







](/obsidianstudio9/article/2060161232360059022/media/2060161181017669632)

ある実測ベンチマークでは、GitHub MCP を使うと月額 $55.20 のコストがかかったケースが、gh CLI 直叩きに置換したら $3.20 に下がりました。17倍のコスト差。別のベンチマークでは Microsoft Graph MCP と mgc CLI を比較して 35倍のコンテキスト消費差 が出ています。Playwright MCP のスナップショットは1回 56KB（トークン化後）も食ってましたが、CLI ならファイルパス参照だけで済む。

このスキルがやることは、現在の MCP 登録一覧を取得し、各 MCP について CLI 代替候補が存在するか確認し、代替可能なら置換手順を提示、代替不可なら理由を記録（リアルタイム性 / UI 操作必須 / 認証フロー 等）する流れ。

年1回、または /context の MCP 消費が 5% を超えたタイミングで実行する設計にしておくと、無駄なコンテキスト消費を抑えられます。「MCP is Dead. Long Live the CLI」の論争が支持を集めるレベルになっている今、棚卸しを文化として持つ意味はここにあります。

共通スキル3: factcheck-ai-cross（2系統ファクトチェック）

3本目は AI×AI ファクトチェック用の factcheck-ai-cross。重要な成果物（事業計画書・記事の数値・法律条文引用 等）の確度を、Claude Code と Codex の2系統で検証するスキルです。

[

![画像](https://pbs.twimg.com/media/HJcprdGagAAyfHM?format=jpg&name=medium)







](/obsidianstudio9/article/2060161232360059022/media/2060161181067935744)

手順は Claude Code で第1次回答を作成（ソースを記録）、Codex を呼んで独立検証、両者一致なら採用、不一致ならユーザー判断、一次ソース未到達なら再依頼、というシンプルな流れ。

ある研究では、メタエージェントとタスクエージェントを「同じモデル系統」で組ませると診断精度が 20% 以上向上した という報告があります。Claude + Claude のペアが Claude + GPT のペアより精度が高い。

ここから派生する判断として、「最終確認は Claude で、計算検証は Codex で」のような役割分離が効くケースが多いです。両者の出力を統合する箇所だけスキル化しておく、というのが現実的な落とし所になります。

共通スキル4: codex-handoff（引き継ぎプロンプト整形）

4本目は Claude → Codex 引き継ぎ用の codex-handoff。Claude Code で「設計・判断」をやって、Codex に「実装・実行」を渡すフローを定型化するスキルです。

引き継ぎプロンプトに必要な要素はタスクの目的（何を達成するか）、制約条件（使う技術 / 避ける手法）、受け入れ基準（完成条件）、参照ファイル（どこを読むか）、完了報告の形式（何をどう返すか）の5つ。

これらをテンプレートに落としておくと、毎回ゼロから書く必要がなくなります。ある企業の実運用事例では、リリース前チェックリストや PR レビューを Skill 化したところ、運用前は「必要ファイル指定 + プロンプト整形」に 2〜3 分かかっていたのが、Skill 後は「Skill 名 + テーマ指示」で 10 秒に短縮された という報告があります。入力時間 80% 削減。

これは Codex に渡す引き継ぎプロンプトでも同じことが起きる。

4本セットで運用する意味

bestpra-checker / cli-vs-mcp-audit / factcheck-ai-cross / codex-handoff。この4本セットで何が起きるかというと、新規スキルは bestpra-checker で品質保証、MCP は cli-vs-mcp-audit で年1回棚卸し、重要成果物は factcheck-ai-cross で2系統検証、Claude → Codex は codex-handoff で定型化、という運用が回り出します。

つまり「共通スキル領域そのものの品質維持」と「3ツール間の連携品質」の両方が、スキル4本で担保される設計になっています。「スキルでスキルを管理する」入れ子構造になっていて、これが「使うほど賢くなるシステム」の核心部分です。

書いた瞬間は仕事を増やしているように見えるけど、3週間後にはこの4本がない世界に戻れなくなります。共通ファイル設計で一番リターンが大きい投資が、この4本だったりするんです。

## 

Step 4: CLAUDE.md と AGENTS.md の共通化・分離

物理的な接続が終わったら、次は思想ファイルの整理です。

Claude Code は CLAUDE.md を、Codex は AGENTS.md を、それぞれ「司令塔ファイル」として読み込む設計になっています。これを別々に書くと、メンテナンスコストが倍になるだけでなく、両者の判断基準がズレていく。

ここでも「共通化できる部分」と「分離すべき部分」を最初に切り分けるのが効きます。

80% は planning, 20% が execution

Every 社が公開している Compound Engineering Plugin の哲学に、こんな一節があります。

The distribution is explicit: 80% is in planning and review, 20% is in execution.

エンジニアリングの 80% は計画とレビュー、20% が実装。開発者のメンタルモデルとしては逆転しがちですが、AI エージェント運用ではこの比率が現実的になります。

CLAUDE.md / AGENTS.md は「20% の実装を成功させるための 80% の計画書」として位置づける。これが共通化の出発点です。

共通化できる3項目

Claude と Codex の両方に同じ内容を書いていい項目は、3つ。

1つ目は L2 スキル定義。\_shared-ai/skills/ 配下のスキルは両ツール共通の判断軸として参照されるので、CLAUDE.md にも AGENTS.md にも「共通スキル領域は \_shared-ai/skills/ に集約」と書き、各スキルの description / version を共有します。

2つ目は AI×AI ファクトチェック手順。「Claude で第1次回答 → Codex で独立検証 → 両者一致なら採用、不一致ならユーザー判断」というフローは両ツール側で同じ記述を持っていい。

3つ目は CLI vs MCP 優先順位表。Playwright / GitHub / Google Workspace / Vercel など CLI 代替が確立しているツールは CLI 優先、リアルタイム性が必要な Slack / Discord 系のみ MCP 保持。この表は両ツールで同じ参照先（\_shared-ai/knowledge/cli-vs-mcp.md のような共通ナレッジ）を見ることで、判断のブレを防ぎます。

分離すべき3項目

逆に、片方にしか書かない方がいい項目もあります。

1つ目は plugin-dev / agent-skill-dev の体系。Claude Code はプラグイン機構（~/.claude/plugins/ 配下に commands / agents / skills / hooks）を持っていますが、Codex はこれを持たない。「プラグインの3層構造（source / cache / 直下）」のような Claude Code 固有の話は AGENTS.md には書きません。

2つ目は環境制約（Sandbox / Permissions）。Windows 環境での Codex sandbox 設定、Claude Code の settings.json パーミッション周り、両者の許可リスト形式は別物です。これを混ぜると、片方で動いていた設定がもう片方で動かない原因が見えなくなる。

3つ目は Model 選択ロジック。Claude Code は Opus / Sonnet / Haiku の3系統、Codex は GPT-5.5 系統。「重い判断は Opus 固定」「定型作業は Sonnet」のような Claude 側のロジックを Codex 側に書いてもノイズです。

「司令塔」と「実行者」の階層を作る

大滝氏が公開している仮想チーム運用記事に、こんな表現があります。

Claude Code は違います。PC 上のファイルを直接読み書きできるし、Web で調べ物もできるし、スクリプトを動かすこともできる。「チャットで相談する相手」ではなく、「実際に手を動かしてくれる者」です。

CLAUDE.md / AGENTS.md は司令塔で、AI 本体は実行者。ここを階層で分けると、司令塔ファイルが軽くなって、実行者が状況に応じて判断する余地が増えます。

推奨レイアウト

実運用での推奨は、こんな階層になります。

CLAUDE.md / AGENTS.md ← 司令塔（200行以内厳守）

└── rules/ ← 状況別ルール（各ファイル50〜100行）

├── shared-conventions.md # 両ツール共通

├── claude-specific.md # Claude Code 固有

├── codex-specific.md # Codex 固有

└── ...

司令塔ファイル本体は薄くして、詳細は rules/ に委任する。両ツールから読まれる shared-conventions.md は \_shared-ai/knowledge/ に置いて junction 経由で共有しても良いです。

CLAUDE.md / AGENTS.md を軽く保つために押さえる原則は「権限境界の明示」と「自分で作業しない宣言」。司令塔ファイルが「自分で作業しよう」とすると、サブエージェント / スキル / コマンドに権限を渡せなくなって、全部が司令塔ファイルに集約されていく。これが context bloat の原因です。

司令塔は「何を誰に任せるか」だけ書く。実装の詳細は委任先に任せる。この階層化ができると、3ツール統合がスケールしても破綻しなくなります。

## 

Step 5: Obsidian Vault を中央ナレッジハブにする

ここで Obsidian Vault が登場します。3ツール統合の中で Vault が担う役割は、両プログラマー（Claude / Codex）の中央リポジトリです。

ここを「ただのノートアプリ」として扱うか、「中央ナレッジハブ」として設計するかで、運用の伸び方が変わります。

Karpathy の LLM Wiki 3層構造

冒頭で引用した Karpathy 氏の GitHub Gist には、Vault の内部構造を3層で整理する設計が書かれています。

Three layers: 1. Raw sources (immutable) 2. The wiki (LLM-maintained markdown) 3. The schema (configuration defining structure and workflows)

3層の責務を翻訳するとこうなります。Raw sources（不変）は取り込んだ生の素材（記事・論文・Slack ログ・議事録）で、AI は読むだけで書き換えない。Wiki（LLM が維持）は AI が抽出・統合・整理した markdown ノート群で、バックリンクで相互参照する。Schema（人間が定義）はフォルダ階層・タグ規則・ワークフローの設計図で、人間が決めて変えない。

この3層分離の何が効くかというと、「AI が暴走しても Raw sources は無傷」という安全装置になることです。Wiki 層は AI が日々書き換えていいが、Raw sources は人間が置いたものを AI は触らない。

Vault の実用構造

実運用では、こんなフォルダ階層に落とし込みます。

Obsidian-Vault/

├── Knowledge/ # AI 生成のナレッジノート（Wiki 層）

├── Library/ # 外部から取り込んだ素材（Raw sources 層）

├── Daily/ # デイリーノート

├── Sessions/ # セッションダイジェスト

└── Tasks/ # タスク履歴

Library に置く外部記事には tier 評価（1=一次データ 〜 5=ソーシャル / 逸話）を付与する。これで「どの情報をどれくらい信頼していいか」が AI 側からも判定可能になります。Knowledge に AI が書き込むノートには source: claude-code のような frontmatter を必須にすると、後で「これは AI 生成か、自分が書いたか」を区別できる。

Obsidian CLI で検索速度を実用化

Obsidian は 1.12 以降で CLI を提供していて、巨大な Vault でも数百ミリ秒で検索が返ります。ある実装事例では Obsidian CLI の検索が 0.26 秒で完了したのに対し、同じ Vault に対する grep が 15.6 秒かかったという報告があります。60倍の速度差。

これが効くのは、Claude や Codex が「ナレッジ検索」を頻繁に行うとき。grep ベースだとセッションあたり数十秒のオーバーヘッドが発生しますが、Obsidian CLI 経由なら無視できるレベルに収まる。

統合型と Polaris 型の選択

Step 1 で触れた Polaris 型（AI 生成物を Vault 外に分離）と、本記事の統合型（Knowledge/ に AI 生成物を統合）。どちらを採用するかは、運用者の哲学次第です。

統合型のメリットは、バックリンクが Vault 内で完結し、AI 生成ノートと人間ノートが相互参照できること。統合型のリスクは、AI 生成の不確実情報が Vault 全体の信頼度を下げること。これを抑える仕組みが tier 評価と frontmatter での出典明示です。

NotebookLM をブリッジに使う

トークン消費が気になる場面では、NotebookLM のような RAG（Retrieval Augmented Generation）ツールを Vault と Claude の間に挟む構成も有効です。Vault の内容を NotebookLM にインデックス化し、Claude / Codex は NotebookLM 経由で必要箇所だけ取得、フル Vault を context に乗せない、という構成。

ある実装事例では、QMD（Quarto Markdown）セマンティックサーチを併用することで 60% のトークン削減 を達成しています。

Greg Isenberg の「24/7 Personal OS」

ある起業家の X 投稿に、こんな簡潔な定式化があります。

write everything in markdown / link your notes together so they mirror how your brain actually thinks

何でも markdown で書く、ノート同士をリンクして自分の脳が考える通りに繋ぐ。Obsidian Vault を「24時間動く個人 OS」として設計する発想です。

Claude Code と Codex は、その個人 OS の中で動く「実行プロセス」。Vault がコアメモリ、共通スキルが共有ヘッダー、CLAUDE.md / AGENTS.md が司令塔という階層が完成すると、3ツール統合の骨格が出来上がります。

## 

Step 6: 運用ルールの設計

設計と接続が終わったら、最後は「運用を続ける仕組み」を組み込みます。

3ツール統合は構築した瞬間が一番美しくて、3週間後に荒れる。これを防ぐ運用ルールを最初に決めておきます。

棚卸しの3サイクル

棚卸し対象は3層で、それぞれ別の頻度で回します。月1回の MCP 棚卸しでは cli-vs-mcp-audit スキルで登録 MCP を列挙して CLI 代替候補を判定、2週間に1回のスキル棚卸しでは \_shared-ai/skills/ と各プラグイン配下スキルを確認して発動していないスキルを抽出、四半期の ephemeral クリーンアップでは cache / debug / paste-cache の蓄積を確認して半年以上更新ないキャッシュを削除する流れです。

「やる時間を決めてカレンダーに入れる」のが現実的。/context で MCP セクションが 5% 超えた瞬間も棚卸しトリガーにします。

Claude Code Routines で自動化する

Anthropic が提供する Claude Code Routines は、cron スケジュール（毎日定刻に実行）・API 呼び出し（外部システムからキック）・GitHub Events（PR 作成 / Issue / Push 等で発火）の3トリガーで自動実行を回せます。

月実行制限はプランごとに違って、Pro 5回 / Max 15回 / Team・Enterprise 25回。例えば「毎週月曜朝に MCP 棚卸し → 結果を Vault の Daily/ に保存」のような自動化を組むと、棚卸しが「カレンダーで決める」から「勝手に走る」に変わります。

GitHub Actions との連携

Claude Code は GitHub Actions と公式連携していて、/install-github-app コマンドで 1 分でセットアップ完了 します。PR や Issue に

[@claude](https://x.com/@claude)

とメンションすると、GitHub Actions 上で Claude Code が起動して、リポジトリ全体を読みながら自動修正 PR を作る。

これと Routines を組み合わせると、「人間が寝ている間に AI が PR を作って、朝レビューだけする」運用が現実になります。

Model Empathy ― 同じ系統のモデルで組む

エージェントを並列で動かすとき、組み合わせの選び方で精度が大きく変わります。

ある研究プロジェクトでは、24 時間自律最適化の実験で 「Claude + Claude」のペアが「Claude + GPT」のペアより診断精度が 20% 以上高かった と報告されています。

Model empathy matters. Same-model pairings understand each other's reasoning. The meta-agent knows where the task agent breaks and why.

同じモデル系統だと、互いの推論クセを理解できる。この知見を3ツール統合に応用すると、設計・判断・コンテキスト管理は Claude Code（Opus 系）が担当、計算検証・並列実装・軽量タスクは Codex（GPT-5.5 系）が担当、重要成果物の最終確認は Claude × Claude のペアでクロスチェック、データ大量処理の検証は Codex × Codex のペアで処理、という役割分担になります。

「機能で分けて、検証は同じ系統で揃える」というハイブリッド設計が、精度とコストの両方で効きます。

自動化ピラミッド Lv.1〜6

[

![画像](https://pbs.twimg.com/media/HJcprdNaYAAx2K5?format=jpg&name=medium)







](/obsidianstudio9/article/2060161232360059022/media/2060161181097287680)

3ツール統合をどこまで自動化するかは、段階を持って進めるのが安全です。

1.  手動で Claude / Codex を呼ぶ
    
2.  スキルで定型タスクを自動化
    
3.  サブエージェントで並列実行
    
4.  Routines でスケジュール実行
    
5.  GitHub Actions / Webhooks でイベント駆動
    
6.  AutoAgent 系の自律改善ループ
    

最上位の AutoAgent は、ある研究実装で SpreadsheetBench 96.5%・TerminalBench 55.1% という記録を出しています。両方とも自律最適化エージェントとして1位で、他のエントリは全て人間が設計したもの。

ただし Lv.6 にいきなり行くと壊れます。Lv.2 〜 Lv.3 で安定運用を作り、定常化したものから Lv.4 → Lv.5 → Lv.6 と段階的に上げていく。「最初から Lv.6」が、冒頭で触れた「40 体エージェント崩壊」の典型パターンです。

運用ルールを README に書き込む

最後に、ここまでの運用ルールを \_shared-ai/README.md に書き込みます。棚卸しの3サイクル（MCP / スキル / ephemeral）、Routines の実行スケジュール一覧、Model Empathy 原則（同系統ペアで検証）、自動化ピラミッドの現在位置（Lv.N で運用中）の4項目。

これがあると、半年後の自分（または共同運用者）が、現在地と次の一手を即座に把握できる。3ツール統合は、この運用ルールがあって初めて「使うほど賢くなるシステム」になります。

## 

「使うほど賢くなる」ループ — まとめ

長くなったので、最後にひとつだけ整理させてください。

[

![画像](https://pbs.twimg.com/media/HJcprdMaAAAE_fA?format=jpg&name=medium)







](/obsidianstudio9/article/2060161232360059022/media/2060161181093068800)

3ツール統合の本質は、「使うほど賢くなるシステム」をどう設計するか、です。ここまで書いてきた構造を1枚に圧縮するとこうなります。

Obsidian Vault が中央リポジトリ（Karpathy「Wiki はコードベース」）、Claude Code が設計・判断・コンテキスト管理を担うプログラマー1号、Codex CLI が実装・並列実行・軽量タスクを担うプログラマー2号、\_shared-ai/ が両プログラマーが共有するヘッダーファイル群、CLAUDE.md / AGENTS.md が司令塔（80% planning, 20% execution）、共通スキル4本が品質・棚卸し・検証・引き継ぎを支える基盤、運用ルール3サイクル（月次MCP / 隔週スキル / 四半期 ephemeral）で維持する。

この階層が組み上がると、スキルを1本追加するだけで Claude と Codex の両方がその瞬間からそのスキルを使える。Vault に書いたノートが両プログラマーの共有知識として瞬時に参照される。CLAUDE.md を更新すると AGENTS.md 側の共通部分も連動して進化する。

「3ツールを別々に育てる」運用から、「1つの共通基盤を育てて、両ツールが共有する」運用に切り替わる。

これが「組織の通貨」という表現の意味です。スキルもナレッジも運用ルールも、ツール単体の資産ではなく、組織（個人を含む）全体の資産として流通する。

ある OSS リポジトリのコンセプトに「stop re-deriving, start compiling」という言葉があります。「毎回再導出するのをやめて、コンパイル済みの記憶を使い始める」。3ツール統合の到達点はここで、これは Karpathy が LLM Wiki で書いていた「Knowledge is compiled once and kept current, not re-derived on every query」と同じ思想です。

「メモを溜めるだけ」を卒業する

ここまで読んでいただいた方には、もう1段先の景色が見えていると思います。

「3ツールを併用するけど、設定はバラバラで、ナレッジも分散している」状態を卒業する。3ツールを「1つの共通基盤の上で動く実行プロセス」として組み直す。

Vault は「メモを溜める場所」ではなく「両プログラマーの中央リポジトリ」。共通スキルは「Claude のスキル」「Codex のスキル」ではなく「両者が読む共有ヘッダー」。CLAUDE.md / AGENTS.md は「2つの司令塔」ではなく「同じ思想の別実装」。

ここまで来ると、ツール選定の悩み（Claude か Codex か）が消えて、「両方を最大限使い倒す」運用に着地できます。

今すぐ試す1歩目

今日からできる第1歩は、これだけです。

1.  クラウドドライブ配下に \_shared-ai/skills/ を作る
    
2.  bestpra-checker/SKILL.md を1本だけ書く
    
3.  junction で ~/.codex/skills/shared/ に接続する
    

15分で終わります。これだけで、Claude Code と Codex の両方から「公式ベスプラ準拠チェック」が呼べるようになる。

ここから2週目に cli-vs-mcp-audit、3週目に factcheck-ai-cross、4週目に codex-handoff を順に追加する。1ヶ月で共通スキル4本が揃って、3ツール統合の骨格が出来上がります。

## 

この記事が少しでも参考になった方へ

ここまで読んでいただいた方へ、最後に少しだけ。

[

![画像](https://pbs.twimg.com/media/HJcprdPacAAjzhW?format=jpg&name=medium)







](/obsidianstudio9/article/2060161232360059022/media/2060161181105680384)

このアカウント

[@obsidianstudio9](https://x.com/@obsidianstudio9)

（りゅう

[@Obsidian](https://x.com/@Obsidian)

ガチ勢） では、Obsidian で 「第2の脳」 を構築するためのナレッジ設計を毎日発信しています

表層ではなく、本質にフォーカスした情報を届けています。

中の人はコンテキスト総量 100 億字規模で Obsidian を運用していて、その実運用から得た知見を翻訳して発信中。思考と情報を永久にローカル資産化したい方はフォロー推奨です👆

普段の発信内容はこんな感じです。

-   Obsidian × AI の実践的構築法
    
-   海外 PKM 最新手法の日本語解説
    
-   プラグイン活用ノウハウ
    
-   Karpathy 式ナレッジベース解説
    

「メモを溜めるだけ」で終わらせない。「使えば使うほど賢くなるナレッジシステム」の作り方を、海外1次情報ベースで日本語で展開していきます。

ご関心のある方は、ぜひフォローしてチェックしてみてください👀 有益だと思います。

## 

参考・引用

-   \[Andrej Karpathy / GitHub Gist\] (2026-04) \[LLM Wiki\](
    
    [https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)
    
    ) — Obsidian × LLM で「Wiki はコードベース」の比喩を提唱、本記事の理論的支柱
    

