# ELF 讀書筆記

## 說在前頭
此份文件是我自己的讀書心得，針對我理解的部份做整理。學海無涯，仍有許多我未了解與不足之處，尚祈見諒。

## ELF格式初探

ELF是Executable and Linking Format的縮寫，是GNU/Linux作業系統的二進位執行檔格式，功能等同於Windows 作業系統的PE(Portable Executable)，在GNU/Linux作業系統下可以使用**file**指令來確認檔案是否為ELF執行檔。

```
$ file /bin/ls
/bin/ls: ELF 64-bit LSB  executable, x86-64, version 1 (SYSV), dynamically linked (uses shared libs), for GNU/Linux 2.6.24, BuildID[sha1]=bd39c07194a778ccc066fc963ca152bdfaa3f971, stripped
```
在**file**指令輸出結果裡出現ELF executable，即表示為ELF執行檔，ELF內容資訊組成眾多，讓我們先來看前面3個標頭，如下：

1. ELF Header
2. Program Header
3. Section Header

## ELF Header
使用**readelf**指令來輸出ELF Header所有資訊，資訊有Class、Machine、Size of program headers等資訊，所顯示的結果，有些資訊一看便了解如Class、OS/ABI、Type、Machine，有些仍然一知半解，如Magic、Start of program headers、Start of section headers。一知半解沒關係，就先放在心底就好。

```
$ readelf -h /bin/ls
ELF Header:
  Magic:   7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00 
  Class:                             ELF64
  Data:                              2's complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI Version:                       0
  Type:                              EXEC (Executable file)
  Machine:                           Advanced Micro Devices X86-64
  Version:                           0x1
  Entry point address:               0x404890
  Start of program headers:          64 (bytes into file)
  Start of section headers:          108288 (bytes into file)
  Flags:                             0x0
  Size of this header:               64 (bytes)
  Size of program headers:           56 (bytes)
  Number of program headers:         9
  Size of section headers:           64 (bytes)
  Number of section headers:         28
  Section header string table index: 27
```

## Program Header

同樣使用**readelf**指令來輸出Program header資訊，輸出結果有Type, PHDR, INTERP, LOAD, DYNAMIC等資訊，一共有9個Program headers資料，此程式輸出內容裡有提到與Section(後面會提到)之對應，從00 - 08，00與07無資料，01-06、08 看起來Program Header跟Section Header有一定的對應關係。先了解有這樣的對應關係就好，我想把重點擺在後面的Section Header。

```
$ readelf -l /bin/ls
Elf file type is EXEC (Executable file)
Entry point 0x404890
There are 9 program headers, starting at offset 64

Program Headers:
  Type           Offset             VirtAddr           PhysAddr           FileSiz            MemSiz              Flags  Align
  PHDR           0x0000000000000040 0x0000000000400040 0x0000000000400040 0x00000000000001f8 0x00000000000001f8  R E    8
  INTERP         0x0000000000000238 0x0000000000400238 0x0000000000400238 0x000000000000001c 0x000000000000001c  R      1
      [Requesting program interpreter: /lib64/ld-linux-x86-64.so.2]
  LOAD           0x0000000000000000 0x0000000000400000 0x0000000000400000 0x0000000000019d44 0x0000000000019d44  R E    200000
  LOAD           0x0000000000019df0 0x0000000000619df0 0x0000000000619df0 0x0000000000000804 0x0000000000001570  RW     200000
  DYNAMIC        0x0000000000019e08 0x0000000000619e08 0x0000000000619e08 0x00000000000001f0 0x00000000000001f0  RW     8
  NOTE           0x0000000000000254 0x0000000000400254 0x0000000000400254 0x0000000000000044 0x0000000000000044  R      4
  GNU_EH_FRAME   0x000000000001701c 0x000000000041701c 0x000000000041701c 0x000000000000072c 0x000000000000072c  R      4
  GNU_STACK      0x0000000000000000 0x0000000000000000 0x0000000000000000 0x0000000000000000 0x0000000000000000  RW     10
  GNU_RELRO      0x0000000000019df0 0x0000000000619df0 0x0000000000619df0 0x0000000000000210 0x0000000000000210  R      1

 Section to Segment mapping:
  Segment Sections...
   00
   01     .interp
   02     .interp .note.ABI-tag .note.gnu.build-id .gnu.hash .dynsym .dynstr .gnu.version .gnu.version_r .rela.dyn .rela.plt .init .plt .text .fini .rodata .eh_frame_hdr .eh_frame
   03     .init_array .fini_array .jcr .dynamic .got .got.plt .data .bss
   04     .dynamic
   05     .note.ABI-tag .note.gnu.build-id
   06     .eh_frame_hdr
   07
   08     .init_array .fini_array .jcr .dynamic .got
```

## Section Header

通常做ELF執行檔靜態分析的時候，我會想要從執行檔的Section Header資訊，推敲程式開發者在撰寫、編譯程式原始碼過程中所留下的蛛絲馬跡，若執行檔處於沒有被strip狀態，則更可透露出更多訊息。在此我們使用自己撰寫的C語言程式helloelf.c，編譯過後，使用**objdump**傾印所產生的ELF執行檔。

- helloelf.c內容

```
#include <stdio.h>

char DNS_ADDR[8] = "8.8.8.8\0";
const int DNS_PORT = 15;

void func01 ()
{
}

int main(void)
{
        return 0;
}
```

- 編譯helloelf.c

