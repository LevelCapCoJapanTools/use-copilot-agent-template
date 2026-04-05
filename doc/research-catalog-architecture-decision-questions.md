# SSOT / Skills / MCP カタログ細分化における設計判断用の質問集

このドキュメントは、`docs/ssot-distribution-system-spec-final-2026-04-04.md` と Issue コメントで示された 12 分割案を突き合わせ、DESIGN Issue に渡す前に決めるべき質問を整理したものである。

## 用語の前置き

- `ssot-catalog`: `docs/ssot-distribution-system-spec-final-2026-04-04.md` で使われている既存の SSOT catalog repo の呼び名を指す。今回の論点は、この既存 repo を `ssot-core` / `ssot-schema` / `ssot-policies` に細分化するかどうかである。
- 上書き競合: 複数の catalog repo が同じ target path へ配布を試み、後から処理された内容が先の内容を上書きしてしまう状態を指す。
- 索引リポジトリ: target repo へ直接コピーする実体ではなく、一覧、互換表、推奨組み合わせのような判断材料を管理する repo を指す。

## 質問1: `ssot-core` / `ssot-schema` / `ssot-policies` を本当に物理 repo として分けるか

### 未確定事項

#### 問題
今の仕様では SSOT は `ssot-catalog` の中でセットごとにまとめる前提になっている。  
一方で 12 分割案では `ssot-core`、`ssot-schema`、`ssot-policies` を別 repo にしたい。  
でも SSOT は `AGENTS.md`、`.github/copilot-instructions.md`、`.github/instructions/**` のように、最終的に target repo のルート直下や `.github/` 配下へ置きたいファイルが多い。  
このまま 3 つに分けると、複数の repo が同じ target path へ配布を試みるため、後勝ちで上書きされる可能性がある。

#### 何が何に影響するか
この判断は `ssot-sync-controller` の単純さと、target repo の `.github/` 直下の安定性に影響する。  
もし `ssot-core` と `ssot-schema` が同じパスへ配ると、後勝ちで上書きされ、SSOT の「単一真実源」が壊れる。  
逆に 1 repo の中で論理分割だけにすると、配布方式は今のまま保ちやすい。

### 選択肢

| 選択肢 | 内容 | 特徴 |
| --- | --- | --- |
| ① SSOT は 1 repo のままにし、repo 内を `core/` `schema/` `policies/` に論理分割する（推奨） | 物理 repo は増やさず、SSOT の中だけを整理する | 今の配布仕様と衝突しにくい |
| ② `ssot-core` / `ssot-schema` / `ssot-policies` を別 repo にする | 役割ごとに完全分離する | 同一パス衝突と、複数 repo のバージョン互換性管理が複雑になりやすい |
| ③ `ssot-core` だけ配布 repo にして、`ssot-schema` と `ssot-policies` は参照専用 repo にする | 一部だけ物理分割する | 折衷案だが、運用ルールがやや複雑になる |

#### 推奨
① SSOT は 1 repo のままにし、repo 内を `core/` `schema/` `policies/` に論理分割する

##### 理由
- 現行仕様の「catalog path をそのまま target repo へコピーする」と最も整合する。
- `ssot-sync-controller` に新しい依存解決や衝突検知を持ち込まずに済む。
- SSOT は全 AI 共通の基盤なので、同じ target path を複数 repo から配らない構造を優先した方が安全である。
- 1 repo 内の論理分割なら、repo 間の上書き競合、つまり後から来た配布内容が先の内容を消してしまう状態を避けやすい。

### 回答

② `ssot-core` / `ssot-schema` / `ssot-policies` を別 repo にする

`セット = 複数repoの束`ではなくて
`構成 = 複数の「軸（レイヤ）」の組み合わせ`に変更する。

セットで用途は決めるが、その中でも分類を組み合わせるようにしたい、原則が違うでしょ？それでSSOTやスキルの運用も違うでしょ？
ssot-core : react-php(言語や構成、php-react, iphoneappなど)
ssot-schema: agile(コーディング運用の方法、agile,pmbokなど)
ssot-policies: rentalserver (展開先やサーバーポリシーなど、レンタルサーバー、サーバーA、自宅サーバーなど)

スキルも外向きと内向きで対応が違う。共通で使わなければならないものがある。

例えばで聞いてほしいけど今はこんな感じ（★そのまま採用するな！）

``` markdown
/// .github/copilot/00-index.md
## 参照順（優先度順）
※ 構成定義レイヤはリポジトリ全体の前提となるため最初に参照してください。番号は通常の昇順で付与しています。

1. [構成定義レイヤ](05-structure/monorepo.md) — モノレポ運用ルール
2. [copilot-instructions.md](../copilot-instructions.md) — 規範層（短く強いルール）
3. [.github/instructions/**/*.instructions.md](../instructions) — 補助的な設計/背景資料レイヤ（`applyTo` は適用範囲を示すメタ情報）
4. [10-requirements.md](10-requirements.md) — 要件とスコープ/受入条件
5. [20-architecture.md](20-architecture.md) — 設計方針・責務分担
6. [30-coding-standards.md](30-coding-standards.md) — コーディング規約
7. [40-testing-strategy.md](40-testing-strategy.md) — テスト戦略
8. [50-security.md](50-security.md) — セキュリティ要求
9. [60-ci-quality-gates.md](60-ci-quality-gates.md) — CI 品質ゲート
10. [70-adr/](70-adr/) — 重要判断の履歴
11. [80-templates/](80-templates/) — plan / PR / review のテンプレート
12. [plans/](plans/) — フェーズ別の実装/設計計画（例: 33-implementation-plan.md）
```

