---
layout: post
title:  "MSX/SCREEN1.5のキャラクタパターン・スプライトを高速に定義する"
date:   2025-02-24 20:00:00 +0900
categories: msx-basic
---

初版 2025/02/25  
改訂 

-----

<br>

MSXではSCREEN1（テキストモード）のまま、VDPをSCREEN2（グラフィックモード）に変更することで、横8ドット単位に2色（前景・背景）を割り当てることができる。  
しかし、BASICプログラムで定義すると時間がかかるため、マシン語を使って高速化する。  
なお、プログラムは`MSXPen`で使用する前提としているが、任意のコンパイラでコンパイルし、バイナリのファイル名を`program.bin`としてBASICプログラムと同じディレクトリに保存（フロッピーディスクの場合）、またはBASICプログラムの後にセーブ（カセットテーブの場合）することで、`MSXPen`環境以外でも利用可能となる。  

<br>

## ASMソース

プログラムは、`DB`定義したデータをVRAMにブロック転送するだけの簡単なもの。  
データは非圧縮（ベタ）で定義されたものであるため、同じようなデータを作成することで他のツールで作成したデータも使用可能。  
なお、グラフィックモードでは、パターンジェネレータテーブルとカラーテーブルは256バイトずつ3つの領域（PAGE0～2）に別れて定義されているが、このプログラムではPAGE0と同じデータをPAGE1,2にも設定する内容としている。各ページのデータを用意し、簡単な改修を行うことで、各PAGEに別データを設定することも可能である。  
また、スプライトパターンも同時に定義するようにしているが、スプライトを使用しない場合は`CALL SET_SPRPTN_TBL`を削除し、ラベル`SET_SPRPTN_TBL`のブロックと、最後の方に定義した`SPRITE_PTN:`のラベルを削除する。  

<br>

データ形式は以下を想定している。  
- パターンジェネレータテーブル・カラーテーブル
	`nMSXtiles`のASM Data形式でエクスポートしたデータ  
- スプライトパターン
	`TinySprite`のASM hexa形式でエクスポートしたデータ  

<br>

`MSXPen`の`ASM`タブに以下コードとデータを貼り付けて使用する。  
なお、画面モード変更は含まれていないため、BASICプログラムで設定する。  

```
START:
LDIRVM:     EQU 0x005C                  ; BIOS LDIRVM
    ORG 0xC000

    ; パターンジェネレータテーブル・カラーテーブル設定
    ; BASICプログラムでスクリーンモード等設定済の前提で呼び出すこと
    CALL SET_PTN_TBL
    CALL SET_COL_TBL
    CALL SET_SPRPTN_TBL
    RET

SET_PTN_TBL:
    ; PATTERN GENERATE TABLE
    ; PAGE 0
    LD DE,0x0000
    CALL SET_PTN_TBL_1
    ; PAGE 1
    LD DE,0x0800
    CALL SET_PTN_TBL_1
    ; PAGE 2
    LD DE,0x1000

SET_PTN_TBL_1:
    LD HL,BANK_PATTERN_0
    LD BC,0x0800
    CALL LDIRVM
    RET

SET_COL_TBL:
    ; COLOR TABLE
    ; PAGE 0
    LD DE,0x2000
    CALL SET_COL_TBL_1
    ; PAGE 1
    LD DE,0x2800
    CALL SET_COL_TBL_1
    ; PAGE 2
    LD DE,0x3000

SET_COL_TBL_1:
    LD HL,BANK_COLOR_0
    LD BC,0x0800
    CALL LDIRVM
    RET

SET_SPRPTN_TBL:
    LD DE,0x3800
    LD HL,SPRITE_PTN
    ; BCレジスタにはパターン数*16の値を設定する
    LD BC,576
    CALL LDIRVM
    RET

BANK_PATTERN_0:
    ; 以下にパターンジェネレータテーブルのデータを貼り付ける。
    ; nMSXtilesでの Export ASM Data で出力したasmデータ形式（BANK_PATTERN_0）を想定。

BANK_COLOR_0:
    ; 以下にカラーテーブルのデータを貼り付ける。
    ; nMSXtilesでの Export ASM Data で出力したasmデータ形式（BANK_COLOR_0）を想定。

SPRITE_PTN:
    ; 以下にスプライトパターンのデータを貼り付ける。
    ; TinySpriteでの Export Sprites - ASM hexa のデータ形式を想定。

END START
```

<br>

## BASICソース

BASICプログラムでは、画面モード変更とマシン語プログラムのロード・実行を行う。  
具体的には、プログラムの先頭に以下の2行を書く。（行番号は任意）  

```
100 SCREEN 1,2,0:WIDTH32:COLOR15,1,1:KEYOFF
110 CLEAR200,&HC000:DEFUSR=&H7E:A=USR(0):BLOAD"PROGRAM.BIN":DEFUSR1=&HC000:A=USR1(0)
```

'&H007E'はBIOSの'SETGRP'のアドレスで、これを呼ぶことでVDPをSCREEN2に設定している。  
次に'&HC000'から配置した前述のマシン語プログラムを呼び出し、キャラクターパターンとスプライトパターンを定義する。  

<br>

ただし、通常の文字（英大文字・小文字と数字）がパターンに含まれていない場合、または他のキャラクターパターンに使っている場合は、エラー発生時にメッセージと行番号がわからなくなる。  
この場合、2行目（上記プログラムでは110行目）をコメントとすることで、通常のSCREEN 1モードとして実行できるため、エラーメッセージが確認可能となる。  

<br>

## 参考：各アドレス

|アドレス|意味|
|---|---|
|0x0000|パターンジェネレータテーブル(0ページ)|
|0x0800|パターンジェネレータテーブル(1ページ)|
|0x1000|パターンジェネレータテーブル(2ページ)|
|0x2000|カラーテーブル(0ページ)|
|0x2800|カラーテーブル(1ページ)|
|0x3000|カラーテーブル(2ページ)|
|0x3800|スプライトジェネレータテーブル|

