---
name: team-task
description: Agent Teamを起動してタスクを自律的に実装する。worktreeで隔離環境を作り、適切なteammateでチームを構成し、タスクを分担実行する。
argument-hint: <タスクの説明>
allowed-tools: Bash, Read, Write, Edit, Glob, Grep, LS, TeamCreate, SendMessage, TaskCreate, TaskUpdate, TaskList, TaskGet
---

## コンテキスト

- 現在のブランチ: !`git branch --show-current`
- プロジェクトルート: !`git rev-parse --show-toplevel 2>/dev/null || pwd`
- Git状態: !`git status --short`
- CLAUDE.md: !`cat CLAUDE.md 2>/dev/null || echo "(CLAUDE.mdなし)"`

## タスク

ユーザーから与えられたタスクを、Agent Teamを使って自律的に実装する。

**ユーザーのタスク:** `$ARGUMENTS`

## 重要な制約

**Lead（このセッション）はSubagent（Agent tool）の使用を禁止する。** コンテキスト利用を分散させるため、必ずTeamCreate + SendMessageでAgent Teamを使用すること。（注: この制約はteammateには適用されない。teammateは必要に応じてSubagentを使用してよい。）

## 実行手順

### Phase 1: Worktree作成

**git-worktreeスキルに従ってworktreeを作成する。**

タスク内容からブランチ名を決定し（`feat/xxx`, `fix/xxx` 等）、git-worktreeスキルの手順に沿ってworktreeを作成する。

### Phase 2: チーム構成 → タスク実行

**agent-teamスキルに従ってチームを構成し、タスクを実行する。**

- タスク内容を分析し、必要なロール・人数・カスタムロールを決定
- TeamCreateでチームを作成（全メンバーのcwdをworktreeのパスに設定）
- SendMessageでタスクを割り当て、進捗を管理
- TaskCreate/TaskUpdateでサブタスクの進捗を追跡する
- 詳細な手順はagent-teamスキルを参照

事前確認が必要な場合（要件が不明確、複数アプローチ、破壊的変更）のみユーザーに確認する。曖昧さがなければそのまま実行する。

### Phase 3: 完了報告

全タスク完了後、以下をユーザーに提示する:

```
## 完了サマリ

### タスク
<元のタスク内容>

### 変更内容
<主要な変更点を箇条書き>

### 変更ファイル
<変更されたファイルの一覧（git diff --stat）>

### テスト結果
<テスト実行結果（ある場合）>

### Worktree
<worktreeのパス>
<ブランチ名>

---

次のアクションを選択してください:
1. **PR作成** — Draft PRを作成する
2. **修正** — 追加の修正を依頼する
3. **破棄** — worktreeを削除して変更を破棄する
```

ユーザーが「PR作成」を選択した場合: git-pushスキルでpushし、git-prスキルでDraft PRを作成する。
ユーザーが「修正」を選択した場合: 修正内容を聞いてPhase 2に戻る。
ユーザーが「破棄」を選択した場合: `git worktree remove` でworktreeを削除する。
