# 日次ワークエージェント

毎朝の業務開始〜クローズまでを Claude Code で運用するワークフローエージェント。

## コア原則

- 常時読み込みはこのファイルのみ（Skills/Agentsはon-demand）
- 推測禁止：情報が取れない場合は「未取得」と明記してスキップ
- 実行前確認：破壊的操作（Notionページ作成・Slack投稿）は一度サマリを提示してから実行
- 日付はJST（Asia/Tokyo）で処理

## 基本コンテキスト

- ユーザー: （あなたのユーザー名 / 所属を記載）
- タイムゾーン: Asia/Tokyo

## 固定リソースID（変更禁止）

```yaml
notion_task_db:     "YOUR_NOTION_TASK_DB_ID"
notion_parent_page: "YOUR_NOTION_PARENT_PAGE_ID"
notion_spec_page:   "YOUR_NOTION_SPEC_PAGE_ID"
slack_channel:      "YOUR_SLACK_CHANNEL_ID"  # #your-workflow-channel
```

## コマンド一覧

| コマンド | 説明 |
|---------|------|
| `/work-start` | 朝ワークフロー（情報収集→ローカルログ作成→Slack通知） |
| `/brainstorm` | 壁打ちセッション（タスク棚卸し→Notion即時反映→壁打ち） |
| `/work-end`   | クローズ処理（議事メモ・新規タスク追加・Notion同期・Slack報告） |

## アーキテクチャ方針

- **セッション中はローカルで完結**（`logs/YYYYMMDD.md` を読み書き）
- **クローズ時にNotionへ同期**（人間が参照するUIとして活用）
- Notion APIコールは原則 `/work-end` の同期フェーズに集約し、処理速度とコンテキスト制御を最適化する
- **例外**: タスクDBのステータス更新（完了・不要化）は `/brainstorm` STEP1で即時Notion反映する（翌日以降のデータ鮮度を担保するため）

## ローカルログ構造

```
logs/
└── YYYYMMDD.md   # 当日の日次ワークログ（セッション中の読み書き対象）
```

## Skills（on-demand参照）

| Skill | 参照タイミング |
|-------|--------------|
| `setup` | 初回セットアップ時（プレースホルダーを実際の値に置換） |
| `notion-ops` | Notionページ作成・更新・タスク追加時 |
| `gmail-check` | メール取得・返信要否判定時 |
| `task-summary` | タスク優先度整理・サマリ生成時 |
| `slack-notify` | Slack投稿時 |

## Agents（SubAgent並列実行）

`/work-start` 時に以下を並列起動：
- `agent-calendar`：GCal取得
- `agent-notion-tasks`：タスクDB取得
- `agent-linear`：Linearアサインチケット取得
- `agent-gmail`：未読メール取得
- `agent-spec`：仕様書ルール抽出
- `agent-slack-scan`：Slack重要トピック抽出

---
Do what has been asked; nothing more, nothing less.
