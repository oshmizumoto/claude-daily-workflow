---
name: setup
description: "claude-daily-workflowの初回セットアップ。プレースホルダー（YOUR_*）を実際のNotion/Slack ID・ユーザー情報に置き換える。Use when「セットアップして」「初期設定」「configure」「初めて使う」「placeholderを埋めて」と言われた時。"
---

# Setup Workflow

このリポジトリをユーザー個別の環境に適合させるための初回セットアップ手順。
プレースホルダー（`YOUR_*`）を実際の値に置換し、固有名詞をユーザー情報に置き換える。

## STEP1｜プレースホルダーの残存スキャン

リポジトリ全体に対して、以下のプレースホルダーが残っているかを `Grep` で確認する：

| プレースホルダー | 用途 |
|----------------|------|
| `YOUR_NOTION_TASK_DB_ID` | NotionタスクDB ID |
| `YOUR_NOTION_PARENT_PAGE_ID` | 日次ページの親ページID |
| `YOUR_NOTION_SPEC_PAGE_ID` | 仕様書・定期ルールページID |
| `YOUR_SLACK_CHANNEL_ID` | 通知先SlackチャンネルID |
| `#your-workflow-channel` | チャンネル表示名 |
| `（あなたのユーザー名 / 所属を記載）` | CLAUDE.md のユーザー欄 |

**全て検出されない場合** → セットアップ済みと判断し、「既にセットアップ済みです。再設定する場合は対象ファイルを直接編集してください。」と報告して終了。

**1つでも検出された場合** → STEP2へ進む。

---

## STEP2｜ユーザーから値を対話で収集

`AskUserQuestion` を使い、検出されたプレースホルダーに対応する値を順次収集する。
各質問に「スキップ」相当のオプションを設けず、必要な値のみを聞く（既に埋まっているものは聞かない）。

### 質問テンプレート

1. **NotionタスクDB ID**
   - 質問: 「未完了タスクを管理しているNotionデータベースのIDを教えてください」
   - 入力形式: 32桁の英数字（ハイフンなし / ハイフンありどちらでも受理）
   - ヒント: NotionデータベースURL `https://www.notion.so/xxxx/<32桁ID>?v=...` の `<32桁ID>` 部分

2. **Notion親ページID**
   - 質問: 「日次ログページを作成する親ページのIDを教えてください」
   - 入力形式: 32桁の英数字

3. **Notion仕様書ページID**（オプショナル / プレースホルダー残存時のみ聞く）
   - 質問: 「定期ルール・仕様書を書いたページのIDを教えてください。なければ後で設定でもOKです」
   - スキップ可能（その場合プレースホルダーのまま残し、後でユーザーが手動設定）

4. **SlackチャンネルID**
   - 質問: 「業務開始・クローズ通知を投稿するSlackチャンネルのIDを教えてください」
   - 入力形式: `C` で始まる11文字以上の英数字
   - ヒント: Slack > チャンネル詳細 > 最下部「チャンネルID」

5. **Slackチャンネル表示名**
   - 質問: 「上記チャンネルの表示名（#xxx形式）を教えてください」
   - 入力形式: `#` で始まる文字列

6. **ユーザー名・所属**
   - 質問: 「CLAUDE.mdに記載するユーザー名と所属を教えてください（例: 山田太郎（株式会社XYZ））」

### 値の正規化

- NotionのIDは32桁（ハイフン除去後）であることを確認。ハイフン付きで入力された場合は除去する
- SlackチャンネルIDは大文字に揃える

---

## STEP3｜置換適用

収集した値で全ファイルを一括置換する。

### 対象ファイル

`.claude/` 配下の全 `.md` および `.json`、ルートの `CLAUDE.md` を対象とする。
`README.md` は STEP5 で別途処理する。

### 置換実装

`Grep` で各プレースホルダーを含むファイルを列挙し、`Edit`（`replace_all: true`）で置換する。

```
プレースホルダー → 収集値
YOUR_NOTION_TASK_DB_ID → <収集したID>
YOUR_NOTION_PARENT_PAGE_ID → <収集したID>
YOUR_NOTION_SPEC_PAGE_ID → <収集したID or 未設定ならそのまま>
YOUR_SLACK_CHANNEL_ID → <収集したID>
#your-workflow-channel → <収集した表示名>
（あなたのユーザー名 / 所属を記載） → <収集したユーザー名・所属>
```

---

## STEP4｜検証（任意・推奨）

置換した値が正しく動作するかを軽く検証する。失敗してもセットアップは継続する（警告のみ）。

### 4a. Notion検証

- `mcp__claude_ai_Notion__notion-fetch` でタスクDB IDと親ページIDを取得試行
- 取得失敗 → 「IDが不正、もしくはClaude AI Notion統合の権限不足の可能性があります。Notion側でClaude統合を承認してください」と警告

### 4b. Slack検証

- `mcp__claude_ai_Slack__slack_read_channel` で投稿可否を確認（限定的）
- 失敗時は警告のみ

検証はベストエフォート。失敗しても STEP5 に進む。

---

## STEP5｜README更新と完了報告

### README更新

`README.md` の「セットアップ」セクションを以下のように書き換える：

```markdown
## セットアップ

✅ セットアップ完了（YYYY-MM-DD）

再設定する場合は `.claude/skills/setup/SKILL.md` を参照、または対象ファイルを直接編集してください。

<details>
<summary>初回セットアップ手順（参考）</summary>

（元のセットアップ手順をここに収納）

</details>
```

元のセットアップ手順本体は `<details>` 内に保持し、再セットアップ時の参考にできるようにする。

### 完了報告

以下を画面に表示：

```
✅ セットアップ完了

置換内容:
- NotionタスクDB: xxxxxxxxxxxx
- Notion親ページ: xxxxxxxxxxxx
- Notion仕様書: xxxxxxxxxxxx (or 未設定)
- Slackチャンネル: Cxxxxxxxxxx (#xxx)
- ユーザー: <name>

次のステップ:
- `/work-start` で朝ワークフローを試してください
- `/cloud-prep` を `/schedule` 登録すると朝の起動が早くなります（STEP6参照）
- 仕様書ページIDが未設定の場合、agent-spec.md と CLAUDE.md を後で手動更新してください
```

---

## STEP6｜早朝事前準備のスケジュール登録（任意）

`AskUserQuestion` で「`/cloud-prep` を平日早朝にスケジュール登録しますか？」と確認する。

- **登録する** → `schedule` Skill を呼び出し、以下の内容で登録：
  - name: `cloud-prep`
  - cron: `0 6 * * 1-5`（平日 6:00 JST、ユーザー要望があれば変更）
  - timezone: `Asia/Tokyo`
  - prompt: `/cloud-prep`
- **後で登録する** → 「README の『スケジュール登録』を参照してください」と案内
- **使わない** → そのまま終了。`/work-start` はクラウドページ非存在時のフルモードで動作する

スケジュール登録が成功したら、`/schedule list` 相当の情報を表示して確認してもらう。

---

## エラー処理

- ユーザーが質問途中で中断した場合 → 既に収集した値だけ置換し「途中までセットアップしました。残りは再度 setup を実行してください」と報告
- 置換中にエラーが発生した場合 → そのファイル名と内容を提示し、手動修正を促す

## Resources

- なし（このSkillは外部assetを使用しない）
