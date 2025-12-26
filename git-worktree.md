# Git Worktree

git worktree は、一つの Git リポジトリに対して複数の作業ツリー（ワーキングディレクトリ）を作成できる機能です。
これにより、同時に複数のブランチで作業を進めることが非常に容易になります。

## git worktree の主な利点

### 複数のブランチでの同時作業

例えば、あるブランチで機能開発をしている最中に、別のブランチのバグ修正を緊急で行う必要が生じた場合でも、git worktree を使えば簡単に切り替えて作業できます。元の作業状態を git stash などで一時保存する必要がありません。

### ブランチの比較やテスト

異なるブランチのコードを同時に開いて比較したり、それぞれでテストを実行したりする際に便利です。

### 長時間かかる処理の実行中に別の作業

あるブランチでコンパイルやテストなど時間のかかる処理を実行している間に、別の作業ツリーで別のブランチの作業を進めることができます。

## git worktree の基本的な使い方

### 前提条件とディレクトリ構造

作業を始める前に、現在の状態を確認しましょう。

**初期状態の例：**

```sh
~/projects/
└── my-project/  # メインリポジトリ（現在 main ブランチにいる想定）
    ├── .git/
    ├── src/
    └── README.md
```

この状態で、`my-project` ディレクトリ内にいることを想定します。

### 1. 新しい作業ツリーの追加

新しい作業ツリーを追加するには、`git worktree add` コマンドを使います。
通常、新しいディレクトリを作成し、そこに既存のブランチをチェックアウトするか、新しいブランチを作成してチェックアウトします。

#### 既存のブランチをチェックアウトする場合

```sh
# 現在のディレクトリ: ~/projects/my-project (main ブランチ)
# main_bugfix ブランチを別の作業ツリーでチェックアウト
git worktree add ../my-project-bugfix main_bugfix
```

**実行後のディレクトリ構造：**

```sh
~/projects/
├── my-project/          # メインリポジトリ（main ブランチ）
│   ├── .git/
│   ├── src/
│   └── README.md
└── my-project-bugfix/   # 新しい作業ツリー（main_bugfix ブランチ）
    ├── .git # ../my-project/.git/worktrees/my-project-bugfix
    ├── src/
    └── README.md
```

- `../my-project-bugfix`: 親ディレクトリ（`~/projects/`）に新しい作業ツリーを作成するパスを指定します。既存のディレクトリを指定するとエラーになります。
- `main_bugfix`: 新しい作業ツリーでチェックアウトする既存のブランチ名を指定します。

#### 新しいブランチを作成してチェックアウトする場合

```sh
# new-feature という新しいブランチを作成してチェックアウト
git worktree add ../my-project-feature -b new-feature
```

**実行後のディレクトリ構造：**

```sh
~/projects/
├── my-project/  # メインリポジトリ（main ブランチ）
│   ├── .git/
│   ├── src/
│   └── README.md
├── my-project-bugfix/  # 作業ツリー（main_bugfix ブランチ）
│   └── ...
└── my-project-feature/  # 新しい作業ツリー（new-feature ブランチ）
    ├── .git  # ../my-project/.git/worktrees/my-project-feature
    ├── src/
    └── README.md
```

- `-b new-feature`: `new-feature` という名前で新しいブランチを作成し、そのブランチを新しい作業ツリーでチェックアウトします。

#### リモートブランチをチェックアウトする場合

```sh
# リモートの feature/xyz ブランチをチェックアウト
git worktree add ../my-project-xyz -b feature/xyz origin/feature/xyz
```

### 2. 既存の作業ツリーの一覧表示

現在存在する作業ツリーの一覧を確認するには、`git worktree list` コマンドを使います。

```sh
git worktree list
```

**出力例：**

```text
~/projects/my-project              abc1234 [main]
~/projects/my-project-bugfix       abc1234 [main_bugfix]
~/projects/my-project-feature      def5678 [new-feature]
```

- 最初の列: 作業ツリーのパス
- 2 番目の列: コミットハッシュ
- 3 番目の列: チェックアウトされているブランチ名（`[ブランチ名]` の形式）

