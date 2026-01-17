---
layout: post
title:  "【Linux】node.jsをインストールする"
date:   2025-11-01 00:00:00 +0900
categories: linux
---

初版 2025/11/01  
改訂 2026/01/18

-----

<br>

## はじめに

node.jsのインストールに少々手こずったため、情報をまとめておく。  

<br>

## 公式リポジトリから入れる

少し古いバージョンだが、最も手っ取り早くインストールできる。  

```shell
sudo apt-get update
sudo apt-get install nodejs npm
```

なお、そのままではnode.jsのバイナリは`nodejs`という名前になっているが、通常は`node`という名前を期待されるため、以下コマンドでリンクを作成する。  

```shell
sudo update-alternatives --install /usr/bin/node node /usr/bin/nodejs 10
```

<br>

## PPAから入れる

バージョンが気になる場合は、PPAから入れると少し最新に近くなる。  
まず、PPAをインストールする。  

```shell
sudo curl -sL https://deb.nodesource.com/setup | sudo bash -
```

その後、nodejsをインストールする。  
PPAからインストールした場合はnpmもインストールされるので、別途インストールする必要はない。  
ただし、一部機能（ソースからビルドする場合に要求されるパッケージの利用など）が使えないため、`build-essential`もインストールする。  

```shell
sudo apt-get install nodejs build-essential
```

<br>

## nvmで入れる

`nvm`(Node.js version manager)を入れ、そこから`node.js`をインストールする方法もある。  
`node.js`の複数バージョンの管理が可能で、最新への更新も簡単なので、こちらを使用するのが良い。  
まず、`nvm`をgithubの公式リポジトリからインストールする。  

[https://github.com/nvm-sh/nvm](https://github.com/nvm-sh/nvm)

> 当記事執筆時点はv0.40.3だが、バージョンによってURLが変わるので上記URLを確認すること。  

```shell
$ curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 16631  100 16631    0     0  32927      0 --:--:-- --:--:-- --:--:-- 32998
=> Downloading nvm from git to '/home/hitoshi/.nvm'
=> Cloning into '/home/hitoshi/.nvm'...
remote: Enumerating objects: 383, done.
remote: Counting objects: 100% (383/383), done.
remote: Compressing objects: 100% (326/326), done.
remote: Total 383 (delta 43), reused 180 (delta 29), pack-reused 0 (from 0)
Receiving objects: 100% (383/383), 391.78 KiB | 2.33 MiB/s, done.
Resolving deltas: 100% (43/43), done.
* (HEAD detached at FETCH_HEAD)
  master
=> Compressing and cleaning up git repository

=> Appending nvm source string to /home/hitoshi/.bashrc
=> Appending bash_completion source string to /home/hitoshi/.bashrc
=> Close and reopen your terminal to start using nvm or run the following to use it now:

export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  # This loads nvm bash_completion
```

インストール後に環境変数を適用し、バージョンを確認する。  

```shell
$ . ~/.bashrc
$ nvm -v
0.40.3
```

続いて`node.js`と`npm`をインストールする。  
推奨される最新のLTS版をインストールする。  

```shell
$ nvm install --lts
Installing latest LTS version.
Downloading and installing node v24.11.0...
Downloading https://nodejs.org/dist/v24.11.0/node-v24.11.0-linux-x64.tar.xz...
############################################################################################# 100.0%
Computing checksum with sha256sum
Checksums matched!
Now using node v24.11.0 (npm v11.6.1)
Creating default alias: default -> lts/* (-> v24.11.0)
```

バージョンを確認する。  

```shell
$ node -v
v24.11.0
$ npm -v
11.6.1
```