ssot-schemaは(../instructions)を担当したり
ssot-policiesは40-testing-strategy.mdや50-security.mdがset指定で変更できたりするわけ
その他はssot-core集約など
★これはあくまであんな訳

```
SSOT = 3つの独立した軸

① 構成（技術・言語）
② 運用（開発プロセス）
③ 環境（制約・ポリシー）
```

---

## 質問2: catalog の物理分割ルールを「役割単位」に広げるか、「path / 権限境界単位」に限定するか

### 未確定事項

#### 問題
今の仕様では、Skills と MCP は「target repo で置き場所が違うなら物理 repo を分ける」と決めている。  
12 分割案では、置き場所が同じでも「役割が違うから分ける」という考え方が入っている。  
つまり「ファイルをどこへ置くか」で分けるのか、「何の仕事か」で分けるのか、分割基準が 2 つ存在している。

#### 何が何に影響するか
この判断は `skills-core`、`skills-provider`、`mcp-tools` などの repo 数だけでなく、衝突の起きやすさにも影響する。  
role 単位で細かく分けても、target repo の同じパスへ置くなら結局ぶつかる。  
逆に path / 権限境界を主ルールにすると、repo 数は増えすぎず、コピー規則も説明しやすい。

### 選択肢

| 選択肢 | 内容 | 特徴 |
| --- | --- | --- |
| ① 物理分割の主基準を `path` と `権限境界` に限定し、役割差は repo 内ディレクトリや metadata で表す（推奨） | 役割だけでは repo を増やさない | 衝突しにくく、現仕様と相性が良い |
| ② 物理分割の主基準を役割単位に切り替える | `skills-core` / `skills-provider` / `skills-domain` のように細かく分ける | 責務は見やすいが、同一パス衝突が起きやすい |
| ③ path 単位と役割単位を案件ごとに選べるようにする | チームごとに判断を変える | 柔軟だが SSOT としてはぶれやすい |

#### 推奨
① 物理分割の主基準を `path` と `権限境界` に限定し、役割差は repo 内ディレクトリや metadata で表す

##### 理由
- 現行仕様の「path 差分がある場合は物理 repo を分離する必要がある」という前提をそのまま使える。
- 「同じ path に届くなら同じ配送単位にまとめる」という方が、同一パスへの複数配布による上書き競合を防ぎやすい。
- 役割の見通しは `catalog.yml` や README などの metadata で補えるため、物理分割に無理に載せる必要がない。

### 回答

## 回答

本件の分割基準については、以下の通り整理する。


### 結論

物理リポジトリの分割基準は **`path / 権限境界` を主とする**。  
役割差による分割は **補助的表現（ディレクトリ・metadata）に限定し、主基準にはしない**。


### 理由

#### 1. 本システムは衝突を仕様として許容しているため

本設計では以下が前提である：

- 同一パスへの出力は許容される
- 衝突時は後勝ちで上書きされる
- ログを出力し、処理は継続される

したがって、リポジトリ分割によって衝突を防ぐことは設計上の主目的ではない。


#### 2. 役割単位での分割は、最終配置パスと独立であり一貫性を持たないため

役割（skills-core / skills-provider など）は論理的分類であり、  
最終的にどのパスへ配置されるかとは直接対応しない。

そのため：

- 同一パスへ複数の役割から出力される可能性がある
- リポジトリ分割と実行結果の関係が直感的でなくなる

#### 3. path / 権限境界は物理配置と直接対応するため

path および権限境界を基準とすることで：

- 「どこに配置されるか」と「どのリポジトリか」が一致する
- 配布単位が明確になる
- CI / 権限管理との整合性が取れる

#### 4. 役割の可読性は他の手段で担保可能

役割の違いは以下で十分に表現できる：

- ディレクトリ構造（例：`skills/core/`, `skills/provider/`）
- catalog.yml の metadata
- README 等のドキュメント

したがって、物理分割に役割を持ち込む必要はない。

### 補足（重要）

本結論は以下を意味する：

- 役割単位でのリポジトリ分割を**禁止するものではない**
- ただし、それは主基準ではなく、必要な場合に限定される

また、以下は引き続き成立する：

- カタログ設計により衝突を回避することは望ましい
- 衝突した場合でも、仕様として処理は継続される

### 最終結論

```text
リポジトリ分割は path / 権限境界を主基準とする。
役割差は物理分割ではなく、ディレクトリ構造または metadata で表現する。
```

---

## 質問3: `skills-provider` と `adapter-layer` の境界をどこで切るか

### 未確定事項

#### 問題
`skills-provider` は AWS / Azure / Sakura の差分を扱いたい。  
`adapter-layer` は OS や実行環境の差分を吸収したい。  
でも実際には「Sakura 上で Bash と PowerShell の違いを吸収してデプロイする」ような場面では、provider 差分と adapter 差分が混ざりやすい。  
この 2 つの責務領域の境目があいまいだと、同じ処理が別 repo に重複する。

#### 何が何に影響するか
この判断は `skills-provider`、`adapter-layer`、`infra-runtime` の責務分担に影響する。  
境界があいまいだと、同じ AWS 用の処理が skill と adapter に二重実装される。  
逆に境界を決めれば、「何を呼び出すか」は skill、「どう実行差分を吸収するか」は adapter と整理しやすくなる。

