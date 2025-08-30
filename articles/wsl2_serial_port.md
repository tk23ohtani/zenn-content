---
title: カスタムビルドしたWSL2でシリアル通信を使う
emoji: 🤖
type: tech
topics:
  - AI
published: false
---

# はじめに

最初に開発環境を分離したいので、ディストリビューションをコピーして専用の作業領域にする。本当はコンテナが使えれば良いのだが、技術的課題に加えてライセンスの課題もあるため、今回は断念してWSL2にて開発環境を整えてみる。

# カーネルのカスタムビルド

標準インストールされるWSLカーネルではデバイスドライバーが足りず接続できないことが多いので、二度手間にならないよう最初から必要なシリアルポートのデバイスドライバーを組み込んだカーネルをビルドし直しておく。

詳細については[GitHub WSL2-Linux-Kernel](https://github.com/microsoft/WSL2-Linux-Kernel)を参照し、最新の情報を確認すること。

## 必要なツールのインストール

```bash
$ sudo apt install build-essential flex bison dwarves libssl-dev libelf-dev cpio qemu-utils
```

## WSL2リポジトリをクローン

```bash
$ git clone https://github.com/microsoft/WSL2-Linux-Kernel.git
$ cd WSL2-Linux-Kernel
```

## WSL2カーネルの設定変更

ここで必要なデバイスドライバを有効にする。

```bash
$ make menuconfig KCONFIG_CONFIG=Microsoft/config-wsl
```

## ビルド

```bash
$ make KCONFIG_CONFIG=Microsoft/config-wsl && make INSTALL_MOD_PATH="$PWD/modules" modules_install
```



# ディストリビューションの複製

詳細については公式ドキュメント[WSL の基本的なコマンド | Microsoft Learn](https://learn.microsoft.com/ja-jp/windows/wsl/basic-commands)に従う。

これは複製の機能があるわけではなく、ディストリビューションのインポート、エクスポートの機能を使う。要はコピー元になるディストリビューションをエクスポートして、コピー先にインポートすれば良い。

- 方法

```powershell
> wsl --list --verbose
> wsl --export <Distribution Name> <FileName>
> wsl --import <Distribution Name> <InstallLocation> <FileName>
```

- 具体例

```powershell
> wsl --list --verbose
> wsl --export Ubuntu DevUbuntu.tar
> wsl --import DevUbuntu .\wsl_manual_install\ .\DevUbuntu.tar
> rm .\DevUbuntu.tar
```

デフォルトのユーザーが`root`で起動するので、規定のユーザーを変更しておく。

普通は次のコマンドで設定できるようだけど、複数のディストリビューションを使っている環境ではエラーになってうまく設定できないので、レジストリを操作する必殺技で設定する。

レジストリはここ。

```
[HKEY_CURRENT_USER\SOFTWARE\Microsoft\Windows\CurrentVersion\Lxss\{8f3af655-ea76-4111-8b33-34a3cbc6e7d4}]
"DefaultUid"=dword:000003e8
```

# USBデバイスを使う

詳細は公式ドキュメント[USB デバイスを接続する | Microsoft Learn](https://learn.microsoft.com/ja-jp/windows/wsl/connect-usb)に従う。

USBIPDというツールを使う。Ver5.0.0になっていたので以前とは少し操作が変わってしまっているようだ。面倒くさいのでWindowsパッケージマネージャーを使ったインストールした。

```powershell
winget install --interactive --exact dorssel.usbipd-win
```

Ubuntu側にインストールする。

```bash
$ sudo apt install linux-tools-virtual hwdata
$ sudo update-alternatives --install /usr/local/bin/usbip usbip /usr/lib/linux-tools/*/usbip 20
```

```powershell
usbipd list
usbipd bind --busid 7-1
usbipd attach --busid 7-1 --wsl DevUbuntu
usbipd list
```

Ubuntu側で確認する。

```bash
$ lsusb
```

実際にデバイスを使用するには読み書き権限を付与する必要がある。

```bash
$ sudo chmod 666 /dev/ttyUSB0
```

USBシリアルドライバがKernelに組み込まれていない場合は、カーネルをリビルドする必要があるようだ。FDTIならデフォルトで対応済みになっているようだ。

```powershell
> usbipd detach --busid 7-1
> usbipd unbind --busid 7-1
```
