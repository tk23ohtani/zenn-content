---
title: Kiro IDEでリモート接続したWSL2上でのKiro CLI
emoji: 👻
type: tech
topics:
  - AI
  - Agent
published: false
---

# Kiro CLI 導入手順書

WSL2環境でKiro CLIをインストールし、カスタムエージェントを生成してMCPサーバーと連携するまでの手順を説明します。

## Kiro IDEとの連携メリット

Kiro CLIをKiro IDEで開いたワークスペースで使用すると、以下のような利点があります：

### 自動的に適用される設定

- **ステアリングルール**: `.kiro/steering/*.md`に定義されたチームの開発規約やコーディング標準が自動的にエージェントに適用されます
- **MCP設定**: `.kiro/settings/mcp.json`で設定されたMCPサーバーがそのまま利用可能
- **エージェントフック**: IDE側で設定したフックがCLIからも利用できます
- **ワークスペースコンテキスト**: プロジェクト固有の設定やコンテキストが共有されます

### 一貫した開発体験

- IDEとCLIで同じエージェント設定を使用できるため、環境による動作の違いがありません
- チーム全体で統一されたAI支援を受けられます
- ローカル開発とCI/CDパイプラインで同じエージェントを活用可能

## 前提条件

- WSL2（Ubuntu）がインストール済みであること
- Kiro IDEでWSL2のワークスペースを開いていること
- ワークスペースにMCPサーバーが設定されていること

## 1. Kiro CLIのインストール

Kiro IDEで新規ターミナルを開く。

![alt text](/images/kiro_cli_on_ide_wsl2-2.png)

WSL2のUbuntuターミナルで以下のコマンドを実行します：

```bash
# 最新版の.debパッケージをダウンロード
wget https://desktop-release.q.us-east-1.amazonaws.com/latest/kiro-cli.deb

# パッケージをインストール
sudo dpkg -i kiro-cli.deb

# 依存関係の問題があれば解決
sudo apt-get install -f

# ダウンロードファイルを削除
rm kiro-cli.deb
```

![alt text](/images/kiro_cli_on_ide_wsl2-3.png)

インストールが完了したら、バージョンを確認します：

```bash
kiro-cli --version
```

正常にインストールされていれば、Kiro CLIのバージョン情報が表示されます。

![alt text](/images/kiro_cli_on_ide_wsl2-5.png)

## 2. Builder IDによる認証

Kiro CLIを使用するには認証する必要があります。

### 認証の開始

初回起動、もしくはログインコマンドで認証が開始されます。

```bash
kiro-cli login
```

複数の認証方法がありますが、個人利用の場合は`Builder ID`による認証を選択します。

![alt text](/images/kiro_cli_on_ide_wsl2-10.png)

認証方法を選択すると、表示されているURLを開けと言ってくるので、Ctrl+クリック（もしくはコピーしてアドレス貼り付け）で開きます。

![alt text](/images/kiro_cli_on_ide_wsl2-11.png)

ここでは、Waiting Listに登録していたりして既にKiro利用の登録済みGoogleアカウントで認証しています。

![alt text](/images/kiro_cli_on_ide_wsl2-6.png)

認証コードが正しいか目視で確認します。

![alt text](/images/kiro_cli_on_ide_wsl2-12.png)

Kiro-CLIの利用を許可します。

![alt text](/images/kiro_cli_on_ide_wsl2-7.png)

リクエストが無事に承認されました。

![alt text](/images/kiro_cli_on_ide_wsl2-8.png)

承認される、ターミナルでもKiro-CLIのログインが成功していることを確認します。

![alt text](/images/kiro_cli_on_ide_wsl2-13.png)

ちなみにKiro-CLIを終了するには`/quit`コマンドを使います。



このコマンドを実行すると、以下のような流れで認証が行われます：

1. ブラウザが自動的に開き、IDCログインページが表示されます
2. IDCアカウントの認証情報（メールアドレスとパスワード）を入力
3. 必要に応じて多要素認証（MFA）を完了
4. 認証が成功すると、ブラウザに成功メッセージが表示されます
5. ターミナルに戻り、認証完了が確認できます

### 認証状態の確認

現在の認証状態を確認するには：

```bash
kiro-cli status
```

出力例：

```
✓ Authenticated as: [email]@example.com
✓ Organization: Your Organization
✓ Token expires: 2025-12-27 10:30:00
```

### 認証トークンの更新

トークンの有効期限が切れた場合は、再度ログインします：

```bash
kiro-cli refresh
```

### ログアウト

認証を解除する場合：

```bash
kiro-cli logout
```

