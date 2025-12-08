## clang-format インストール

エディタとしてVSCodeを使う場合に`C/C++ Extension`を導入するとclang-formatが含まれるのでインストール作業は不要

参考
- [【完全ガイド】clang-formatでコードスタイルを完璧に統一する7つの実践テクニック]([【完全ガイド】clang-formatでコードスタイルを完璧に統一する7つの実践テクニック | Dexall公式テックブログ](https://dexall.co.jp/articles/?p=1706))
## Windowsへの導入方法

### 方法は2つ

1. Windows用LLVMをインストールする方法
	パッケージにclang-formatが含まれている
	・LLVM公式サイトからLLVMのインストーラをダウンロード
	・インストール時に「Add LLVM to the system PATH」にチェック
	・コマンドプロンプトで動作確認
```
	　>clang-format --version
	　clang-format version 21.1.0
```

2. MSYS2環境でインストールする
	・

VSCodeとの連携

・拡張機能のインストール

　C/C++ Extension をインストール

　extenstionにはclang-formatが含まれている。

1. .setting.json に設定追加　

{

 "C_Cpp.clang_format_path": "clang-format実行ファイルへのパス",

 "C_Cpp.clang_format_style": "file",

 "editor.defaultFormatter": "ms-vscode.cpptools",

 "editor.formatOnSave": true,

 "[cpp]": {

 "editor.defaultFormatter": "ms-vscode.cpptools"

 }

}

CIパイプラインへの組み込み

GitLab CIでの例

clang-format:

 stage: lint

 image: ubuntu:latest

 before_script:

 - apt-get update && apt-get install -y clang-format git

 script:

 - find . -regex '.*\.\(cpp\|hpp\|cc\|cxx\)' -exec clang-format -style=file -i {} \;

 - git diff --exit-code

git コミット前にフォーマットチェックを行う

#!/bin/bash

# format-check.sh

# プロジェクト内の全C++ファイルをフォーマットチェック

# エラー時に中断

set -e

# 変更のあるファイルをフォーマットチェック

git diff --name-only HEAD | grep -E '\.(cpp|hpp|cc|h)$' | while read -r file; do

 if [ -f "$file" ]; then

 clang-format -style=file -i "$file"

 fi

done

# 変更があればエラー

git diff --exit-code || {

 echo "フォーマットエラーが見つかりました。"

 echo "clang-formatを実行して修正してください。"

 exit 1

}

git pre-commitにフックとして設定

設定ファイル

.clang-formaファイルをプロジェクトのルートディレクトリに配置

ベーススタイルの設定をすべて設定ファイルに出す方法

clang-format -style=llvm -dump-config

→VSCode の Unity Test