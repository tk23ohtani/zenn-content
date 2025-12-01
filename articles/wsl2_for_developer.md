---
title: インストール先を指定したWSL2のセットアップ
emoji: 🐧
type: tech
topics:
  - WSL2
  - Ubuntu
published: false
---

# インストール先を指定したWSL2のセットアップ

コンテンツそのものはGit等にリモート保存するので作業用ディスクとしている`D:`ドライブ等にディストリビューションを保存することで`C:`ドライブの空きを確保しておくようなオレオレな使い方の手順を示す。

## ハイパーバイザーを有効にしておく

前処理として、WSL2を使えるようにするには、ホストマシンの仮想アプリケーションを使えるようにしておく必要がある。超ニッチ？なオレオレ環境は、VMWare上のWindowsがホストマシンになるので、VMWare仮想マシンのプロセッサ設定で「この仮想マシンでハイパーバイザーアプリケーションを有効にする」にチェックを入れておく。

![alt text](/images/wsl2_for_developer-1.png)

## WSLコンポーネントの導入

まず、WSL2のコンポーネントをインストールする。ディストリビューションは後から手動で入れる。

```ps
# 必要なオプション コンポーネントのみをインストール
wsl --install --no-distribution
```

![alt text](/images/wsl2_for_developer-3.png)

## ディストリビューションの導入

- ディストリビューション名
- インストール先

```ps
# インストールパスのフォルダを準備（例えばD:ドライブに置く場合に使う）
mkdir C:¥WSL

# インストールパス及びディストリビューション名を指定してUbuntuをインストール
wsl --install Ubuntu --location C:¥WSL --name DevUbuntu
```

インストールの課程で、使用者のユーザー名とパスワードを設定して、セットアップを完了する。

![alt text](/images/wsl2_for_developer-2.png)

# 既存のディストリビューションをC:ドライブ以外に複製（移動）する

標準的なインストールによってC:ドライブにディストリビューションをセットアップしたものの、後から移動したくなった場合は、次のような手順で複製もしくは移動することができる。なお、WSL2は標準インストールされているものとする。

## ディストリビューションを準備

標準の`Ubuntu`ディストリビューションを複製して、D:ドライブに`DevUbuntu`として新たにLinuxディストリビューションを作る。インストールされている状態を確認する。

```powershell
wsl --list --verbose
```

問題なくインストールされていれば、標準の`Ubuntu`ディストリビューションが見えるはずなので、これをエクスポートする。

```
  NAME         STATE           VERSION
* Ubuntu       Stopped         2
```

標準の`Ubuntu`ディストリビューションをエクスポートしてD:ドライブ上に圧縮ファイルを作る。

```powershell
wsl --export Ubuntu D:\DevUbuntu.tar
```

## ディストリビューションを作成

作成した圧縮ファイルをD:ドライブ上にインポートして新たに`DevUbuntu`として新たにLinuxディストリビューションを作る。インポート先のディレクトリは予め作っておくと良い。圧縮ファイルが不要な場合は、`rm`コマンドで削除しておくと良い。

```powershell
mkdir D:\WSL2
wsl --import DevUbuntu D:\WSL2 D:\DevUbuntu.tar
rm D:\DevUbuntu.tar
```

## デフォルトのディストリビューションを指定（任意）

標準ディストリビューションは可能な限り温存しておくので、デフォルトのディストリビューションを今回作成したディストリビューションに変更しておく。

```powershell
wsl --set-default DevUbuntu
wsl --list --verbose
```

デフォルトのディストリビューションには`*`が付く。

```
  NAME         STATE           VERSION
* DevUbuntu    Stopped         2
  Ubuntu       Stopped         2
```

## 不要になるディストリビューションを削除する（任意）




