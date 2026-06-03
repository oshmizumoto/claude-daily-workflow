# MCP接続リトライ手順

各agentがMCPツール呼び出しに失敗した場合、以下の手順でリトライする。

## 判定条件

以下のエラーはMCP接続エラーとみなし、リトライ対象とする：
- `MCP server not found` / `MCP server disconnected`
- `Tool not found` （ToolSearchでツールが見つからない場合）
- `connection refused` / `connection reset` / `timeout`
- `ECONNREFUSED` / `ECONNRESET` / `ETIMEDOUT`

以下はリトライ対象外（即スキップ）：
- 認証エラー（`401`, `403`, `unauthorized`）
- リソース不在（`404`, `not found`）
- レート制限（`429`, `rate limit`）
- リクエスト不正（`400`, `bad request`）

## リトライ手順

1. **1回目失敗**: 5秒待機後、ToolSearchを再実行してツールを再ロードし、同じ操作をリトライ
2. **2回目失敗**: 10秒待機後、再度ToolSearch→リトライ
3. **3回目失敗**: 接続不可と判断し、エラー出力テンプレートで返す

最大リトライ回数: 2回（初回含め計3回試行）

## 実行フロー

```
ToolSearch → ツール呼び出し
  ├─ 成功 → 正常出力
  └─ MCP接続エラー
       ├─ リトライ1: 5秒待機 → ToolSearch再実行 → ツール呼び出し
       │    ├─ 成功 → 正常出力
       │    └─ MCP接続エラー
       │         └─ リトライ2: 10秒待機 → ToolSearch再実行 → ツール呼び出し
       │              ├─ 成功 → 正常出力
       │              └─ 失敗 → エラー出力（スキップ）
       └─ リトライ対象外エラー → エラー出力（スキップ）
```