### 選択肢

| 選択肢 | 内容 | 特徴 |
| --- | --- | --- |
| ① provider 固有の業務操作は `skills-provider`、OS / shell / path 差分は `adapter-layer` に限定する（推奨） | 「何をするか」と「どう吸収するか」を分ける | 重複しにくい |
| ② provider 差分も adapter-layer に寄せる | 環境差分を 1 か所へ集める | adapter が肥大化しやすい |
| ③ adapter-layer をなくし、全部 `skills-provider` に寄せる | repo 数を減らせる | portability が落ち、再利用しにくい |

#### 推奨
① provider 固有の業務操作は `skills-provider`、OS / shell / path 差分は `adapter-layer` に限定する

##### 理由
- `skills-provider` は「AWS にデプロイする」「Azure の設定を読む」といった利用者に見える能力として定義しやすい。
- `adapter-layer` は Bash / PowerShell、Linux / Windows、ファイルパス差分のような下位吸収に限定した方が再利用性が高い。
- この分け方なら依存方向を `skill -> adapter` に固定でき、逆流しにくい。

## 回答

① provider 固有の業務操作は `skills-provider`、OS / shell / path 差分は `adapter-layer` に限定する

---

## 質問4: `skills-*` と `mcp-tools` の境界をどう定義するか

### 未確定事項

#### 問題
12 分割案では `skills-core` は「複数の tool を組み合わせた業務レベルの能力」、`mcp-tools` は「単一の基本操作を提供する tool 定義」と読める。  
でも実際には `shell`、`file`、`http` のような tool を組み合わせると、そのまま skill のように見える。  
境界がないままだと、「ファイルを読む」は skill なのか tool なのか、「AWS の状態確認」は skill なのか tool なのかが毎回ぶれる。

#### 何が何に影響するか
この判断は `mcp-server-core` の API 設計と、`skills-core` の再利用単位に影響する。  
もし `mcp-tools` が高級機能まで持つと、skill は薄いラッパーになり、責務が逆転する。  
逆に `skills-*` を業務単位、`mcp-tools` を基本操作単位に固定すれば、依存方向をきれいに保てる。

### 選択肢

| 選択肢 | 内容 | 特徴 |
| --- | --- | --- |
| ① `mcp-tools` は基本操作、`skills-*` は複数 tool を束ねた利用者向け能力に限定する（推奨） | 低レベルと高レベルを分離する | 依存方向が明確 |
| ② provider ごとの操作も `mcp-tools` に含める | tool 側に機能を寄せる | skill の存在意義が薄くなる |
| ③ `skills-*` をなくして `mcp-tools` だけで表現する | 構造は単純になる | 使い方の意味づけが弱くなる |

#### 推奨
① `mcp-tools` は基本操作、`skills-*` は複数 tool を束ねた利用者向け能力に限定する

##### 理由
- `mcp-server-core` は「安全に呼び出せる部品」を提供し、skill は「部品の使い方」を表す方が自然である。
- この分割なら `mcp-tools -> adapter-layer`、`skills-* -> mcp-tools` の依存方向を崩しにくい。
- issue で求められている「実行時責務と定義時責務の分離」にも合う。

## 回答

① `mcp-tools` は基本操作、`skills-*` は複数 tool を束ねた利用者向け能力に限定する

---

## 質問5: `infra-runtime` を配布対象に含めるなら、何まで入れてよいか

### 未確定事項

#### 問題
`infra-runtime` は「実行環境・デプロイ・secret」を扱う想定になっている。  
でも今の配布方式は catalog の中身をそのまま target repo にコピーする方式である。  
この方式で secret まで repo に配るのは危険で、そもそもやってはいけない。  
公開 repo や共有 repo なら機密情報が見えてしまうし、Git 履歴に残ると後から完全に消すのも難しい。  
つまり `infra-runtime` に入れてよいものと、絶対に入れてはいけないものを先に決める必要がある。

#### 何が何に影響するか
この判断は `infra-runtime` と `target repo` のセキュリティ境界に影響する。  
境界がないと、secret、認証情報、環境依存の実値が catalog に混ざる危険がある。  
逆に「配ってよいのはテンプレートや非機密設定だけ」と決めれば、今の pull 型でも安全に運用しやすい。

### 選択肢

| 選択肢 | 内容 | 特徴 |
| --- | --- | --- |
| ① `infra-runtime` には非機密の runtime 定義、deploy 手順、IaC テンプレートだけを入れ、secret 実値は対象外にする（推奨） | 配布可能なものを限定する | 現行方式でも安全性を保ちやすい |
| ② `infra-runtime` に secret 参照定義と実値の両方を持たせる | repo 単体で完結させる | セキュリティ上の危険が大きい |
| ③ `infra-runtime` は配布対象から外し、別管理にする | catalog を軽くできる | version 管理が分裂しやすい |

#### 推奨
① `infra-runtime` には非機密の runtime 定義、deploy 手順、IaC テンプレートだけを入れ、secret 実値は対象外にする

##### 理由
- 現行の「catalog path をそのまま target repo へコピーする」方式でも、非機密設定なら安全に配布できる。
- Secret 実値は `infra-runtime` に入れず、Vault / GitHub Secrets / 環境変数など別経路で注入する方が責務が明確である。
- `infra-runtime` を完全に除外すると、deploy 定義だけ version 固定の枠外に落ちるため、管理がぶれやすい。

