### 例として KCITS の CortexM4 に対して ビルド環境として　ARMCLANG を使った場合の手順を示す

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
2.  