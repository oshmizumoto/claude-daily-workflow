# claude-daily-workflow

Claude Code（CLI）で運用する日次ワークフローのテンプレート。  
朝の情報収集（Calendar / Notionタスク / Linear / Gmail / Slack）→ 壁打ち → クローズ報告までを `/work-start` `/brainstorm` `/work-end` の3コマンドで回す。

## ワークフロー概要

| コマンド | 用途 |
|---------|------|
| `/cloud-prep` | 早朝バッチ。Notion・Linear・Slack・仕様書を収集して当日のNotion日次ページを事前生成（`/schedule` で平日早朝に自動実行） |
| `/work-start` | カレンダー・Gmailを補完しつつ当日ログを生成、Slackで業務開始通知 |
| `/brainstorm` | タスク棚卸し（Notion即時反映）→ 壁打ち。トピック区切りごとに当日ログへ追記 |
| `/work-end` | ローカルログをNotionに同期、アクションアイテムをタスクDBへ追加、Slackでクローズ報告 |

設計の特徴:

- **セッション中はローカル完結**（`logs/YYYYMMDD.md` に読み書き）
- **クローズ時に一括でNotion同期**（API往復を最小化）
- **SubAgent並列実行**（朝の情報収集を6つのagent-*.mdで並列化）
- **早朝事前準備をスケジュール化**（`/cloud-prep` を `/schedule` 登録、起動時の体感速度を上げる）

## セットアップ

### 1. リポジトリを取得

```bash
git clone https://github.com/oshmizumoto/claude-daily-workflow.git
cd claude-daily-workflow
```

### 2. Claude Codeを起動してセットアップSkillを実行

```bash
claude
```

起動後、以下のいずれかを送信してセットアップSkillを呼び出す：

- 「セットアップして」
- 「初期設定」
- 「configure」

`setup` Skill が対話形式で以下を順次収集し、リポジトリ内のプレースホルダーを置換する：

| 収集する値 | 取得方法 |
|----------|---------|
| NotionタスクDB ID | NotionデータベースURLから抽出（32桁） |
| Notion親ページID | 日次ログを作成する親ページのURL |
| Notion仕様書ページID | 定期ルール記載ページ（任意） |
| SlackチャンネルID | Slackチャンネル詳細の「チャンネルID」（Cで始まる） |
| Slackチャンネル表示名 | `#xxx` 形式 |
| ユーザー名・所属 | CLAUDE.md に記載するアイデンティティ |

置換完了後、Notion・Slack接続の検証を自動実行し、結果を報告する。

### 3. MCP接続を整える

Claude Code（CLI）で以下のMCPサーバーが利用できる状態にする：

- Notion（`mcp__claude_ai_Notion__*`）
- Slack（`mcp__claude_ai_Slack__*`）
- Gmail（`mcp__claude_ai_Gmail__*`）
- Google Calendar（`mcp__claude_ai_Google_Calendar__*`）
- Linear（`mcp__linear__*`）

`.claude/settings.json` で許可済み。CLIから claude.ai のビルトイン統合を有効化すること。

### 4. 動かす

```
/work-start
```

### 5. （任意）早朝事前準備をスケジュール登録

`/cloud-prep` を `/schedule` Skill で平日早朝に登録すると、`/work-start` 起動時に既存のNotion日次ページを再利用でき、待ち時間が短くなります。

```
/schedule create
- name: cloud-prep
- cron: 0 6 * * 1-5      # 平日 6:00（JST想定）
- timezone: Asia/Tokyo
- prompt: /cloud-prep
```

スケジュール内容を変更する場合は `/schedule list` で現状を確認し、必要に応じて更新してください。

## ディレクトリ構成

```
.claude/
├── agents/          # SubAgent（並列情報収集）
│   ├── agent-calendar.md
│   ├── agent-gmail.md
│   ├── agent-linear.md
│   ├── agent-notion-tasks.md
│   ├── agent-slack-scan.md
│   ├── agent-spec.md
│   └── mcp-retry.md
├── commands/        # スラッシュコマンド
│   ├── work-start.md
│   ├── brainstorm.md
│   └── work-end.md
├── skills/          # on-demand参照されるスキル
│   ├── setup/       # 初回セットアップ（このリポジトリ固有）
│   ├── gmail-check/
│   ├── notion-ops/
│   ├── slack-notify/
│   └── task-summary/
└── settings.json    # MCP permission許可リスト
CLAUDE.md            # プロジェクト全体の常時読み込みコンテキスト
```

## カスタマイズ

- **クライアント別ルール**: `.claude/skills/gmail-check/SKILL.md` の「クライアント別特別ルール」セクションを編集
- **Slack注目トピック**: `.claude/agents/agent-slack-scan.md` の検索キーワード・重要度判定を編集
- **デイリーページテンプレート**: `.claude/skills/notion-ops/assets/daily_page_template.md` を編集

## 手動セットアップ（参考）

Skillを使わず手動でプレースホルダーを置換する場合の sed スクリプト例：

```bash
NOTION_TASK_DB="<your-id>"
NOTION_PARENT="<your-id>"
NOTION_SPEC="<your-id>"
SLACK_CH="<your-id>"
SLACK_NAME="#your-channel"

grep -rl "YOUR_NOTION_TASK_DB_ID" . | xargs sed -i "" "s/YOUR_NOTION_TASK_DB_ID/$NOTION_TASK_DB/g"
grep -rl "YOUR_NOTION_PARENT_PAGE_ID" . | xargs sed -i "" "s/YOUR_NOTION_PARENT_PAGE_ID/$NOTION_PARENT/g"
grep -rl "YOUR_NOTION_SPEC_PAGE_ID" . | xargs sed -i "" "s/YOUR_NOTION_SPEC_PAGE_ID/$NOTION_SPEC/g"
grep -rl "YOUR_SLACK_CHANNEL_ID" . | xargs sed -i "" "s/YOUR_SLACK_CHANNEL_ID/$SLACK_CH/g"
grep -rl "#your-workflow-channel" . | xargs sed -i "" "s|#your-workflow-channel|$SLACK_NAME|g"
```

## ライセンス

MIT
