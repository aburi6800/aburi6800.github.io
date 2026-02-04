---
layout: page
title:  "【z88dk】C言語/変数(3) インラインアセンブラでの変数操作(1)"
date:   2026-01-21 00:00:00 +0900
categories: z88dk
---

初版 2026/01/21  
改訂 2026/01/25  

-----

var3.c

```C
#include <stdio.h>
#include <stdlib.h>
#include <stdint.h>

// uint8_t = unsigned 8bit integer(0～255)
uint8_t value8 = 100;


uint8_t fnc_inc(uint8_t *distaddr, uint8_t value) {

#ifndef __INTELLISENSE__
#asm
    ; 引数取得
    ;   引数は第1引数から順にスタックに積まれるため、
    ;   取得する際は末尾の引数から取得する。
    ;   HLレジスタで読む以外には、IXレジスタを使う方法もあるが、SPは壊さないこと。
    LD      HL, 2       ; スタックの先頭2byte(リターンアドレス)読み飛ばし
    ADD     HL, SP      ;
    LD      A, (HL)     ; 第2引数(1byte)をAレジスタにロード
    INC     HL          ;
    INC     HL          ; 引数はスタックに積まれる(=2byte)のため、1byteスキップ
    LD      C, (HL)     ; 第1引数(2byte)をBCレジスタにロード
    INC     HL          ;
    LD      B, (HL)     ;

    ; 処理ロジック
    PUSH    BC          ; スタック経由でBCレジスタの値をHLレジスタにセット
    POP     HL
    ADD     A, (HL)     ; AレジスタにHLレジスタのアドレスの値を加算

    ; 戻り値の設定
    ;   戻り値はHLレジスタの値が渡される。
    ;   当サンプルのように1byteの値で良い場合は、
    ;   Hレジスタにゼロ、Lレジスタに値を設定する。
    LD      H, 0
    LD      L, A

    RET
#endasm
#endif

}

void main() {

    // 変数の値を表示する
    printf("value : %u\n", value8);

    printf("value update(function call).\n");

    // 変数の値を更新・表示する
    value8 = fnc_inc(&value8, 2);
    printf("value : %u\n", value8);

    value8 = fnc_inc(&value8, 2);
    printf("value : %u\n", value8);

    return;

}
```

- ポインタ変数の定義は`*`を付け、代入する変数には`&`を付ける。
- ポインタ変数から値を参照・操作する際は、ポインタ変数に`*`を付ける。

> `&`：アドレス演算子  
> 変数が配置されているメモリのアドレスを静的に参照するための演算子。  
> 変更などの操作はできない。  

> `*`：間接参照演算子  
> 変数が配置されているメモリのアドレスを動的に参照するための演算子。  
> 値を変更できる。  
> また、ポインタ変数の先頭に`*`を付けることで、ポインタ変数の型に従いそのアドレスのメモリの値を取得できる。  

<br>

## コンパイル

compile.sh

```shell
#!/bin/sh
zcc +msx -lmsxbios --list -subtype=msxdos $1.c -o $1.com
```

<br>

```shell
$ ./compile.sh ./var3
```

<br>

## 実行結果

![2026-01-21-z88dk-c-valiable1_02.png](/resources/2026-01-21-z88dk-c-valiable3_01.png)

- [直接操作](/z88dk/2026/01/18/z88dk-c-valiable1.html)と同じ結果になることがわかる。

※MSXDOS.SYSとCOMMAND.COM、コンパイルして作成した.COMを配置したディレクトリをopenMSXでマウント、MSXDOSから実行
