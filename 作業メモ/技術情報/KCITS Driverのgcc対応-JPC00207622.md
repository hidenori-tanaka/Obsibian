### 1. startup_XXXX.cをgcc向けに変更
- ベクタテーブルの修正
	cmsis_gcc.h で __VECTOR_TABLE_ARRRIBUTEで使っているベクタテーブルのセクション名が ".vectors" になっているのでリンカでも　".vectors" が.textの先頭に来るように変更　
	__INITIAL_SP は __StackTop となっているので、リンカで ".stack" セクションを作成したら。その最終アドレスを __StackTop となるように修正
	
- 割り込みハンドラの修正
	”.after_vectors" セクションにして、リンカで ".vectors" セクションのあとに配置されるように修正。Reset_handlerは先頭に配置しておきたいので独自セクション .entry を定義　　
	
- リセットハンドラの修正
	armccでは標準ライブラリの初期化ルーチン `__main` でデータ、BSSの初期化を実行してたが、newlib では実行してくれないのでリセットハンドラで実行する必要がある。
	newlibを使う場合にC++を使わなければ初期化で呼び出す処理はなし。
		CMSISのdata/bss初期化ルーチンをもってきたら、転送テーブルの構造体のサイズがバイト数となっていた、転送はワードで行われるのでサイズをバイトに修正

## 標準ライブラリのprintfをUART出力させる

1. ARMCLANGの場合
	Retargeting output to UARTを参考にする。
	1.  UARTに対して入出力をおこなう fgetc/fputc を作成。retarget.c に記述。
	2. UARTの初期化関数もretarget.cに追加。
	これだけで、標準出力、標準入力がUARTに変更できる。
	
参考
- [[作業メモ/retargeting_output_to_uart_102440_0100_02_en.pdf]]

2.  GCCの場合
	