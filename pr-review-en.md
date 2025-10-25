# Task

Conduct a code review for the specified Pull Request and create **PENDING** status review comments using the **GitHub API**.

- \<owner\>: $1
- \<repo\>: $2
- \<PR number\>: $3

For each review comment, make sure to include one of the following labels **at the beginning** of the comment, followed by a newline:

- **[change request]**: Use this label when a change is _mandatory_.
- **[suggestion]**: Use this label when a change is _not required_ but you want to _propose an improvement_.
- **[imo]**: Use this label when you want to express an _opinion_.
- **[imho]**: Use this label when you want to express an _opinion modestly_.
- **[nits]**: Use this label when the change is _not required_ because it doesn't affect functionality, but you want to _suggest a minor improvement_.
- **[question]**: Use this label when you want to _ask a question_ about the implementation’s intent or whether certain considerations were made.

---

# Execution Procedure

## 1. Retrieve PR Information

```bash
# Check PR details
gh pr view <PR number>

# Get PR diff
gh pr diff <PR number>
```

## 2. Create Review Comments

Use the **GitHub REST API** to create a **PENDING** review.

**Important specifications:**

- **event**: Omit this field to keep the review in **PENDING** status.
- **line**: Specify the _actual line number_ within the file.
- **side**: Specify `"RIGHT"` (new code) or `"LEFT"` (old code).
- **path**: Specify the _relative path_ of the target file.
- **position** parameter: **Do not use** (deprecated).

**API Format:**

```json
{
  "body": "Overall summary comment for the review",
  "comments": [
    {
      "path": "backend/path/to/file.py",
      "line": 42,
      "side": "RIGHT",
      "body": "Specific comment on the line"
    }
  ]
}
```

**Execution Command:**

```bash
echo '<above JSON>' | gh api -X POST repos/<owner>/<repo>/pulls/<PR number>/reviews --input -
```

---

## 3. Verify Review Creation

### Retrieve Review ID

```bash
gh api repos/<owner>/<repo>/pulls/<PR number>/reviews --jq '.[] | {id, state, user:.user.login}'
```

### Check if Comments Were Registered Correctly

```bash
gh api repos/<owner>/<repo>/pulls/<PR number>/reviews/<review ID>/comments --jq '.[] | {path, line, body}'
```

---

## Error Handling

### When Comments Are Not Displayed

1. Check the number of comments associated with the review.
2. If the count is **0**, delete the review and recreate it.

**Delete Review:**

```bash
gh api -X DELETE repos/<owner>/<repo>/pulls/<PR number>/reviews/<review ID>
```
