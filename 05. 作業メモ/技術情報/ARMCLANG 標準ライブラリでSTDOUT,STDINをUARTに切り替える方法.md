
1.  fputc(). fgetc() をリターゲットする場合
	下記文章を参考に、 fputc().fgetc()を実装する。
	この場合、標準ライブラリの初期化ルーチン内のsemihosting用のブレークが有効になり、何回かブレークがかかる。
	これを避けるために、　`__asm(".global __use_no_semihosting");` を定義する必要があるが、この場合低レベルIOをリターゲットする必要がある。

<font color="#ff0000">実装例</font>

- includeファイル
```
/*############################################################################
 * Copyright 2025 Sony Semiconductor Solutions Corporation.
 */
/**
 * @file        retarget.h
 *
 * @brief       retarget sdtin/stdout to UART for ARMCLANG header file.
 *
 * @date        2025/12/04
 */
/*##########################################################################*/

#ifndef RETARGET_H_
#define RETARGET_H_

#include <stdio.h>

#if !defined(DBG_UART)
#define DBG_UART    0
#endif

#if DBG_UART == 0
#define MY_UART CXD2888_UART0
#elif DBG_UART == 1
#define MY_UART CXD2888_UART1
#else
#error unknown uart number.
#endif

void uart_init(void);
void _stop(void);

int fputc(int c, FILE *f);
int fgetc(FILE *f);

#endif /* RETARGET_H_ */

```

- srcファイル
```
/*############################################################################
 * Copyright 2024 Sony Semiconductor Solutions Corporation.
 */
/**
 * @file        retarget.c
 *
 * @brief       retarget sdtin/stdout to UART for ARMCLANG
 *
 * @date        2025/12/04
 */
/*##########################################################################*/

#include <stdio.h>
#include <string.h>
#include <stdarg.h>
#include "CXD2888.h"
#include "retarget.h"

void uart_init(void)
{
#if DBG_UART == 0
    /*
     * Set IOMUX and IOFONFIG, Reset, Supply clock UART0
     */
    CXD2888_IOCFG->IOCFG_UARTRXD0
        &= ~(IOCFG_IE_MSK | IOCFG_OE_MSK | IOCFG_PULL_MSK);
    CXD2888_IOCFG->IOCFG_UARTRXD0
        |= (IOCFG_IE_EN | IOCFG_OE_EN | IOCFG_PULL_UP);

    CXD2888_IOCFG->IOCFG_UARTTXD0
        &= ~(IOCFG_IE_MSK | IOCFG_OE_MSK | IOCFG_PULL_MSK);
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

void _stop(void)
{
    UART_Stop(MY_UART);
}

/* ------------------------------------------------------------ */
/* 標準出力リターゲット：fputc                                 */
/* printf / puts / putchar などは最終的にここを呼ぶ            */
/* ------------------------------------------------------------ */
int fputc(int c, FILE *f)
{
    (void)f; /* 未使用 */

    int32_t ret;

    const uint32_t size = 1;
    uint32_t rest;

    /* FIFO 上書き対策　*/
    while (!(MY_UART->UARTFR & UARTFR_TXFE_MSK)) {
    }

    ret = UART_WriteFifo(MY_UART, size, (uint8_t *)&c, &rest);
    if (ret != 0) {
        return EOF;
    }

    /* モデルによっては \n のとき \r を追加した方が見やすい */
    if ((char)c == '\n') {
        const uint8_t r = '\r';
        ret             = UART_WriteFifo(MY_UART, size, (uint8_t *)&r, &rest);
        if (ret != 0) {
            return EOF;
        }
    }

    return (unsigned char)c;
}

/* ------------------------------------------------------------ */
/* 標準入力リターゲット：fgetc                                 */
/* getchar / scanf / fgets(stdin) などが最終的にここを呼ぶ      */
/* ------------------------------------------------------------ */
int fgetc(FILE *f)
{
    (void)f; /* 未使用 */

    int32_t ret;
    uint8_t c = '\0';

    uint32_t read = 0;

    while (read == 0) {
        ret = UART_ReadFifo(MY_UART, 1, &c, &read);
        if (ret != 0) {
            return EOF;
        }
    }

    /* CR を LF に変換（好みに応じて） */
    if (c == '\r') {
        c = '\n';
    }
    return (int)c;
}

```
参考
Retargeting output to UART
![[作業メモ/retargeting_output_to_uart_102440_0100_02_en.pdf]]

2. 低レベルI/Oをリターゲットする場合
	実装例
	注意する点
		以下のように標準入出力名を定義すること。これを行わないと３つとも `:tt` という名前で '__sys_open ' 呼び出しされてしまう 'sys_read(), sys_write()' のfdが  __sys_open() で :tt に返却した fd で呼び出されるため、方向が区別できない。
		__sys_read()は複数バイト指定でリードかかるが1byte読む毎に返らないと、getc() 系の1バイト読み出しが指定バイト完了するまで終わらなくなる。
	`const char __stdin_name[]  = "STDIN";'
	`const char __stdout_name[] = "STDOUT";'
	`const char __stderr_name[] = "STDERR";`


