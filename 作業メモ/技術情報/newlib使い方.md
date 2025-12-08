- リセットハンドラ処理
	- data,bssの初期化はリセットハンドラで行う
	- 初期化ルーチンとして、`____libc_init_array()`を呼び出す。
	- ユーザプログラムを呼び出す
- ビルド
	- リンカは arm-none-eabi-ld.exeではなくarm-none-eabi-gcc.extを使う,newlibを指定するための `--specs=nosys.specs --specs=nano.specs`が使えないため
	- リンカにライブラリ場所を指定するとarmモードのライブラリがリンクされてしまうため`-LXXXX/lib`などは使わない、`--cpu=cortex-m4+nofp --mthumb` とコア種別とモードを指定することで適切なライブラリがリンクされる。
	- リンカ設定例
```
	TOOLCHAIN ?= "C:/Program Files (x86)/GNU Arm Embedded Toolchain/10 2021.10"
	PREFIX    := arm-none-eabi
	LD        := $(TOOLCHAIN_BIN)/$(PREFIX)-gcc.exe
	LDFLAGS   := -mcpu=cortex-m4+nofp -mthumb --specs=nosys.specs --specs=nano.specs
	LDLIBS    := -lc -lm -lg
	LDSCRIPT  := $(DRIVER_ROOT)/startup/cm4/src/GCC/CXD2888_cm4.ld
	
	$(TARGET) : $(OBJS)
	    $(LD) $(LDFLAGS) $(LDLIBS) -T $(LDSCRIPT) -o $@ $(OBJS)
``` 
- リンカスクリプト
	- 念の為、以下のセクションをいれる。※組み込みではexit()を使うことがないので.finiは不要かもしれない。
```
    .init :
    {
        KEEP(*(.init))
    } > ROM

    .fini :
    {
        KEEP(*(.fini))
    } > ROM

    /* 例外インデックス */
    __exidx_start = .;
    .ARM.exidx : { *(.ARM.exidx* .gnu.linkonce.armexidx.*) } > ROM
    __exidx_end = .;
```
- システムコールの実装
	- syscalls.cなどのファイルを作成して、`_write(), etc.`の必要な処理を実装する。 
	- syscalls.cのテンプレート
		- 標準週出力をUARTに置き換える場合 `_write(),_read()`を
```
// syscalls.c - newlib のシステムコール実装（UART 使用版）
// Bare-metal ARM Cortex-M 用
// （必須：_write, _sbrk, _close, _read など最低限を定義）

#include <errno.h>
#include <sys/stat.h>
#include <sys/types.h>

// ------------------------------------------------------
// UART 出力：ユーザが実装する必要あり
// ------------------------------------------------------
extern void uart_putc(char c);

// ------------------------------------------------------
// write() : printf など標準出力が使用
// ------------------------------------------------------
int _write(int file, const char *ptr, int len)
{
    (void)file; // STDOUT/STDERR 区別しない場合

    for (int i = 0; i < len; i++) {
        if (ptr[i] == '\n') {
            uart_putc('\r');  // CR+LF にしたい場合
        }
        uart_putc(ptr[i]);
    }
    return len;
}

// ------------------------------------------------------
// read() : 標準入力（必要なければ未実装でOK）
// ------------------------------------------------------
int _read(int file, char *ptr, int len)
{
    (void)file;
    // UART からの入力が必要な場合に実装
    // 例：
    // for (i

```

	- OSを使ってmallocをフレッドセーフにする場合
		以下の２つのフックを用意して、ヒープ操作の前後で呼び出す。
		- `__malloc_lock(struct _reent *r)`
		- `__malloc_unlock(struct _reent *r)`
		FreeRTOSを使った場合の記述例
```
#include "FreeRTOS.h"
#include "semphr.h"

static SemaphoreHandle_t malloc_mutex;

void malloc_lock_init(void)
{
    malloc_mutex = xSemaphoreCreateMutex();
}

// newlib から呼ばれる（名前は固定）
void __malloc_lock(struct _reent *r)
{
    (void)r;
    if (malloc_mutex) {
        xSemaphoreTake(malloc_mutex, portMAX_DELAY);
    }
}

void __malloc_unlock(struct _reent *r)
{
    (void)r;
    if (malloc_mutex) {
        xSemaphoreGive(malloc_mutex);
    }
}

```
	2025年12月05日 printfの実装までは完了。ただしUARTからの標準入力がうまく動作しないので調査が必要。
	　→月曜日は newlib をコンパイルして、ソースコードデバックしてみる。