---

## 質問6: `skill-catalog` は配布 repo なのか、設計用のメタレジストリなのか

### 未確定事項

#### 問題
`skill-catalog` は「スキル一覧・依存・version 管理」と書かれている。  
でも今の仕様で配布されるのは「target repo にコピーされる実体ファイル」である。  
`skill-catalog` が実体ファイルを配らないなら、それは catalog というより「索引」や「台帳」に近い。  
ここを決めないと、`ssot-bot.yml` に `skill-catalog/...` を書くべきかどうかも決まらない。

#### 何が何に影響するか
この判断は `ssot-bot.yml` の書き方と、`ssot-sync-controller` の責務に影響する。  
もし `skill-catalog` を配布対象にすると、controller が追加の特別処理を持つ可能性が高い。  
逆に metadata 専用と決めれば、実行時の処理は増えず、DESIGN 時の判断材料として使える。

### 選択肢

| 選択肢 | 内容 | 特徴 |
| --- | --- | --- |
| ① `skill-catalog` は metadata 専用の索引リポジトリにし、target repo へは配らない（推奨） | 設計判断の索引として使う | 実行時責務を増やさない |
| ② `skill-catalog` を配布 repo として扱い、複合セットを直接 target repo に配る | 入口を 1 つにまとめやすい | セット依存に近い処理が必要になる |
| ③ `skill-catalog` を廃止し、情報を `skills-core` に寄せる | 構成は減る | `skills-core` の責務が太くなる |

#### 推奨
① `skill-catalog` は metadata 専用の索引リポジトリにし、target repo へは配らない

##### 理由
- `skill-catalog` を配布対象にしないことで、`ssot-sync-controller` に新しい解決ロジックを足さずに済む。
- version 互換表や推奨セット一覧は、DESIGN Issue の判断材料として持つ方が使いやすい。
- 「実行するもの」と「選ぶための情報」を分けることで、実行時責務と定義時責務を分離できる。
- もし metadata 専用に寄せるなら、命名変更は別の設計判断事項として切り出し、`skill-catalog` のまま続けるかを別途決めた方が論点を混ぜにくい。

## 回答

① `skill-catalog` は metadata 専用の索引リポジトリにし、target repo へは配らない

```
flowchart TD
  A[User Input]
  B[Planner]
  C[Skill Catalog]
  D[Plan]
  E[Executor]
  F[MCP]
  
  A --> B
  B --> C
  C --> B
  B --> D
  D --> E
  E --> F
```

---

## 質問7: include 基準を `set.yml` 相対に統一するか、層ごとに別ルールを残すか

### 未確定事項

#### 問題
今の仕様では SSOT は `set.yml` からの相対パス、Skills / MCP は catalog ルート基準になっている。  
これだけでも少し覚えにくいのに、12 分割で repo が増えると、どの repo がどの書き方かを毎回考えることになる。  
パス解決の基準が repo ごとに異なる状態なので、書く人が間違えやすい。

#### 何が何に影響するか
この判断は set 定義を書く人のミス率と、`ssot-sync-controller` の include 展開実装に影響する。  
基準がばらばらだと、同じ `include:` を書いても repo ごとに意味が変わる。  
逆に 1 つに統一すると、説明も実装もかなり単純になる。

### 選択肢

| 選択肢 | 内容 | 特徴 |
| --- | --- | --- |
| ① すべての catalog を `set.yml` からの相対パスに統一する（推奨） | ルールを 1 つにそろえる | 書き方が分かりやすい |
| ② すべての catalog を repo ルート基準に統一する | Skills / MCP に合わせる | SSOT の既存説明を直す必要がある |
| ③ 層ごとに別ルールを維持する | 既存仕様を崩さない | 12 分割後の認知負荷が高い |

#### 推奨
① すべての catalog を `set.yml` からの相対パスに統一する

##### 理由
- set 定義ファイルを動かすときに、そのファイル自身を基準に読めるため、repo 分割後も理解しやすい。
- `ssot-sync-controller` の include 展開ロジックを 1 パターンに寄せられる。
- `ssot-core`、`mcp-tools`、`infra-runtime` のように repo が増えても、書き方が変わらない。

### 回答

③ 層ごとに別ルールを維持する

---

## 質問8: 複数 repo の version 組み合わせ表を誰が持つか

### 未確定事項

#### 問題
12 分割にすると、たとえば React 系の target repo で `ssot-core`、`skills-core`、`mcp-tools`、`infra-runtime` の組み合わせが必要になる。  
でも今の仕様は「セット依存なし」で、`ssot-sync-controller` は単純に `use:` を順に読むだけである。  
このままだと「どの version の組み合わせが安全か」を誰も持たないまま、target repo 側の人が手でそろえることになる。

#### 何が何に影響するか
この判断は `ssot-bot.yml` の運用負荷と、障害時の切り戻しのしやすさに影響する。  
互換表がないと、`skills-provider` だけ更新して `mcp-tools` と合わなくなる事故が起きやすい。  
逆に互換表を 1 か所に持てば、pull 型と version 固定を保ったまま、変更レビューがしやすくなる。

### 選択肢

| 選択肢 | 内容 | 特徴 |
| --- | --- | --- |
| ① `skill-catalog` などの metadata repo に「推奨 version tuple」を持たせ、DESIGN 時に参照する（推奨） | 実行時ではなく設計時に整合を確認する | 現行 controller を大きく変えない |
| ② `ssot-sync-controller` に互換性チェック機能を追加する | 実行時に不整合を止められる | 依存解決機能が肥大化しやすい |
| ③ target repo 側で完全手動管理にする | 実装変更は少ない | 利用者の負担が大きい |

