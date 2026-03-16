---
name: git-push
description: "push", "git push", "プッシュして", "リモートにpush"といった操作や、Agent Teamのteammateが自律的にpushする場面で使用する。リモート状態を確認し、確認なしでpushする。main/masterへの直接pushは拒否する。
---

# Git Push（自律実行）

確認なしでgit pushを実行する。

## 手順

1. `git branch --show-current` で現在のブランチを確認
2. `git remote -v` でリモートを確認
3. `git status` でコミット済みかを確認（未コミットの変更があればgit-commitスキルに従いコミットする）
4. `git push` を実行（上流ブランチ未設定なら `-u origin <branch>` を使用）

## ルール

- **確認不要**: ユーザーへの確認は一切行わず、即座にpushする
- **main/masterへの直接pushは禁止**: main/masterブランチにいる場合はエラーを報告する
- **force pushは禁止**: `--force` や `-f` は使用しない
- **全操作を1メッセージで完了させる**

## エラー時

- push拒否された場合: `git pull --rebase` してから再push
- コンフリクト発生時: コンフリクト内容を報告し、解決を待つ

## 使用するツール

- `Bash(git branch:*)`
- `Bash(git remote:*)`
- `Bash(git status:*)`
- `Bash(git push:*)`
- `Bash(git pull:*)`
