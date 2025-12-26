# Git clone –bare とは

git clone --bare は、Git リポジトリをベアリポジトリ（bare repository）としてクローンするコマンドです。

## 通常のクローンとの違い

通常の git clone：

- ワーキングディレクトリ（作業ディレクトリ）が作成される
- .git フォルダ内に Git の管理データが格納される
- ファイルを直接編集・操作できる

git clone --bare：

- ワーキングディレクトリが作成されない
- リポジトリ全体が Git の管理データのみで構成される
- ファイルの直接編集はできない

## 主な用途

サーバー用リポジトリの作成

- 他の開発者が push/pull するための中央リポジトリとして使用
- GitHub や GitLab のようなリモートリポジトリの役割

バックアップ: リポジトリの完全なバックアップを作成

ミラーリング: 別の場所にリポジトリの複製を作成

## 使用例

### ベアリポジトリとしてクローン

```sh
git clone --bare https://github.com/user/repo.git repo.git
```

### 通常のリポジトリからベアリポジトリを作成

```sh
git clone --bare /path/to/local/repo repo.git
```

ベアリポジトリは通常 .git 拡張子を付けて命名することが慣例となっています。
