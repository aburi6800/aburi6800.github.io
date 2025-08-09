---
layout: post
title:  "Linux/termuxにsshサーバを設定する"
date:   2025-08-09 00:00:00 +0900
categories: termux android
---

初版 2025/08/09  
改訂 

-----

<br>

## はじめに

ローカルネットワーク上の他PCのLinuxから、Andoidにインストールしたtermuxへ接続できるよう、sshサーバを設定する。  
なお、termux上に別のディストリビューションを動作させている場合は、一旦抜けてtermuxで作業する。  

<br>

## termux側での作業（初回のみ）

### ユーザーIDを確認

現在のユーザーIDを確認する。  
ここに表示されたuidを控える。  

```
id
```

>作業時はu0_a264だった

<br>

### パスワードを設定

ssh接続に必要ぱパスワードを設定する。  

```
passwd
```

>作業時はpasswdと設定した

<br>

### IPアドレスを確認

接続先から必要なIPアドレスを確認する。  
`wlan0`で表示されるIPアドレスを控える。  

```
ifconfig
```

>作業時は192.168.0.85だった

<br>

### opensshをインストール

```
pkg install openssh
```

>作業時は既にインストールされていた


### sshdを起動

```
sshd
```

<br>

## 接続先での作業

### ssh接続を行う

以下コマンドを実行する。  
引数は事前に確認した内容に適宜読み替えること。  

```
ssh -p 8022 u0_a264@192.168.0.85
```

<br>

>初回は以下が表示される。  
>`yes`を入力し、一度接続が閉じられるので、再度`ssh`コマンドを実行する。  
>
>```
>The authenticity of host '[192.168.0.85]:8022 ([192.168.0.85]:8022)' can't be established.
>ED25519 key fingerprint is SHA256:6kG5+7Mvmtaen5mBeNM2WJRSeqBbnGdLDbAd2oJKhEU.
>This key is not known by any other names
>Are you sure you want to continue connecting (yes/no/[fingerprint])?
>```

<br>

接続するとパスワードを求められるので、設定したパスワードを入力。  

```
u0_a264@192.168.0.85's password:
```

接続成功。  

```
Welcome to Termux!

Docs:       https://termux.dev/docs
Donate:     https://termux.dev/donate
Community:  https://termux.dev/community

Working with packages:

 - Search:  pkg search <query>
 - Install: pkg install <package>
 - Upgrade: pkg upgrade

Subscribing to additional repositories:

 - Root:    pkg install root-repo
 - X11:     pkg install x11-repo

For fixing any repository issues,
try 'termux-change-repo' command.

Report issues at https://termux.dev/issues
hitoshi@localhost:~$
```

失敗した場合は、表示されるメッセージを元に調査・対応すること。  


