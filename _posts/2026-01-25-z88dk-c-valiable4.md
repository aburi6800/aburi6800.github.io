---
layout: post
title:  "【z88dk】C言語/変数(4) インラインアセンブラでの変数操作(2)"
date:   2026-01-25 00:00:00 +0900
categories: z88dk
---

初版 2026/01/25  
改訂  

-----

var4.c

```C
#include <stdio.h>
#include <stdlib.h>
#include <stdint.h>

#define ADD_VALUE            10;

// uint8_t = unsigned 8bit integer(0～255)
uint8_t value8 = 100;
uint8_t *value8_p = &value8;

void main() {

    // 変数の値を表示する
    printf("value   : %u\n", value8);

    // 変数の値をインラインアセンブラで更新する
    printf("value update(inline asm).\n");

#ifndef __INTELLISENSE__
#asm
    LD      HL, (_value8_p)
    LD      A, (HL)
    ADD     A, ADD_VALUE
    LD      (HL), A
#endasm
#endif

    printf("value   : %u\n", value8);

    return;
}
```

- コンパイル時に`-m`オプションを付けると生成される`.map`ファイルを確認するとわかるが、変数はコンパイル時に`_`＋変数名のラベルで定義される。アセンブリからはその名前を参照できる。
- 値を直接参照する場合は`_`＋変数名、格納アドレスを参照する場合は`_`＋ポインタ変数名を指定する。
- ただし、関数内で宣言したローカル変数はスタック領域に確保されるため、この手法は使えない。操作対象の変数はグローバル変数として定義すること。
- また、定数はその名前を直接書く。上記コードでは`ADD_VALUE`が定数となる。

<br>

## コンパイル

compile.sh

```shell
#!/bin/sh
zcc +msx -lmsxbios --list -subtype=msxdos $1.c -o $1.com
```

<br>

```shell
$ ./compile.sh ./var4
```

<br>

## 実行結果

![2026-01-25-z88dk-c-valiable4_01.png](/resources/2026-01-25-z88dk-c-valiable4_01.png)

※MSXDOS.SYSとCOMMAND.COM、コンパイルして作成した.COMを配置したディレクトリをopenMSXでマウント、MSXDOSから実行
