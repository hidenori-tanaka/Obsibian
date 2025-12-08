参考
- [WindowsでGCCを使おう：MSYS2のインストールと設定方法](https://qiita.com/koyukitukimino/items/df76f7fdb8512920d75b)

以下から Intallation にある 1. Download the installerから msys2-x86_64-YYYYMMDD.exeをダウンロード&インストール
- [MSYS2](https://www.msys2.org)

普段使うシェルは MSYS2 UCRT64 を選択, コマンドプロンプトからコマンドが使えるように PATH に
`C:\msys64\usr\bin` 
`C:\msys64\ucrt64\bin` 
を追加

UCRT64環境向けに gcc のインストール

- MSYS2 UCRT64 シェルを起動
- パッケージデータベースを更新
	pacman -Syu
- gccをインストール
	pacman -S --needed base-devel mingw-w64-ucrt-x86_64-toolchain

`

'


