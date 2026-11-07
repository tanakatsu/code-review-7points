---
name: code-review-7points
description: >
  コード変更をレビューし、設計・理解容易性・命名・コードスタイル・機能/性能・テスト・ドキュメントの
  7観点から、重要度付きで具体的な改善点を指摘する。PRレビュー、差分レビュー、実装レビュー、
  リファクタリングレビュー、テストレビューを依頼されたときに使用する。
---

# Code Review Skill

## 目的

コード変更をレビューし、実際に問題となる箇所を重要度付きで具体的に指摘する。

レビューの目的は指摘数を増やすことではない。
変更によって導入・顕在化する現実的な問題を高い精度で見つけ、
誤検知や根拠の弱い指摘をできるだけ除外する。

レビュー基準そのものはこのファイルに重複して記載せず、
必ず `references/code_review_7points.md` を参照する。

このSkillはレビューを行うものであり、ユーザーから明示的に依頼されない限りコードを修正しない。

## Reference

レビュー開始時に、必ず以下を読む。

- `references/code_review_7points.md`

このReferenceをレビュー基準の一次情報として扱う。

Referenceに定義された7観点:

1. 設計
2. 理解容易性
3. 命名
4. コードスタイル
5. 機能・性能
6. テスト
7. ドキュメント

Referenceの全項目を機械的に列挙するのではなく、
変更内容に関連する観点を選び、コード上で実際に問題が確認できるものを指摘する。

## 基本方針

- correctness、security、data integrity、regressionを最優先する
- 指摘数よりsignalの高さを優先する
- 指摘にはコード上の具体的な根拠を示す
- 問題の発生条件と影響を説明できるものを優先する
- 推測と確認済み事実を区別する
- 好みの違いだけの指摘を避ける
- 将来の仮定だけに基づく過剰な抽象化を要求しない
- 変更と無関係な既存問題は原則として指摘しない
- 問題カテゴリだけから重大度を決めない
- 問題がなければ無理に指摘を作らない
- 不確実な指摘を大量に出すより、少数の確実な指摘を優先する

## レビュー対象の把握

最初に、レビュー対象と変更の意図を把握する。

diffだけで判断せず、必要に応じて以下を確認する。

- PR description
- issue / task description
- commit message
- 関連する既存コード
- 関連テスト
- README / API仕様 / ADRなどの関連ドキュメント
- CLAUDE.md、AGENTS.md、CONTRIBUTING.mdなどのプロジェクト固有ルール

確認範囲は変更内容との関連性に応じて判断する。
レビューの妥当性に必要な情報を省略してはならない。

必要な情報を確認できない場合は推測で補わず、
その制約をレビュー結果に明示する。

PRやdiffが与えられている場合は変更箇所を中心にレビューする。
ただし、変更による不整合や回帰を判断するために必要な周辺コードは確認する。

## レビュー手順

### 1. Intent / Scopeを理解する

変更によって何を実現しようとしているかを把握する。

例:

- 新機能
- バグ修正
- リファクタリング
- 性能改善
- テスト追加
- 設定変更
- dependency update
- migration

可能であれば次を整理する。

- 何が変わるべきか
- 何が変わってはいけないか
- どの入力・状態・利用者が影響を受けるか
- どのコンポーネントやデータが影響を受けるか

変更目的が不明な場合は、推測を事実として扱わない。
必要なら「意図が〜であれば」と条件付きで評価する。

変更目的と実際のdiffを比較し、以下を確認する。

#### Missing implementation

要求された振る舞いの一部が実装されていない、
または必要な関連箇所の変更が漏れていないか。

#### Scope creep

変更目的と直接関係しない変更、
不要なリファクタリング、
意図しない振る舞い変更が混入していないか。

#### Partial update

同じ契約・データ・API・状態を扱う複数箇所のうち、
一部だけ変更されて不整合になっていないか。

Scope上の問題も、
具体的な根拠・発生条件・影響を説明できる場合のみFindingとする。

### 2. Change inventoryとリスクを把握する

変更されたファイルと周辺コードを確認し、
重点的にレビューすべき領域を判断する。

例:

- authentication / authorization
- user-controlled input
- database / transaction
- schema / migration
- concurrency / async processing
- external API
- filesystem
- cache
- background job
- money / billing
- permissions
- serialization
- public API / compatibility
- AI / LLM integration
- test / CI / quality gate

