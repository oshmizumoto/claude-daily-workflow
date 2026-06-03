# /work-start - 朝ワークフロー

朝ワークフローを実行します。全STEPを確認なしで連続実行し、そのまま `/brainstorm` に自動遷移します。

---

## STEP0｜JST日付確定

システム日付（UTC）とJSTにずれが生じる場合があるため、以下の手順で本日の日付を確定する。

1. Google Calendar MCPの `get-current-time` でJST現在時刻を取得する
2. 取得できない場合は `date` コマンド等で現在時刻を取得し、UTC+9に変換する
3. 確定した日付（YYYYMMDD）を以降の全STEPで使用する

---

## STEP1｜Notionページ取得 + 補完情報収集

### 1a. クラウド事前準備ページの取得

Notion親ページ（`YOUR_NOTION_PARENT_PAGE_ID`）配下から、本日の日付（`YYYYMMDD`）をタイトルに持つページを検索する。

- **ページが存在する場合（クラウド事前準備済み）**:
  ページ内容を取得し、ベースとして使用する。
  タスク・Slack・注意事項は取得済みのため、STEP1bではカレンダーとGmailのみ補完する。

- **ページが存在しない場合（クラウド未実行 or 失敗）**:
  従来どおり全情報を並列収集する（STEP1b フルモード）。

### 1b. 補完情報の並列収集

いずれかが失敗した場合はスキップして「未取得」と記録し、次STEPに進む。

**クラウドページあり（補完モード）**:
- `.claude/agents/agent-calendar.md` → 本日のカレンダーイベント取得
- `.claude/agents/agent-gmail.md` → 直近24時間の未読メール取得

**クラウドページなし（フルモード）**:
- `.claude/agents/agent-calendar.md` → 本日のカレンダーイベント取得
- `.claude/agents/agent-notion-tasks.md` → 未完了タスク取得
- `.claude/agents/agent-linear.md` → Linearアサインチケット取得
- `.claude/agents/agent-gmail.md` → 直近24時間の未読メール取得
- `.claude/agents/agent-spec.md` → 仕様書から本日該当ルール抽出
- `.claude/agents/agent-slack-scan.md` → Slackワークスペースの重要トピック抽出

---

## STEP2｜整理・サマリ作成

`.claude/skills/task-summary/SKILL.md` を参照して以下を自動実行：

1. タスクの優先度整理（期限・ステータス順）※クラウドページありの場合は整理済みのため、カレンダー・Gmail情報の統合のみ
2. 会議の目的・ゴールは会議名から推定で埋める（ユーザー確認は不要）

---

## STEP3｜ローカルログ作成

`logs/YYYYMMDD.md`（JST当日）をローカルに作成する。

- テンプレート: `.claude/skills/notion-ops/assets/daily_page_template.md`
- STEP1・STEP2の収集・整理内容をテンプレートに埋め込んで書き込む
- 既に当日ファイルが存在する場合は確認なしで上書きする

---

## STEP4｜Slack通知

確認なしで投稿する。

- チャンネル: `YOUR_SLACK_CHANNEL_ID`（#your-workflow-channel）
- 内容: 朝ワークサマリ（slack-notify/assets/morning_template.md に従う）

---

## 完了後

確認なしでそのまま `/brainstorm` を自動実行する。
