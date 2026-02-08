---
layout: page
title:  "【MSX】VDP操作によるVRAMアクセスについて"
date:   2026-01-27 00:00:00 +0900
categories: MSX
---

初版 2026/01/27  
改訂 2026/01/28  

-----

## 概要

- VRAMは、CPUからI/O経由でVDP経由でアクセスする。そのため、I/O操作命令（OUT/IN）を使用する。
- なお、MSXでは互換性の保証のため、周辺ハードウェアへのアクセスはBIOS経由を推奨しているが、VRAMへのアクセスについては直接I/O経由でのアクセスが認められている。

![2026-01-27-msx-vdp-access.svg](/resources/2026-01-27-msx-vdp-access_01.svg)

- VDPは大きく「コマンド発行」「コマンド実行」の２つの手順が必要。
- VDPのレジスタ操作は最低12ステート、読み書きは最低21ステートの間隔を開けること。ただし、`OUT (C), r`の場合は連続して書いても問題ない。[^1][^2]
- I/O操作中は割り込みがかからないようにすること。[^3]

<br>

## VRAM1バイト読み込み

```asm
    ; ---- SET READ ----
    DI

    LD      A, (0x0006)         ; get read port #0 address
    LD      C, A
    INC     C                   ; port #1 address

    LD      A, L                ; HL = read addr.
    OUT     (C), A
    LD      A, H
    OUT     (C), A

    ; ---- READ ----
    DEC     C                   ; port #1 address
    IN      A, (C)              ; A = data

    EI
```

<br>

## VRAM1バイト書き込み

- 書き込みをする際は、ポート#1に送るアドレスのビット7をONにする。

```asm
    ; ---- SET WRITE ----
    DI

    LD      A, (0x0007)         ; get write port #0 address
    INC     A
    LD      C, A                ; port #1 address

    LD      A, E                ; DE = dist addr.
    OUT     (C), A
    LD      A, D
    OR      0x40                ; write mode
    OUT     (C), A

    ; ---- WRITE ----
    LD      A, (HL)             ; HL = data addr.
    DEC     C                   ; port #0 address
    OUT     (C), A

    EI
```

<br>

## VRAM連続書き込み

- `OUTI`を実行すると、データのアドレス（HL）は自動的にインクリメント、カウンタ（B）はデクリメントされ、カウンタがゼロになった時にゼロフラグが立つ。
- 書き込み先のVRAMアドレスは、VDPにより自動的にインクリメントされる。これを利用して、メモリにある連続したデータを高速にVRAMに送ることができる。

```asm
    ; ---- SET WRITE ----
    DI

    LD      HL, _data                   ; HL = data addr.
    LD      DE, 0x1800                  ; DE = dist addr.

_loop:
    ; ---- SET WRITE ----
    LD      A, (0x0007)                 ; get write port #0 address
    LD      C, A
    INC     C                           ; port #1 address

    LD      A, E                        ; set dist addr
    OUT     (C), A
    LD      A, D
    OR      0x40                        ; write mode
    OUT     (C), A

    ; ---- WRITE ----
    DEC     C                           ; port #0 address
    LD      B, 10                       ; col num

_loop_1:
    OUTI
    JP      NZ, _loop_1

    EI

    RET

_data:
    DB 0x41   ; A
    DB 0x42   ; B
    DB 0x43   ; C
    DB 0x44   ; D
    DB 0x45   ; E
    DB 0x46   ; F
    DB 0x47   ; G
    DB 0x48   ; H
    DB 0x49   ; I
    DB 0x4A   ; J
```

[^1]: https://beach.biwako.ne.jp/~beaver/msx/msxtecho/vdptimi.htm

[^2]: MSXの機種によってはもっと短くても良い場合もあるが、全ての機種で問題無く動作するための安全ラインはこの値となる。なお、MSX1（TMS9918）の場合で、MSX2（V9938）では15ステートで良い。

[^3]: MSX2（V9938）のアドレスカウンターはレジスタの影響を受けないため、この考慮は不要。
