### 例として KCITS の CortexM4 に対して ビルド環境として　ARMCLANG を使った場合の手順を示す

参考ページ
# [Running the RTOS on a ARM Cortex-M Core](https://www.freertos.org/Documentation/02-Kernel/03-Supported-devices/04-Demos/ARM-Cortex/RTOS-Cortex-M3-M4)

- 割り込み優先度に関して
	CortexM4 の割り込み優先度は 3bit で定義されているので 0~7の優先度を持つ。

		上記ページ和訳 [[Running the RTOS on a ARM Cortex-M Core]]
	[[Running the RTOS on a ARM Cortex-M Core]]
	[Running the RTOS on a ARM Cortex-M Core]
- ディレクトリ構成
```
ROOT 
|- SONY
|   |- CXD2888
|       |- sample
|           |- FreeRTOS_CM4 <- 新規に記述したファイルを入れる
|           |- FreeRTOS_CM33 <- 新規に記述したファイルを入れる
|- FreeRTOS <- FreeROTS GitHub をサブモジュールとして入れる、このフォルダは書き換えない
```
1.  FreeRTOSのソースコードを取り込む
		FreeRTOSのGitHub をサブフォルダとして Root に配置
2.  FreeRTOSConfig.h を　FreeROTS-Kerne/template_configuration.hからコピー
	- KCITS CortexM4向けに修正
		1. configCPU_CLOCK_HZ を 120MHz に設定
		2. configTICK_RATE_HZ　は 1000Hz （Tick 割り込み間隔 1ms に設定）
		3. configUSE_NEWLIB_REENTRANT　とりあえず 0 で運用
		4. configMAX_SYSCALL_INTERRUPT_PRIORITY は 1 とする（0は禁止のため）
		5. configMAX_API_CALL_INTERRUPT_PRIORITYはconfigMAX_SYSCALL_INTERRUPT_PRIORITYと同じ値にする
		6. configTOTAL_MPU_REGIONS　は MPU を実装していないので  0とする
			1.  port.c 内で protMPU_ENABLE = 0 に定義すれば良さそう
		7. secureconfigMAX_SECURE_CONTEXTSは０に定義
		8. configENABLE_TRUSTZONEは 0
		9. configENABLE_MPU は０に設定
		10. configENABLE_FPUは０に設定
		11. configENABLE_MVEは０に設定
3.  ソースファイルには FreeRTIOS.h をインクルードする
4.  FreeRTOS-Kernel 以下の taskc.s/list.c/queue.c をソースに加える。残りは　Optional なので必要に従って追加
5. 自分用の protable フォルダを作成して、これに FreeRTOS/portable/ARM_CM4Fの内容をコピーしてソースに追加
6. FreeRTOS/protable/MemMang から使いたい heepX.c をソースに加える。heap4.c を使ってみる
7. ここでコンパイル





https://www.freertos.org/Documentation/02-Kernel/03-Supported-devices/04-Demos/ARM-Cortex/RTOS-Cortex-M3-M4