すべての変更にすべての専門チェックを適用しない。
変更が該当するリスク領域を含む場合だけ深掘りする。

### 3. Gate Integrityを確認する

以下のような品質ゲートに関係するファイルが変更されている場合は、
変更によって問題を解決したのではなく、
検出能力や品質基準を弱めていないか確認する。

例:

- tests
- test fixtures
- snapshots / golden files
- CI configuration
- lint configuration
- formatter configuration
- type-check configuration
- coverage configuration
- build configuration

特に以下を確認する。

- assertionが不必要に弱くなっていないか
- テストケースが削除・skipされていないか
- coverage対象が不必要に狭められていないか
- lint / type checkのルールが不必要に無効化されていないか
- snapshot更新によって意図しない変更を承認していないか
- CIから必要な検証が外されていないか

設定変更そのものが変更目的の場合は、その意図を考慮する。

### 4. Referenceに沿ってレビューする

`references/code_review_7points.md` に基づき、
今回の変更に関連する観点を選んで確認する。

特に、次の優先順位で確認する。

1. 正しさ
2. セキュリティ
3. データ整合性
4. エラー処理
5. 性能・リソース
6. 設計・保守性
7. テスト
8. 命名・コードスタイル
9. ドキュメント

7観点はFinding数のノルマではない。
問題が存在しない観点についてFindingを作らない。

### 5. リスクに応じて深掘りする

変更内容が特定のリスク領域に該当する場合のみ、
その領域を重点的に確認する。

#### Security

trust boundaryを特定し、
untrusted inputがsensitive operationへ到達する経路を確認する。

関連する場合は以下を確認する。

- authentication bypass
- authorization / IDOR
- SQL / NoSQL injection
- command injection
- XSS
- SSRF
- path traversal
- secret leakage
- unsafe deserialization
- insecure redirect
- race / TOCTOU
- AI / LLM prompt injection and tool misuse

脆弱性カテゴリに該当し得るという理由だけでFindingにしない。
実際のデータフローと到達可能性を確認する。

#### Database / Data Integrity

関連する場合は以下を確認する。

- transaction boundary
- uniqueness / constraint
- concurrent update
- partial failure
- retry / idempotency
- migration compatibility
- existing data
- mixed-version operation

#### Concurrency / Async

関連する場合は以下を確認する。

- race condition
- shared mutable state
- ordering dependency
- duplicate execution
- cancellation
- timeout
- retry behavior
- resource cleanup

#### External API / I/O

関連する場合は以下を確認する。

- timeout
- retry
- error response handling
- partial response
- rate limit
- idempotency
- resource cleanup
- compatibility assumptions

これらは独立したレビュー観点ではなく、
Referenceの7観点を正しく評価するための補助チェックとして扱う。

### 6. テストを確認する

変更に対するテストの妥当性を確認する。

テスト不足を指摘する場合は、
単に「テストが足りない」とせず、次を明確にする。

- 何を検証すべきか
- どの条件で問題を検出できるか
- なぜ既存テストでは不足するか

実装詳細を固定するだけのテストを安易に要求しない。

### 7. 実行可能な検証を行う

環境で可能かつレビュー判断に有用な場合は、
関連する検証を実行する。

例:

- unit test
- integration test
- lint
- formatter check
- type check
- build

検証結果はFindingの根拠として利用してよい。

レビュー対象と無関係な広範囲の修正は行わない。

## Finding Validation

Finding候補をそのまま最終出力してはならない。

各候補について以下を確認する。

### Evidence

コード、diff、テスト、仕様、設定、実行結果などに検証可能な具体的根拠があるか。

### Trigger

問題が発生する入力・状態・タイミング・環境・操作を説明できるか。

### Impact

Triggerが成立した場合に何が起こるかを具体的に説明できるか。

### Scope relevance

今回の変更によって導入・顕在化した問題、
または今回の変更を正しく成立させるために必要な問題か。

### Actionability

現実的な改善方向を示せるか。

### Confidence

Evidence、Trigger、Impactのつながりに十分な確信があるか。

以下の場合は原則としてFindingにしない。

- 発生可能性を想像しているだけ
- Triggerを説明できない
- コード、diff、テスト、仕様、設定、実行結果などに検証可能な具体的根拠がない
- 変更との関連性がない
- チーム規約などの根拠がない単なる好み
- frameworkや共通処理によって対策済みか確認していない

確信が不足している場合は追加調査する。

