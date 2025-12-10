
下記文章を参考に、 fputc().fgetc()を実装する。
実装例

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
![[retargeting_output_to_uart_102440_0100_02_en.pdf]]