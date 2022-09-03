---
layout: post
title:  "Makefile tiếng Việt"
image: ''
date: 2022-09-03 14:00 +0700
tags: [makefile]
description: 'Cách sử dụng Makefile'
categories: [build-system]
---

## Introduction

Thứ đưa tôi đến với Makefile là `OSdev`. Với tuổi trẻ bồng bột, nghĩ mình có thể build 1 hệ điều hành from scratch (thật ra là có thể). Tôi bỏ dở hệ điều hành của mình khi nó vừa đọc xong bàn phím và ghi lên màn hình :)). 

Trước khi đến với `OSdev` tôi đã làm việc với một số MCU `8051` và `AVR`. Xài Keil C (crack - điều hiển nhiên nếu ở VN) để biên dịch cho `8051`, xài `AVR Studio` rồi đến `Atmel Studio `(sau là `Microchip Studio`) cho `AVR`. Vì tôi cảm thấy vô cùng tội lỗi khi xài đồ crack nên không lâu sau chuyển hoàn toàn sang dùng AVR với `Atmel Studio` :)). Tất nhiên là 8051 có `SDCC` nhưng tối lúc đó "chưa đủ tuổi" để xài `SDCC` (xời, tuổi gì xài SDCC :D). 

`Atmel Studio` vỏ  `Visual Studio`, ruột `Eclispe`, lõi `avr-gcc`. Sau khi trở về từ `OSdev` tôi có thể tự mình config các build flag cho `toolchain` trong `Eclipse` của `Atmel Studio`, đặc biệt là macro `F_CPU`. Thề, `F_CPU` đặt trong `main.c` đeo' bao giờ mà build ra nhận giá trị đúng. Bởi, `F_CPU` cần phải đặt là *global macro* thì tất cả các file `.c` `.h` mới nhận được giá trị đúng. Đến khi tôi tìm được flag `-D` của `gcc` thì mọi chuyện được giải quyết. Từ lúc đó tôi vẫn sử dụng toolchain config (Project properties) để config `F_CPU` cho project của mình, cho tới khi tìm được chân ái của cuộc đời mình là `Makefile`.

## Sơ lược gcc, toolchain và quá tình biên dịch và liên kết

GCC là một trình biên dịch của GNU, hỗ trợ nhiều nền tảng khác nhau từ phần cứng cho tới hệ điều hành. Với phần cứng, dễ thấy nhất là `arm-none-eabi-gcc` hay dùng lập trình cho STM32, hay `avr-gcc` cho AVR, với các SoC ARM chạy linux ta có `arm-linux-gnueabihf-gcc`,... GCC là thành phần chính của `GNU toolchain`, là trình biên dịch chuẩn cho hầu hết các dự án của GNU và cả Linux.

`Toolchain` là một tập hợp các công cụ để tạo ra phần mềm. Một công cụ lấy input từ công cụ trước đó để tạo ra output, output này trở thành input cho công cụ tiếp theo, tạo thành một chuỗi (chain). Các thành phần trong `GNU toolchain`:

- GNU make: công cu quản lí và tự động hóa quá trình build
- GCC: trình biên dịch
- glibc: thư viện gồm: header file, thư viện, loader
- GNU binutils: tập hợp các công cụ gồm: linker, assembler,...
- GNU Bison: trình phân tích cú pháp
- GNU m4: trình xử li macro
- GDB: trình gỡ lỗi
- GNU Libtool: công cụ thư viên động.

Biên dịch và liên kết một file thực thi:

![quá trình biên dịch và liên kết một file thuc thi](/assets/makefile-tieng-viet/build-order.svg){:style="filter: invert(100%);"}

Trong ảnh trên tôi đã lược bỏ đi quá trình preprocess và assembly vì trong bài này tôi không tập trung vào quá trình biên dịch file C. Các file `.c` sẽ được biên dịch thành các file `.o`, sau đó các file này được liên kết với một số thư viện (etc. glibc) và thêm vào một số phần code startup (etc. crt.a).

