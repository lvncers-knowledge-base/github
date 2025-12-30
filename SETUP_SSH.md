# SSH 認証について

別のパソコンで Git を使って既存のリポジトリにプッシュする際、いくつかのベストプラクティスと重要なポイントがあります。

以下に手順を整理しつつ、ユーザー設定やブランチの取り扱いについて説明します。

## 手順：別のパソコンでリポジトリを使う

### 1. Git をインストール

必要であれば、Git をインストールします。[公式サイト](https://git-scm.com/)から最新版をダウンロードできます。

### 2. SSH キーを設定

新しいパソコンからリモートリポジトリにアクセスするには、認証が必要です。

1. SSH キーを生成する：

   ```bash
   ssh-keygen -t ed25519 -C "your_email@example.com"
   ```

2. 公開鍵を Git サービス（GitHub, GitLab など）に登録します。

   ```bash
   cat ~/.ssh/id_ed25519.pub
   ```

   **_公開鍵のフォーマット_**

   公開鍵ファイル（通常は ~/.ssh/id_ed25519.pub）には、以下のような 1 行のテキストが含まれています：

   ```text
   ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIAAA...任意の長い文字列... 　　　　your_email@example.com
   ```

#### GitHub に登録する

1. GitHub アカウントにログイン。
2. 右上のプロフィール写真をクリックし、「Settings（設定）」を選択。
3. 左側メニューから「SSH and GPG keys」をクリック。
4. 「New SSH key」を選択。
5. Title（名前）を入力し、コピーした公開鍵を貼り付けて「Add SSH key」をクリック。
6. SSH 接続を確認：

   ```sh
   ssh -T git@github.com
   ```

以下のメッセージが表示されれば正常。

```sh
ssh -T git@github.com
The authenticity of host 'github.com ()' can't be established.
ED25519 key fingerprint is SHA256:.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'github.com' (ED25519) to the list of known hosts.
Hi ! You've successfully authenticated, but GitHub does not provide shell access.
```

### 3. リポジトリをクローン

初めてそのパソコンで作業する場合、リポジトリをクローンします：

```sh
git clone git@github.com:username/repository.git
cd repository
```

### 既存の HTTPS リモート URL を変更

既存のリポジトリで HTTPS を使っていた場合、リモート URL を SSH 形式に変更します：

```sh
git remote set-url origin git@github.com:username/repository.git
```

## 4. Git ユーザー設定

個別のリポジトリで設定する場合：

```bash
git config user.name "Your Name"
git config user.email "your_email@example.com"
```

グローバル設定は、複数のリポジトリに共通：

```bash
git config --global user.name "Your Name"
git config --global user.email "your_email@example.com"
```

## 5. 新しいブランチを作成

他のパソコンで作業する際、新しいブランチを作るのが安全です。

```bash
git checkout -b new-feature
```

## 6. 作業後にプッシュ

ブランチをリモートにプッシュします：

```bash
git push -u origin new-feature
```

## 7. 変更のマージ

プルリクエスト（GitHub）やマージリクエスト（GitLab）を利用して、コードレビュー後に変更を統合します。

### ユーザー設定とブランチについてのポイント

#### ユーザー設定

- 別パソコンでも、ユーザー名とメールアドレスを必ず正しく設定してください。これにより、コミット履歴が一貫性を保ちます。
- メールアドレスはプライバシーを保護するため、GitHub の場合は GitHub 専用のメール（`username@users.noreply.github.com`）を使うこともできます。

### トラブルシューティング

- 権限エラー

  SSH キーの設定が間違っている可能性があります。公開鍵が正しく登録されているか確認してください。

これらのポイントを押さえれば、複数のパソコン間で効率的に Git を利用できます！

## 別の PC からデータを残さないようにデータを削除したい

別の PC からデータを削除して痕跡を残さないようにする方法を説明します。
ここでは、Git リポジトリや SSH キーなどの認証情報、キャッシュファイルの削除に焦点を当てます。

### 1. Git 関連のデータを削除する

#### リポジトリデータを削除

- ローカルにクローンしたリポジトリを完全に削除します。

  ```bash
  rm -rf /path/to/your/repository
  ```

- `/path/to/your/repository` をリポジトリのフォルダパスに置き換えてください。

#### Git のキャッシュや設定を削除

1. グローバル設定の確認：

   ```bash
   git config --global --list
   ```

   これで保存された設定（`user.name` や `user.email`）が表示されます。

2. 設定を削除

   ```bash
   git config --global --unset user.name
   git config --global --unset user.email
   ```

3. すべてのグローバル設定をリセットする場合：

   ```bash
   rm ~/.gitconfig
   ```

### 2. SSH キーを削除する

#### ローカル SSH キーの削除

- SSH キーが保存されているディレクトリ（通常 `~/.ssh/`）を削除します：

  ```bash
  rm -rf ~/.ssh
  ```

#### 特定の SSH キーのみを削除

- 特定の鍵ファイルを削除：

  ```bash
  rm ~/.ssh/id_ed25519
  rm ~/.ssh/id_ed25519.pub
  ```

### 3. 認証情報キャッシュを削除する

Git が HTTPS を使用している場合、認証情報がキャッシュされている可能性があります。

#### キャッシュされた認証情報を削除

- 以下のコマンドで Git のキャッシュをクリア：

  ```bash
  git credential-cache exit
  ```

#### OS の認証情報マネージャーを確認

- Windows の場合：

  - 「資格情報マネージャー」を開き、Git 関連の資格情報を削除します。

- macOS の場合：

  - 「Keychain Access（キーチェーンアクセス）」で GitHub や GitLab 関連の情報を削除。

### 4. ログや履歴を削除する

作業後にシェルやターミナルのコマンド履歴を削除します。

#### Bash/Zsh の履歴を削除

1. 履歴ファイルをクリア：

   ```bash
   history -c
   ```

2. 履歴ファイルを削除：

   ```bash
   rm ~/.bash_history
   rm ~/.zsh_history
   ```

#### PowerShell の履歴を削除

- 以下のファイルを削除：

  ```text
  %USERPROFILE%\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
  ```

### 5. その他の不要ファイルを削除

Git や SSH 以外で作業中に生成された不要な一時ファイルや設定ファイルが残っている場合も削除します。

- 確認：

  - クローンや設定時に作成した特定のフォルダやファイル。
  - メモや作業ログ。

- 削除：

  ```bash
  rm -rf /path/to/temp/files
  ```

### 6. 完全消去が必要な場合

一般的な削除ではデータ復元が可能な場合があります。本当に痕跡を残したくない場合、以下を検討してください：

1. **shred コマンド**（Linux/MacOS）

   - ファイルを完全消去します：

     ```bash
     shred -u /path/to/your/file
     ```

2. **専用ツール**

   - Windows：CCleaner
   - macOS：Secure Empty Trash
   - Linux：BleachBit

上記の手順で、データを削除して痕跡を最小限に抑えることができます。必要に応じてご相談ください！
