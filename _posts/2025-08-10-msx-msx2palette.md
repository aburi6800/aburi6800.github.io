---
layout: post
title:  "【MSX】MSX2でMSX1の色合いにする"
date:   2025-08-10 00:00:00 +0900
categories: msx msx2
---

初版 2025/08/10  
改訂 

-----

<br>

## はじめに

MSX2はMSX1と比較し、全体的にコントラストが強い色合いになっているため、画面の印象が変わる。
しかし、各色のパレットを変更することでMSX1に近い色合いにすることができる。  
ここでは、その処理について記載する。  

<br>

## パレットを初期化する

BIOSの`INIPLT`を使用する。

・INIPLT (0141H/SUB)  
　機能：パレットの初期化 (現在のパレットは VRAM にセーブされている)  
　入力：なし  
　出力：なし  
　レジスタ：AF, BC, DE  

```
INIPLT		equ 0x0141
EXTROM		equ 0x015F

    ld      ix, INIPLT      ; Set BIOS entry address
    call    EXTROM          ; Returns here
```

<br>

## パレットを変更する

BIOSの`SETPLT`を使用し、各色にカラーコードを設定する。  

・SETPLT (014DH/SUB)  
　機能：カラーコードをパレットにセット  
　入力：Dレジスタ にパレット番号 (0～15)  
　　　　Aレジスタ の上位 4 ビットに赤のコード  
　　　　Aレジスタ の下位 4 ビットに青のコード  
　　　　レジスタE の下位 4 ビットに緑のコード  
　出力：なし  
　レジスタ：AF  

```
SETPLT		equ 0x014D
EXTROM		equ 0x015F

    ld      ix, SETPLT      ; Set BIOS entry address
    ; color 0
    ld      d, 0
    ld      a, 0b00000000
    ld      e, 0b00000000
    call    EXTROM          ; interslot call SUB-ROM 
    ; color 1
    ld      d, 1
    ld      a, 0b00000000
    ld      e, 0b00000000
    call    EXTROM
    ; color 2
    ld      d, 2
	:
	:
	:
```

とベタで書くのもアレなので、こんな感じで

```
SETPLT		equ 0x014D
EXTROM		equ 0x015F

SET_PALETTE:
    ld      a, (MSXVER)
    or      a
    ret     Z

    ld      ix, SETPLT      ; Set BIOS entry address
    ld      hl, PALLETE_DATA
    ld      d, 0            ; pallet no

SET_PALETTE_L1:
    ld      (hl), a
    inc     hl
    ld      (hl), e
    inc     hl
    call    EXTROM          ; interslot call SUB-ROM 
    inc     d
    ld      a, 15
    sub     d
    jr      nz, SET_PALETTE_L1

SET_PALETTE_EXIT:
    ret

PALLETE_DATA:
    ; +0 : RED(4bit) + blue(4bit)
    ; +1 : 0b0000 + green(4bit)
    db  00, 00              ; 0
    db  00, 00              ; 1
    db  22, 05              ; 2
    db  33, 06              ; 3
    db  26, 02              ; 4
    db  47, 03              ; 5
    db  52, 03              ; 6
    db  37, 06              ; 7
    db  62, 03              ; 8
    db  73, 04              ; 9
    db  63, 05              ; 10
    db  64, 06              ; 11
    db  22, 04              ; 12
    db  55, 03              ; 13
    db  66, 06              ; 14
    db  77, 07              ; 15
```
