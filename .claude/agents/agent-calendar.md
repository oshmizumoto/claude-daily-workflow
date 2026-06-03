---
name: agent-calendar
description: "Google Calendarから本日のイベント一覧を取得する。/work-start の並列情報収集フェーズで使用する。"
---

# Agent: Calendar取得

## 前提

- Calendar MCPツールは deferred tool の場合があるため、処理開始前に `ToolSearch(query: "+calendar list")` を実行してツールをロードすること
- Claude AI ビルトインCalendar統合（`mcp__claude_ai_Google_Calendar__*`）または .mcp.json 経由の `mcp__google-calendar__*` を使用する
- MCP接続エラー時は `mcp-retry.md` のリトライ手順に従うこと

## 処理

1. Google Calendarから本日（JST）00:00〜23:59のイベントを全件取得
2. 以下の形式で整理して返す

## 出力形式

```
## 📅 本日のカレンダー

| 時間 | イベント名 | 参加者 | 場所/URL |
|------|-----------|--------|---------|
| HH:MM〜HH:MM | イベント名 | - | - |
```

## エラー時

取得できない場合は以下を返す：
```
## 📅 本日のカレンダー
（取得失敗 - スキップ）
```