#### 推奨
① `skill-catalog` などの metadata repo に「推奨 version tuple」を持たせ、DESIGN 時に参照する

##### 理由
- `ssot-sync-controller` の責務を「コピーと差分生成」に留めやすい。
- version 固定と手動 rollback の方針を崩さずに、判断材料だけを整備できる。
- 配布時ではなく設計時に整合性を見ることで、実行時の複雑化を避けられる。

### 回答

③ target repo 側で完全手動管理にする

---

## 質問9: `allowed-paths.yml` の変更権限をどこに置くか

### 未確定事項

#### 問題
今の仕様書では、書き込める path の一覧は `ssot-sync-controller/config/allowed-paths.yml` に集約されている。  
12 分割で repo が増えると、新しい catalog が新しい path を使いたい場面が増える。  
ここで「各 catalog が自分で allowed path を増やせる」のか、「controller 側でだけ増やせる」のかを決めないと、セキュリティ境界がぶれる。

#### 何が何に影響するか
この判断は `ssot-sync-controller` の安全装置と、catalog 追加時の手間に影響する。  
catalog 側で自由に path を増やせると、危険な場所まで配布範囲が広がる可能性がある。  
逆に controller 側に集約すれば手続きは増えるが、「どこに書いてよいか」を中央で止められる。

### 選択肢

| 選択肢 | 内容 | 特徴 |
| --- | --- | --- |
| ① `allowed-paths.yml` は `ssot-sync-controller` に中央集約し続ける（推奨） | path 許可は中央承認にする | セキュリティ境界が明確 |
| ② 各 catalog repo に allowed path 定義を置く | catalog 単位で自己完結できる | 許可の一貫性が崩れやすい |
| ③ 中央の禁止リストだけを残し、それ以外は許可する | 追加は簡単 | 安全側に倒しにくい |

#### 推奨
① `allowed-paths.yml` は `ssot-sync-controller` に中央集約し続ける

##### 理由
- path の許可は機能追加ではなくセキュリティ判断なので、catalog 側に自己申告させない方が安全である。
- `ssot-sync-controller` が「どこに書けるか」の最終門番である方が、責務境界が明確になる。
- 新 path 追加時に controller 側 PR が必要でも、危険な path 拡張を防げるメリットの方が大きい。

### 回答

① `allowed-paths.yml` は `ssot-sync-controller` に中央集約し続ける

---

**追加調査で見つかった未確定事項**

## 質問10: `ssot-core` / `ssot-schema` / `ssot-policies` の3軸が同じ target path に出力するとき、何を優先するか

かんたんに言うと、**3つの repo が同じファイルに書こうとしたとき、だれの内容を採用するのか**を決める質問である。

### 未確定事項

#### 問題
質問1の回答セクションでは、推奨案とは別の選択肢として、SSOT を `ssot-core` / `ssot-schema` / `ssot-policies` の3軸に分ける案が示されている。  
一方で質問2の回答セクションでは、物理 repo 分割の主基準は `path / 権限境界` だとしている。  
この「質問1の回答セクションにある3軸案」と「質問2の回答セクション」を同時に採用すると、たとえば `.github/copilot/00-index.md`、`.github/instructions/**`、`40-testing-strategy.md`、`50-security.md` のような同一 target path に対して、複数軸から配布する場面が発生しうる。  
そのとき「どの軸が優先されるか」が未定義である。

#### 何が何に影響するか
この判断は `ssot-sync-controller` の差分生成順序、PR で見える最終ファイル、target repo 側の理解しやすさに影響する。  
優先順がないまま後勝ちに任せると、`ssot-core` の基底ルールを `ssot-policies` が偶然上書きし、設計意図が見えにくくなる。  
逆に優先ルールや専有 path を決めれば、3軸構成と path 基準の両立条件を明文化できる。

### 選択肢

| 選択肢 | 内容 | 特徴 |
| --- | --- | --- |
| ① 論理分割または物理分割した軸ごとに担当ファイルを分担し、同じ target path へは出力しない（推奨） | `ssot-core` / `ssot-schema` / `ssot-policies` が担当ファイルを分担する | 質問1の推奨案と回答案のどちらでも使える整理になる |
| ② 3軸の優先順を固定し、同一 path ではその順で後勝ちにする | 例: `core -> schema -> policies` | 実装は簡単だが、暗黙の上書きが増える |
| ③ target repo 側で merge する前提にし、catalog は部分ファイルだけを持つ | controller は断片を集めるだけにする | 仕組みが大きく変わり、現行仕様から離れる |

#### 推奨
① 論理分割または物理分割した軸ごとに担当ファイルを分担し、同じ target path へは出力しない

##### 理由
- 質問1の「3軸を独立した軸として扱う」と、質問2の「path / 権限境界を主基準にする」を両立しやすい。
- `ssot-sync-controller` に新しい merge ロジックを持ち込まずに済む。
- 後勝ちを事故対応の最後の逃げ道に留め、平常運用では上書き競合を起こさない設計へ寄せられる。

### 回答

① 論理分割または物理分割した軸ごとに担当ファイルを分担し、同じ target path へは出力しない

---

## 質問11: `ssot-schema` と `ssot-policies` は、`00-index.md` や `40-testing-strategy.md` / `50-security.md` をどう変えるのか

