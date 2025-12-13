### 例として KCITS の CortexM4 に対して ビルド環境として　ARMCLANG を使った場合の手順を示す

参考ページ
# [Running the RTOS on a ARM Cortex-M Core](https://www.freertos.org/Documentation/02-Kernel/03-Supported-devices/04-Demos/ARM-Cortex/RTOS-Cortex-M3-M4)

- 割り込み優先度に関して
	CortexM4 の割り込み優先度は 3bit で定義されているので 0~7の優先度を持つ。

0xE00ED0C AIRCR レジスタの値  0xFA05_0000

![[Pasted image 20251213145418.png|700]]

ENDIANNESS : 0 - Little
PRIGROUP     :  0
	優先度の上位2ビットが優先度グループ、下位1bitが順番として用いられる

FreeRTOSでは優先度ビットをすべてプリエンプト優先度ビットに割当、サブ優先度ビットを0に割り当てることが推奨される。
→PRIGRPUTはこのまま変更しなくてよい。

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
		3. configUSE_PORT_OPTIMISED_TASK_SELECTION　は 1に設定、CortexM4にはCLZ命令があるため
		4. configUSE_NEWLIB_REENTRANT　とりあえず 0 で運用		5. 
		5. configMAX_SYSCALL_INTERRUPT_PRIORITY は 1 とする（0は禁止のため）
		6. configMAX_API_CALL_INTERRUPT_PRIORITYはconfigMAX_SYSCALL_INTERRUPT_PRIORITYと同じ値にする
		7. configTOTAL_MPU_REGIONS　は MPU を実装していないので  0とする
			1.  port.c 内で protMPU_ENABLE = 0 に定義すれば良さそう
		8. secureconfigMAX_SECURE_CONTEXTSは０に定義
		9. configENABLE_TRUSTZONEは 0
		10. configENABLE_MPU は０に設定
		11. configENABLE_FPUは０に設定
		12. configENABLE_MVEは０に設定
3.  ソースファイルには FreeRTIOS.h をインクルードする
4.  FreeRTOS-Kernel 以下の taskc.s/list.c/queue.c をソースに加える。残りは　Optional なので必要に従って追加
5. 自分用の protable フォルダを作成して、これに FreeRTOS/portable/ARM_CM4Fの内容をコピーしてソースに追加
6. FreeRTOS/protable/MemMang から使いたい heepX.c をソースに加える。heap4.c を使ってみる
7. ここでコンパイル





https://www.freertos.org/Documentation/02-Kernel/03-Supported-devices/04-Demos/ARM-Cortex/RTOS-Cortex-M3-M4