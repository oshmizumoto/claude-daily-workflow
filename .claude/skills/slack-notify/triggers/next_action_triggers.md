# Next Action Triggers

## 起動条件
| ID | 起動条件 | 次アクション |
|----|---------|------------|
| T1 | /work-start STEP4 | 朝ワークサマリをSlack投稿（morning_template.md） |
| T2 | /work-end STEP3 | クローズ報告をSlack投稿（close_template.md） |

## スキップ条件
- APIエラーが発生した場合はスキップし「未取得」と記録して次に進む
- ユーザーが明示的にスキップを指定した場合
