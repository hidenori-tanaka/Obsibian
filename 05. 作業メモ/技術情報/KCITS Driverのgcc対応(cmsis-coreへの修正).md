## startup_XXXX.cをgcc向けに変更
- ベクタテーブルの修正
	cmsis_gcc.h で __VECTOR_TABLE_ARRRIBUTEで使っているベクタテーブルのセクション名が ".vectors" になっているのでリンカでも　".vectors" が.textの先頭に来るように変更　
	__INITIAL_SP は __StackTop となっているので、リンカで ".stack" セクションを作成したら。その最終アドレスを __StackTop となるように修正
	
- 割り込みハンドラの修正
	”.after_vectors" にセクション名を変更して、リンカスクリプトで ".vectors" セクションのあとに配置されるように修正。Reset_handlerは先頭に配置しておきたいので独自セクション .entry を定義　　
	
- リセットハンドラの修正
	armccでは標準ライブラリの初期化ルーチン `__main` でデータ、BSSの初期化を実行してたが、newlib では実行してくれないのでリセットハンドラで実行する必要がある。
	newlibを使う場合にC++を使わなければ初期化で呼び出す処理はなし。
		CMSISのdata/bss初期化ルーチンをもってきたら、転送テーブルの構造体のサイズがバイト数となっていた、転送はワードで行われるのでサイズをバイトに修正

## newlib のprintfをUART出力させる

- `_wirte(),_read(),etc.`を実装する
	実装例
	この実装例では `_sbrk()`も実装しており、malloc()も利用可能。
	KCITS実機で`printf(),getc(),pusc()`の動作は確認。ただしgets()はうまく動作しない。
