# GitHub Actions で NPM パッケージ自動公開フロー

npm で二要素認証（2FA）を有効にしている場合、通常のアクセストークンでは自動公開できません。

以下の解決方法があります。

## 解決方法 1: Automation Access Token を使用（推奨）

npm の Automation Access Token を使用すれば、2FA が有効でも自動公開可能です。

### 1. Automation Token の作成

#### コマンドラインから作成

```sh
# npm にログイン
npm login

# Automation 用トークンを作成
npm token create --type=automation
```

作成されたトークンは一度しか表示されないため、必ずコピーして保存してください。

#### npm 公式サイトから作成

1. [https://www.npmjs.com/settings/tokens](https://www.npmjs.com/settings/tokens) にアクセス
2. 「Generate New Token」をクリック
3. Type: **Automation** を選択
4. 必要な権限を設定して作成
5. 表示されたトークンをコピー（一度しか表示されません）

### 2. GitHub Secrets に追加

1. リポジトリの Settings → Secrets and variables → Actions に移動
2. 「New repository secret」をクリック
3. Name: `NPM_TOKEN`、Value: 作成した Automation Token を入力
4. 「Add secret」をクリック

### 3. GitHub Actions ワークフローの設定

`.github/workflows/publish.yml` を作成（例）：

```yaml
name: Publish to npm

on:
  release:
    types: [created]
  workflow_dispatch:

jobs:
  publish:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: "20"
          registry-url: "https://registry.npmjs.org"

      - name: Install dependencies
        run: npm ci

      - name: Run tests
        run: npm test

      - name: Publish to npm
        run: npm publish
        env:
          NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}
```

**重要なポイント：**

- `NODE_AUTH_TOKEN` 環境変数に `secrets.NPM_TOKEN` を設定する必要があります
- `actions/setup-node@v4` の `registry-url` を指定することで、自動的に認証が設定されます
- Automation Token を使用していれば、2FA コードの入力なしに自動公開できます

## 解決方法 2: .npmrc での設定

リポジトリに `.npmrc` ファイルを作成して認証情報を設定する方法です。

### .npmrc ファイルの作成

プロジェクトルートに `.npmrc` ファイルを作成：

```ini
//registry.npmjs.org/:_authToken=${NODE_AUTH_TOKEN}
```

この方法でも、GitHub Secrets の `NPM_TOKEN` を `NODE_AUTH_TOKEN` 環境変数として設定する必要があります。

**注意点：**

- `.npmrc` ファイルを Git にコミットする場合は、トークンを直接書かないでください
- 環境変数を使用する方法（上記）が推奨されます

## 解決方法 3: 組織アカウントの場合

組織のパッケージの場合、組織レベルで Automation Token の設定が必要な場合があります。

### 組織の Automation Token 設定

```sh
# 組織の 2FA 設定を automation-only に変更
npm org set <org-name> two-factor-auth automation-only
```

これにより、組織のメンバーは Automation Token のみを使用して CI/CD で公開できるようになります。

## トークンの種類と違い

| トークンタイプ | 2FA 必須 | CI/CD 使用 | 用途                   |
| -------------- | -------- | ---------- | ---------------------- |
| Automation     | ❌       | ✅         | CI/CD、自動化          |
| Read-only      | ❌       | ✅         | パッケージ読み取りのみ |
| Publish        | ✅       | ❌         | 手動公開時             |

### 現在のトークンタイプの確認

```sh
npm token list
```

出力例：

```sh
┌─────────────┬────────────┬────────────┬──────────┐
│ id          │ type       │ created    │ readonly │
├─────────────┼────────────┼────────────┼──────────┤
│ abc123...   │ automation │ 2024-01-01 │ no       │
│ def456...   │ publish    │ 2023-12-01 │ no       │
└─────────────┴────────────┴────────────┴──────────┘
```

**重要：** すでに作成済みの Token が Publish type の場合は、新しく Automation type の Token を作成し直してください。

## 便利な使い方

### バージョン管理と自動公開

`package.json` のバージョンを更新してから自動公開する例：

```yaml
- name: Bump version and publish
  run: |
    npm version patch --no-git-tag-version
    npm publish
  env:
    NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}
```

### 条件付き公開（タグやブランチによる制御）

特定のブランチやタグが作成された時のみ公開：

```yaml
on:
  push:
    tags:
      - "v*" # v1.0.0 のようなタグがプッシュされた時

jobs:
  publish:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: "20"
          registry-url: "https://registry.npmjs.org"
      - run: npm ci
      - run: npm publish
        env:
          NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}
```

### プレリリース版の公開

ベータ版やアルファ版を公開する場合：

```yaml
- name: Publish beta version
  run: npm publish --tag beta
  env:
    NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}
```

### 公開前のビルド

TypeScript などのビルドが必要な場合：

```yaml
- name: Build
  run: npm run build

- name: Publish to npm
  run: npm publish
  env:
    NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}
```

### エラーハンドリング

公開に失敗した場合の通知を追加：

```yaml
- name: Publish to npm
  id: publish
  run: npm publish
  env:
    NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}
  continue-on-error: true

- name: Notify on failure
  if: steps.publish.outcome == 'failure'
  uses: actions/github-script@v7
  with:
    script: |
      github.rest.issues.create({
        owner: context.repo.owner,
        repo: context.repo.repo,
        title: 'npm publish failed',
        body: 'The npm publish step failed. Please check the workflow logs.'
      })
```

### 公開前のバージョン確認

既に同じバージョンが公開されている場合、公開をスキップ：

```yaml
- name: Check if version exists
  id: check-version
  run: |
    VERSION=$(node -p "require('./package.json').version")
    if npm view ${{ github.event.repository.name }}@$VERSION version > /dev/null 2>&1; then
      echo "exists=true" >> $GITHUB_OUTPUT
    else
      echo "exists=false" >> $GITHUB_OUTPUT
    fi

- name: Publish to npm
  if: steps.check-version.outputs.exists == 'false'
  run: npm publish
  env:
    NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}
```

## トラブルシューティング

### エラー: "You must provide a one-time passcode"

- Automation Token を使用しているか確認
- `NODE_AUTH_TOKEN` 環境変数が正しく設定されているか確認
- GitHub Secrets の `NPM_TOKEN` が正しく設定されているか確認

### エラー: "You do not have permission to publish"

- トークンに適切な権限があるか確認
- 組織のパッケージの場合、組織の設定を確認

### トークンの削除

不要になったトークンを削除：

```sh
# トークン一覧を確認
npm token list

# 特定のトークンを削除
npm token delete <token-id>
```

## まとめ

Automation Token を使用すれば、GitHub Actions で 2FA コードの入力なしに自動公開できます。
これが最も確実で推奨される方法です。
