---
name: git-pr
description: "PR作成", "create a PR", "pull request", "PRを出して"といった操作や、Agent Teamのteammateが自律的にPRを作成する場面で使用する。gh CLIを使い、Summary/Changes/Test planを含む構造化されたdraft PRを確認なしで作成する。
---

# Git PR作成（自律実行）

確認なしでGitHub Draft PRを作成する。

## 前提条件

```bash
# gh CLIがインストール・認証済みか確認
gh auth status
```

`gh` CLIが利用できない場合はエラーを報告し、インストール・認証を促す。

## 手順

1. `git branch --show-current` で現在のブランチを確認
2. `gh repo view --json defaultBranchRef -q '.defaultBranchRef.name'` でベースブランチを特定
3. `git log --oneline <base>..HEAD` でコミット一覧を取得
4. `git diff <base>...HEAD --stat` で変更ファイルの概要を把握
5. 未pushのコミットがあれば、**git-pushスキルに従ってpushする**（安全チェック含む）
6. `gh pr create --draft` でDraft PRを作成

## PRの形式

```bash
gh pr create --draft --base "<base_branch>" --title "<title>" --body "$(cat <<'EOF'
## Summary
<変更内容を1-3行で説明>

## Changes
<主要な変更点をbulletで列挙>

## Test plan
<テスト方法をbulletで列挙>
EOF
)"
```

## ルール

- **確認不要**: ユーザーへの確認は一切行わず、即座にPRを作成する
- **常にDraft PRとして作成する** — `--draft` フラグを必ず付ける
- **`--base` フラグを必ず指定する** — Step 2で取得したベースブランチを明示的に渡す
- **タイトル**: 70文字以内、英語、変更の要約
- **説明**: 英語で記述。Summary、Changes、Test planのセクションを含む
- **pushにはgit-pushスキルを使う** — インラインでpushせず、git-pushスキルの安全チェック（main/master拒否、force push禁止）を経由する
- **全操作を1メッセージで完了させる**
- **`gh` CLIが利用可能であることを事前に確認する**

## 使用するツール

- `Bash(git branch:*)`
- `Bash(git log:*)`
- `Bash(git diff:*)`
- `Bash(gh repo view:*)`
- `Bash(gh pr create:*)`
- `Bash(gh pr view:*)`
- `Bash(gh auth:*)`