- include(変更無し)
- srcファイル
```
/*############################################################################
 * Copyright 2024 Sony Semiconductor Solutions Corporation.
 */
/**
 * @file        retarget.c
 *
 * @brief       retarget sdtin/stdout to UART for ARMCLANG
 *
 * @date        2025/12/04
 */
/*##########################################################################*/

#include <stdio.h>
#include <string.h>
#include <rt_misc.h>
#include <rt_sys.h>
#include "CXD2888.h"
#include "retarget.h"

__asm(".global __use_no_semihosting");

const char __stdin_name[]  = "STDIN";
const char __stdout_name[] = "STDOUT";
const char __stderr_name[] = "STDERR";

/* UART 初期化が完了したかどうかのフラグ */
static int uart_ready = 0;

static const char CR = '\r';
static const char LF = '\n';

void uart_init(void)
{
#if DBG_UART == 0
    /*
     * Set IOMUX and IOFONFIG, Reset, Supply clock UART0
     */
    CXD2888_IOCFG->IOCFG_UARTRXD0
        &= ~(IOCFG_IE_MSK | IOCFG_OE_MSK | IOCFG_PULL_MSK);
    CXD2888_IOCFG->IOCFG_UARTRXD0
        |= (IOCFG_IE_EN | IOCFG_OE_EN | IOCFG_PULL_UP);

    CXD2888_IOCFG->IOCFG_UARTTXD0
        &= ~(IOCFG_IE_MSK | IOCFG_OE_MSK | IOCFG_PULL_MSK);
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

    uart_ready = 1;
}

/*---------------------------------------------------------------------------*/
/* 低レベル I/O：ファイルオープン */
/*---------------------------------------------------------------------------*/

FILEHANDLE _sys_open(const char *name, int openmode)
{
    (void)openmode;

    /* "STDIN", "STDOUT", "STDERR" しか扱わない簡易版 */
    if (name == NULL) {
        return -1;
    }

    /* 端末デバイス (console) を表す ":tt" の扱い */
    if (strcmp(name, ":tt") == 0 || strcmp(name, ":TT") == 0) {
        /* 適当なハンドル番号を返す。ここでは stdout と同じ扱いにする */
        return 1; /* STDOUT 相当 */
    }

    if (strcmp(name, "STDIN") == 0) return 0;  /* stdin  */
    if (strcmp(name, "STDOUT") == 0) return 1; /* stdout */
    if (strcmp(name, "STDERR") == 0) return 2; /* stderr */

    /* 普通のファイルは未サポート */
    return -1;
}

/*---------------------------------------------------------------------------*/
/* 低レベル I/O：クローズ（ここでは何もしない） */
/*---------------------------------------------------------------------------*/
int _sys_close(FILEHANDLE fh)
{
    (void)fh;
    return 0;
}

/*---------------------------------------------------------------------------*/
/* 低レベル I/O：書き込み (printf などが最終的にここに来る) */
/*---------------------------------------------------------------------------*/
int _sys_write(FILEHANDLE fh, const unsigned char *buf, unsigned len, int mode)
{
    (void)mode;

    int32_t ret;
    uint32_t rest;

    /* stdout / stderr だけ UART に流す */
    if (fh == 1 || fh == 2) {
        if (!uart_ready) {
            /* まだ UART が使えないタイミング (__rt_lib_init 中など)
             * では何もせず成功扱い */
            return 0;
        }

        for (unsigned i = 0; i < len; i++) {
            if (buf[i] == LF) {
                do {
                    ret = UART_WriteFifo(MY_UART, 1, &CR, &rest);
                    if (ret != 0) {
                        return -1;
                    }
                } while (rest != 0);
            }
            do {
                ret = UART_WriteFifo(MY_UART, 1, (const char *)&buf[i], &rest);
                if (ret != 0) {
                    return -1;
                }
            } while (rest != 0);
        }
        return 0; /* 0 = 成功（Arm C
                     ライブラリ仕様）:contentReference[oaicite:2]{index=2}
                   */
    }

    /* 未サポートのファイルハンドル */
    return -1;
}

/*---------------------------------------------------------------------------*/
/* 低レベル I/O：読み込み (scanf, getchar 等) */
/*---------------------------------------------------------------------------*/
int _sys_read(FILEHANDLE fh, unsigned char *buf, unsigned len, int mode)
{
    (void)mode;

    if (fh == 0) { /* stdin */
        if (!uart_ready) {
            return (int)len;
        }

        if (len == 0) {
            return (int)len;
        }

        int32_t ret;
        int32_t read;
        char c = -1;

        /* ブロッキング版 */
        do {
            ret = UART_ReadFifo(MY_UART, 1, &c, &read);
            if (ret != 0) {
                return (int)(len);
            }
        } while (read != 1);

        buf[0] = c;

        /* Arm C ライブラリでは「読めなかったバイト数」を返す仕様 */
        return (int)(len - 1);
    }

    /* stdin 以外は未サポート */
    return (int)len;
}

/*---------------------------------------------------------------------------*/
/* その他のサポート関数 */
/*---------------------------------------------------------------------------*/

int _sys_istty(FILEHANDLE fh)
{
    /* stdin/stdout/stderr は端末扱い */
    if (fh == 0 || fh == 1 || fh == 2 || fh == 3) {
        return 1;
    }
    return 0;
}

int _sys_seek(FILEHANDLE fh, long pos)
{
    (void)fh;
    (void)pos;
    return -1; /* シーク未サポート */
}

long _sys_flen(FILEHANDLE fh)
{
    (void)fh;
    return 0;
}

/* putchar の最終出口としても呼ばれることがある */
void _ttywrch(int ch)
{
    uint32_t rest;
    if (!uart_ready) return;
    if (ch == LF) {
        do {
            UART_WriteFifo(MY_UART, 1, &CR, &rest);
        } while (rest != 0);
    }
    do {
        UART_WriteFifo(MY_UART, 1, (const char *)&ch, &rest);
    } while (rest != 0);
}

/* abort() や exit() 経由で呼ばれる */
void _sys_exit(int return_code)
{
    (void)return_code;
    while (1) {
        __NOP();
        /* ここで WFI してもよい */
    }
}
```
```

```