```
$ gcc -o helloelf helloelf.c
```

- 使用**objdump**指令傾印所產生的ELF執行檔(helloelf)，以section分段印出內容

```
$ objdump -s ./helloelf

./helloelf:     file format elf64-x86-64

[略]
Contents of section .text:
 400400 31ed4989 d15e4889 e24883e4 f0505449  1.I..^H..H...PTI
 400410 c7c07005 400048c7 c1000540 0048c7c7  ..p.@.H....@.H..
 400420 f3044000 e8b7ffff fff4660f 1f440000  ..@.......f..D..
 400430 b8471060 0055482d 40106000 4883f80e  .G.`.UH-@.`.H...
 400440 4889e577 025dc3b8 00000000 4885c074  H..w.]......H..t
 400450 f45dbf40 106000ff e00f1f80 00000000  .].@.`..........
 400460 b8401060 0055482d 40106000 48c1f803  .@.`.UH-@.`.H...
 400470 4889e548 89c248c1 ea3f4801 d048d1f8  H..H..H..?H..H..
 400480 75025dc3 ba000000 004885d2 74f45d48  u.]......H..t.]H
 400490 89c6bf40 106000ff e20f1f80 00000000  ...@.`..........
 4004a0 803d990b 20000075 11554889 e5e87eff  .=.. ..u.UH...~.
 4004b0 ffff5dc6 05860b20 0001f3c3 0f1f4000  ..].... ......@.
 4004c0 48833d58 09200000 741eb800 00000048  H.=X. ..t......H
 4004d0 85c07414 55bf200e 60004889 e5ffd05d  ..t.U. .`.H....]
 4004e0 e97bffff ff0f1f00 e973ffff ff554889  .{.......s...UH.
 4004f0 e55dc355 4889e5b8 00000000 5dc36690  .].UH.......].f.
 400500 41574189 ff415649 89f64155 4989d541  AWA..AVI..AUI..A
 400510 544c8d25 f8082000 55488d2d f8082000  TL.%.. .UH.-.. .
 400520 534c29e5 31db48c1 fd034883 ec08e875  SL).1.H...H....u
 400530 feffff48 85ed741e 0f1f8400 00000000  ...H..t.........
 400540 4c89ea4c 89f64489 ff41ff14 dc4883c3  L..L..D..A...H..
 400550 014839eb 75ea4883 c4085b5d 415c415d  .H9.u.H...[]A\A]
 400560 415e415f c366662e 0f1f8400 00000000  A^A_.ff.........
 400570 f3c3                                 ..              
[略]
Contents of section .rodata:
 400580 01000200 0f000000                    ........        
[略]
Contents of section .data:
 601028 00000000 00000000 00000000 00000000  ................
 601038 382e382e 382e3800                    8.8.8.8.        
Contents of section .comment:
 0000 4743433a 20285562 756e7475 20342e38  GCC: (Ubuntu 4.8
 0010 2e342d32 7562756e 7475317e 31342e30  .4-2ubuntu1~14.0
 0020 342e3129 20342e38 2e340047 43433a20  4.1) 4.8.4.GCC: 
 0030 28556275 6e747520 342e382e 322d3139  (Ubuntu 4.8.2-19
 0040 7562756e 74753129 20342e38 2e3200    ubuntu1) 4.8.2. 
```

在此我刻意忽略掉其他Section headers的輸出結果，先針對.text、.rodata、.data、.comment 重要的Section以表格做說明

|Section Name|說明|對應helloelf.c
|---|---|---
|.text| 放置機器語言程式碼的區段|main()、func01()函式內容
|.data| 放置程式可覆寫資料區段| char DNS_ADDR[8]變數
|.rodata| 放置程式唯讀資料區段| const int DNS_PORT變數
|.comment|放置程式編譯資訊| gcc 編譯

進一步使用**objdump**指令反組譯的功能(-d)查看.rodata與.data Section的內容，可以發現helloelf.c裡面的變數名稱、對應的值、自訂函式名稱與對應的反組譯內容就顯示出來了，面對陌生的ELF執行檔，使用此指令，可以先掌握所使用的變數名稱與對應的內容、自訂函式名稱，去推敲可能的程式功能。

```
$ objdump -d -j .data ./helloelf

./helloelf:     file format elf64-x86-64
Disassembly of section .data:
[略]
0000000000601038 <DNS_ADDR>:
  601038:	38 2e 38 2e 38 2e 38 00                             8.8.8.8.

$ objdump -d -j .rodata ./helloelf
./helloelf:     file format elf64-x86-64
Disassembly of section .rodata:
[略]

0000000000400584 <DNS_PORT>:
  400584:	0f 00 00 00                                         ....

$ objdump -d -j .text ./helloelf

./helloelf:     file format elf64-x86-64
Disassembly of section .text:
[略]
00000000004004ed <func01>:
  4004ed:       55                      push   %rbp
  4004ee:       48 89 e5                mov    %rsp,%rbp
  4004f1:       5d                      pop    %rbp
  4004f2:       c3                      retq

00000000004004f3 <main>:
  4004f3:       55                      push   %rbp
  4004f4:       48 89 e5                mov    %rsp,%rbp
  4004f7:       b8 00 00 00 00          mov    $0x0,%eax
  4004fc:       5d                      pop    %rbp
  4004fd:       c3                      retq
  4004fe:       66 90                   xchg   %ax,%ax
[略]
```
