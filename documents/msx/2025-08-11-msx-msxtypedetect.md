---
layout: page
title:  "【MSX】MSXのバージョンを判定する"
date:   2025-08-11 12:00:00 +0900
categories: msx msx2
---

初版 2025/08/11  
改訂 2025/11/01

-----

<br>

## 判定方法

作成するプログラムはMSX1/2両用プログラムだが、パレット変更のようにMSX2のみの処理を入れたい、ということがある。  
この場合は、BASICのバージョンを判定することで実現できる。  
具体的には、以下のどちらかとなる。  

1. MAIN-ROMの0x002Dの値を判定する。0=ver1.0(MSX1), 1=ver2.0(MSX2)、2=Ver3.0(MSX2+)、3=Ver4.0(TurboR)となる。  
2. EXBRSA(0xFAF8)にSUB-ROMのスロットアドレスを判定する。入っていなければ(0であれば)MSX1、以外はMSX2以降となる。  

<br>

## 実装例

上記1.の方法で、アセンブラでのサンプルプログラムを記載する。  

```asm
MSXVER              equ 0x002d

VERCHK:
    LD      A, (MSXVER)
    OR      A
    RET     Z
      :
      <MSX2以降の処理>
      :
```

BASICの場合は、以下のようにする。

```basic
IF PEEK(&H002D) THEN <MSX2以降の処理>
```
