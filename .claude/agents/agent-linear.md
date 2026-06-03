---
name: agent-linear
description: "Linearからアサインされたチケットを取得する。/work-start の並列情報収集フェーズで使用する。"
---

# Agent: Linearチケット取得

## 前提

- Linear MCPツールは deferred tool の場合があるため、処理開始前に `ToolSearch(query: "+Linear list")` を実行してツールをロードすること
- Claude AI ビルトインLinear統合（`mcp__claude_ai_Linear__*`）を使用する
- MCP接続エラー時は `mcp-retry.md` のリトライ手順に従うこと

## 処理

1. `mcp__claude_ai_Linear__list_issues` で自分にアサインされた未完了チケットを取得する
   - フィルタ: ステータスが完了・キャンセル以外（Backlog / Todo / In Progress / In Review 等）
   - assignee: 自分（"me" または現在のユーザー）
2. 各チケットからタスク名・ステータス・優先度・期限・プロジェクト・チケットIDを抽出する

## 出力形式

```
## 🎯 Linearチケット（XX件）

| チケット | ステータス | 優先度 | 期限 | プロジェクト |
|---------|-----------|--------|------|------------|
| LIN-123 タイトル | In Progress | Urgent | 3/27 | プロジェクト名 |
```

優先度の表記: Urgent / High / Medium / Low / None

## エラー時

```
## 🎯 Linearチケット
（取得失敗 - スキップ）
```
