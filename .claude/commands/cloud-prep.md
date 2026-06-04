# /cloud-prep - 早朝事前準備（スケジュール実行用）

`/work-start` の前にバックグラウンドで動かす事前準備処理。  
当日のタスク・Slack・定期ルールを Claude Code のスケジュール機能（`/schedule`）で早朝に収集し、Notion 日次ページとして書き出しておく。

朝の `/work-start` 実行時に既存ページを検出すれば、その内容をベースに使い、カレンダー・Gmailだけリアルタイム補完する（高速化＋当日朝の安定動作）。

`/cloud-prep` は **冪等**で、同日に再実行された場合は既存ページを上書き更新する。

---

## STEP0｜JST日付確定

1. `mcp__google-calendar__get-current-time` で JST 現在時刻を取得
2. 取得できない場合は `date` コマンド等でフォールバック
3. 確定した `YYYYMMDD` を以降の全STEPで使用

---

## STEP1｜情報収集（SubAgents並列実行）

以下のAgentを並列起動：

- `.claude/agents/agent-notion-tasks.md` → Notion未完了タスク取得
- `.claude/agents/agent-linear.md` → Linearアサインチケット取得
- `.claude/agents/agent-slack-scan.md` → Slack重要トピック抽出（直近24時間）
- `.claude/agents/agent-spec.md` → 仕様書から当日該当する定期ルール抽出

いずれかが失敗した場合はスキップして「未取得」と記録し、次STEPに進む。

カレンダー・Gmailは含めない（リアルタイム性を重視し朝の `/work-start` 実行時に取得する）。

---

## STEP2｜整理・サマリ作成

`.claude/skills/task-summary/SKILL.md` を参照して以下を自動実行：

1. Notion + Linear を統合し、優先度順にソート（重複はLinear優先）
2. 【本日フォーカス】【30分検討枠】【今週中対応】【来週以降】【Linearチケット】【バックログ / 保留】を組み立てる

---

## STEP3｜Notion日次ページ生成

`.claude/skills/notion-ops/SKILL.md` の手順に従い、当日ページを作成または更新する。

1. 親ページ（`YOUR_NOTION_PARENT_PAGE_ID`）配下に `YYYYMMDD` ページが既に存在するか確認する
2. **存在する場合** → `notion-update-page`（`replace_content`）で内容を上書き
3. **存在しない場合** → `notion-create-pages` で新規作成

ページ内容のテンプレート: `.claude/skills/notion-ops/assets/daily_page_template.md`

書き込むセクション:
- 今日のタスク（TOP5 / 30分検討枠 / 今週中 / 来週以降 / Linear / バックログ）
- 今日の注意事項（agent-spec 抽出分）
- Slack注目トピック（agent-slack-scan 抽出分）

書き込まないセクション（朝の `/work-start` で補完）:
- 今日の会議体
- 返信要メール
- 壁打ちメモ
- 今日のアクションアイテム

---

## STEP4｜完了報告

スケジュール実行のため UI 出力は最小限とする。

- 成功時: NotionページURL を1行で出力
- 失敗時: 失敗したSTEPと理由を出力

Slack通知は行わない（朝の `/work-start` STEP4 で投稿する）。

---

## スケジュール登録

このコマンドは `/schedule` Skill で平日早朝に自動実行することを想定している。

例（平日 6:00 JST）:

```
/schedule create
- name: cloud-prep
- cron: 0 6 * * 1-5
- timezone: Asia/Tokyo
- prompt: /cloud-prep
```

詳細は `README.md` の「スケジュール登録」セクションを参照。
