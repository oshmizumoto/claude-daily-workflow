---
name: agent-notion-tasks
description: "NotionタスクDBから未完了タスクを取得する。/work-start の並列情報収集フェーズで使用する。"
---

# Agent: タスクDB取得

## 前提

- Notion MCPツールは deferred tool の場合があるため、処理開始前に `ToolSearch(query: "+notion query")` を実行してツールをロードすること
- Claude AI ビルトインNotion統合（`mcp__claude_ai_Notion__*`）を使用する
- MCP接続エラー時は `mcp-retry.md` のリトライ手順に従うこと

## 処理

NotionタスクDB（ID: `YOUR_NOTION_TASK_DB_ID`）から
ステータスが「バックログ / TODO / 対応中 / レビュー / 保留」のタスクを全件取得。

## 出力形式

```
## 📋 未完了タスク（XX件）

| タスク名 | ステータス | 優先度 | 期限 | プロジェクト |
|---------|-----------|--------|------|------------|
| タスク名 | TODO | 高 | 2/25 | - |
```

## エラー時

```
## 📋 未完了タスク
（取得失敗 - スキップ）
```
