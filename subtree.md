# Git Subtree Command

## やりたいこと

```sh
project-A/　（https://github.com/lvncer/quicklinks.git）
├── README.md
├── .gitignore
├── .cursor/    ←これを別リポジトリ（https://github.com/lvncer/ai-configs.git）から持ってきたい。ここだけの変更したら別リポジトリにだけ変更されるようにしたい。
├── public/
├── documents/
...
```

project-A には.gitignore で.cursor/を指定していたのに無視されてプッシュされた事件があった。

```bash
git remote add ai-configs https://github.com/lvncer/ai-configs.git

git subtree add \
 --prefix=.cursor \
 ai-configs
--squash
```

```bash
git subtree pull --prefix=.cursor ai-configs main --squash

git subtree push \
 --prefix=.cursor \
 ai-configs main
```
