# タスク
指定されたPull Requestに対してコードレビューを実施し、GitHub APIを使用してPENDINGステータスのレビューコメントを作成してください。
<owner>: $1
<repo>: $2
<PR番号>: $3

レビューの各コメントの冒頭には以下のラベルを必ず付与するようにしてください。ラベルの後は改行してください。
 - [change request]: 必ず変更してほしいときに使用するラベル
 - [suggestion]: 変更必須ではないが、変更をを案したいときに使用するラベル
 - [imo]: 意見を述べるときに使用するラベル
 - [imho]: のののない意見を述べるときに使用するラベル
 - [nits]: 動作に影響はしないので変更必須ではないが、些細な改善を提案する時に使用するラベル
 - [quetion]: 意図の確認や考慮の必要性など実装内容について質問するときに使用するラベル

# 実行手順

## 1. PR情報の取得
```bash
# PR詳細の確認
gh pr view <PR番号>

# PR差分の取得
gh pr diff <PR番号>

## 2. レビューコメントの作成

GitHub REST APIを使用してPENDINGレビューを作成します。

重要な仕様:
- event: PENDINGステータスにするため指定しない
- line: ファイル内の実際の行番号を指定
- side: "RIGHT"（新しいコード）または"LEFT"（古いコード）を指定
- path: レビュー対象ファイルの相対パス
- positionパラメータは使用しない(非推奨)

APIフォーマット:
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

実行コマンド:
echo '<上記JSON>' | gh api -X POST repos/<owner>/<repo>/pulls/<PR番号>/reviews --input -

## 3. レビュー作成後の確認

### レビューIDを取得
gh api repos/<owner>/<repo>/pulls/<PR番号>/reviews --jq '.[] | {id, state, user:.user.login}'

### コメントが正しく登録されているか確認
gh api repos/<owner>/<repo>/pulls/<PR番号>/reviews/<レビューID>/comments --jq '.[] | {path, line, body}'

## エラー時の対処

### コメントが表示されない場合

1. レビューに紐づくコメント数を確認
2. 0件の場合は、レビューを削除して再作成

*レビュー削除*
gh api -X DELETE repos/<owner>/<repo>/pulls/<PR番号>/reviews/<レビューID>

*レビューのSubmit*

ユーザーがUI上で確認・編集してからSubmitすることを想定している場合は、PENDINGのまま残す。

## 注意事項

- PENDINGレビューはレビュー作成者のみUI上で確認可能
- Files changedタブでコメントを確認するには、少なくとも1つの行コメントが必要
- 本文のみのレビューはUIに表示されない
- 行番号はファイルの実際の行番号を使用（diff表示の行番号ではない）