### 3. 作業ツリーの削除

作業ツリーでの作業が完了したら、`git worktree remove` コマンドで削除できます。

```sh
# my-project-bugfix 作業ツリーを削除
git worktree remove ../my-project-bugfix
```

または、作業ツリーのディレクトリ内から削除する場合：

```sh
cd ../my-project-bugfix
git worktree remove .
```

**注意点：**

- 作業ツリーに未コミットの変更がある場合、削除できません。
- 変更をコミットするか、`git worktree remove --force` で強制的に削除する必要があります（ただし、未コミットの変更が失われるので注意が必要です）。
- 作業ツリーのディレクトリ自体は手動で削除する必要はありません（`git worktree remove` が自動的に削除します）。

### 4. 作業ツリーの状態確認

メインのリポジトリから、各作業ツリーのブランチの状態を確認できます。

```sh
# 作業ツリーの一覧と詳細情報を表示
git worktree list --verbose

# 各作業ツリーのブランチ状態を確認
git branch -a
```

## 便利な使い方

### 作業ツリーの移動

作業ツリーを別の場所に移動したい場合：

```sh
# 作業ツリーを新しい場所に移動
git worktree move ../my-project-bugfix ../my-project-bugfix-new
```

### 作業ツリーのロック/アンロック

作業ツリーを保護したい場合（例：外部ストレージにマウントされている場合など）：

```sh
# 作業ツリーをロック（削除を防ぐ）
git worktree lock ../my-project-bugfix

# ロックを解除
git worktree unlock ../my-project-bugfix
```

ロックされた作業ツリーは `git worktree list` で `(locked)` と表示されます。

### 削除された作業ツリーのクリーンアップ

作業ツリーのディレクトリを手動で削除してしまった場合、メインリポジトリに残っている参照をクリーンアップできます：

```sh
# 削除された作業ツリーの参照を削除
git worktree prune

# 確認しながらクリーンアップ（推奨）
git worktree prune --verbose
```

### 特定のコミットをチェックアウト

特定のコミットハッシュをチェックアウトして作業ツリーを作成：

```sh
# デタッチドHEAD状態で特定のコミットをチェックアウト
git worktree add ../my-project-old-commit abc1234
```

### 作業ツリーの最大数の確認

Git はデフォルトで 1 つのリポジトリに対して最大 5 つの作業ツリーを作成できます。制限を変更したい場合：

```sh
# 設定を確認
git config extensions.worktreeConfig

# 最大数を変更（例：10個に設定）
git config --global extensions.worktreeConfig true
# 注意: 実際の最大数は Git のバージョンによって異なります
```

## git worktree の内部的な仕組み

git worktree は、メインのリポジトリ（`.git` ディレクトリがある場所）の他に、追加の作業ツリーごとにミニマルな `.git` ファイルを作成します。
この `.git` ファイルは、実際のオブジェクトデータベース（リポジトリの履歴やデータ）がメインのリポジトリにあることを指し示しています。
そのため、リポジトリのデータ自体が重複してディスク容量を消費することはありません。

**内部構造の例：**

```text
~/projects/my-project/.git/
├── worktrees/
│   ├── my-project-bugfix/
│   │   ├── HEAD
│   │   ├── index
│   │   └── logs/
│   └── my-project-feature/
│       ├── HEAD
│       ├── index
│       └── logs/
└── ... (通常の .git ディレクトリの内容)
```

## 注意点

- git worktree はローカルの機能です。リモートリポジトリに影響を与えるものではありません。
- 複数の作業ツリーで同じブランチをチェックアウトすることはできません。
- 作業ツリーを削除する際は、未コミットの変更がないことを確認するか、必要に応じて強制削除オプションを使用してください。
- 作業ツリーのパスは絶対パスでも相対パスでも指定できますが、相対パスの場合はメインリポジトリからの相対パスとして解釈されます。
- 作業ツリーは通常の Git リポジトリと同じように動作しますが、`.git` がファイル（シンボリックリンク）になっている点が異なります。
