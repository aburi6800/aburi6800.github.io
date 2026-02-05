---
layout: page
title:  "【z88dk】C言語/変数(2) ポインタを介した操作"
date:   2026-01-19 00:00:00 +0900
categories: z88dk
---

初版 2026/01/19  
改訂

-----

var2.c

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

    // ポインタを取得する
    // ポインタ変数の型は変数と同じとし、先頭に'*'を、代入対象の変数には先頭に'&'をつける
    int *value_p = &value;
    uint16_t *value16_p = &value16;
    uint8_t *value8_p = &value8;

    // 変数の値を表示する
    // ポインタから変数の値を参照する際は、ポインタ変数の先頭に`*`を付ける
    printf("value:\n");
    printf(" integer : %d\n", *value_p);
    printf(" uint8_t : %u\n", *value8_p);
    printf(" uint16_t: %u\n", *value16_p);

    // 変数のポインタを表示する
    printf("pointer:\n");
    printf(" integer : 0x%p\n", value_p);
    printf(" uint8_t : 0x%p\n", value8_p);
    printf(" uint16_t: 0x%p\n", value16_p);

    // ポインタ経由で値を更新する
    printf("\nvalue update.\n");
    *value_p = 2000;
    *value8_p = 255;
    *value16_p = 65535;

    // 変数の値を表示する
    // ポインタから変数の値を参照する際は、ポインタ変数の先頭に`*`を付ける
    printf("value:\n");
    printf(" integer : %d\n", *value_p);
    printf(" uint8_t : %u\n", *value8_p);
    printf(" uint16_t: %u\n", *value16_p);

    // 変数のポインタを表示する
    printf("pointer:\n");
    printf(" integer : 0x%p\n", value_p);
    printf(" uint8_t : 0x%p\n", value8_p);
    printf(" uint16_t: 0x%p\n", value16_p);

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
$ ./compile.sh ./var2
```

<br>

## 実行結果

![2026-01-19-z88dk-c-valiable1_02.png](/resources/2026-01-19-z88dk-c-valiable2_01.png)

- [直接操作](/z88dk/2026/01/18/z88dk-c-valiable1.html)と同じ結果になることがわかる。

※MSXDOS.SYSとCOMMAND.COM、コンパイルして作成した.COMを配置したディレクトリをopenMSXでマウント、MSXDOSから実行