## Mục tiêu

Trong giai đoạn nghiên cứu và phát triển đam mê, tôi quen rất nhiều người bạn, những người có khả năng lập trình nhưng vẫn sử dụng IDE, một số người bày tỏ mong muốn tìm hiểm về Makefile nhưng

- không có thời gian,
- không tìm được tài liệu chất lượng (tiếng Việt),
- không đủ trình độ nghiên cứu tài liệu tiếng Anh cho một vấn đề cụ thể.

Tôi mong muốn viết bài này cho mọi người cùng học hỏi và để tra lại khi cần
thiết. Vì kiến thức tích lũy bao năm qua, tôi sợ một ngày nào đó mình sẽ quên
dần đi. Tôi định viết bài này rất lâu rồi nhưng vì `lười`. Tranh thủ kì hè năm 2 rảnh rỗi ngồi viết dần, để tích lũy bài viết lên blog.

## Vấn đề

Khi viết bài này, tôi hay vấp phải vần đề  `trứng hay gà`, bởi vì kiến thức tôi
trình bày trong một phần yêu cầu kiến thức trong phần khác và ngược lại. Tôi giả
sử bạn có bootstrap kiến thức của một trong 2 phần đó .Tôi sẽ trình bày theo
hướng bottom-up, đó là đi từ vấn đề cốt lõi tại sao sinh ra `Makefile`.

### Đặt vấn đề 	

Giả sử, khi lập trình, với C, bạn có thể code mọi thứ trên 1 file .c, nhưng sẽ khó khăn khi maintain

- Một file .c hàng chục nghìn dòng tốn thời gian để build hơn 1 file có ít dòng.
- Chỉ cần sửa 1 lỗi nhỏ nhưng phải build lại toàn bộ file.
- Quá trình maintain lại muốn sử dụng thư viện ngoài, `.h` `.c`

### Hướng giải quyết

Có 1 cách giải quyết đó là chia file `main.c` thành các file `.c`, rồi build các file `.c` thành `.o`. Việc này có thể thực hiện dễ dàng bằng `shell script`.

Giả sử  tôi có 3 file `foo0.c`, `foo1.c` và `main.c`, tôi muốn build 3 file này thành 1 file thực thi. 

``` c
// main.c
#include <stdio.h>
#include "foo0.h"
#include "foo1.h"

int main() 
{
	foo0_func();
	foo1_func();
	return 0;
}
```

``` c
// foo0.c
#include <foo0.h>
void foo0_func() 
{
	printf("printf from foo0.c");
}
```

``` c
// foo1.c
#include <foo1.h>
void foo0_func() 
{
	printf("printf from foo1.c");
}
```

``` c
// cùng với 2 file:
// foo0.h
#include <stdio.h>
void foo0_func();

// foo1.h
#include <stdio.h>
void foo1_func();
```

**Build các file .c thành .o bằng gcc, link các file .o thành file thực thi**

``` sh
$ gcc foo0.c -c # biên dịch foo0.c thành foo0.o
$ gcc foo1.c -c # biên dịch foo1.c thành foo1.o
$ gcc main.c -c # biên dịch main.c thành main.o
$ gcc main.o foo0.o foo1.o # link các file .o thành file thực thi a.out
```

Mặc định gcc sẽ link 1 file .c thành file thực thi. Cờ -c để gcc biết rằng chỉ
cần biên dịch ra file .o mà không cần link thành file thực thi. Nếu không có lỗi
gì xảy ra thì 3 file là foo0.o, foo1.o, main.o và a.out sẽ được tạo ra trong thư
mục chứa file .c.

Việc build từng file `.c` thành từng file `.o` vấp phải một vấn đề triết học
mang tên `trứng hay gà`. Bởi vì trình biên dịch chỉ biên dịch 1 file mỗi lần, mà
trong file đó chứa lời gọi đến hàm trong 1 file .c khác. Cách giải quyết đó là
gán một tên ảo cho hàm đó, giả định hàm đó sẽ có đâu đó trong danh sách các hàm
trong quá trình link. Hay gọi ngắn là `prototype` hay chung quy hơn là `khai
báo` (declaration).

