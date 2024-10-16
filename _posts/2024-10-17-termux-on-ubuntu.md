---
layout: post
title:  "Android端末にtermux on ubuntu環境を構築する"
date:   2024-10-17 01:18:00 +0900
categories: android termux ubuntu
---

初版 2024/10/17  

-----

<br>

Android端末に、LinuxのCUI環境を構築する。
ここでは、root権限を取得しなくても良いTermuxを使用する。
このTermuxはAndroid OSの上に別の仮想Linux環境を構築するもので、端末のルートディレクトリにはアクセス不可となっている。そのため、Android端末を壊すことはない。
ただし、Debian系のディストリビューションをベースにしているため、一般的に使われているUbuntuを追加インストールして使用する。  

> なお、当方ではAndroidOS 12の端末を使用しています。他のバージョンでは、想定しない問題が起こる可能性があることをご承知おきください。

<br>

## CUI環境の構築

### Termuxのインストール

現在、TermuxはF-DROIDでapkファイルが配布されている。  
https://f-droid.org/

上記サイトからF-DROIDのapkをダウンロード・インストール。  
その後、F-DROIDからTermuxを検索してapkをダウンロード、インストールする。  

> 以前はapkをインストールするためにOSの設定変更をする必要があったが、Android12では、apkを実行するときに実行するかを選択するように変わっていた。  

<br>

### 最初の設定

インストール後にTermuxを起動。  
まずは、Termuxから本体のストレージへのアクセスを可能とするため、アプリを起動後のコンソールで以下を実行する。  

```shell
$ termux-setup-storage
```

権限の許可を求めるダイアログが出るので、許可する。  
その後、パッケージリストを最新化する。  

```shell
$ pkg update
```

<br>

### ubuntuのインストール

まず、以下パッケージをインストールする。

```shell
$ apt install proot
$ apt install proot-distro
```

続いて、以下のコマンドでubuntuをインストール。  

```shell
$ proot-distro install ubuntu
```

> (2023/02/19)
> `proot-distro`は、ubuntu以外にもディストリビューションを選択可能。  
> 対応しているディストリビューションとコマンドライン引数は、`proot-distro list` で確認できる。  

以下はインストール中の表示例。  

```
	[*] Installing Ubuntu...
	[*] Creating directory '/data/data/com.termux/files/usr/var/lib/proot-distro/installed-rootfs/ubuntu-20.04'...
	[*] Creating directory '/data/data/com.termux/files/usr/var/lib/proot-distro/dlcache'...
	[*] Downloading rootfs tarball...

	  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
									 Dload  Upload   Total   Spent    Left  Speed
	100   663  100   663    0     0   1398      0 --:--:-- --:--:-- --:--:--  1401
	100 25.0M  100 25.0M    0     0   325k      0  0:01:18  0:01:18 --:--:--  320k

	[*] Checking integrity, please wait...
	[*] Extracting rootfs, please wait...
	[*] Writing '/data/data/com.termux/files/usr/var/lib/proot-distro/installed-rootfs/ubuntu-20.04/etc/profile.d/termux-proot.sh'...
	[*] Writing resolv.conf file (NS 1.1.1.1/1.0.0.1)...
	[*] Writing hosts file...
	[*] Registering Android-specific UIDs and GIDs...
	[*] Running distro-specific configuration steps...
	[*] Installation finished.

	Now run 'proot-distro login ubuntu' to log in.
```

インストールが完了したら、以下のコマンドでubuntuにログインする。  

```shell
$ proot-distro login ubuntu
```

ログインが確認できたら、一度`exit`してTermuxに戻り、以下コマンドでTermuxの`~/.bashrc`に起動時のコマンドを追記すると、次回からTermux開始時に自動的にubuntuに入れる。  

```shell
$ echo 'proot-distro login ubuntu' > ./.bashrc
```

なお、ここでインストールした`ubuntu`は必要最低限の状態であるため、必要なパッケージは随時インストールする必要がある。  

<br>

### 一般ユーザーを追加する

Termuxではrootでの利用を前提としているため、ubuntuでもrootでの利用となる。  
しかし、このままでは`sudo`を使用した手順がそのまま利用できないため、ユーザーを追加する。  

```shell
$ apt dist-upgrade
$ apt install -y sudo
$ adduser <ユーザ名>
$ echo "<ユーザ名> ALL=(ALL:ALL) ALL" > /etc/sudoers
```

完了したら、`su`してみる。

```shell
$ su <ユーザ名>
```

パスワードの入力を求められるので入力。
プロンプトの先頭がユーザ名に変更され、コマンドが実行できればOK。  
ここで一応、`apt update && apt upgrade`しておく。  

また、Ternuxの`.bashrc`に追加したubuntu起動コマンドを以下に変更すると、次から指定したユーザでログインした状態で起動する。

```shell
$ proot-distro login --user <ユーザ名> ubuntu
```

<br>

### エディタのインストール

この後での作業で設定ファイルなどを書き換えられるよう、テキストエディタをインストールする。  
`nano`が便利なので入れておく。

```shell
$ apt install nano
```

<br>

### 日本ロケールへの変更

今の状態を確認する。

```shell
$ locale -a
C
C.utf8
POSIX
```

以下コマンドを実行する。

```shell
$ apt install language-pack-ja -y
```

再度確認。

```shell
$ locale -a
C
C.utf8
POSIX
ja_JP.utf8
```

続いて以下コマンドを実行。

```shell
$ echo 'export LANG=ja_JP.UTF-8' >> ~/.bashrc
$ echo 'export LANGUAGE="ja_JP:ja"' >> ~/.bashrc
$ . ~/.bashrc
```

これでメッセージが日本語で表示されるようになる。

<br>

### タイムゾーンの変更