```
/*############################################################################
 * Copyright 2025 Sony Semiconductor Solutions Corporation.
 */
/**
 * @file        syscalls.c
 *
 * @brief       retarget sdtin/stdout to UART for GCC
 *
 * @date        2025/12/04
 */
/*##########################################################################*/

#include <errno.h>
#include <sys/stat.h>
#include <sys/types.h>
#include "CXD2888.h"

#if !defined(DBG_UART)
#define DBG_UART 0
#endif

#if DBG_UART == 0
#define MY_UART CXD2888_UART0
#elif DBG_UART == 1
#define MY_UART CXD2888_UART1
#else
#error unknown uart number.
#endif

/* ------------------------------------------------------------ */
/* urat_init : UART初期化処理　　　　　                          */
/* UARTの初期化,etc.                                             */
/* ------------------------------------------------------------ */
void uart_init(void)
{
#if DBG_UART == 0
    /*
     * Set IOMUX and IOFONFIG, Reset, Supply clock UART0
     */
    CXD2888_IOCFG->IOCFG_UARTRXD0 &= ~(IOCFG_IE_MSK | IOCFG_OE_MSK | IOCFG_PULL_MSK);
    CXD2888_IOCFG->IOCFG_UARTRXD0 |= (IOCFG_IE_EN | IOCFG_OE_EN | IOCFG_PULL_UP);

    CXD2888_IOCFG->IOCFG_UARTTXD0 &= ~(IOCFG_IE_MSK | IOCFG_OE_MSK | IOCFG_PULL_MSK);
    CXD2888_IOCFG->IOCFG_UARTTXD0 |= (IOCFG_OE_EN | IOCFG_PULL_UP);

    CXD2888_RESET_CTRL->GRP4 &= ~(RST_SERAPB_UART0_X_MSK);
    CXD2888_RESET_CTRL->GRP4 |= (RST_SERAPB_UART0_X_MSK);
    CXD2888_CLOCK_EN->GRP4 |= (CKEN_SERAPB_UART0_MSK);
#elif DBG_UART == 1
    /*
     * Set IOMUX and IOFONFIG, Reset, Supply clock UART1
     */
    CXD2888_IOMUX->IOMUX_ALTSEL0 &= ~(IOMUX_ALT0_I2CS_MSK);
    CXD2888_IOMUX->IOMUX_ALTSEL0 |= (IOMUX_ALT0_I2CS_UART1);

    CXD2888_RESET_CTRL->GRP4 &= ~(RST_SERAPB_UART1_X_MSK);
    CXD2888_RESET_CTRL->GRP4 |= (RST_SERAPB_UART1_X_MSK);
    CXD2888_CLOCK_EN->GRP4 |= (CKEN_SERAPB_UART1_MSK);
#endif

    UART_Initialize(MY_UART);

    uart_ctrl_data_t para;
    para.baudrate    = 115200;
    para.databit     = UARTLCRH_DATABIT8;
    para.parity      = UART_PARITY_NO;
    para.stopbit     = UARTLCRH_STOPBIT1;
    para.rtsflowctrl = false;
    para.ctsflowctrl = false;
    para.clock       = SystemGetSerialClock();
    para.fen         = true;
    para.rxifl       = UART_IFL_18FULL;
    para.txifl       = UART_IFL_18FULL;
    para.rxe         = true;
    para.txe         = true;

    UART_SetCtrlData(MY_UART, &para);

    UART_Start(MY_UART);
}

// ------------------------------------------------------
// write() : 標準出力
// ------------------------------------------------------
int _write(int file, const char *ptr, int len)
{
    (void)file;  // 未使用

    int32_t ret;
    uint32_t rest;

    /* FIFO 上書き対策　*/
    while (!(MY_UART->UARTFR & UARTFR_TXFE_MSK)) {
    }

    ret = UART_WriteFifo(MY_UART, len, (uint8_t *)ptr, &rest);
    if (ret != 0) {
        return -EIO;
    }

    return (len - rest);
}

// ------------------------------------------------------
// read() : 標準入力
// ------------------------------------------------------
int _read(int file, const void *ptr, int len)
{
    (void)file;  // 未使用

    int32_t ret;
    int32_t rest = len;
    int32_t read = 0;

    while (rest != 0) {
        ret = UART_ReadFifo(MY_UART, (uint32_t)len, (uint8_t *)ptr, &read);
        if (ret != 0) {
            errno = ENOSYS;
            return -1;
        }
        rest -= read;
    }

    return len;
}

// ------------------------------------------------------
// close() : ファイルを持たないので未実装
// ------------------------------------------------------
int _close(int file)
{
    (void)file;
    return -1;
}

// ------------------------------------------------------
// fstat() : デバイスであることを返す
// ------------------------------------------------------
int _fstat(int file, struct stat *st)
{
    (void)file;
    st->st_mode = S_IFCHR;  // キャラクタデバイス
    return 0;
}

// ------------------------------------------------------
// isatty() : 端末デバイスであると返す
// ------------------------------------------------------
int _isatty(int file)
{
    (void)file;
    return 1;
}

// ------------------------------------------------------
// lseek() : シーク不可
// ------------------------------------------------------
int _lseek(int file, int ptr, int dir)
{
    (void)file;
    (void)dir;
    errno = ESPIPE;
    return -1;
}

// ------------------------------------------------------
// sbrk() : newlib の malloc が使用するメモリ領域拡張
// ------------------------------------------------------
// リンカスクリプトで定義されているヒープ境界を利用
extern char __end__;      // ヒープ の開始位置
extern char __HeapLimit;  // ヒープ の終端

static char *heap_end = 0;

caddr_t _sbrk(int incr)
{
    if (heap_end == 0) {
        heap_end = &__end__;
    }

    char *prev_heap_end = heap_end;
    char *new_heap_end  = heap_end + incr;

    // スタックと衝突防止（安全チェック）
    if (new_heap_end >= &__HeapLimit) {
        errno = ENOMEM;
        return (caddr_t)-1;
    }

    heap_end = new_heap_end;
    return (caddr_t)prev_heap_end;
}

```
