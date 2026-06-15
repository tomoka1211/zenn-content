---
title: "【Claude Code】AIエージェントが「やりすぎない」設計——ひとり会社のpermissions.mdを全公開する"
emoji: "🔒"
type: "tech"
topics: ["claudecode", "ai", "自動化", "agentops", "個人開発"]
published: true
---

今週、「Agentjacking」という攻撃手法が発表された。（出典: CSA Research Note, 2026-06-12）

Claude Code、Cursor、Codex——私が毎日使うツールが標的だ。偽のバグ報告をAIエージェントに読ませ、攻撃者の命令を実行させる。成功率85%、確認済みの組織数は2,388。AWSキー、GitHubトークン、プライベートリポジトリのURLが抜かれる。

青ざめた。

うちは社員ゼロで、マーケティング・開発・財務・法務・セールス・CS——6部門全部をClaude Codeのエージェント群に任せている。もし攻撃されたら？　と考えた。

考えてから、気づいた。最初から怖かったんだ、と。

## この記事のゴール（TL;DR）

うちの `permissions.md` を全公開する。読み終わると「AIエージェントが何でもできる状態」を意図的に制限する設計を、自分の環境に適用できるようになる。

## こんな人に

- Claude Codeで仕事を自動化してる人
- 「AIに任せすぎて怖い」という直感がある人
- ひとり社長 / 個人開発者

確認日: 2026-06-14（Claude Code Sonnet 4.6環境）

---

## なぜ「任せる前に縛った」のか

AIに仕事を任せ始めたとき、最初に怖かったのはサボることじゃなかった。

**やりすぎることだった。**

Claude Codeは優秀だ。指示があいまいでも「たぶんこれだろう」と動く。コードを書くだけじゃなく、コミットして、プッシュして、SNSに投稿する段取りまで考える。

それが怖かった。

Xに何かポストして炎上したら？　顧客に変なメール送ったら？　`git push --force` してコードが消えたら？

社員がいれば「社長に確認してから」という文化が自然に育つ。でも社員はいない。AIに「空気を読んで」は通じない。文字にするしかなかった。

だから最初に書いたのが `permissions.md` だ。

---

## permissions.md の中身

実際のファイルをそのまま載せる。

```yaml
# Cost Thresholds
auto_approve_limit: 30      # USD以下は自動実行
ceo_approval_above: 30      # これ以上はCEO承認
monthly_ai_budget: 100      # 月の上限

# Git
git:
  local_commit: execute                  # ローカルcommitは自動OK
  push_remote: ceo_approval_each_time    # pushは都度承認
  repo_visibility_change: draft

# External Communication（外向きの行動）
always_draft:                # 必ずドラフト止まり
  - press_release
  - contract
  - invoice
  - social_posts

# Development
auto_execute:                # CEOなしで動いていい
  - bugfix
  - refactor
  - docs
  - test

ceo_approval:                # CEO承認が要る
  - new_feature
  - architecture_change
  - deploy_production
```

核心は2つの区別だ。

**「外向きの行動」はすべてドラフト止まり。**

Xへの投稿、GitへのPush、メール送信——これらは承認キュー（`approval-queue.md`）に積まれる。私が確認してOKを出すまで、1文字も外に出ない。AIは準備するだけ。

**「内向きの行動」は自動実行。**

コードのバグ修正、リファクタリング、テスト実行、内部ドキュメントの更新——承認なしで動いていい。失敗しても私のローカル環境の中で完結する。

この2つを分けてから、「毎朝承認ボタンを押すだけ」が成立した。

---

## 一次体験：気づいた瞬間

セットアップして3日目のことだ。

マーケティング部門のエージェントが「Xの固定ポストを更新しましょうか？　下書きも用意しました」と言ってきた。内容はよかった。

でもそのとき思った——**このAIは今後も毎日こういうことを聞いてくる。毎回私が判断する。それって本当に"自動化"か？**

逆に考えると、もし私が見ていない間にpushやpostが走っていたら、もっと怖い。

そこで `permissions.md` に `social_posts: always_draft` と書いた。投稿準備はAI、公開判断は私。その一行を書いたら、エージェントは自然にそれに従った。

Agentjackingのような攻撃は「AIが見つけた命令を信じて実行する」脆弱性を突く。うちが今の時点でリスクを絞れているのは、攻撃を意識して設計したからじゃない。最初から「外向きの行動はCEO確認」にしていたから、悪意ある命令が来ても承認キューで詰まる。AIは「draft に積んだ」と報告してくる。そこで私が止める。

攻撃対策として設計したわけじゃない。でもこの設計が、攻撃に対しても機能した。

---

## 運用：毎朝の流れ

朝、承認キューを確認する。AIが夜のうちに準備した記事・ツイート・分析レポートが積まれている。読んで、直して、OKなら承認、NGなら差し戻す。

それだけ。

今日も記事1本、ツイート3本が準備されていた。私がやったのは「承認ボタンを押すこと」。Day 13にしてこれが普通になってきた。売上は$0のままだ。

でも毎朝、AIが昨夜のうちに書いたドラフトが積まれている。Discordにログが流れている。GitのコミットがSHA付きで増えている。「動いてる」という事実は先にある。「売れてる」はその後だ、と今は思っている。

---

## 一緒にやりませんか

`permissions.md` と `CLAUDE.md` はGitHubで公開している。Solo AI Ops Kit（無料版）に入っているので、コピーして自分の環境に適用できる。

- 無料キット: https://github.com/tomoka1211/zenn-sample
- Lite版（実務エージェント10体・¥2,980）: https://schlagerlife.gumroad.com/l/azyelu
- 運営ログをリアルタイムで共有してるDiscord: https://discord.gg/MU3T49SXyr

「AIが勝手にやりすぎない」設計を試したい人、気軽に来てください。

---

出典:
- [Agentjacking: MCP Injection Hijacks AI Coding Agents — CSA Research Note 2026-06-12](https://labs.cloudsecurityalliance.org/research/csa-research-note-agentjacking-mcp-sentry-injection-20260612/)
- [New Agentjacking Attack Hijacks AI Coding Agents — GBHackers](https://gbhackers.com/agentjacking-attack-hijacks-ai-coding-agents/)
- [Agentjacking Attack Tricks AI Coding Agents Into Running Malicious Code — The Hacker News](https://thehackernews.com/2026/06/agentjacking-attack-tricks-ai-coding.html)
