# Atomic Commit Plugin

変更された差分を分析し、意味のある単位でコミットをまとめる Claude Code プラグイン。各コミットは常にコンパイル可能で、ロールバック可能な状態を維持します。

## 特徴

- **Atomic Commits**: 各コミットは1つの論理的な変更のみを含む
- **Compilable**: すべてのコミットがビルド/コンパイルチェックをパス
- **Revertable**: 各コミットは独立してロールバック可能
- **Intelligent Analysis**: AI エージェントが変更を分析し、最適なコミット戦略を提案
- **Dependency Aware**: ファイル間の依存関係を考慮した順序でコミット

## インストール

### マーケットプレイスから

```bash
# マーケットプレイスを追加
/plugin marketplace add watarus/watarus-claude-plugins

# プラグインをインストール
/plugin install atomic-commit
```

### ローカルから

```bash
# リポジトリをクローン
git clone https://github.com/watarus/watarus-claude-plugins.git

# プラグインディレクトリを Claude Config にシンボリックリンク
ln -s /path/to/watarus-claude-plugins/atomic-commit ~/.claude/plugins/atomic-commit
```

## 使い方

### 基本的な使用方法

```bash
# 現在の変更を分析してアトミックコミットを作成
/atomic-commit
```

### 実行フロー

1. **変更の分析**
   - Git status と diff を取得
   - プロジェクトのビルドシステムを検出

2. **コミット計画の作成**
   - commit-analyzer エージェントが変更を分析
   - 論理的なグループ化と依存関係の特定
   - 各コミットのメッセージを生成

3. **計画の確認**
   - 提案されたコミット計画をレビュー
   - 必要に応じて修正

4. **コミットの実行**
   - 各コミットグループを順番に処理
   - ビルド/コンパイルチェックを実行
   - 成功したらコミット、失敗したら対処を選択

5. **完了報告**
   - 作成されたコミットのサマリー

## 使用例

### シナリオ 1: 新機能の追加

```
変更内容:
- user.go (新しいユーザー認証機能)
- database.go (ユーザーテーブルスキーマ)
- user_test.go (テスト)
- README.md (ドキュメント更新)

コミット計画:
1. feat(database): add user table schema
2. feat(auth): implement user authentication
3. test(auth): add authentication tests
4. docs: update README with auth usage
```

### シナリオ 2: バグ修正とリファクタリング

```
変更内容:
- api.go (バグ修正とリファクタリング)
- api_test.go (テスト更新)

コミット計画:
1. fix(api): correct error handling in POST endpoint
2. refactor(api): extract validation logic to helper
3. test(api): update tests for new validation
```

### シナリオ 3: 密結合した変更

```
変更内容:
- 複数ファイルが密接に関連している

コミット計画:
1つのコミット: すべての変更を含む
理由: 変更が密結合しており、分割すると
      コンパイルできない中間状態になる
```

## コンポーネント

### コマンド

#### `/atomic-commit`

現在の Git 作業ディレクトリの変更を分析し、アトミックコミットを作成します。

**フロー:**
1. Git status/diff を取得
2. ビルドコマンドを検出
3. commit-analyzer エージェントを起動
4. コミット計画を確認
5. 各コミットを実行（ビルドチェック付き）
6. サマリーを報告

### エージェント

#### `commit-analyzer`

変更の分析とコミット戦略の立案を行う専門エージェント。

**責任:**
- 変更されたファイルの内容を詳細に分析
- 論理的に関連する変更をグループ化
- 各グループが独立してコンパイル可能か判断
- 依存関係を考慮してコミット順序を決定
- 各コミットに適切なメッセージを生成（Conventional Commits 形式）

**使用ツール:**
- Bash (git 操作)
- Read (ファイル内容の読み取り)
- Grep (依存関係の検索)
- Glob (関連ファイルの検出)

## 対応言語/フレームワーク

プラグインは以下のビルドシステムを自動検出します:

- **Node.js**: package.json → `npm run build` / `npm test`
- **Go**: go.mod → `go build ./...` / `go test ./...`
- **Rust**: Cargo.toml → `cargo build`
- **Python**: setup.py → `python setup.py build`
- **Java**: build.gradle / pom.xml → `./gradlew build` / `mvn compile`
- **Make**: Makefile → `make` / `make build`

検出できない場合は、ビルドコマンドを入力するよう求められます。

## コミットメッセージ形式

Conventional Commits 形式を使用:

```
<type>(<scope>): <subject>

<body>

<footer>
```

**Types:**
- `feat`: 新機能
- `fix`: バグ修正
- `refactor`: リファクタリング
- `test`: テストの追加/更新
- `docs`: ドキュメント
- `style`: コードスタイル
- `chore`: ビルドプロセス等

## 設定

現在、設定ファイルはありません。すべての設定は実行時に自動検出または対話的に決定されます。

## トラブルシューティング

### ビルドが失敗する

コミット作成中にビルドが失敗した場合、以下の選択肢が提示されます:

1. **Skip**: そのコミットをスキップして次へ
2. **Modify and Retry**: ファイルを修正してからリトライ
3. **Abort**: プロセス全体を中止

### 変更が複雑すぎる

エージェントが分析に苦労する場合:

1. 一部の変更を手動でコミット
2. 残りの変更で `/atomic-commit` を再実行
3. または、エージェントの提案を参考に手動でコミット

## 制限事項

- リモートへの自動プッシュは行いません（レビュー後に手動でプッシュ）
- 大量の変更（50+ ファイル）は分析に時間がかかる場合があります
- バイナリファイルの変更は内容分析できません
- マージコンフリクトがある場合は事前解決が必要です

## ライセンス

MIT License

## 作者

watarus

## 貢献

Issue や Pull Request を歓迎します！

## 関連リンク

- [Claude Code Documentation](https://docs.claude.com/en/docs/claude-code)
- [Conventional Commits](https://www.conventionalcommits.org/)
- [Atomic Commits Guide](https://www.freshconsulting.com/insights/blog/atomic-commits/)
