---
name: git-commit
description: "commit", "git commit", "コミットして", "変更をコミット"といった操作や、Agent Teamのteammateが自律的にコミットする場面で使用する。変更内容を分析し、Conventional Commits形式のメッセージを生成して確認なしでコミットする。
---

# Git Commit（自律実行）

確認なしでgitコミットを作成する。

## 手順

1. `git status` と `git diff HEAD` で変更内容を把握
2. `git log --oneline -5` で直近のコミットメッセージのスタイルを確認
3. 変更内容を分析し、Conventional Commits形式でメッセージを生成
4. 関連ファイルを `git add` でステージング
5. `git commit` を実行

## ルール

- **確認不要**: ユーザーへの確認は一切行わず、即座にコミットする
- **Conventional Commits**: `feat:`, `fix:`, `refactor:`, `docs:`, `test:`, `chore:` などのprefixを使用
- **メッセージは英語**: コミットメッセージは英語で書く
- **1コミット1目的**: 論理的に異なる変更は分割してコミットする
- **秘密情報を含むファイル（.env等）はステージングしない**
- **全操作を1メッセージで完了させる**: tool callを並列実行し、テキスト出力は最小限にする（エラー時のみ報告）

## コミットメッセージ形式

```
<type>(<scope>): <subject>

<body（必要な場合のみ）>
```

- subject: 50文字以内、英語、命令形
- body: なぜその変更が必要かを説明（複雑な変更の場合のみ）

## 使用するツール

- `Bash(git status:*)`
- `Bash(git diff:*)`
- `Bash(git log:*)`
- `Bash(git add:*)`
- `Bash(git commit:*)`
