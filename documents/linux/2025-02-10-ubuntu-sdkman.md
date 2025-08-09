---
layout: post
title:  "【Linux】ubuntuでJavaを使う"
date:   2025-02-10 16:00:00 +0900
categories: java ubuntu sdkman
---

初版 2025/02/10  
改訂 

-----

<br>

## 概略

Javaはubuntuのリポジトリに存在しているものがあるが、最新ではない。
インストールは各配布先からパッケージを入手し手動で行えるが、複数のバージョンを使い分けることも考慮し、SDKMAN!を利用する。

<br>

## SDKMAN!のインストール
```
$ curl -s https://get.sdkman.io | bash
$ source "/home/ユーザー名/.sdkman/bin/sdkman-init.sh"
$ sdk version
```

<br>

## インストール可能なJDKを調べる
```
$ sdk list java
```
![2025-02-10-ubuntu-sdkman-1.png](/resources/2025-02-10-ubuntu-sdkman-1.png)

<br>

## JDKをインストールする

```
$ sdk install java <対象JDKのIdentifierの文字列>
```

例えばOpenJDK 25.ea.1-openをインストールする場合は以下となる。
```
$ sdk install java 25.ea.1-open
```

インストール後、デフォルトにするか聞かれるので、するなら`y`、しないなら`n`を入力する。
```
In progress...

################################################################################################################# 100.0%

Repackaging Java 25.ea.1-open...

Done repackaging...

Installing: java 25.ea.1-open
Done installing!

Do you want java 25.ea.1-open to be set as default? (Y/n):
```

<br>

## 使用するJDKを変更する

```
$ sdk default java 25.ea.9-open
```

<br>

## インストール済のJDKを削除する

```
$ sdk uninstall java 25.ea.1-open
```

<br>

## 補足

- AntやMaven、Gradleなどのツールも同様にSDKMAN!でインストール可能。
- 例えばGradleの場合は以下となる。
```
$ sdk list gradle
$ sdk install gradle 6.8
```