かんたんに言うと、**大事なファイルを「まるごと入れ替える」のか、「一部だけ変える」のか**を決める質問である。

### 未確定事項

#### 問題
質問1の回答セクション（選択肢②の説明例）では、`ssot-schema` は `.github/instructions/**` を担当し、`ssot-policies` は `40-testing-strategy.md` や `50-security.md` を set 指定で変えられる想定が示されている。  
しかし、何を「丸ごと差し替える」のか、どこを「選択的に変える」のか、まだ定義されていない。  
たとえば `00-index.md` の参照順や `50-security.md` の内容を変えるなら、target repo 側では「どの set が効いているか」を追跡できる必要がある。

#### 何が何に影響するか
この判断は `ssot-bot.yml` の記法、`ssot-sync-controller` の読み方、設計レビューのしやすさに影響する。  
仕組みが曖昧なままだと、「schema の set を変えたのか」「policies の set を変えたのか」が PR だけでは分かりにくい。  
逆に set の効き方を明文化すれば、3軸の責務と変更範囲をレビューしやすくなる。

### 選択肢

| 選択肢 | 内容 | 特徴 |
| --- | --- | --- |
| ① `ssot-schema` / `ssot-policies` は担当ファイルを丸ごと差し替える単位として扱う（推奨） | set ごとに完成済みファイルを持つ | controller が単純なままで済む |
| ② 1ファイルの一部だけを軸ごとに差し込む | 章や節だけを切り替える | merge 仕様が増え、複雑になる |
| ③ `00-index.md` などの基底ファイルは `ssot-core` 固定にし、schema/policies は周辺ファイルだけ変える | 変更対象を限定する | 安全だが柔軟性は下がる |

#### 推奨
① `ssot-schema` / `ssot-policies` は担当ファイルを丸ごと差し替える単位として扱う

##### 理由
- 現行仕様の「catalog path をそのまま target repo へコピーする」に最も近い。
- 「一部差し込み」を避けることで、どの軸がどのファイルを担当しているかを明確にできる。
- 必要なら質問10の専有 path ルールと組み合わせて、担当範囲をはっきりさせられる。

### 回答

① `ssot-schema` / `ssot-policies` は担当ファイルを丸ごと差し替える単位として扱う

---

## 質問12: `skill-catalog` を配布しない索引リポジトリにするなら、誰がいつ参照するのか

かんたんに言うと、**`skill-catalog` は実行用の部品ではなく、設計のときだけ見る参考書なのか**を決める質問である。

### 未確定事項

#### 問題
質問6の回答では `skill-catalog` は配布しない索引リポジトリとされている。  
一方で質問8には、推奨の「DESIGN 時に推奨 version tuple を参照する」と、別選択肢としての「target repo 側で完全手動管理」が並んでいる。  
この状態だと、`skill-catalog` は存在するが、実際に誰が・いつ・どの手順で使うのかが曖昧になる。  
質問6の回答セクションにある図では Planner が `skill-catalog` を参照しているが、その責務が本文ではまだ固定されていない。

#### 何が何に影響するか
この判断は DESIGN Issue の作り方、Planner の責務、target repo 側の version 更新手順に影響する。  
参照主体が決まらないままだと、`skill-catalog` は「あるが使われない台帳」になりやすい。  
逆に参照タイミングを決めれば、完全手動管理と索引 repo の役割分担を整理できる。

### 選択肢

| 選択肢 | 内容 | 特徴 |
| --- | --- | --- |
| ① Planner / DESIGN フェーズだけが `skill-catalog` を参照し、Executor / controller は参照しない（推奨） | 設計時だけ使う索引にする | 実行時責務を増やさない |
| ② `ssot-sync-controller` が実行時に `skill-catalog` を読んで互換性を判定する | 実行時に不整合を止められる | controller が重くなる |
| ③ `skill-catalog` を廃止し、各 catalog repo の README に推奨組み合わせを書く | 構成は単純になる | 推奨情報が散らばる |

#### 推奨
① Planner / DESIGN フェーズだけが `skill-catalog` を参照し、Executor / controller は参照しない

##### 理由
- 質問6で整理した「実行時責務と定義時責務の分離」に合う。
- 質問8の「target repo 側で手動管理する」とも両立しやすく、手動更新の判断材料として `skill-catalog` を使える。
- `ssot-sync-controller` は現行どおり、指定された version をそのまま配るだけの単純な役割を保てる。

### 回答

回答：

①で確定で良い。そのうえで曖昧になっているポイントは「誰が使うか」ではなく「いつ固定されるか」。

`skill-catalog` は **実行のための仕組みではなく、設計のための判断材料**。
参照主体は明確に **Planner（= DESIGN Issueを作る人 / AI）だけ**。

時系列で整理するとこうなる：

1. DESIGNフェーズ
   Plannerが `skill-catalog` を読む
   → 「安全な version 組み合わせ（tuple）」を決める
   → その結果を DESIGN Issue に書く（ここで確定）

2. IMPLEMENTフェーズ
   target repo 側は DESIGN Issue に書かれた version をそのまま採用
   → `ssot-sync-controller` はそれを配るだけ（何も判断しない）

3. 実行時
   Executor / MCP / controller は **catalogを一切見ない**
   → 既に固定された version を使うだけ

つまり構造はこう：

