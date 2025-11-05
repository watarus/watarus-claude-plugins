# Testing Guide for Atomic Commit Plugin

## ローカルテスト

### 1. プラグインのインストール

```bash
# マーケットプレイスを追加
/plugin marketplace add /Users/zdzd/src/github.com/watarus/watarus-claude-plugins/marketplace.json

# プラグインをインストール
/plugin install atomic-commit

# インストール確認
/plugin list
```

### 2. テストシナリオ

#### シナリオ A: シンプルな変更（2-3ファイル）

1. テスト用の変更を作成:
```bash
# テストリポジトリを作成
mkdir -p /tmp/atomic-commit-test
cd /tmp/atomic-commit-test
git init

# 初期ファイル
echo "package main" > main.go
echo "func main() {}" >> main.go
git add main.go
git commit -m "Initial commit"

# 複数の変更を追加
echo "func Hello() string { return \"Hello\" }" >> main.go
echo "package main" > utils.go
echo "func Add(a, b int) int { return a + b }" >> utils.go
echo "# Test Project" > README.md
```

2. プラグインを実行:
```bash
/atomic-commit
```

3. 期待される動作:
- Git status/diff が取得される
- ビルドコマンドが検出される（go build）
- commit-analyzer が変更を分析
- 論理的なコミット計画が提示される
- 確認後、各コミットが作成される
- 各コミット後に `go build` が実行される

4. 検証:
```bash
# コミット履歴を確認
git log --oneline

# 各コミットが個別にビルド可能か確認
git checkout HEAD~2
go build
git checkout HEAD~1
go build
git checkout HEAD
go build
```

#### シナリオ B: 依存関係のある変更

1. テスト用の変更を作成:
```bash
cd /tmp/atomic-commit-test

# 基礎となる型定義
echo "package main" > types.go
echo "type User struct { Name string }" >> types.go

# 型を使用する実装
echo "package main" > service.go
echo "func GetUser() User { return User{Name: \"test\"} }" >> service.go

# テスト
echo "package main" > service_test.go
echo "import \"testing\"" >> service_test.go
echo "func TestGetUser(t *testing.T) {}" >> service_test.go
```

2. プラグインを実行:
```bash
/atomic-commit
```

3. 期待される動作:
- types.go が最初のコミット（基礎）
- service.go が2番目のコミット（実装）
- service_test.go が3番目のコミット（テスト）
- 各コミットが独立してビルド可能

#### シナリオ C: ビルドエラーの処理

1. テスト用の変更を作成:
```bash
cd /tmp/atomic-commit-test

# わざと壊れたコードを追加
echo "func Broken() { undefined_function() }" >> main.go
```

2. プラグインを実行:
```bash
/atomic-commit
```

3. 期待される動作:
- ビルドエラーが検出される
- Skip/Retry/Abort の選択肢が提示される
- ユーザーが対処方法を選択できる

#### シナリオ D: 密結合した変更

1. テスト用の変更を作成:
```bash
cd /tmp/atomic-commit-test

# 相互に依存する変更
echo "type Config struct { Value string }" > config.go
sed -i '' 's/func main() {}/func main() { cfg := Config{Value: "test"} }/' main.go
```

2. プラグインを実行:
```bash
/atomic-commit
```

3. 期待される動作:
- エージェントが密結合を検出
- 単一のコミットとして提案
- 理由が明確に説明される

### 3. エージェントの個別テスト

commit-analyzer エージェントを直接テストする場合:

```bash
# Task ツールで直接起動
Use Task tool with subagent_type=commit-analyzer

Prompt:
Analyze the following changes and create an atomic commit plan:

Git Status:
[paste git status output]

Staged Changes:
[paste git diff --staged output]

Unstaged Changes:
[paste git diff output]

Build Command: go build ./...

Please provide:
1. Logical grouping of changes
2. Commit order with dependency analysis
3. Commit messages for each group
4. Files to include in each commit
```

期待される出力:
- 構造化されたコミット計画
- 各コミットの理由
- 依存関係グラフ
- Conventional Commits 形式のメッセージ

### 4. 異なる言語/フレームワークでのテスト

#### Node.js/TypeScript

```bash
mkdir -p /tmp/atomic-commit-test-js
cd /tmp/atomic-commit-test-js
npm init -y
echo '{"compilerOptions": {"strict": true}}' > tsconfig.json

# 変更を作成
echo "export interface User { name: string }" > user.ts
echo "import { User } from './user'; export const getUser = (): User => ({ name: 'test' })" > service.ts

# テスト
/atomic-commit
```

#### Python

```bash
mkdir -p /tmp/atomic-commit-test-py
cd /tmp/atomic-commit-test-py
touch setup.py

# 変更を作成
echo "class User: pass" > user.py
echo "from user import User\ndef get_user(): return User()" > service.py

# テスト
/atomic-commit
```

### 5. エッジケース

#### 変更なし
```bash
cd /tmp/atomic-commit-test
git status  # clean

/atomic-commit
# 期待: "No changes to commit" メッセージ
```

#### ステージ済み変更のみ
```bash
cd /tmp/atomic-commit-test
echo "test" > test.txt
git add test.txt

/atomic-commit
# 期待: ステージ済み変更を分析するか、unstaged も含めるか確認
```

#### 大量の変更（50+ファイル）
```bash
for i in {1..60}; do echo "file $i" > file$i.txt; done
/atomic-commit
# 期待: 分析に時間がかかることを警告、進捗を表示
```

## デバッグ

### デバッグモードでの実行

```bash
claude --debug
# プラグインのロードメッセージとエラーを監視
```

### ログの確認

```bash
# Claude Code のログを確認
tail -f ~/.claude/logs/claude.log
```

### 一般的な問題

1. **プラグインが見つからない**
   - `plugin.json` が正しい場所にあるか確認
   - マーケットプレイスのパスが正しいか確認

2. **エージェントが起動しない**
   - `agents/commit-analyzer.md` のYAMLフロントマターを確認
   - ツール名のスペルミスをチェック

3. **ビルドコマンドが検出されない**
   - サポートされているビルドファイルが存在するか確認
   - 手動でビルドコマンドを指定できるようにする

## 成功基準

すべてのテストシナリオで:
- ✓ プラグインが正常にロードされる
- ✓ コマンドが利用可能
- ✓ エージェントが正しく起動する
- ✓ 変更が適切に分析される
- ✓ コミット計画が論理的
- ✓ 各コミットがビルド可能
- ✓ エラーが適切に処理される
- ✓ ユーザーフィードバックが明確

## レポート

テスト結果を記録:

```markdown
## テスト結果

**日付**: [date]
**バージョン**: 1.0.0

### シナリオ A: シンプルな変更
- Status: ✓ Pass / ✗ Fail
- Notes: [observations]

### シナリオ B: 依存関係のある変更
- Status: ✓ Pass / ✗ Fail
- Notes: [observations]

### シナリオ C: ビルドエラーの処理
- Status: ✓ Pass / ✗ Fail
- Notes: [observations]

### シナリオ D: 密結合した変更
- Status: ✓ Pass / ✗ Fail
- Notes: [observations]

### 発見された問題
1. [issue description]
2. [issue description]

### 改善提案
1. [suggestion]
2. [suggestion]
```
