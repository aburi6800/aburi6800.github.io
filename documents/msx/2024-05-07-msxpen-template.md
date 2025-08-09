---
layout: page
title:  "【MSX】MSXPenのプログラムテンプレート"
date:   2024-05-07 01:30:00 +0900
categories: msxpen msx-basic z80
---

初版 2024/05/07  
改訂 2025/02/10

-----

<br>

## 概略

[MSXPen](https://msxpen.com/)では、「RUN」ボタンを押したタイミングでプログラムが格納されたディスクイメージを作成し、WebMSXの起動時にロードされる仕組みになっている。  
ディスクイメージには、BASICプログラムはアスキーセーブ形式で保存され、アセンブリソースはコンパイル後に `PROGRAM.BIN` というファイル名で保存される。  
なお、MSXPenのアセンブラは[Pasmo](:/7fff9d2403274518ab7f1b6f4e362b48)が使用されている。  

<br>

## BASICプログラムのテンプレート

MSXPenの「Basic」タブに記載するテンプレート。
小数演算が必要な場合は、`DEFINTA-Z`の部分を適宜修正する。

```
100 SCREEN 1,2,0:WIDTH 32:KEY OFF:COLOR 15,1,1:DEFINT A-Z
110 CLEAR 200,&HC000:DEFUSR=&HC000:R=RND(-TIME)
120 BLOAD "PROGRAM.BIN"
130 R=INT(RND(1)*9)+1:PRINT R
130 POKE &HC100,R
140 A=USR(0)
150 PRINT "IN : " ; PEEK(&amp;HC100)
160 PRINT "OUT: " ; PEEK(&amp;HC101)
```

<br>

## アセンブリソースのテンプレ

MSXPenの「Asm」タブに記載するテンプレート。
ソース内に複数のORG指定が可能。

```
START:
	; 開始アドレス
	ORG	0xC000

	; ロジックを記述
	LD A,(INDATA)
	ADD A,1
	LD (OUTDATA),A

	; BASICに戻る
	RET


	; 開始アドレス
	ORG	0xC100			

INDATA:
	DB	$00

OUTDATA:
	DB	$00

END START
```
