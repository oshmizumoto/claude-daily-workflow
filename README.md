# claude-daily-workflow

Claude Code（CLI）で運用する日次ワークフローのテンプレート。  
朝の情報収集（Calendar / Notionタスク / Linear / Gmail / Slack）→ 壁打ち → クローズ報告までを `/work-start` `/brainstorm` `/work-end` の3コマンドで回す。

## ワークフロー概要

| コマンド | 用途 |
|---------|------|
| `/work-start` | カレンダー・タスク・メール・Slackを並列収集し、当日ログを生成、Slackで業務開始通知 |
| `/brainstorm` | タスク棚卸し（Notion即時反映）→ 壁打ち。トピック区切りごとに当日ログへ追記 |
| `/work-end` | ローカルログをNotionに同期、アクションアイテムをタスクDBへ追加、Slackでクローズ報告 |

設計の特徴:

- **セッション中はローカル完結**（`logs/YYYYMMDD.md` に読み書き）
- **クローズ時に一括でNotion同期**（API往復を最小化）
- **SubAgent並列実行**（朝の情報収集を6つのagent-*.mdで並列化）

## セットアップ

### 1. リポジトリを取得

```bash
git clone https://github.com/oshmizumoto/claude-daily-workflow.git
cd claude-daily-workflow
```

### 2. プレースホルダーを自分の環境に置き換える

以下の値を `CLAUDE.md` および `.claude/` 配下で全置換する。

| プレースホルダー | 置き換える値 |
|----------------|------------|
| `YOUR_NOTION_TASK_DB_ID` | あなたのタスクDB（Notion）のID |
| `YOUR_NOTION_PARENT_PAGE_ID` | 日次ページの親ページ（Notion）のID |
| `YOUR_NOTION_SPEC_PAGE_ID` | 仕様書・定期ルールを書いたページ（Notion）のID |
| `YOUR_SLACK_CHANNEL_ID` | 通知先Slackチャンネル ID |
| `#your-workflow-channel` | 表示用チャンネル名 |

`CLAUDE.md` の「基本コンテキスト」セクション（ユーザー名・組織）も書き換える。

例（bash, macOS/Linux）:

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

### 3. MCP接続を整える

Claude Code（CLI）で以下のMCPサーバーが利用できる状態にする:

- Notion（`mcp__claude_ai_Notion__*`）
- Slack（`mcp__claude_ai_Slack__*`）
- Gmail（`mcp__claude_ai_Gmail__*`）
- Google Calendar（`mcp__claude_ai_Google_Calendar__*`）
- Linear（`mcp__linear__*`）

`.claude/settings.json` で許可済み。CLIから claude.ai のビルトイン統合を有効化すること。

### 4. ローカルログ・docs用ディレクトリを作成

`logs/`, `docs/` は `.gitignore` 済み。実行時にローカル生成される。

### 5. 動かす

```bash
claude
```

起動後、`/work-start` を実行。

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

## ライセンス

MIT
