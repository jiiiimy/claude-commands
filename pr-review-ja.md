# タスク

指定された Pull Request に対してコードレビューを実施し、GitHub API を使用して PENDING ステータスのレビューコメントを作成してください。

- \<owner\>: $1
- \<repo\>: $2
- \<PR 番号\>: $3

レビューの各コメントの冒頭には以下のラベルを必ず付与するようにしてください。ラベルの後は改行してください。

- **[change request]**: 必ず変更してほしいときに使用するラベル
- **[suggestion]**: 変更必須ではないが、変更をを案したいときに使用するラベル
- **[imo]**: 意見を述べるときに使用するラベル
- **[imho]**: のののない意見を述べるときに使用するラベル
- **[nits]**: 動作に影響はしないので変更必須ではないが、些細な改善を提案する時に使用するラベル
- **[quetion]**: 意図の確認や考慮の必要性など実装内容について質問するときに使用するラベル

# 実行手順

## 1. PR 情報の取得

```bash
# PR詳細の確認
gh pr view <PR番号>

# PR差分の取得
gh pr diff <PR番号>
```

## 2. レビューコメントの作成

GitHub REST API を使用して PENDING レビューを作成します。

**重要な仕様:**

- event: PENDING ステータスにするため指定しない
- line: ファイル内の実際の行番号を指定
- side: "RIGHT"（新しいコード）または"LEFT"（古いコード）を指定
- path: レビュー対象ファイルの相対パス
- position パラメータは使用しない(非推奨)

API フォーマット:

```
{
  "body": "レビュー全体の概要コメント",
  "comments": [
    {
    "path": "backend/path/to/file.py",
    "line": 42,
    "side": "RIGHT",
    "body": "行に対する具体的なコメント"
    }
  ]
}
```

実行コマンド:
echo '<上記 JSON>' | gh api -X POST repos/<owner>/<repo>/pulls/<PR 番号>/reviews --input -

## 3. レビュー作成後の確認

### レビュー ID を取得

```
gh api repos/<owner>/<repo>/pulls/<PR 番号>/reviews --jq '.[] | {id, state, user:.user.login}'
```

### コメントが正しく登録されているか確認

```
gh api repos/<owner>/<repo>/pulls/<PR 番号>/reviews/<レビュー ID>/comments --jq '.[] | {path, line, body}'

```

## エラー時の対処

### コメントが表示されない場合

1. レビューに紐づくコメント数を確認
2. 0 件の場合は、レビューを削除して再作成

**レビュー削除**
gh api -X DELETE repos/<owner>/<repo>/pulls/<PR 番号>/reviews/<レビュー ID>

**レビューの Submit**

ユーザーが UI 上で確認・編集してから Submit することを想定している場合は、PENDING のまま残す。

## 注意事項

- PENDING レビューはレビュー作成者のみ UI 上で確認可能
- Files changed タブでコメントを確認するには、少なくとも 1 つの行コメントが必要
- 本文のみのレビューは UI に表示されない
- 行番号はファイルの実際の行番号を使用（diff 表示の行番号ではない）
