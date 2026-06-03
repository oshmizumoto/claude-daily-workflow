# Notion Ops 評価ガイド

## Most（必須修正）

以下のいずれかがFailの場合、操作を確定しない。

| 項目 | 基準 | 判定 |
|------|------|------|
| 正しいIDを使用しているか | CLAUDE.mdの固定リソースIDと一致 | Pass/Fail |
| テンプレート構造を崩していないか | 全セクションが存在する | Pass/Fail |
| 破壊的操作前の確認を取ったか | ユーザーへの提示・承認がある | Pass/Fail |

## More（推奨）

| 観点 | 基準 | 配点 |
|------|------|------|
| 会議ゴールの補完 | 不明な場合に「※推定」と付記している | 30 |
| タスク優先度の反映 | task-summaryの判定に沿っている | 40 |
| エラー記録の明確さ | スキップ理由が明記されている | 30 |

---

## ページ編集（pull/push）操作のチェック

### Push前チェック（Must — 1つでもFailならpush禁止）

| 項目 | 基準 | 判定 |
|------|------|------|
| frontmatter存在 | `notion_page_id` が有効なUUID形式である | Pass/Fail |
| frontmatter status | `editing` 状態である（`synced` のまま再pushしていない） | Pass/Fail |
| 子ページ・DB保持 | pull時に含まれていた `<page url="...">` `<database url="...">` タグが本文に残っている | Pass/Fail |
| ユーザー確認 | push内容のサマリをユーザーに提示し、承認を得ている | Pass/Fail |
| スナップショット鮮度 | `pulled_at` から24時間以内である（超過時は再pull推奨を提示） | Pass/Fail |
