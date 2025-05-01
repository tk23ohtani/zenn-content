---
title: 初めてのMCP
emoji: 🤖
type: tech
topics:
  - AI
  - MCP
published: false
---

# 初めてのMCP

Created by T.Ohtani at 2025/04/30

## MCP Serverを使ってみる

まずはGitHubが用意してくれているGitHub MCP Serverを使ってみることで、MCPサーバーとはなんなのかを知る。いくつか方法がるが、Dockerから動作させるのが一番お手軽っぽい。

### WSL2

Docker Desktopは商用利用が有償になってしまったのでWSL2にCLI版のDockerを入れて使うことにする。インストール手順など詳細は、[公式ドキュメント](https://learn.microsoft.com/ja-jp/windows/wsl/setup/environment)による。
管理者モードでPowerShellを開き、インストールコマンドを入力する。

```ps
wsl --install
```

![alt text](/images/try-mcp-img-2.png)

再起動する。

![alt text](/images/try-mcp-img-3.png)

セットアップのベストプラクティスに従ってセットアップを続ける。

#### ユーザー名とパスワードの設定

```ps
wsl -u root
```

![alt text](/images/try-mcp-img-4.png)

#### パッケージの更新とアップグレード
Windowsターミナルを開き、Ubuntuを開く。

```bash
sudo apt update && sudo apt upgrade
```

#### ディストリビューションの複製

すぐに素の状態からやり直せるように、作業用のディストリビューションは複製して作る。なお、このステップは個人的趣向によるものなので、省略可。

```ps
wsl --list --verbose
  NAME                   STATE           VERSION
* Ubuntu                 Stopped         2
wsl --export Ubuntu DevUbuntu.tar
wsl --import DevUbuntu .\wsl_manual_install\ .\DevUbuntu.tar
rm .\DevUbuntu.tar
```
ディストリビューションがインポートされた時点でWindwosターミナルに新たなプロファイルが追加されるので、それを起動する。

#### Docker CLIのインストール

この辺り[Docker公式 for Ubuntu](https://docs.docker.com/engine/install/ubuntu/)に従ってインストールを行う。

- Setup Docker's `apt` repository.

```bash
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

- Install Docker Pakages

```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

- Verify that the installation is successful by running the `hello-world` image:

```bash
sudo docker run hello-world
```
![alt text](/images/try-mcp-img-5.png)

#### GitHubアクセストークンの生成

you will need to [Create a GitHub Personal Access Token](https://github.com/settings/personal-access-tokens). The MCP server can use many of the GitHub APIs, so enable the permissions that you feel comfortable granting your AI tools (to learn more about access tokens, please check out the documentation).

アクセストークン例（7日間有効のもので確認）

アクセス制限はガッチガチにしておく。要は全部アクセスなしにする。

トークン名：`McpSample`
トークン：メモっておく。初回アクセス時にトークン聞かれるので。

#### MCPサーバーの起動

- VS CodeをAgent Modeにする。
- `setting.json`にサーバーを追加する。

色々ハマりどころがある。
まず、直接dockerコマンドを実行できないので、WSLから起動させるようにする。
いちびって複数ディストリビューションにしてしまったので、ディストリビューションを指定してWSLを起動する必要がある。

```json
    "mcp": {
        "inputs": [
            {
            "type": "promptString",
            "id": "github_token",
            "description": "GitHub Personal Access Token",
            "password": true
            }
        ],        
        "servers": {
            "github": {
                "command": "wsl",
                "args": [
                    "-d",
                    "DevUbuntu",
                    "bash",
                    "-c",
                    "docker run --rm -i -e GITHUB_PERSONAL_ACCESS_TOKEN ghcr.io/github/github-mcp-server"
                ],
                "env": {
                    "GITHUB_PERSONAL_ACCESS_TOKEN": "${input:github_token}",
                    "WSLENV": "GITHUB_PERSONAL_ACCESS_TOKEN/u"
                }
            }
        }
    },
```

このように△Startが表示されるので、それを押すか、
![alt text](/images/try-mcp-img-6.png)

コマンドパレットから`MCP: List Servers`を押して、githubを選び、サーバーを起動する。

![alt text](/images/try-mcp-img-7.png)

![alt text](/images/try-mcp-img-8.png)

起動すれば、ツールリストに出てくる。
![alt text](/images/try-mcp-img-9.png)

動作確認。ここでは、直接MCPに対するコマンドを指定している。実際にはプロンプトで上手に指定と命令を行う。

![alt text](/images/try-mcp-img-10.png)

無事にMCPサーバー経由でGitHubへのアクセスができた。
