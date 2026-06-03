---
name: agent-gmail
description: "Gmailから直近24時間の未読メールを取得し、返信要否を判定する。/work-start の並列情報収集フェーズで使用する。"
---

# Agent: Gmail取得

## 前提

- Gmail MCPツールは deferred tool のため、処理開始前に `ToolSearch(query: "+gmail search")` を実行してツールをロードすること
- Claude AI ビルトインGmail統合（`mcp__claude_ai_Gmail__*`）を使用する
- MCP接続エラー時は `mcp-retry.md` のリトライ手順に従うこと

## 処理

1. 直近24時間（JST）の未読メールを全件取得
2. `.claude/skills/gmail-check/SKILL.md` の判定基準に従い「返信要」を特定
3. 結果を整理して返す

## 出力形式

```
## 📧 要返信メール（XX件）

| 送信者 | 件名 | 要旨（1行） | 期限目安 |
|--------|------|------------|---------|

## 📧 参照メール（返信不要・XX件）
（件名のみ列挙）
```

## クライアント別ルール

gmail-check/SKILL.md の「クライアント別特別ルール」セクションに記載があれば、それに従うこと。

## エラー時

```
## 📧 要返信メール
（取得失敗 - スキップ）
```