> **注意**: WSL2環境でブラウザが自動的に開かない場合は、表示されたURLを手動でブラウザにコピー＆ペーストしてください。

## 3. Kiro CLIの初期設定

ワークスペースのルートディレクトリに移動し、Kiro CLIを初期化します：

```bash
cd /path/to/your/workspace
kiro-cli init
```

このコマンドにより、`.kiro`ディレクトリが作成され、必要な設定ファイルが生成されます。

> **Kiro IDEで既に開いているワークスペースの場合**: `.kiro`ディレクトリが既に存在する場合、既存の設定（ステアリングルール、MCP設定、エージェントフックなど）がそのまま利用されます。`kiro-cli init`をスキップして次のステップに進んでください。

## 4. カスタムエージェントの作成

カスタムエージェントを生成します：

```bash
kiro-cli agent create my-custom-agent
```

対話形式で以下の情報を入力します：

- エージェント名：`my-custom-agent`
- 説明：エージェントの目的や役割
- 使用するモデル：GPT-4やClaude等から選択
- 追加の設定：必要に応じてカスタマイズ

生成されたエージェント設定ファイルは `.kiro/agents/my-custom-agent.json` に保存されます。

## 5. エージェント設定のカスタマイズ

生成されたエージェント設定ファイルを編集します：

```bash
code .kiro/agents/my-custom-agent.json
```

設定例：

```json
{
  "name": "my-custom-agent",
  "description": "MCPサーバーと連携するカスタムエージェント",
  "model": "gpt-4",
  "systemPrompt": "あなたは開発支援エージェントです。MCPツールを活用してタスクを実行してください。",
  "temperature": 0.7,
  "maxTokens": 4000,
  "tools": ["mcp"]
}
```

## 6. MCPサーバーの設定確認

ワークスペースのMCP設定を確認します：

```bash
cat .kiro/settings/mcp.json
```

MCPサーバーが正しく設定されていることを確認してください。例：

```json
{
  "mcpServers": {
    "sticky-note-canvas": {
      "command": "node",
      "args": ["./mcp-server/build/index.js"],
      "disabled": false,
      "autoApprove": []
    }
  }
}
```

## 7. カスタムエージェントの実行

カスタムエージェントを起動します：

```bash
kiro-cli agent run my-custom-agent
```

または、対話モードで起動：

```bash
kiro-cli chat --agent my-custom-agent
```

## 8. MCPツールの動作確認

エージェントとの対話で、MCPツールが使用可能か確認します：

```
> MCPサーバーで利用可能なツールを教えてください
```

エージェントがMCPツールの一覧を返答すれば、正常に連携できています。

実際にツールを使ってみます：

```
> 新しいフレームを作成してください。ラベルは "test_frame" でお願いします。
```

エージェントがMCPサーバーの `create_frame` ツールを呼び出し、フレームが作成されれば成功です。

## 9. エージェントのデバッグ

問題が発生した場合は、デバッグモードで実行します：

```bash
kiro-cli agent run my-custom-agent --debug
```

ログファイルを確認：

```bash
cat .kiro/logs/agent-debug.log
```

## トラブルシューティング

### 認証エラー

**症状**: `kiro-cli auth login`でブラウザが開かない

- WSL2の場合、表示されたURLを手動でブラウザにコピー
- `wsl-open`や`wslu`パッケージをインストールすると自動でブラウザが開くようになります：
  ```bash
  sudo apt install wslu
  ```

**症状**: 認証トークンの有効期限切れ

```bash
kiro-cli auth refresh
```

### MCPサーバーに接続できない

- MCPサーバーのプロセスが起動しているか確認
- `mcp.json`のパスが正しいか確認
- ポート番号の競合がないか確認

### エージェントがツールを認識しない

- エージェント設定で `"tools": ["mcp"]` が含まれているか確認
- MCPサーバーを再起動してみる
- Kiro IDEのMCP Server viewで接続状態を確認

### パーミッションエラー

```bash
chmod +x .kiro/agents/my-custom-agent.json
```

## 次のステップ

- エージェントにカスタムプロンプトを追加
- 複数のMCPサーバーを統合
- エージェントフックを設定して自動化
- チーム用のステアリングルールを作成

## 参考リンク

- [Kiro CLI Documentation](https://docs.kiro.ai/cli)
- [MCP Protocol Specification](https://modelcontextprotocol.io)
- [Custom Agent Examples](https://github.com/kiro-ai/examples)
- [Kiro CLI の紹介：Kiro エージェントをあなたのターミナルへ](https://aws.amazon.com/jp/blogs/news/introducing-kiro-cli/)

---
wrote by Kiro

