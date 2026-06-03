---
name: slack-notify
description: "Slackへの朝ワーク開始通知・クローズ通知を投稿する。/work-start STEP4と/work-end ④で使用する。"
---

# Slack Notify Workflow

## 投稿先

チャンネル: `YOUR_SLACK_CHANNEL_ID`（#your-workflow-channel）

## 投稿パターン

### 朝ワーク開始通知（`./assets/morning_template.md`）

```
【朝ワーク開始 YYYYMMDD】

📝 {NotionページURL}

📌 サマリ
- タスク: {未完了件数}件（本日フォーカス: {件数}件）
- 会議: {件数}件
- 要返信: {件数}件
```

### クローズ通知（`./assets/close_template.md`）

```
【壁打ち完了 YYYYMMDD】

📝 {NotionページURL}

✅ サマリ
- テーマ: {主テーマ}
- 決定事項: {件数}件
- 追加タスク: {件数}件
```

## 注意事項

- 投稿前のユーザー確認は不要。そのまま投稿する。
- APIエラー時はスキップして最後にまとめて報告する

## Resources

- assets/morning_template.md
- assets/close_template.md
- triggers: `./triggers/next_action_triggers.md`
