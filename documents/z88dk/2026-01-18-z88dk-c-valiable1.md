---
layout: page
title:  "【z88dk】C言語/変数(1) 直接操作"
date:   2026-01-18 00:00:00 +0900
categories: z88dk
---

初版 2026/01/18  
改訂

-----

## ソース

var1.c

```C
#include <stdio.h>
#include <stdlib.h>
#include <stdint.h>

// integer
int value = -2000;
// uint8_t = unsigned 8bit integer(0～255)
uint8_t value8 = 100;
// uint16_t = unsigned 16bit integer(0～65535)
uint16_t value16 = 65000;

void main() {

    // 変数の値を表示する
    // printfの書式：
    //   符号あり：%d
    //   符号なし：%u
    printf("value:\n");
    printf(" integer : %d\n", value);
    printf(" uint8_t : %u\n", value8);
    printf(" uint16_t: %u\n", value16);

    // 変数のポインタを表示する
    // 先頭に'&'をつけることで取得できる
    // printfの書式：
    //   ポインタ：%p
    printf("pointer:\n");
    printf(" integer : 0x%p\n", &value);
    printf(" uint8_t : 0x%p\n", &value8);
    printf(" uint16_t: 0x%p\n", &value16);

    // 値を更新する
    printf("\nvalue update.\n");
    value = 2000;
    value8 = 255;
    value16 = 65535;

    // 変数の値を表示する
    printf("value:\n");
    printf(" integer : %d\n", value);
    printf(" uint8_t : %u\n", value8);
    printf(" uint16_t: %u\n", value16);

    // 変数のポインタを表示する
    printf("pointer:\n");
    printf(" integer : 0x%p\n", &value);
    printf(" uint8_t : 0x%p\n", &value8);
    printf(" uint16_t: 0x%p\n", &value16);

    return;
}
```

- 変数は主にint、uint16_t、uint8_tを使用する。(uint16_t、uint8_tはunsigned)
- ポインタ（変数の値が格納されているアドレス）を参照する際は`&`を付ける。

<br>

## コンパイル

compile.sh

```shell
#!/bin/sh
zcc +msx -lmsxbios --list -subtype=msxdos $1.c -o $1.com
```

<br>

実行

```shell
./compile.sh ./var1
```

<br>

## 実行結果

![/resources/2026-01-18-z88dk-c-valiable1_01.png](/resources/2026-01-18-z88dk-c-valiable1_01.png)

- 各型の長さは、int=2byte、uint8_t=1byte、uint16_t=2byteとなる。

※MSXDOS.SYSとCOMMAND.COM、コンパイルして作成した.COMを配置したディレクトリをopenMSXでマウント、MSXDOSから実行
