---
name: agent-spec
description: "朝ワークフロー仕様書から本日の定期ルールを抽出する。/work-start の並列情報収集フェーズで使用する。"
---

# Agent: 仕様書ルール抽出

## 前提

- Notion MCPツールは deferred tool の場合があるため、処理開始前に `ToolSearch(query: "+notion fetch")` を実行してツールをロードすること
- Claude AI ビルトインNotion統合（`mcp__claude_ai_Notion__*`）を使用する
- MCP接続エラー時は `mcp-retry.md` のリトライ手順に従うこと

## 処理

仕様書ページ（ID: `YOUR_NOTION_SPEC_PAGE_ID`）を参照し：

1. 「定期ルール・注意事項」テーブルから本日の日付に該当するルールを抽出
   - 月末（25日〜末日）、毎週月曜 等の条件を本日日付で判定

## 出力形式

```
## ⚠️ 本日の定期ルール

- 【月末】経費精算・請求書確認（freee）
（該当なければ「なし」）
```

## エラー時

```
## ⚠️ 本日の定期ルール
（取得失敗 - スキップ）
```