コードだけでは確認できない場合は断定せず、必要に応じて確認事項またはレビュー上の制約として扱う。

ConfidenceはFindingを採用するかどうかの内部判断に使用する。
原則として最終出力へ数値スコアを表示しない。

## サブエージェントの利用

変更規模が大きい、複数レイヤーにまたがる、または専門領域を独立して確認する価値がある場合のみ、
レビューをサブエージェントへ分担してよい。

小規模な変更では、原則として単一エージェントでレビューする。
観点ごとに機械的にサブエージェントを起動しない。

分担する場合は、以下の4グループを基本とする。

### Architecture

担当:

- 設計
- 責務分離
- 依存関係
- API / contract
- エラー設計
- テスト容易性
- 既存アーキテクチャとの一貫性

### Correctness & Security

担当:

- 正常系 / 異常系
- 境界条件
- 状態変更 / 副作用
- DB / transaction
- concurrency / async
- external API
- security
- data integrity
- performance / resource usage
- query efficiency / N+1
- algorithmic complexity
- AI / LLM固有リスク

### Maintainability

担当:

- 理解容易性
- 命名
- コードスタイル
- 重複
- 複雑度
- 公開範囲
- 保守性

### Tests & Documentation

担当:

- テストケース設計
- boundary / error / side-effect coverage
- テストコードの品質
- documentation
- README / API仕様 / コメント / 運用情報

### サブエージェントへの指示

各サブエージェントは担当観点について以下を返す。

1. Finding候補（存在する場合）
2. 重要な未確認事項（存在する場合）
3. レビュー上の制約（存在する場合）

未確認事項や制約をFindingと混同せず、明確に区別する。

次を必ず守る。

- `references/code_review_7points.md` をレビュー基準として参照する
- 必要に応じて周辺コード・テスト・プロジェクトルールを確認する
- 担当領域についてFindingを作ることをノルマにしない
- Evidence、Trigger、Impactを説明できない候補は出さない
- コード上の具体的な根拠がない候補は出さない
- 推測の場合は断定しない
- 変更と無関係な既存問題は原則として扱わない
- 重大度は暫定値として扱う
- 暫定的な重大度について、その理由を説明できる状態で返す

### メインエージェントによる統合

サブエージェントの結果を単純に連結して最終回答にしてはならない。

メインエージェントは、すべてのFinding候補について以下を行う。

1. Evidenceを再確認する
2. Triggerを再確認する
3. Impactを再確認する
4. 変更内容との関連性を確認する
5. 重複した指摘を統合する
6. 同一root causeに由来する指摘を可能な限りまとめる
7. false positiveや推測だけの指摘を除外する
8. Findingとして出す十分なConfidenceがあるか確認する
9. 重大度を再評価する
10. 重大度とその理由が整合しているか確認する
11. 改善案が実際に適用可能か確認する
12. 重要な未確認事項は必要に応じて追加調査し、解消できない場合は確認事項またはレビュー上の制約として扱う

最終的なFindingの採用可否と重大度は、メインエージェントが決定する。

### 分担の判断目安

サブエージェントの利用を検討する例:

- 変更ファイル数が多い
- UI / Application / Domain / Infrastructureなど複数レイヤーにまたがる
- DB、非同期処理、認証・認可、外部APIなど複数の専門領域を含む
- 大規模なリファクタリング
- 複数の機能を同時に変更するPR
- diffが大きく、単一エージェントでは観点の見落としリスクが高い

単一エージェントを優先する例:

- 数行〜数十行程度の小規模な修正
- 単純なバグ修正
- 命名変更
- コメントやドキュメントのみの変更
- 対象範囲が1つの関数や1つのコンポーネントに限定される変更

## 重大度

各指摘に以下の重大度を付ける。

重大度は問題カテゴリだけで決めず、以下を総合して判断する。

- **Impact**: 問題が発生した場合の影響の大きさ
- **Reachability**: 発生条件に現実的に到達できるか
- **Blast radius**: 影響を受けるユーザー、データ、機能、システムの範囲
- **Recoverability**: 自動復旧できるか、手動対応が必要か、不可逆か

### Critical

マージすると重大な障害やセキュリティ事故につながる可能性が高い。

例:

- 認証・認可の欠落
- データ破壊
- 機密情報漏洩
- サービス全体が停止する不具合

### High

主要機能の誤動作、明確な回帰、重大な運用問題につながる。