---
>**Ví dụ khi biên dịch main.c**
>
>`main.c `chứa 2 lời gọi hàm đó là `foo0_func()` và `foo1_func()`. Hai hàm này
>đều nằm trong các file khác, để trình biên dịch biết rằng 2 hàm này nằm ở đâu
>đó ngoài kia, những vẫn có một điểm nào đó để lời gọi hàm này trỏ tới, ta khai
>báo 2 hàm này trong `main.c`, hay `#include fooX.h`. Sau khi biên dịch main.o
>sẽ có 2 lời gọi `hàm chưa được định nghĩa`, đợi đến lúc link, 2 lời gọi hàm này
>sẽ trỏ tới phần thân hàm trong `fooX.c`.

---

Để dễ hình dung hơn, bạn có thể dùng [`nm`](https://linux.die.net/man/1/nm) - công cụ để list symbols từ file object (.o). `nm` là 1 công cụ trong bộ [`binutils`](https://www.gnu.org/software/binutils/), rất hữu ích trong việc trích xuất và xem nội dung file thực thi cũng như file object. Chạy `nm main.o foo1.o foo0.o` để list các symbol trong 3 file .o trên:

``` sh
$ nm main.o foo1.o foo0.o

main.o:
                 U foo0_func
                 U foo1_func
0000000000000000 T main

foo1.o:
0000000000000000 T foo1_func
                 U puts

foo0.o:
0000000000000000 T foo0_func
                 U puts
```

Ta có thể thấy output của `nm` chia thành 3 cột: giá trị, loại, tên theo thứ thụ từ trái sang phải. Với output trên, có 2 loại symbol là U và T. U nghĩa là "undefined", symbol chưa được định nghĩa (hay chưa có thân hàm). T nghĩa là "text", symbol nằm trong [.text sections](https://en.wikipedia.org/wiki/Code_segment), hay hiểu đơn giản là symbol này là một hàm, giá trị của nó là địa chỉ của hàm.

Đối với main.o, 2 hàm foo0_func(), foo1_func() chưa được định nghĩa, mong đợi được định nghĩa khi link.

Đối với `foo0.o`, hàm `foo0_func()`, đã được định nghĩa nhưng và có giá trị là 0, tức là địa chỉ của hàm `foo0_func()` trong `foo0.o` là 0. Địa chỉ này sẽ được thay đổi khi các .o link lại với nhau. Hàm `puts()` chưa được định nghĩa, vì ta dùng hàm `printf()`, `printf()` là một hàm wrapper cho `puts()`. Hàm `puts()` này sẽ được link với thư viện động khi link. Tương tự với `foo1.o`.

**Sau khi link thành file thực thi thì có gì ?**

Well, còn tùy thuộc vào nền tảng chạy file thực thi đó mà output của `nm` sẽ khác nhau, vì nó sẽ link với thư viện động của hệ điều hành. Ví dụ, tôi đang sử dụng Linux (`Arch Linux` btw) và `x86_64-pc-linux-gnu` để build thành file `a.out` ở trên. Đây là output khi chạy `nm` với `a.out`:

``` sh
$ nm a.out

0000000000003df8 d _DYNAMIC
0000000000004000 d _GLOBAL_OFFSET_TABLE_
0000000000002000 R _IO_stdin_used
                 w _ITM_deregisterTMCloneTable
                 w _ITM_registerTMCloneTable
0000000000002120 r __FRAME_END__
0000000000002030 r __GNU_EH_FRAME_HDR
0000000000004030 D __TMC_END__
000000000000039c r __abi_tag
0000000000004030 B __bss_start
                 w __cxa_finalize@GLIBC_2.2.5
0000000000004020 D __data_start
00000000000010e0 t __do_global_dtors_aux
0000000000003df0 d __do_global_dtors_aux_fini_array_entry
0000000000004028 D __dso_handle
0000000000003de8 d __frame_dummy_init_array_entry
                 w __gmon_start__
                 U __libc_start_main@GLIBC_2.34
0000000000004030 D _edata
0000000000004038 B _end
0000000000001184 T _fini
0000000000001000 T _init
0000000000001040 T _start
0000000000004030 b completed.0
0000000000004020 W data_start
0000000000001070 t deregister_tm_clones
0000000000001139 T foo0_func
000000000000114f T foo1_func
0000000000001130 t frame_dummy
0000000000001165 T main
                 U puts@GLIBC_2.2.5
00000000000010a0 t register_tm_clones
```

Ta có thể thấy một mớ symbol lằng nhằng được thêm vào sau khi link thành file thực thi, những symbol này có liên quan tới trình biên dịch và nền tảng thực thi. Bạn có thể google những symbol lằng nhằng kia nếu tò mò.

`foo0_func`, `foo1_func`, `main` đã có địa chỉ. Tuy nhiên, địa chỉ này chỉ là tương đối, khi thực thi, file thực thi sẽ được load vào ram và sẽ có địa chi `<địa chỉ loader> + <địa chỉ tương đối>`.

**Quay về vấn đề chính**

Tôi có thể build nhiều file .c thành 1 file thực thi, thế quái nào tôi lại cần `Makefile` làm gì ? Vấn đề, `shell script` sẽ chạy từ trên xuống, bất kể file .c có chỉnh sửa chưa, `shell script` vẫn sẽ build lại file đó. Hãy tưởng tượng bạn có hàng nghìn file .c, mỗi file build mất 5s, bạn chỉ sửa lại một lỗi nhỏ trong 1 file .c bé tẹo, nhưng phải chờ vài tiếng đồng hồ để build lại cả project. Đó là lúc `Makefile` nhảy vào.

## Make, makefile là gì ?

`Make` là công cụ tự động build lại những file .c đã có sự thay đổi. Make có thể sử dụng không chỉ trong lập trình, có thể sử dụng bất cứ đâu mà file output cần phải tự động update khi những file input có sự thay đổi.

Khi chạy `make`, `make` sẽ tìm thư mục hiện tại có file tên là `Makefile`, `makefile` hoặc `GNUmakefile` để tìm cách build đã vẽ ra trong những file đó. Nếu không tìm thấy `makefile` nào, `make` sẽ mặc định chạy *implicit rule*.

Nếu bạn cảm thấy bài post này chán, nhưng vẫn muốn tìm hiểu vê `make`, here you go, [make](https://www.gnu.org/software/make/manual/make.html).

## Rules

Rule có dạng:

``` makefile
target … : prerequisites …
        recipe
        …
        …
```

- *target* thông thường là file, được tạo ra khi rule này chạy, giống kiểu output.
- *prerequisites* là một hoặc nhiều file input cần để tạo ra file target.
- *recipe* là hành động tạo ra file target, có thể là một lệnh hoặc nhiều lệnh, trên cùng một dòng hoặc nhiều dòng. **Lưu ý:** Cần phải đặt một dấu `tab` trước môi dòng recipe, thì make mới hiểu đó là dòng recipe.

Thông thường, recipe trong rule sẽ thực thi, để tạo ra target, khi có bất cứ file nào thay đổi ở prerequisites. Rule có thể có prerequisites hoặc không.

Rule giải thích cho make,

- khi nào cần build lại file target (prerequisites)
- và làm sao build được nó (recipe)

Đơn giản, recipe sẽ chạy khi rule có:

- target chưa tồn tại, prerequisites tồn tại, trường hợp này target sẽ được **tạo ra**,
- target tồn tại, prerequisites bị thay đổi, trường hợp này target cần **update**.

Quay lại với vấn đề build 3 file `foo0.c`, `foo1.c`, `main.c`. Tôi có makefile mẫu:

``` makefile
# Thêm foo0.h và foo1.h để chắc chắn 2 file này tồn tại để 
# main.c có thể include được. 2 file này có thể thêm vào prerequisites
# hoặc không
main.o: main.c foo0.h foo1.h 
	gcc main.c -o main.o -c

foo0.o: foo0.c foo0.h
	gcc foo0.c -o foo0.o -c

foo1.o: foo1.c foo1.h
	gcc foo1.c -o foo1.o -c

# Linking
example.elf: main.o foo0.o foo1.o
	gcc main.o foo0.o foo1.o
```

Khi chạy `make example.elf`:

``` sh
$ make example.elf 

gcc main.c -o main.o -c
gcc foo0.c -o foo0.o -c
gcc foo1.c -o foo1.o -c
gcc main.o foo0.o foo1.o -o example.elf
```

`Make` sẽ tìm tới rule `example.elf`, tìm 3 file prerequisites, nếu 3 file đó chưa được tạo ra thì tìm tới từng rule để tạo ra file đó. Giả sử `main.o`, make sẽ tìm tới rule `main.o`, kiểm tra `prerequisites`, nếu có đủ, chạy recipe, sau khi chạy xong sẽ trở về rule `example.elf`, kiểm tra prerequisites tiếp theo là foo0.o, cứ tiếp tục như vậy đến khi prerequisites có đủ các file thì recipe của rule `example.elf` sẽ chạy để tạo ra `example.elf`.

### Pattern Rules

`Makefile` ở trên có thể viết gọn hơn dùng Pattern Rules:

``` makefile
%.o: %.c
	gcc $^ -o $@ -c

# Linking
example.elf: main.o foo0.o foo1.o
	gcc main.o foo0.o foo1.o
```

Bất cứ khi nào prerequisites của rule nào đó match với target của pattern rule, 
thì pattern rule sẽ được thực thi với `%` là phần còn lại không match sẽ thay thế target và prerequisites. Ví dụ, khi rule example.elf được thực thi, nó tìm main.o, main.o match với `%.o: %.c` lúc này pattern rule sẽ trở thành `main.o: main.c`.

`$^` là `biến tự động` chứa tên của tất cả prerequisites. `$@` là `biến tự động` chứa tên của target. Ngoài ra còn có `$<` chứa tên của một prerequisites đầu tiên.
Ví dụ: khi match với main.o thì pattern rule sẽ giống như này:

``` makefile
main.o: main.c
	gcc main.c -o main.o -c
```

Việc sử dụng pattern rule giúp makefile đơn giản hơn và dễ quản lí. Mặc định, `%.o: %.c` là một trong số những Implicit Rule tích hợp sẵn trong make (xem ví dụ 2 Implicit Rule).

### Implicit rule

**Ví dụ 1:**

Thông thường khi chạy make mà không có `makefile`, make sẽ dùng *Implicit rule*.
Rất hữu ích khi muốn build nhanh một file .c thành file thực thi.

``` c
#include <stdio.h>

int main() {
	printf("Test make");
	return 0;
}
```

Lưu file với tên test.c trong một thư mục không có *Makefile*. Chạy `make test`:

``` sh
$ make test

cc     test.c   -o test
```

File thực thi `test` sẽ được tạo ra. `cc     test.c   -o test` là dòng ngầm định khi build không có makefile. `cc` là một symlink tới `gcc`, về cơ bản là giống nhau.
Để sửa `cc` thành `gcc` hay thêm flag để dòng ngầm định trên trở thành `gcc -g test.c -o test` chẳng hạn thì ta chỉ cần gán cho các biến ngầm định như CC và CFLAGS:

``` makefile
CC = gcc
CFLAGS = -g
```

Output khi chạy `make test` với `makefile` trên:

``` sh
$ make test

gcc -g    test.c   -o test
```

Bạn có thể tra các biến ngầm định tại [đây](https://www.gnu.org/software/make/manual/make.html#Implicit-Variables)

**Ví dụ 2:**

Giả sử, ta có makefile:

``` makefile
example.elf: main.o foo1.o foo2.o
	gcc $^ -o example.elf
```
Ta không viết rule nào cho `main.o`, nhưng makefile sẽ tự động tìm *implicit rule* cho main.o. Việc chỉnh sửa recipe cho *implicit rule* này dựa vào *implicit variable* giống như ví dụ 1. Để biết các *implicit rule* và cách sắp xếp của *implicit variable* trong *implicit rule* ban có thể tra ở [đây](https://www.gnu.org/software/make/manual/html_node/Catalogue-of-Rules.html#Catalogue-of-Rules)

## Phony target

Phony target chỉ là một cái tên để luôn chạy recipe mỗi khi được gọi, không nhất thiết là một file cần được tạo ra. 

```makefile
clean: 
	rm -rf $(OBJS) example.elf
```
Trong ví dụ trên, môi khi chạy `make clean`, thì recipe sẽ luôn chay. Nhưng trên
đường đời tấp nập ta vô tình tạo ra một file tên `clean` thì make file kiểm tra thư mục hiện tai ([working directory](https://en.wikipedia.org/wiki/Working_directory)) có file clean đã được tạo ra và do không có dependency nên file này đuoc coi như là đã update nên recipe của rule này sẽ không chạy nếu ta chạy `make clean`. Để khắc phục vấn để này, để recipe này luôn luôn chạy mỗi khi `make clean` thì ta dùng phony target:

```makefile
.PHONY: clean
clean: 
	rm -rf $(OBJS) example.elf
```

## Makefile hoàn chỉnh

Thông thường, để quản lí project, makefile chỉ cần viết một lần. Các khi thêm một module vào project, ta chỉ cần sửa makefile thêm đường dẫn tới file .c bằng biến `SRCS`. Các file .o sau khi build sẽ lưu ở một thư mục khác (e.g. build, obj). Dưới đây là makefile mẫu cho một project C:

``` makefile
CFLAGS	=-g -O0
CC 	= gcc
SRCS	= main.c foo0.c foo1.c
OBJS	= $(patsubst %.c, %.o, $(SRCS)) # chuyển suffix .c thành .o: OBJS = main.o foo.o foo1.o

.PHONY: all clean

all: example.elf # make se mặc định chạy example.elf

example.elf: $(OBJS)
	$(CC) $(CFLAGS) $^ -o $@

# Implicit Rule, cái này có hay không cũng được, nhung ngầm đinh nó sẽ như thế này:
# %.o: %.c #
# 	$(CC) -c $(CFLAGS) $(CPPFLAGS) -o $@ $<

clean: 
	rm -rf $(OBJS) example.elf

```

## Một số preset build makefile link với .a (code không có hàm `main()`)

Với kiểu build này, người lập trình sẽ code trong một hoặc nhiều hàm, có flow khác với kiểu code trong main() truyền thống. Ví du tiêu biểu nhất là Arduino IDE, người lập trình sẽ code trong setup() và loop(), khi chương trình bắt đầu, sau khi chạy startup (của trình biên dịch), chương trình nhảy vào setup() 1 lần, sau đó nhảy vào một vòng lặp và lặp loop() đến chết. RTOS, SDK của xtensa cho esp8266 cũng theo kiểu này.

Mục đích của kiểu link này là đê giấu phần code của hệ thống đi, chỉ để lại phần usercode và API cho người dùng gọi tới. Phần API, system flow sẽ được build trong file .a kia, còn phần usercode sẽ được viết trong 1 vài hàm, khi system flow gọi tới hàm trong usercode thì phần code đó sẽ được thực thi.

Dưới đây là makefile cho esp8266 với esp8266 non-os SDK, build trên wsl-ubuntu:
```makefile
XTENSA		?=

# Mac and linux
SDK_BASE	?= /mnt/c/otp/Espressif/ESP8266_NONOS_SDK
ESPTOOL		?= /mnt/c/otp/Espressif/utils/esptool/esptool.py

# Windows with unofficial dev kit (default install location is C:/Espressif)
 SDK_BASE_WIN	?= C:/otp/Espressif/ESP8266_NONOS_SDK
 ESPTOOL_WIN	?= C:/otp/Espressif/utils/esptool/esptool.py

SDK_LIBS 	:= -lc -lgcc -lhal -lphy -lpp -lnet80211 -lwpa -lmain -llwip -lcrypto -ljson
CC			:= $(XTENSA)xtensa-lx106-elf-gcc
LD			:= $(XTENSA)xtensa-lx106-elf-gcc
AR			:= $(XTENSA)xtensa-lx106-elf-ar

LDFLAGS		= -nostdlib -Wl,--no-check-sections -u call_user_start -Wl,-static -Xlinker -Map=esp.map
CFLAGS 		= -g -Wpointer-arith -Wundef -Wl,-EL -fno-inline-functions -nostdlib\
			  -mlongcalls -mtext-section-literals -ffunction-sections -fdata-sections\
			  -fno-builtin-printf -DICACHE_FLASH\
			  -I.
LD_SCRIPT	= -T$(SDK_BASE)/ld/eagle.app.v6.ld

all: main.bin

main.bin: main.out
	xtensa-lx106-elf-objdump -S $< > esp.lss
	$(ESPTOOL) elf2image $(ESPTOOL_FLASHDEF) main.out -o main
	
main.out: main.a
	@echo "LD $@"
	$(LD) -L$(SDK_BASE)/lib $(LD_SCRIPT) $(LDFLAGS) -L$(SDK_BASE)/lib -Wl,--start-group $(SDK_LIBS) main.a -Wl,--end-group -o main.out

main.a: main.o
	@echo "AR main.o"
	$(AR) cru main.a main.o rf_init.o
	
main.o:
	@echo "CC main.c & rf_init.c"
	$(CC) -I'$(SDK_BASE)/include/' $(CFLAGS) -c main.c -o main.o
	$(CC) -I'$(SDK_BASE)/include/' $(CFLAGS) -c rf_init.c -o rf_init.o
	
clean:
	rm -rf *.o *.bin *.a *.out

flash:
	python.exe $(ESPTOOL_WIN) --port \\\\.\\COM3 \
			   --baud 480600 \
			   write_flash --flash_freq 40m --flash_mode dio --flash_size 32m \
			   0x00000 main0x00000.bin \
			   0x10000 main0x10000.bin \
			   0x3fc000 $(SDK_BASE_WIN)/bin/esp_init_data_default.bin

.PHONY: all clean
```

```c
// main.c
#include <stdio.h>
#include "osapi.h"
#include "user_interface.h"
static os_timer_t led_timer;
static int led_value = 0;

void app_init() {
	os_printf("Helloo blahbla\r\n");
}
void user_init(void) {
	system_init_done_cb(app_init);
}
```
> Makefile tôi lấy từ 1 project có sẵn, có file rf_init.c nữa nhưng tôi không thêm vào để đỡ rối.

Ở `main.out:` main.a sẽ được link với `-lc -lgcc -lhal -lphy -lpp -lnet80211 -lwpa -lmain -llwip -lcrypto -ljson`. Sau đó được trích thành file .bin bằng esptool.py.

## Kết luận

Trên đây là kiến thức cơ bản Makefile cho những người mới bắt đầu, nếu bạn đọc tới đây thì bạn có thể tao bất kì project nào với Makefile kể từ bây giờ, không chỉ là code mà bât cứ project nào có dependency file. 

Bởi vì giới hạn về thời gian nên bài viết còn rât nhiều thiếu sót, bài viết này cần feedback để hoàn thiện hơn. Feel free to leave comments, tôi sẽ giải đáp thắc mắc và tiếp thu ý kiến từ bạn.