```text
skill-catalog →（参照するのはここだけ）→ Planner（DESIGN時）
                                               ↓
                                      DESIGN Issueに固定
                                               ↓
target repo / controller / executor（参照しない）
```

ここを曖昧にすると破綻する理由は2つ：

* 実行時にcatalogを見始めると「その場で判断」が発生して決定性が崩れる
* controllerに判定責務を持たせると「配布システム」が「依存解決システム」に変質する

逆にこの設計だと役割が完全に分離される：

* `skill-catalog`：おすすめの組み合わせを提示するだけ（知識）
* Planner：それを見て決める（意思決定）
* DESIGN Issue：決定結果（契約）
* controller：配るだけ（無思考）
* executor：実行するだけ（無思考）

重要なのはここ：

```text
catalogは「選ぶためのもの」であって「動かすためのものではない」
```

したがって、

* 誰が参照するか → Plannerのみ
* いつ参照するか → DESIGN時のみ
* どこに結果を残すか → DESIGN Issue
* 実行系は参照するか → 一切しない

これで責務は完全に固定される。

---

## 質問13: include 基準を層ごとに残すなら、`ssot-sync-controller` の複雑性はどこまで許容するか

かんたんに言うと、**repo ごとに別の読み方を増やしても大丈夫か、それとも増やしすぎない方がよいか**を決める質問である。

### 未確定事項

#### 問題
質問7の選択肢③および回答では、include 基準は「層ごとに別ルールを維持する」となっている。  
しかし質問1の回答案のように SSOT を3軸化する場合は、SSOT 内でも論理的な管理単位が増える。  
この状態で SSOT / Skills / MCP / 追加の索引 repo がそれぞれ別の include 基準を持つと、`ssot-sync-controller` は repo 種別または論理単位ごとに path 解決ルールを持つ必要がある。  
その複雑性をどこまで許容するかが未確定である。

#### 何が何に影響するか
この判断は controller 実装の保守性、設定ミス時の検出しやすさ、今後 repo を増やしたときの説明コストに影響する。  
ルールが増えすぎると、同じ `include:` でも repo によって意味が変わり、利用者が誤記しやすい。  
逆に許容範囲を先に決めれば、「例外を増やすのか」「統一へ寄せるのか」を判断しやすい。

### 選択肢

| 選択肢 | 内容 | 特徴 |
| --- | --- | --- |
| ① include 基準の例外は SSOT と Skills/MCP の2系統までに制限する（推奨） | 追加 repo でも既存2系統のどちらかに寄せる | 複雑性を抑えやすい |
| ② repo ごとに独自基準を持てるようにする | 柔軟性は高い | controller の理解と実装が重くなる |
| ③ 追加 repo を作る前に、include 基準を全面統一する | 仕様を整理し直す | 変更範囲は大きいが最も分かりやすい |

#### 推奨
① include 基準の例外は SSOT と Skills/MCP の2系統までに制限する

##### 理由
- 既存仕様との差分を最小化しつつ、例外の増殖を止められる。
- 質問7の回答を尊重しながらも、controller 実装の複雑化に上限を設けられる。
- DESIGN では「新 repo は既存2系統のどちらへ寄せるか」を判断すればよくなる。

### 回答

① include 基準の例外は SSOT と Skills/MCP の2系統までに制限する

---

## 質問14: `infra-runtime` の「非機密」と `allowed-paths.yml` の中央管理をどう結びつけるか

かんたんに言うと、**配ってよい設定ファイルの見本と、絶対に配ってはいけない本物の値をどう分けるか**を決める質問である。

### 未確定事項

#### 問題
質問5の回答では `infra-runtime` は非機密の runtime 定義だけを配るとしており、質問9の回答では `allowed-paths.yml` は `ssot-sync-controller` に中央集約し続けるとしている。  
しかし、この2つだけでは「何を非機密と判定するのか」「誰が新 path を承認するのか」がまだ不明である。  
たとえば IaC テンプレート、サンプルの環境変数、default 値入り設定ファイルのどこまでを `infra-runtime` に含めてよいかが未定義である。

#### 何が何に影響するか
この判断は security review、catalog 追加フロー、`infra-runtime` の repo 境界に影響する。  
基準がないと、catalog 作成者ごとに「これは非機密」と解釈が分かれ、中央管理の意味が弱くなる。  
逆に判定基準と承認者を決めれば、質問5と質問9の回答を実運用に落とし込みやすい。

### 選択肢

| 選択肢 | 内容 | 特徴 |
| --- | --- | --- |
| ① `infra-runtime` は template / placeholder だけを許可し、実値を含む設定は常に禁止と明文化する（推奨） | allowed-paths の審査基準をはっきりさせる | security review しやすい |
| ② `infra-runtime` の非機密判定は catalog repo ごとの裁量に任せる | 追加は速い | 判定がぶれやすい |
| ③ `infra-runtime` 自体を配布対象から外す | 危険は減る | deploy 定義の version 管理が分裂する |

#### 推奨
① `infra-runtime` は template / placeholder だけを許可し、実値を含む設定は常に禁止と明文化する

##### 理由
- 質問5の「secret 実値は別経路」と、質問9の「中央管理」をつなぐ運用ルールになる。
- `allowed-paths.yml` の承認時に、path だけでなく内容基準も確認しやすくなる。
- `infra-runtime` を残しつつ、機密混入リスクを最小化できる。

### 回答

① `infra-runtime` は template / placeholder だけを許可し、実値を含む設定は常に禁止と明文化する

---