例:

- 一般的な入力で誤った結果になる
- 例外処理不足で主要処理が失敗する
- DB整合性が壊れる
- 非同期競合で結果が不定になる

### Medium

特定条件での不具合、保守性や性能への現実的な悪影響がある。

例:

- 境界値で失敗する
- N+1
- 責務混在により変更影響が大きい
- 重要ケースのテスト不足

### Low

直ちに不具合にはならないが、明確な改善価値がある。

例:

- 読みづらい命名
- 軽微な重複
- 将来バグにつながりやすい分かりづらい構造

### Nit

任意修正。

原則として、チーム規約違反など明確な根拠がある場合のみ使用する。
好みだけのNitは出さない。

重大度を決定する際は、
Impact、Reachability、Blast radius、Recoverabilityのうち
そのFindingの判断に重要な要素を確認する。

問題カテゴリだけを根拠に重大度を決めてはならない。

例えば以下のような判断をしない。

- security issueだから自動的にCritical
- race conditionだから自動的にHigh
- crashだから自動的にHigh

重大度とその理由が整合しない場合は、重大度を再評価する。

## 指摘を書く条件

指摘は、以下をすべて満たす場合に出す。

1. コード、diff、テスト、仕様、設定、実行結果などに検証可能な具体的根拠がある
2. 何が問題なのかを具体的に説明できる
3. その問題が発生する入力・状態・タイミング・環境などの条件を説明できる
4. その条件で発生した場合の影響を説明できる
5. 修正方法または望ましい方向性を示せる
6. 変更対象と関連している
7. Evidence、Trigger、Impactのつながりに十分なConfidenceがある
8. 重大度の理由を説明できる

発生条件を具体的に特定できず、問題が起こる可能性だけを推測している場合は、原則としてFindingにしない。

コードだけでは発生条件を確認できない場合は、断定せず確認事項として扱う。

## 指摘フォーマット

各指摘は以下の情報を含める。

```text
[Severity] 短いタイトル

対象: path/to/file.py:L10-L18
観点: 設計 / 理解容易性 / 命名 / コードスタイル / 機能・性能 / テスト / ドキュメント

問題:
何が問題なのか。

発生条件:
どの入力・状態・タイミング・環境などで問題が発生するのか。

影響:
その条件で問題が発生した場合、ユーザーやシステムにどのような影響があるのか。

重大度の理由:
Impact、Reachability、Blast radius、Recoverabilityのうち判断に重要な要素を用いて、
なぜこの重大度なのかを1〜2文で簡潔に説明する。

改善案:
どのように修正するのがよいか。
必要なら短いコード例を示す。
```

コードレビューUI上でインラインコメントを書く場合は、
テンプレートをそのまま冗長に使わず、
「問題 → 影響 → 改善案」が伝わる簡潔な文章にする。

## 出力形式

最終結果は以下の順序で出力する。

### 1. Review summary

2〜5文程度でまとめる。

- 変更の目的
- 全体評価
- 重要なリスク
- マージ可否の判断材料

### 2. Findings

重大度の高い順に並べる。

```markdown
## Findings
### [High] エラー応答を成功として保存している

`src/foo.py:42-51`

**観点:** 機能・性能

`response.json()` の結果をHTTPステータスに関係なく保存しているため、
4xx/5xx時にもエラーレスポンスが正常データとして永続化されます。

**重大度の理由:** 一般的なAPIエラー時に誤ったデータが保存され、主要機能の正しさに影響するためHighと判断します。

`response.raise_for_status()` 相当の確認を保存前に行い、
エラー時は保存処理へ進まないようにしてください。
```

### 3. Missing tests

不足テストがある場合のみ記載する。

追加すべきテストケースを具体的に示す。

### 4. Optional improvements

マージを妨げない改善案がある場合のみ記載する。

重要なFindingsと混ぜない。

## 問題が見つからない場合

問題が見つからない場合は明示する。

```markdown
## Findings

重大な問題は見つかりませんでした。
```

そのうえで、確認した主要観点や、実行できなかった検証があれば簡潔に記載する。

## 不確実な指摘

コードだけでは判断できない場合は断定しない。

悪い例:

> このAPIは必ずタイムアウトします。

良い例:

> 呼び出し側でタイムアウトが設定されていることを確認できませんでした。
> 共通HTTPクライアント側で設定されていない場合、応答待ちが長時間継続する可能性があります。
