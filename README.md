# zenn-content

[Zenn](https://zenn.dev/) の記事を GitHub 連携で管理するリポジトリ。

## 使い方
```bash
npm install zenn-cli      # 初回のみ
npx zenn preview          # http://localhost:8000 でプレビュー
npx zenn new:article --slug <スラッグ>   # 新規記事の雛形
```

## 公開フロー
1. `articles/<スラッグ>.md` を編集
2. frontmatter の `published` を `true` にする
3. `git push` → Zenn が自動公開

## 記事
- `articles/claude-code-solo-ai-kit.md` — 「コマンドを覚えない」Claude Code運用術
