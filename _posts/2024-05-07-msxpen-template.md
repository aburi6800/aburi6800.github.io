---
layout: post
title:  "MSXPenのプログラムテンプレート"
date:   2024-05-07 01:30:00 +0900
categories: msx msxpen
---

初版 2024/05/07  
改訂  

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
100 SCREEN1,2,0:WIDTH32:KEYOFF:COLOR15,1,1:DEFINTA-Z
110 CLEAR200,&HBFFF:DEFUSR=&HC000:R=RND(-TIME)
120 BLOAD "PROGRAM.BIN"
130 R=INT(RND(1)*9)+1:PRINT R
130 POKE &HC100,R
140 A=USR(0)
150 PRINT PEEK(&HC101)
```

<br>

## アセンブリソースのテンプレ

MSXPenの「Asm」タブに記載するテンプレート。
ソース内に複数のORG指定が可能。

```
START:
	; 開始アドレス
	ORG	0xC000

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