## 今回の回答で新しく見えた追加質問（やさしい言い方）

以下は、今回追記された 12 repo 案、5 つの層、依存関係の整理内容を読んで、新しく見えた質問である。  
言い方をなるべくやさしくしているが、中身は DESIGN で決めるべき重要な論点である。

## 質問15: 12個の repo のうち、どの repo がどのファイルを担当するのか

かんたんに言うと、**「このファイルはだれの仕事か」を最初に決める必要がある**という質問である。

### 未確定事項

#### 問題
今回の回答では、repo を 12 個に細かく分けたい意図がかなりはっきりした。  
ただし、配布方法は変えないので、最終的には target repo の決まった path にファイルを置くことになる。  
このとき、`ssot-core` / `ssot-schema` / `ssot-policies` や `skills-core` / `skills-provider` / `skills-domain` が、**どのファイルを自分の担当にするのか**を先に決めないと、同じ場所に複数 repo が書きにいく危険が残る。

#### 何が何に影響するか
この判断は path 衝突の防止、repo の責務、レビューのしやすさに影響する。  
担当が決まっていないと、「この変更はどの repo で直すべきか」が毎回ぶれる。  
逆に担当表を作れば、「そのファイルはこの repo の責任」とはっきり言える。

### 選択肢

| 選択肢 | 内容 | 特徴 |
| --- | --- | --- |
| ① repo ごとに担当 path 一覧を先に決める（推奨） | 「この repo はここだけ書く」を明文化する | ぶつかりにくい |
| ② 実際に衝突したらその都度決める | 先に決めない | 運用で迷いやすい |
| ③ 後勝ちで吸収する前提にする | 衝突を仕様で飲み込む | 設計意図が見えにくい |

#### 推奨
① repo ごとに担当 path 一覧を先に決める

##### 理由
- 配布方法を変えないなら、最後は「どこへ置くか」が一番大事だからである。
- 12 repo へ細分化しても、担当表があれば責務を追いやすい。
- 後勝ちを通常運転にせず、例外時だけの逃げ道にできる。

## 質問16: version が合っていても、中身の相性が悪いときはどうするのか

かんたんに言うと、**番号は合っていても、書いてある内容がかみ合わないことを誰が見つけるのか**という質問である。

### 未確定事項

#### 問題
今回の回答では、`skill-catalog` は DESIGN 時だけ見る参考情報としてかなり整理された。  
しかし、`ssot-core` が前提にしている技術と、`ssot-schema` の運用ルールや `ssot-policies` の制約が**内容として合っているか**までは、version 番号だけでは分からない。  
たとえば、技術選択は React / PHP なのに、運用ルールが別の前提で書かれていたら、番号が合っていても実際には食い違いが起きる。

#### 何が何に影響するか
この判断は Planner の責務、DESIGN Issue の書き方、レビュー時の確認項目に影響する。  
ここを決めないと、「version は正しいのに内容が食い違う」状態を見落としやすい。  
逆に確認方法を決めれば、DESIGN の段階で危ない組み合わせを止めやすい。

### 選択肢

| 選択肢 | 内容 | 特徴 |
| --- | --- | --- |
| ① Planner が DESIGN 時に内容の相性も確認する（推奨） | version だけでなく意味も見る | 実行時を単純に保てる |
| ② controller が実行時に内容まで検査する | 配る直前に止める | controller が重くなる |
| ③ version だけ合っていればよいとする | 内容確認をしない | 食い違いを見逃しやすい |

#### 推奨
① Planner が DESIGN 時に内容の相性も確認する

##### 理由
- `skill-catalog` を設計用の参考情報にした考え方と合う。
- controller を「配るだけ」の役割に保ちやすい。
- 実行前ではなく DESIGN の段階で直せるため、手戻りが少ない。

## 質問17: みんなが読む共通ファイルの最終版は、だれが決めるのか

かんたんに言うと、**`00-index.md` のように全体に関わるファイルは、最後にだれが責任を持つのか**という質問である。

### 未確定事項

#### 問題
今回のメモでは、12 repo を 5 層に分け、依存の流れも整理されている。  
しかし、`00-index.md` のように「全体の入口」になるファイルや、複数層が前提にする共通ファイルは、1つの repo だけでは決めにくい可能性がある。  
ここで最終責任者がいないと、「みんな少しずつ関係あるが、だれも最後に決めない」状態になりやすい。

#### 何が何に影響するか
この判断は SSOT の入口設計、レビューの責任範囲、変更時の合意手順に影響する。  
最終責任者が曖昧だと、共通ファイルの変更が止まりやすい。  
逆に責任者を決めれば、12 repo に細分化しても入口の整合を保ちやすい。

### 選択肢

| 選択肢 | 内容 | 特徴 |
| --- | --- | --- |
| ① 共通ファイルは `ssot-core` を最終責任者にする（推奨） | 入口の責任を 1 か所へ寄せる | 分かりやすい |
| ② ファイルごとに責任 repo を別途決める | 柔軟に分担する | 管理表が必要になる |
| ③ 関係する repo の合議で毎回決める | 全員で決める | 遅くなりやすい |

#### 推奨
① 共通ファイルは `ssot-core` を最終責任者にする

##### 理由
- `ssot-core` は「全 AI の単一真実源」という役割と相性がよい。
- 入口ファイルの最終責任者が 1 つだと、変更判断がぶれにくい。
- 他の repo は周辺ルールを担当し、入口は core がまとめる形にしやすい。
