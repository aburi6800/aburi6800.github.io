---
layout: post
title:  "github pagesのローカル開発メモ"
date:   2024-05-04 03:00:00 +0900
categories: jekyll github
---

github pagesで公開する前にローカルでの動作確認をするための環境を用意するメモ。  
ドキュメントを読んでもさっぱりわからず、色々と調べてみたのでまとめておく。

<br>

## 使用環境

- ubuntu 22.04 (WSL2上でも可)
- vscode

<br>

## rubyの環境設定

静的コンテンツを生成するツールとして、[jekyll](https://jekyllrb-ja.github.io/)を使用する。
jekyllはruby上で動作するので、まずはrubyのセットアップを行う。
```
$ sudo apt-get install ruby-full build-essential zlib1g-dev
```

rubyのライブラリはgemと呼ばれ、ホームディレクトリの`gems/`に格納される。
環境設定を行い、gemsへのパスを通しておく。
```
$ echo '# Install Ruby Gems to ~/gems' >> ~/.bashrc
$ echo 'export GEM_HOME="$HOME/gems"' >> ~/.bashrc
$ echo 'export PATH="$HOME/gems/bin:$PATH"' >> ~/.bashrc
$ source ~/.bashrc
```

<br>

## jekyllの環境設定

続いて、jekyllをインストールする。
jekyllはrubyライブラリとして提供されているため、`gem`コマンドを使用する。

> bundlerはruby2.6.0以降から標準添付されているため、明示的にインストールする必要はない。

```
$ gem install jekyll
```

その後、プロジェクトのルートディレクトリで以下を実行し、サイトの初期化を行う。
```
$ bundle exec jekyll new ./ --force
```

`Gemfile`ファイルが出来ているので、以下の内容を追記する。
```
gem "github-pages", "~> 227", group: :jekyll_plugins
gem "webrick"
```

その後、`bundler`を使用してインストールする。
```
$ bundle install
```

> `bundle`はbundlerを起動するコマンドで、`install`はgemをインストールするサブコマンド。
> このコマンドでbundlerをGemfileを読み、依存するgemのバーションを調べ、`Gemfile.lock`というファイルに書きだす。
> `bundle exec`とすると、このGemfile.lockに書かれたバージョンのgemを起動する。

<br>

## jeykillの起動

以下のコマンドを実行する。
```
$ bundle exec jekyll serve
```

> jekyll単体でも動作できるが、gemのバージョンが固定されないため、jekyllが使用するgemのバージョンが異なるなど、想定しない事象が発生する可能性がある。
> そのため、必ずbundle経由で起動すること。

警告メッセージが出るが、動作には支障がないので無視して構わない(`jekyll-github-metadata`の警告はgithub apiの認証が出来ていないことを示すもの)。
```
hitoshi@infinity:~/Development/git/aburi6800/aburi6800.github.io.git$ bundle exec jekyll serve
Configuration file: none
To use retry middleware with Faraday v2.0+, install `faraday-retry` gem
            Source: /home/hitoshi/Development/git/aburi6800/aburi6800.github.io.git
       Destination: /home/hitoshi/Development/git/aburi6800/aburi6800.github.io.git/_site
 Incremental build: disabled. Enable with --incremental
      Generating...
   GitHub Metadata: No GitHub API authentication could be found. Some fields may be missing or have incorrect data.
                    done in 1.894 seconds.
                    Auto-regeneration may not work on some Windows versions.
                    Please see: https://github.com/Microsoft/BashOnWindows/issues/216
                    If it does not work, please upgrade Bash on Windows or run Jekyll with --no-watch.
 Auto-regeneration: enabled for '/home/hitoshi/Development/git/aburi6800/aburi6800.github.io.git'
    Server address: http://127.0.0.1:4000
  Server running... press ctrl-c to stop.
```

ブラウザで`localhost:4000`にアクセスすると、github pagesで公開されるものと同じページが表示される。

<br>

## githubリポジトリの作成

- githubに「`<user>.github.io`」の名前でリポジトリを作成し、ローカルにcloneする。大文字は使用不可。また「Initialize this repository with a README」にチェックを入れ、可視性はpublicにする。

<br>

## githubビルド設定

- 対象リポジトリのSettingsに入り、左メニューからPagesを選択。「Branch」のところで、対象のブランチとホームディレクトリとするパスを選択。
- push後は自動的にビルドとデプロイが行われ、公開される。urlはリポジトリ名と同じ。ビルド時にエラーが発生した場合は、対象リポジトリのActionから状況が確認できる。

<br>

## ページの記載について

- jekyllは、以下のフロントマターが書かれた.mdファイルをhtmlに変換する。
```
---
layout: default
title: <ページのタイトル>
description: <ページの説明>
permalink: <html生成先のパス・ファイル名>
---
```
- layoutは各テンプレートで定義されたレイアウト名を指定する。
- permalinkは、たとえば`docs/test`と書くと、`_site/docs/test.html`のファイル名で生成される。省略時は.mdファイルと同じ構造で生成される。

<br>

