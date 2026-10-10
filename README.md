# code-review-7points

コード変更を **7つの観点** からレビューするエージェントスキルです。`SKILL.md` 形式のスキルに対応した AI コーディングエージェント（[Claude Code](https://claude.com/claude-code) など）で利用できます。

レビュー基準は『[コードレビューの教科書 ――なんとなく承認から抜け出すための観点と判断基準](https://gihyo.jp/book/2026/978-4-297-15768-5)』（技術評論社、著者：佐藤晶彦）をもとにしています。

## 概要

PRやdiff、実装、リファクタリング、テストなどのレビューを依頼すると、次の7観点から、重要度付きで具体的な改善点を指摘します。

1. 設計
2. 理解容易性
3. 命名
4. コードスタイル
5. 機能・性能
6. テスト
7. ドキュメント

### 特徴

- 実際にコード上で根拠が確認できる問題だけを指摘する（好みの違いや無関係な既存問題は指摘しない）
- 各指摘に重大度（Critical / High / Medium / Low / Nit）を付与し、問題・影響・改善案を示す
- 変更規模が大きい場合は、Architecture / Correctness & Security / Maintainability / Tests & Documentation の4グループでサブエージェントに分担し、メインエージェントが統合する（サブエージェントに対応していない環境では単一エージェントでレビューします）
- 出力は Review summary → Findings → Missing tests → Optional improvements の順

## ディレクトリ構成

```text
skills/
└── code-review-7points/
    ├── SKILL.md                         # スキル本体（レビュー手順・出力形式）
    └── references/
        └── code_review_7points.md       # レビュー基準（7観点の詳細）
```

## インストール方法

リポジトリを `git clone` し、`skills/code-review-7points` フォルダを利用するエージェントのスキルディレクトリへコピーするか、シンボリックリンクを張ります。

> `SKILL.md` と `references/` はフォルダごと配置してください。`SKILL.md` は `references/code_review_7points.md` を参照します。

### 1. clone

```bash
git clone https://github.com/tanakatsu/code-review-7points.git
cd code-review-7points
```

### 2. スキルフォルダを配置

配置先は、利用するエージェントのスキルディレクトリです。以下は Claude Code の例です。他のエージェントを使う場合は、そのエージェントのドキュメントに記載されたスキルディレクトリに読み替えてください。

| 範囲 | 配置先（Claude Code の例） |
| --- | --- |
| 全プロジェクトで使う（ユーザー単位） | `~/.claude/skills/` |
| 特定プロジェクトのみ | `<プロジェクト>/.claude/skills/` |

以降の例では、配置先を `SKILLS_DIR` として記載します。

```bash
SKILLS_DIR=~/.claude/skills   # 利用するエージェントのスキルディレクトリに変更
mkdir -p "$SKILLS_DIR"
```

#### 方法A: コピー

```bash
cp -R skills/code-review-7points "$SKILLS_DIR/"
```

#### 方法B: シンボリックリンク（推奨）

`git pull` で更新を取り込むだけで最新版になります。

```bash
ln -s "$(pwd)/skills/code-review-7points" "$SKILLS_DIR/code-review-7points"
```

### 3. 確認

エージェントを（再）起動し、`code-review-7points` がスキル一覧に認識されていることを確認してください。

## 使い方

エージェント上でレビューを依頼すると、スキルが自動的に選択されます。

```text
現在の差分をレビューして
```

```text
このPRをコードレビューして
```

スキル名を指定して呼び出せるエージェントでは、`code-review-7points` と名前を指定して呼び出すこともできます（Claude Code では `/code-review-7points`）。

## 更新

```bash
cd code-review-7points
git pull
```

シンボリックリンクで配置した場合はこれだけで反映されます。コピーした場合は再度 `cp -R` で上書きしてください。

## アンインストール

```bash
rm -rf "$SKILLS_DIR/code-review-7points"
```

シンボリックリンクの場合は `rm "$SKILLS_DIR/code-review-7points"` でリンクのみ削除されます（clone したリポジトリは残ります）。
