---
name: notion-ops
description: "Notionのページ作成・セクション追記・タスク追加・ページ編集（pull/push）を行う。日次ページ作成やタスクDB操作、Notionページの大幅編集を依頼されたときに使用する。"
---

# Notion Operations Workflow

## Instructions

1. **Preflight**:
   - **ToolSearchでNotionツールをロードする**: `ToolSearch(query: "+notion update page")` を実行し、`mcp__claude_ai_Notion__notion-update-page` 等のツールを利用可能にする。これを行わないとNotion APIコールが失敗する
   - 操作対象のページIDまたはDBIDを確認する（CLAUDE.mdの固定リソースIDを参照）
   - `./assets/daily_page_template.md` を先に読み、テンプレート構造を確認する
   - 破壊的操作（ページ作成・タスク追加）は実行前にユーザーに内容を提示して確認する

2. **操作パターン別手順**:

   ### ページ作成（日次ページ）
   - `notion-create-pages` ツールを使用
   - `parent.page_id` に `YOUR_NOTION_PARENT_PAGE_ID` を指定
   - `properties.title` に `YYYYMMDD`（JST当日）を指定
   - `content` に `./assets/daily_page_template.md` の内容を埋めて指定

   ### セクション追記（壁打ちメモ等）
   - まず `notion-fetch` で現在のページ内容を取得
   - `notion-update-page`（`insert_content_after`）で追記
   - `selection_with_ellipsis` で対象セクション見出しを指定

   ### タスク追加
   - まず `notion-fetch` でDBのスキーマ（data_source_id）を確認
   - `notion-create-pages`（`parent.data_source_id`）でタスク追加
   - 必須プロパティ: タスク名（title）、ステータス: `TODO`

   ### ページ編集（pull → ローカル編集 → push）
   大幅な書き出し・肉付き等、複数回の編集が発生するNotion文書の更新に使用する。
   直接Notion APIを叩く往復を減らし、ローカル編集で高速化する。

   #### Step 1: Pull（Notion → ローカル）
   - `notion-fetch` で対象ページの内容を取得
   - `docs/notion-snapshots/{slug}.md` に保存
   - ファイル冒頭にfrontmatterを付与（フォーマットは下記参照）
   - `{slug}` はページ内容から判別できる短い英字スラッグ（例: `sabi-ki-outline`, `ist-seminar-design`）

   #### Step 2: Edit（ローカル編集）
   - Read/Edit/Write ツールでローカルファイルを自由に編集
   - Notion APIは呼ばない（高速・コンテキスト節約）
   - 編集中はfrontmatterの `status` を `editing` にする

   #### Step 3: Push（ローカル → Notion）
   - ローカルファイルからfrontmatterを除いた本文を取得
   - `notion-update-page`（`replace_content`）で一括反映
   - `page_id` はfrontmatterの `notion_page_id` を使用
   - 反映後、frontmatterの `status` を `synced`、`synced_at` を現在日時に更新
   - 破壊的操作のため、push前にユーザーに差分サマリを提示して確認する

   #### Frontmatterフォーマット
   ```yaml
   ---
   notion_page_id: "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
   notion_url: "https://www.notion.so/..."
   title: "ページタイトル"
   pulled_at: "2026-02-25T10:00:00+09:00"
   synced_at: "2026-02-25T12:00:00+09:00"  # push完了時に更新
   status: "editing"  # editing | synced
   ---
   ```

   #### 注意事項
   - pull時点のスナップショットであり、他ユーザーの並行編集との競合は検知しない
   - 長時間（1日以上）経過したスナップショットは再pullを推奨
   - 子ページ・データベースを含むページは `replace_content` 時に削除されないよう、タグを保持すること
   - `notion-search` が大量データを返す場合は、`notion-fetch` でページIDを直接指定して取得を優先する
   - 会議準備ドキュメント・アウトライン等もpull/push対象として `docs/notion-snapshots/` に保管可能

3. **エラー時**: エラー内容を記録してスキップし、最後にまとめて報告する

## Resources

- assets: `./assets/daily_page_template.md`
- questions: `./questions/notion_ops_questions.md`
- evaluation: `./evaluation/evaluation_guide.md`
- triggers: `./triggers/next_action_triggers.md`
- snapshots: `docs/notion-snapshots/`

## Next Action

- triggers: `./triggers/next_action_triggers.md`
