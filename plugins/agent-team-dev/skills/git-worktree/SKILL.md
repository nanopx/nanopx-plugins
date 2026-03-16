---
name: git-worktree
description: "worktree作成", "create a worktree", "並行開発環境", "ブランチを隔離して作業"といった操作や、Agent Teamが隔離された開発環境を必要とする場面で使用する。.worktrees/配下にworktreeを作成し、依存関係のセットアップまで行う。
---

# Git Worktree管理

プロジェクトルートの `.worktrees/` にworktreeを作成・管理する。

## 前提条件

```bash
# gh CLIがインストール・認証済みか確認
gh auth status
```

`gh` CLIが利用できない場合はエラーを報告し、インストール・認証を促す。

## Worktree作成手順

### 1. プロジェクトルートの特定

```bash
project_root=$(git rev-parse --show-toplevel)
```

### 2. 作業状態の確認

```bash
cd "$project_root"
original_branch=$(git branch --show-current)
# 未コミットの変更がないか確認
if ! git diff --quiet || ! git diff --cached --quiet; then
  echo "未コミットの変更があります。stashしてから続行します。"
  git stash push -m "auto-stash before worktree creation"
fi
```

### 3. 最新のベースを取得

```bash
git fetch origin
base_branch=$(gh repo view --json defaultBranchRef -q '.defaultBranchRef.name')
git checkout "$base_branch"
git pull origin "$base_branch"
```

### 4. .worktrees/ の準備

```bash
mkdir -p "$project_root/.worktrees"

# .gitignoreに.worktrees/がなければ追加
if ! git check-ignore -q .worktrees 2>/dev/null; then
  echo ".worktrees/" >> .gitignore
  git add .gitignore
  git commit -m "chore: add .worktrees/ to .gitignore"
fi
```

### 5. Worktree作成

```bash
branch_name="<feature-branch-name>"
git worktree add "$project_root/.worktrees/$branch_name" -b "$branch_name"
cd "$project_root/.worktrees/$branch_name"
```

### 6. 依存関係のセットアップ

worktreeディレクトリ（`$project_root/.worktrees/$branch_name`）で、プロジェクトに応じて自動検出・実行（該当ファイルがない場合はスキップ）:

```bash
cd "$project_root/.worktrees/$branch_name"
```

| ファイル | コマンド |
|---------|---------|
| package.json | `npm install` |
| Cargo.toml | `cargo fetch` |
| requirements.txt | `pip install -r requirements.txt` |
| pyproject.toml | `poetry install` |
| go.mod | `go mod download` |

### 7. ベースラインテスト（任意）

worktreeが正常かを確認するため、テストを実行して結果を報告する。

### 8. stashの復元（該当する場合）

```bash
# Step 2でstashした場合、元のブランチに戻してstashを復元
cd "$project_root"
git checkout "$original_branch"
git stash pop
```

## ブランチ名の規則

- `feat/<short-description>` — 新機能
- `fix/<short-description>` — バグ修正
- `refactor/<short-description>` — リファクタリング
- `docs/<short-description>` — ドキュメント

## Worktreeの削除

```bash
git worktree remove "$project_root/.worktrees/$branch_name"
```

## ルール

- **worktreeは必ず `.worktrees/` 配下に作成する**
- **.gitignoreへの追加を忘れない**
- **最新のデフォルトブランチから分岐する** — `gh repo view` でベースを特定し、fetch + pullしてから作成
- **未コミットの変更がある場合は自動stashする** — worktree作成後に復元する
- **`gh` CLIが利用可能であることを事前に確認する**
