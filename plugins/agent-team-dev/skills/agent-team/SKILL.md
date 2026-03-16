---
name: agent-team
description: "チームでやって", "team task", "Agent Teamで実行", "並行で開発して", "タスクを分担"といった場面や、/team-taskコマンド実行時に使用する。Subagent（Agent tool）ではなくTeamCreate/SendMessageを使い、コンテキストを分散させて複数のteammateが並行して作業する。
---

# Agent Team起動

Subagent（Agent tool）ではなく、Agent Team（TeamCreate + SendMessage）を使用してチームベースの開発を行う。

## なぜAgent Teamを使うか

- **コンテキスト分散**: 各teammateが独立したコンテキストを持ち、メインセッションの負荷を軽減
- **並行作業**: 複数のteammateが同時に異なるタスクに取り組める
- **役割分担**: architect、implementer、reviewer等の専門ロールで品質向上
- **メッセージベース協調**: SendMessageによる明示的なコミュニケーション

## チーム作成手順

### 1. タスク分析

タスクの内容を分析し、必要なロールと**人数**を決定する:

| ロール | いつ必要か | 複数起動の目安 |
|--------|-----------|---------------|
| architect | 設計判断が必要な場合 | 通常1人で十分 |
| implementer | コード実装が必要な場合（ほぼ常に必要） | 独立したサブタスクの数に応じて複数 |
| reviewer | 品質保証が必要な場合 | 通常1人で十分 |
| tester | テスト作成・実行が必要な場合 | テスト対象の独立性に応じて複数 |
| docs-writer | ドキュメント更新が必要な場合 | 通常1人で十分 |

**同一ロールの複数起動**: タスクを独立したサブタスクに分割できる場合、同じロールのteammateを複数立てて並列に処理する。特にimplementerは並列実装が効果的なケースが多い。

### 2. TeamCreate実行

TeamCreateツールを呼び出してチームを作成する:

- **name**: タスクに関連した名前（例: `feat-auth-team`）
- **description**: チームの目的を簡潔に記述
- **members**: 必要なロールのteammateを配列で指定。各メンバーにはname、agentType、cwdを設定

**命名規則（複数起動時）**: 同一ロールを複数立てる場合、担当領域で命名する。
- 例: `implementer-api`, `implementer-ui`, `implementer-db`
- 例: `tester-unit`, `tester-e2e`

**全メンバーのcwdはworktreeのパスに設定する。**

### 3. タスク割り当て

SendMessageツールで各teammateにタスクを割り当てる:

1. **architect** に設計方針を依頼（必要な場合） → 結果を待つ
   - 複数implementerがいる場合、各implementerの担当範囲を明確に定義してもらう
2. **implementer（複数可）** に実装を依頼 → 各自が独立したサブタスクを並列で実装
   - architectの設計方針があればそれを含める
   - **各implementerの担当範囲が重複しないようにする**（ファイル競合を防ぐ）
3. 全implementerの実装完了後、**reviewer** にレビューを依頼 → 指摘があれば該当のimplementerに修正依頼
4. **tester（複数可）** にテスト作成を依頼（実装と **並行可能**）
5. **docs-writer** にドキュメント更新を依頼（実装と **並行可能**）

**並行化のポイント**:
- 複数のimplementerは同時に作業する（担当範囲が独立していること）
- testerとdocs-writerもimplementerと並行作業可能（ただし実装対象コードが必要な場合は完了を待つ）

### 4. 進捗管理

- 各teammateからのメッセージを確認し、進捗を把握
- ブロッカーがあれば調整・指示を出す
- 全タスク完了後、結果をまとめてユーザーに報告

## Predefined Teammates

このプラグインのagent定義ファイル（`agents/` ディレクトリ）に以下のテンプレートが用意されている:

- **architect** (`agents/architect.md`): 設計・アーキテクチャ策定、実装方針決定
- **implementer** (`agents/implementer.md`): コード実装、既存コードの修正 — **複数起動推奨**
- **reviewer** (`agents/reviewer.md`): コードレビュー、品質チェック
- **tester** (`agents/tester.md`): テスト作成・実行、テストカバレッジ確認
- **docs-writer** (`agents/docs-writer.md`): ドキュメント作成・更新

これらはあくまでテンプレート。タスクに応じて必要なメンバー・人数を選択する。

## カスタムTeammate

predefinedにないロールが必要な場合、**Leadがその場でカスタムteammateを定義して起用できる**。

### カスタムteammateの例

| 名前 | 用途 |
|------|------|
| analyzer | コードベースの調査・分析 |
| migrator | データやスキーマのマイグレーション |
| security-auditor | セキュリティ監査 |
| performance-optimizer | パフォーマンス最適化 |
| devops | CI/CD・インフラ設定 |

### カスタムteammateの作成方法

TeamCreateのmembersに以下を指定する:

- **name**: ロールを表す名前（例: `analyzer`, `security-auditor-1`）
- **agentType**: 任意の識別子（predefined agentと一致しなくてよい）
- **model**: タスクの複雑度に応じて選択
  - 分析・判断が必要: `opus`
  - 機械的な作業: `sonnet` または `haiku`
- **tools**: タスクに必要な最小限のツールセット
- **systemPrompt**: そのteammateの役割・手順・ルールを記述

### systemPrompt設計のガイドライン

```
あなたは[ロール名]として、[担当領域]を担当する。

## 役割
- [主要な責務]

## 手順
1. [具体的なステップ]

## ルール
- [守るべき制約]
```

- 役割を簡潔に定義する（長すぎるとコンテキストを圧迫する）
- 具体的な手順を含める
- ファイル変更を伴うロールには「作業完了後、変更があればgit-commitスキルに従いコミットする」手順を含める
- 調査・分析のみのロール（analyzerなど）にはコミット手順は不要

## ルール

- **同一ロールの複数起動は積極的に行う** — 独立したサブタスクがあれば並列化する
- **複数implementer間でファイル競合を避ける** — 担当範囲を明確に分離する
- **各teammateには明確なタスクを与える** — 曖昧な指示は避ける
- **teammateの作業結果を確認してから次のステップに進む**
