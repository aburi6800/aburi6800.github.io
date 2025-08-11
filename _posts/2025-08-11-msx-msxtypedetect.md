---
layout: post
title:  "【MSX】MSXのバージョンを判定する"
date:   2025-08-11 00:00:00 +0900
categories: msx msx2
---

初版 2025/08/11  
改訂 

-----

<br>

MSX1/2両用プログラムだが、[パレット変更](2025-08-10-msx-msx2palette.md)のようにMSX2のみの処理を入れたいことがある。  
この場合は、BASICのバージョンを判定することで実現できる。  
具体的には、以下のどちらかとなる。  

1. MAIN-ROMの0x002Dの値を判定する。0=ver1.0(MSX1), 1=ver2.0(MSX2)、2=Ver3.0(MSX2+)、3=Ver4.0(TurboR)となる。  
2. EXBRSA(0xFAF8)にSUB-ROMのスロットアドレスを判定する。入っていなければ(0)、MSX1となる。  

<br>

上記1.の方法でのサンプルプログラムを記載する。  

```
MSXVER              equ 0x002d

VERCHK:
	LD		A, (MSXVER)
	OR		A
	RET		Z
	:
	<MSX2以降の処理>
	:
```