datetimectlコマンドで変更できるそうだがデフォルトでは入っていないため、以下でインストールする。

```shell
$ sudo apt install tzdata
```

インストール中に地域を聞かれるので、以下で設定。

```shell
Configuring tzdata
------------------

Please select the geographic area in which you live. Subsequent configuration questions will narrow this down by presenting a list of cities,
representing the time zones in which they are located.

  1. Africa  2. America  3. Antarctica  4. Australia  5. Arctic  6. Asia  7. Atlantic  8. Europe  9. Indian  10. Pacific  11. US  12. Etc
Geographic area: 6

Please select the city or region corresponding to your time zone.

  1. Aden      11. Baku        21. Damascus     31. Hong_Kong  41. Kashgar       51. Makassar      61. Pyongyang  71. Singapore      81. Ujung_Pandang
  2. Almaty    12. Bangkok     22. Dhaka        32. Hovd       42. Kathmandu     52. Manila        62. Qatar      72. Srednekolymsk  82. Ulaanbaatar
  3. Amman     13. Barnaul     23. Dili         33. Irkutsk    43. Khandyga      53. Muscat        63. Qostanay   73. Taipei         83. Urumqi
  4. Anadyr    14. Beirut      24. Dubai        34. Istanbul   44. Kolkata       54. Nicosia       64. Qyzylorda  74. Tashkent       84. Ust-Nera
  5. Aqtau     15. Bishkek     25. Dushanbe     35. Jakarta    45. Krasnoyarsk   55. Novokuznetsk  65. Rangoon    75. Tbilisi        85. Vientiane
  6. Aqtobe    16. Brunei      26. Famagusta    36. Jayapura   46. Kuala_Lumpur  56. Novosibirsk   66. Riyadh     76. Tehran         86. Vladivostok
  7. Ashgabat  17. Chita       27. Gaza         37. Jerusalem  47. Kuching       57. Omsk          67. Sakhalin   77. Tel_Aviv       87. Yakutsk
  8. Atyrau    18. Choibalsan  28. Harbin       38. Kabul      48. Kuwait        58. Oral          68. Samarkand  78. Thimphu        88. Yangon
  9. Baghdad   19. Chongqing   29. Hebron       39. Kamchatka  49. Macau         59. Phnom_Penh    69. Seoul      79. Tokyo          89. Yekaterinburg
  10. Bahrain  20. Colombo     30. Ho_Chi_Minh  40. Karachi    50. Magadan       60. Pontianak     70. Shanghai   80. Tomsk          90. Yerevan
Time zone: 79


Current default time zone: 'Asia/Tokyo'
Local time is now:      Tue Oct 15 01:57:10 JST 2024.
Universal Time is now:  Mon Oct 14 16:57:10 UTC 2024.
Run 'dpkg-reconfigure tzdata' if you wish to change it.
```

最後に適用されていることを確認。

```shell
$ date
2024年 10月 15日 火曜日 02:01:18 JST
```

<br>

## 開発環境の構築

### Pythonのインストール

何かと便利なのと、関連パッケージでだいたい必要なものが揃うので、まずはPythonをインストールする。  

```shell
$ sudo apt update && apt upgrade
$ sudo apt install python3
$ sudo apt install python3-venv
$ sudo apt install python3-pip
$ sudo apt install python-is-python3
```

> 通常は`python3`コマンドになるが、最後のパッケージを入れることで、`python`コマンドでPython3が実行されるようになる。  

pipの実行例。

```shell
$ sudo pip install --upgrade pip
```

venvでの仮想環境作成は以下。

```shell
$ sudo python -m venv [仮想環境名]
```

仮想環境への切り替えは以下のように、仮想環境名のディレクトリの`bin/activate`ファイルをsourceする。
実行するとプロンプトの先頭に仮想環境名が表示される。

```shell
$ . [仮想環境名]/bin/activate
```

<br>

### Javaのインストール

特に要件がなければ、ubuntuで標準で提供されているopenjdkをインストールする。

```shell
sudo apt install default-jdk
```

参考
https://www.digitalocean.com/community/tutorials/how-to-install-java-with-apt-on-ubuntu-20-04-ja

<br>

### gitのインストール・設定

開発では必須のgitをインストールする。

```shell
$ sudo apt install git
```

最低限、以下を設定しておく。

```shell
$ git config --global user.name "<ユーザー名>"
$ git config --global user.emale "<メールアドレス>"
```

<br>

### codeserverのインストール・設定

ブラウザでVSCodeを動かすためのcodeserverをインストールする。

```shell
$ curl -faSL https://code-server.dev/install.sh | sh
```

インストール後、以下で起動する。

```shell
$ code-server --auth none &
```

その後、ブラウザで以下アドレスにアクセス。

```
http://127.0.0.1:8080/
```

拡張機能は、最低限、japanese language pack拡張を入れる。
後は必要な拡張機能を入れていく。

<br>

### cmakeのインストール

ビルドツールで使用するcmakeをインストールする。

```shell
$ sudo apt install cmake
```

<br>

## Tips

### ファイルをAndroidに渡す

以下ディレクトリにファイルをコピーすることでAndroidとのファイルの受け渡しが可能。

```
/sdcard
```

> ディレクトリ名はsdcardだが、実体は本体ストレージとなる。

なお、termuxのtermux-setup-storageでセットアップされる、各ディレクトリは以下に作成される。

```
/data/data/com.termux/files/home/storage
```

この中に`dcim`、`downloads`などのディレクトリがあるが、当然AndroidとLinuxではファイルシステムが違うため、Termuxからシンボリックリンクを作成することはできない。

<br>

### Androidのアプリ(Webブラウザ等)にファイルを認識させる

基本的に本体ストレージを使う。
本体ストレージのルートに受け渡し用のディレクトリを作成するのが確実。

<br>
