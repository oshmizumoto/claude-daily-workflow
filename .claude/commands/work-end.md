# /work-end - クローズ処理（Notion同期・Slack通知）

外部サービスへの同期・通知を実行する。
`/brainstorm` 終了後に任意で実行する。ローカルログは `/brainstorm` 内で記録済みの前提。
全STEPを確認なしで連続実行する。

---

## STEP1｜ローカルログ補完

`logs/YYYYMMDD.md` を読み込み、壁打ちメモ・アクションアイテムが記録済みか確認する。

- 記録済み → そのままSTEP2へ
- 未記録（`/brainstorm` を経由せずに直接実行された場合） → セッションの会話内容をもとに追記する
  - テンプレート: `.claude/skills/notion-ops/assets/daily_page_template.md`
- ファイル自体が存在しない場合 → テンプレートから当日ログを生成してから追記する

---

## STEP2｜Notion同期（一括実行）

以下を一括で実行する。個別の確認は不要。

1. **デイリーページの同期**:
   - 親ページ（`YOUR_NOTION_PARENT_PAGE_ID`）配下に `YYYYMMDD` ページが既に存在するか確認する（クラウド事前準備ページ or 既存同期ページ）
   - **既存ページあり** → `notion-update-page`（`replace_content`）で `logs/YYYYMMDD.md` の内容を上書きする（重複ページ作成を防止）
   - **既存ページなし** → `notion-create-pages` で新規作成し、`logs/YYYYMMDD.md` の内容を転写する
2. STEP1で追加したアクションアイテムをタスクDB（`YOUR_NOTION_TASK_DB_ID`）に追加
3. Notion同期が失敗した場合はスキップしてSTEP3に進む

---

## STEP3｜Slack クローズ報告

`.claude/skills/slack-notify/SKILL.md` を参照して投稿：

- チャンネル: `YOUR_SLACK_CHANNEL_ID`（#your-workflow-channel）
- 内容: `slack-notify/assets/close_template.md` に従う
- Notion同期が成功していればNotionページURLを含める
- 失敗していれば「Notion同期未完了」と付記する
- 投稿前の確認は不要。そのまま投稿する。

---

## 完了後

「クローズ完了」と報告する。ワークフロー改善提案がある場合のみ簡潔に提示する。
