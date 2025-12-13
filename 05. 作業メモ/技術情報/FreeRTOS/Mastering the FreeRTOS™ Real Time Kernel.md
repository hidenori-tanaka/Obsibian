# 2 The FreeRTOS Kernel Distribution
## 2.1 Introduction

FreeRTOS のカーネルファイルやディレクトリ構成を理解しやすくするために、この章では次の内容を扱います：

- **FreeRTOS のディレクトリ構造のトップレベル概要を示す**
- **特定の FreeRTOS プロジェクトに必要なソースファイルを説明する**
- **デモアプリケーションを紹介する**
- **新しい FreeRTOS プロジェクトを作成する方法について説明する**

ここでの説明は **公式の FreeRTOS 配布物（official distribution）** の構成に基づいています。  
本書に付属しているサンプルは、  
公式配布物とは **やや異なる構成** を使用しています。
## 2.2 Understanding the FreeRTOS Distribution
### 2.2.1 Definition: FreeRTOS Port

FreeRTOS は **約 20 種類のコンパイラ** でビルドすることができ、  
**40 種類以上のプロセッサアーキテクチャ** 上で動作します。

これらの **コンパイラとプロセッサアーキテクチャの組み合わせごと** に  
FreeRTOS のポート（移植版）が存在し、  
それぞれの組み合わせを **FreeRTOS ポート（FreeRTOS port）** と呼びます。
### 2.2.2 Building FreeRTOS

FreeRTOS は、  
もともと **シングルスレッドのベアメタルアプリケーション**であるものに、  
**マルチタスク機能を提供するライブラリ**です。

FreeRTOS は **C 言語のソースファイル一式**として提供されています。

- 一部のソースファイルは **すべてのポートで共通**  
- 一部のソースファイルは **特定のポート（コンパイラ＋CPU アーキテクチャ）専用**

これらのソースファイルをプロジェクトの一部としてビルドすることで、  
アプリケーションから FreeRTOS API を利用できるようになります。
### 📦 公式ポートごとにデモアプリケーションが付属

各公式 FreeRTOS ポートには、  
リファレンスとして利用できる **デモアプリケーション** が付属しています。

デモアプリケーションは次のように設定済みです：

- 正しい FreeRTOS ソースファイルをビルドするよう構成済み
- 必要なヘッダファイルを含むように設定済み

---

### 🧪 デモは作成時点では「そのままビルド成功」するようになっている

各デモアプリケーションは、  
作成時点では **コンパイルエラーや警告なしでビルドできる状態（out of the box）** にされています。

もしビルドツールの更新などにより  
この状態でビルドできなくなっている場合は、  
FreeRTOS サポートフォーラムで報告してください：

[https://forums.FreeRTOS.org](https://forums.FreeRTOS.org)

---

デモアプリケーションについての詳細は **セクション 2.3** で説明されています。
### 2.2.3 FreeRTOSConfig.h

FreeRTOS のカーネルは、  **FreeRTOSConfig.h というヘッダファイルに定義された定数**によって設定されます。

⚠ **注意：FreeRTOSConfig.h をソースファイルから直接 include してはいけません！**

代わりに **FreeRTOS.h** を include してください。  
FreeRTOS.h が適切なタイミングで FreeRTOSConfig.h を取り込みます。

FreeRTOSConfig.h は、  **特定のアプリケーション用に FreeRTOS カーネルをカスタマイズするための設定ファイル**です。
例として、
- `configUSE_PREEMPTION`  
    → FreeRTOS が **協調型（co-operative）** と **プリエンプティブ（pre-emptive）** のどちらのスケジューリング方式を使うかを定義します。  
    ※ スケジューリングアルゴリズムは **4.13節** で詳説

FreeRTOSConfig.h は **特定アプリケーションに依存する設定**を行うため、  
**アプリケーション側のディレクトリに配置すべき**であり、  
FreeRTOS のソースコードが入ったディレクトリに置くべきではありません。

公式 FreeRTOS 配布物には、  **各ポート（CPU + コンパイラ）向けのデモアプリケーション**が含まれており、  
それぞれが独自の **FreeRTOSConfig.h** を持っています。

推奨される方法は、使用している FreeRTOS ポートのデモに付属する FreeRTOSConfig.h をベースとして 、必要な部分を自分のアプリケーション用に調整すること。
１から作成するより遥かに安全で確実です。

FreeRTOS のリファレンスマニュアルや以下のリンクに  
FreeRTOSConfig.h に含まれる定数の説明があります：
[https://www.freertos.org/a00110.html](https://www.freertos.org/a00110.html)
なお、FreeRTOSConfig.h には **すべての定数を書き込む必要はありません**。  
多くの定数は、ファイルに記述しなければ **デフォルト値が自動的に使用**されます。
### 2.2.4 Official Distributions

FreeRTOS の各ライブラリ（カーネルを含む）は、  **それぞれ専用の GitHub リポジトリ**から入手でき、  
**ZIP アーカイブ**としても提供されています。

個別のライブラリを入手できるのは、  FreeRTOS を本番コード（production code）で利用する際には便利です。  
しかし、**学習や開発を開始する段階では、メインの FreeRTOS ディストリビューションをダウンロードする方が適しています。**  
なぜなら、それには **ライブラリだけでなくサンプルプロジェクトも含まれている**からです。

メインディストリビューションには以下がすべて含まれています：
- FreeRTOS の **すべてのライブラリのソースコード**
- FreeRTOS の **すべてのカーネルポート（移植版）**
- FreeRTOS の **すべてのデモアプリケーションのプロジェクトファイル**
   
ファイル数の多さに圧倒されないでください！  
実際のアプリケーションに必要なのは **そのごく一部** だけです。

最新版の FreeRTOS ディストリビューション ZIP は以下から入手できます：
[https://github.com/FreeRTOS/FreeRTOS/releases/latest](https://github.com/FreeRTOS/FreeRTOS/releases/latest)

以下の Git コマンドを使用すると、  メインの FreeRTOS ディストリビューションをクローンし、  さらに個別ライブラリを各リポジトリから **サブモジュールとして取得**できます：
（※原文ではこのあと具体的な git clone コマンドが続きます）

```
git clone https://github.com/FreeRTOS/FreeRTOS.git --recurse-submodules
git clone git@github.com:FreeRTOS/FreeRTOS.git --recurse-submodules
```
Figure 2.1 にFreeRTOSディストリビューションの第1,2階層のディレクトリを示します。

![[Pasted image 20251212155425.png]]
Figure 2.1 Top level directories within the FreeRTOS distribution

このディストリビューションには、**FreeRTOS カーネルのソースファイルは 1 つのコピーしか含まれていません**。

すべてのデモプロジェクトは、  
**カーネルのソースファイルが FreeRTOS/Source ディレクトリに存在することを前提として**います。

そのため、このディレクトリ構造を変更すると、  
**デモプロジェクトがビルドできなくなる可能性があります。**

### FreeRTOS Source Files Common to All Ports

`tasks.c` と `list.c` は **FreeRTOS カーネルのコア機能を実装**しており、  
**必ず必要となるファイル**です。  
これらは **FreeRTOS/Source** ディレクトリに配置されています（図 2.2 を参照）。

同じディレクトリには、以下の **オプションのソースファイル** も含まれています。
- queue.c
	`queue.c` は、**キューとセマフォ** の機能を提供します（本書の後半で解説）。  
	ほぼすべての FreeRTOS アプリケーションで必要となるため、  **事実上ほぼ常にビルドされます。**

- timers.c
	`timers.c` は **ソフトウェアタイマー** の機能を提供します（後述）。  
	アプリケーションがソフトウェアタイマーを使用する場合にのみビルドします。

- event_groups.c
	`event_groups.c` は **イベントグループ** の機能を提供します（後述）。  
	アプリケーションがイベントグループを使用する場合にのみビルドします。

- stream_buffer.c
	`stream_buffer.c` は **ストリームバッファ** と **メッセージバッファ** の機能を提供します（後述）。  
	これらの機能を使用する場合にのみビルドします。

- croutine.c
	`croutine.c` は **FreeRTOS のコルーチン機能**を実装します。
	ただし：
	- 非常に小型のマイクロコントローラ向けの機能である
	- 現在ではほとんど使用されない
    - **今後の設計での使用は推奨されない**
    - メンテナンスもすでに行われていない
    - 本書でも説明対象外
    そのため、アプリケーションがコルーチンを使用しない限り、ビルドする必要はありません。
    ![[Pasted image 20251212155854.png]]
Figure 2.2 Core FreeRTOS source files within the FreeRTOS directory tree

ZIP 形式で配布されている FreeRTOS のファイル名は、  
**すでに同名のファイルを使っているプロジェクトと名前衝突（namespace clash）を起こす可能性がある**  
ことは認識されています。

ユーザー側で必要に応じて **ファイル名を変更することは可能** です。

しかし、**FreeRTOS の配布物に含まれるファイル名そのものを変更することはできません。**  
理由は次の通りです：
- ファイル名を変更すると、既存のユーザーのプロジェクトとの互換性が失われる
- また、FreeRTOS 対応の開発ツールとの互換性も壊れてしまう

### 2.2.7 FreeRTOS Source Files Specific to a Port

FreeRTOS/Source/portable ディレクトリには、  **各 FreeRTOS ポート（＝特定のコンパイラ＋アーキテクチャの組み合わせ）に依存するソースファイル**が含まれています。
portable ディレクトリは階層構造になっており、
1. **コンパイラ別**
2. **プロセッサアーキテクチャ別**

という順序で分類されています。  
図 2.3 にその階層が示されています。

CPU アーキテクチャ `'architecture'` に対して コンパイラ `'compiler'` を使用して FreeRTOS を実行するには、
**FreeRTOS のコアソースファイルに加えて、  以下のディレクトリ内のファイルもビルドする必要があります：**
`FreeRTOS/Source/portable/[compiler]/[architecture]`

第3章「ヒープメモリ管理」で説明されているように、  FreeRTOS は **ヒープメモリの割り当てもポータブル層の一部** とみなします。
- `configSUPPORT_DYNAMIC_ALLOCATION = 0` の場合  
    → プロジェクトにヒープメモリ割り当て方式を **含めてはいけません**
   
FreeRTOS はヒープ実装の例を以下のディレクトリに提供しています：
`FreeRTOS/Source/portable/MemMang`
FreeRTOS が動的メモリ割り当てを使用する設定になっている場合は、
このディレクトリ内のいずれかのヒープ実装ファイルをプロジェクトに追加 、**または**、独自のヒープ実装を用意する必要があります。

⚠ **注意：複数のヒープ実装（heap_1.c, heap_2.c など）を同時に含めてはなりません！**

```
FreeRTOS
└─Source
     └─portable          ← すべてのポート固有ソースファイルが入る        
         ├─MemMang       ← 代替ヒープ実装ファイル         
         ├─[compiler 1]  ← コンパイラ1向けポートファイル         
         │   ├─[architecture 1]  ← コンパイラ1 × アーキテクチャ1用ファイル
         │   ├─[architecture 2]  ← コンパイラ1 × アーキテクチャ2用ファイル
         │   └─[architecture 3]  ← コンパイラ1 × アーキテクチャ3用ファイル
         └─[compiler 2]  ← コンパイラ2向けポートファイル
            ├─[architecture 1]  ← コンパイラ2 × アーキテクチャ1用ファイル
            ├─[architecture 2]  ← コンパイラ2 × アーキテクチャ2用ファイル
            └─ …`
```
Figure 2.3 Port specific source files within the FreeRTOS directory tree

### 2.2.8 Include Paths

FreeRTOS を使用するには、  コンパイラの **インクルードパス（include path）** に 次の 3 つのディレクトリを追加する必要があります。
	**1. FreeRTOS カーネルの基本ヘッダファイルへのパス**
	`FreeRTOS/Source/include`
	   これは FreeRTOS のコア API が定義されている重要なヘッダ群です。
	**2. 使用している FreeRTOS ポート（コンパイラ＋アーキテクチャ）固有のヘッダへのパス**
	`FreeRTOS/Source/portable/[compiler]/[architecture]`
	 このディレクトリには、対象プラットフォーム向けに最適化された  
	FreeRTOS ポート層のファイルが含まれています。
	**3. FreeRTOSConfig.h の正しい場所へのパス**
	FreeRTOSConfig.h はアプリケーション固有の設定ファイルであり、  プロジェクト側に配置する必要があります。
	コンパイラはこのファイルを最終的にインクルードできる必要があるため、  	**FreeRTOSConfig.h を置いたディレクトリをインクルードパスに追加する必要があります。**

### 2.2.9 Header Files

FreeRTOS API を使用するソースファイルでは、まず **FreeRTOS.h** をインクルードする必要があります。
その上で、使用する API 関数のプロトタイプが含まれているヘッダファイル 
（以下のいずれか）をインクルードします：

- `task.h`
- `queue.h`
- `semphr.h`
- `timers.h`
- `event_groups.h`
- `stream_buffer.h`
- `message_buffer.h`
- `croutine.h`
 
	⚠ **注意**
	これら以外の FreeRTOS 関連ヘッダを **明示的に include してはいけません**。
	`FreeRTOS.h` を include することで、  **FreeRTOSConfig.h を含む必要な内部ヘッダは自動的にインクルードされます。**

## 2.3 Demo Applications

各 FreeRTOS ポートには **少なくとも 1 つのデモアプリケーション**が付属しています。  
これらは作成時点では **そのままビルドできる状態（out-of-the-box）** であり、  コンパイルエラーや警告は発生しません。

もしツールチェーンの更新などによりビルドできなくなっている場合は、  
以下のサポートフォーラムに報告してください：

[https://forums.FreeRTOS.org](https://forums.FreeRTOS.org)

FreeRTOS は **Windows / Linux / macOS** を使用して開発・テストされており、  様々な組み込み用・一般向けツールチェーンに対応しています。

しかし時折、ツールやバージョン違いによりビルドエラーが発生する場合があります。  そのような場合も、サポートフォーラムへ報告してください。

デモアプリケーションには次の役割があります：
- **正しいファイル構成とコンパイラ設定がされた動作例を提供する**
- **ほぼ何も準備せずに実験を開始できる環境を提供する**
- **FreeRTOS API の使用例を示す**
- **実アプリケーションを作る際のベースとなるプロジェクトを提供する**
- **FreeRTOS カーネル実装のストレステストを行う**

各デモプロジェクトは、  `FreeRTOS/Demo` ディレクトリの下にある **固有のサブディレクトリ**に配置されています。

サブディレクトリの名前は、  
**そのデモが対応する FreeRTOS ポート（CPU + コンパイラ）** を表しています。

FreeRTOS.org の各デモアプリケーションのページには以下の情報があります：
- FreeRTOS ディレクトリ構造の中で **どこにプロジェクトファイルがあるか**
- デモが使用する **ハードウェアまたはエミュレータ**
- デモを実行するための **ハードウェア設定方法**
- デモの **ビルド方法**
- デモの **期待される動作**

すべてのデモプロジェクトは、  `FreeRTOS/Demo/Common/Minimal` にある **共通デモタスクの一部** を生成します。

共通デモタスクの目的は：
- FreeRTOS API の使用例を提供する
- FreeRTOS ポート（移植版）をテストする

であり、  **特別な実用機能を実装しているわけではありません。**

多くのデモプロジェクトは、  「**blinky**」スタイルの簡易スタータープロジェクトに切り替えることもできます。
通常は：
- 2つの RTOS タスク
- 1つのキュー
を作成する簡単な構成になります。

すべてのデモプロジェクトには、  `main()` 関数を含む **main.c** ファイルがあります。
main.c の役割：
- デモアプリケーションのタスクを作成する 
- FreeRTOS カーネルを起動する

個々のデモ固有の情報については、  各 main.c 内のコメントを参照してください。

## 2.4 Creating a FreeRTOS Project

## 2.5 Data Types and Coding Style Guide

### 2.5.1 Data Types

FreeRTOS の各ポート（CPU + コンパイラ組み合わせ）には、  そのポート専用の **portmacro.h** ヘッダファイルが存在します。

この portmacro.h には、他にも様々な定義が含まれていますが、  特に重要なのは **2つのポート依存データ型** の定義です：
- **TickType_t**    
- **BaseType_t**    

以下に、それぞれのデータ型で使用される `macro` または `typedef` と、  実際の型（C 言語型）がどのようになっているか説明します。

- **TickType_t**    
	FreeRTOS は **tick 割り込み（ティック割り込み）** と呼ばれる **一定周期の割り込み**を生成するように設定されています。
	FreeRTOS アプリケーション開始後に   **何回 tick 割り込みが発生したか**を示す値を **tick カウント（tick count）** と呼びます。
    この tick カウントを FreeRTOS 内部の **時間の基準**として使用します。
	
	2 回の tick 割り込みの間隔を **tick period（ティック周期）** と呼び、  FreeRTOS の API では **時間指定を tick の倍数で表現します**。
	
	`TickType_t` は次の用途で使用されるデータ型です：
	- **tick カウントの値を保持する**
	- **API で時間を指定する**
    
	TickType_t は、次のいずれかの **符号なし整数型** になります：
	- `uint16_t`（16ビット）  
	- `uint32_t`（32ビット）
	- `uint64_t`（64ビット）
	どの型が選ばれるかは、  FreeRTOSConfig.h の **configTICK_TYPE_WIDTH_IN_BITS** によって決まります。
	- この設定は **CPU アーキテクチャに依存**します。
    - FreeRTOS ポートは、この設定が有効かどうかをチェックします。  
	
	16ビットの TickType_t を使うと：
	- 8ビット・16ビットアーキテクチャでは **効率が大幅に向上**する  
    （CPU が小さいほど、16ビット処理が軽い）
	しかしその一方で：
	- **最大ブロック時間（vTaskDelay などで指定できる最大時間）が非常に小さくなる**
    
	32ビットまたは64ビット CPU では、  **16ビット TickType_t を使う理由は全くありません。**
	
	過去の FreeRTOS では`configUSE_16_BIT_TICKS`という設定を使っていましたが、  
	現在は **より広い幅（32～64ビット）に対応するため、以下の設定に置き換えられました：**
	`configTICK_TYPE_WIDTH_IN_BITS`
	新しい設計では **必ず configTICK_TYPE_WIDTH_IN_BITS を使用**してください。
	
| configTICK_TYPE_WIDTH_IN_BITS | 8ビット CPU | 16ビット CPU | 32ビット CPU | 64ビット CPU |
| ----------------------------- | -------- | --------- | --------- | --------- |
| **TICK_TYPE_WIDTH_16_BITS**   | uint16_t | uint16_t  | uint16_t  | N/A       |
| **TICK_TYPE_WIDTH_32_BITS**   | uint32_t | uint32_t  | uint32_t  | N/A       |
| **TICK_TYPE_WIDTH_64_BITS**   | N/A      | N/A       | uint64_t  | uint64_t  |
Table 2 TickType_t data type and the configTICK_TYPE_WIDTH_IN_BITS configuration

- BaseType_t
	これは、アーキテクチャにとって **最も効率的なデータ型** として常に定義されます。
	一般的には次のようになります：
	- **64ビットアーキテクチャ** → 64ビット型
    - **32ビットアーキテクチャ** → 32ビット型
    - **16ビットアーキテクチャ** → 16ビット型
    - **8ビットアーキテクチャ** → 8ビット型
    
	`BaseType_t` は通常、  **ごく限られた値の範囲しか取らない返り値** や `pdTRUE` / `pdFALSE` といった **ブール値** のために使用されます。

### 2.5.2 Variable Names

変数名には、その型を示す **プレフィックス（接頭辞）** が付けられます：
- `c` : `char` 
- `s` : `int16_t`（short）
- `l` : `int32_t`（long）
- `x` : `BaseType_t` およびその他の非標準型  
    （構造体、タスクハンドル、キューハンドルなど）

変数が **unsigned 型** の場合は、さらに `u` を付けます。  
変数が **ポインタ** の場合は、さらに `p` を付けます。

 例：
- `uint8_t` → **`uc`** がプレフィックス
- `char *`（char 型へのポインタ）→ **`pc`** がプレフィックス

### 2.5.3 Function Names

関数名には、**戻り値の型** と **定義されているファイル名を示すプレフィックス** が付けられます。
例：
- **`vTaskPrioritySet()`**    
    - 戻り値：`void`        
    - 定義ファイル：`tasks.c`        
- **`xQueueReceive()`**    
    - 戻り値：`BaseType_t`        
    - 定義ファイル：`queue.c`        
- **`pvTimerGetTimerID()`**    
    - 戻り値：`void *`（ポインタ to void）        
    - 定義ファイル：`timers.c`        

ファイルスコープ（外部公開しない）関数には  **`prv`** というプレフィックスが付きます。
例：`prvProcessReceivedCommands()`

### 2.5.4 Formatting

いくつかのデモアプリケーションでは、  **タブ文字が使用されており、1タブは常に4スペースに設定**されています。
一方、**FreeRTOS カーネル本体では、現在タブは使用していません。**

### 2.5.5 Macro Names

多くのマクロは **大文字**で記述され、  さらにそのマクロが **どこで定義されているかを示すプレフィックス**（小文字）が付けられます。

以下の表は代表的なプレフィックス一覧です。
## **表 3：マクロのプレフィックス**

| プレフィックス    | マクロ例                   | 定義されている場所                  |
| ---------- | ---------------------- | -------------------------- |
| **port**   | `portMAX_DELAY`        | portable.h または portmacro.h |
| **task**   | `taskENTER_CRITICAL()` | task.h                     |
| **pd**     | `pdTRUE`               | projdefs.h                 |
| **config** | `configUSE_PREEMPTION` | FreeRTOSConfig.h           |
| **err**    | `errQUEUE_FULL`        | projdefs.h                 |
Table 3 Macro prefixes

セマフォ API は、ほとんどが **マクロとして実装**されていますが、  
**マクロの命名規則ではなく、関数の命名規則に従っている** ことに注意してください。

表 4 に示されているマクロは、FreeRTOS のソースコード全体で使用されています。

| マクロ         | 値   |
| ----------- | --- |
| **pdTRUE**  | 1   |
| **pdFALSE** | 0   |
| **pdPASS**  | 1   |
| **pdFAIL**  | 0   |
Table 4 Common macro definitions
### 2.5.6 Rationale for Excessive Type Casting

FreeRTOS のソースコードは多くの異なるコンパイラでコンパイルされます。  
しかし、それらコンパイラは **警告（warning）を生成する条件やタイミングが異なります**。

特に、コンパイラごとに **キャスト（型変換）の要求方法が異なる** ことが問題になります。

そのため、FreeRTOS のソースコードでは、  
通常よりも **多くの型キャストを記述せざるを得ない** 状況になっています。

# 3 Heap Memory Management

## 3.1 Introduction

### 3.1.1 Prerequisites

FreeRTOS を使用するためには、  **十分な C 言語プログラミング能力が前提**となります。

そのため、この章では読者が以下の概念にすでに精通していることを前提としています：
- C プロジェクトをビルドする際の **コンパイルフェーズ** と **リンクフェーズ** の違い    
- **スタック（stack）** と **ヒープ（heap）** の意味    
- 標準 C ライブラリの `malloc()` と `free()` 関数

### 3.1.2 Scope

この章では次の内容を扱います：

- **FreeRTOS が RAM を割り当てるタイミング**    
- **FreeRTOS が提供している 5 つのメモリ割り当て方式（ヒープ管理方式）**    
- **どのメモリ割り当て方式を選択すべきか**

### 3.1.3 Switching Between Static and Dynamic Memory Allocation

次の章では、**タスク・キュー・セマフォ・イベントグループ** などのFreeRTOS のカーネルオブジェクトについて説明されます。

これらのオブジェクトを保持するために必要な RAM は：
- **コンパイル時に静的に割り当てる（Static Allocation）**    
- **実行時に動的に割り当てる（Dynamic Allocation）**    
のどちらかで確保できます。

 動的割り当ての特徴（Dynamic Allocation）
- 設計や計画の手間を軽減する    
- API がシンプルになる    
- 全体の RAM 使用量（フットプリント）を最小化しやすい    

 静的割り当ての特徴（Static Allocation）
- より **決定論的（predictable）** な動作を得られる    
- メモリ割り当て失敗への対処が不要になる    
- **ヒープ断片化（fragmentation）** のリスクを避けられる    
    - 断片化：ヒープ全体としては十分な空きがあるのに、  
        連続した領域がないため割り当てできない状態

静的メモリでカーネルオブジェクトを作成する FreeRTOS API は、
`configSUPPORT_STATIC_ALLOCATION = 1`
のとき **のみ利用可能** です。

動的メモリを使ってカーネルオブジェクトを作成する API は、
`configSUPPORT_DYNAMIC_ALLOCATION = 1`
または
`configSUPPORT_DYNAMIC_ALLOCATION が未定義`
のときに利用可能です。

`configSUPPORT_STATIC_ALLOCATION = 1 configSUPPORT_DYNAMIC_ALLOCATION = 1`
のように **同時に有効** にすることも可能です。

より詳しい情報は、  
**3.4 Static Memory Allocation** セクションで説明されています。
### 3.1.4 Using Dynamic Memory Allocation

動的メモリ割り当ては **C 言語の一般的な概念**であり、 FreeRTOS やマルチタスク固有の概念ではありません。

FreeRTOS と関連する理由は、  **カーネルオブジェクトを動的メモリで作成することが選択可能である**ためです。

しかし、C 標準ライブラリの `malloc()` や `free()` は、 次の理由から **組み込みシステムや FreeRTOS で利用するには適切でない場合があります**：

1. 小規模組み込みシステムでは利用できないことがある
	標準ライブラリが存在しない、または削られているケースが多い。
	
2. 実装サイズが大きく、貴重なコード領域を消費する
	組み込みではコードサイズが非常に重要。
	
3. スレッドセーフでないことが多い
	マルチタスク環境では安全に動作しない可能性がある。
	
 4. 実行時間が決定論的ではない
	呼び出すたびに実行時間が変化し、  リアルタイムシステムでは問題となる。
	
 5. メモリ断片化の問題がある
	ヒープ全体には空きがあっても、  **連続した領域が確保できず割り当てに失敗する**ことがある。
	
6. リンカ設定が複雑になることがある
	ヒープ領域と他の領域の配置調整が必要になる。
	
7. ヒープが他の変数を侵食すると、デバッグが困難なバグの原因になる
	ヒープがスタックやグローバル変数領域と衝突することで  	**原因不明の暴走やクラッシュ**につながりやすい。

FreeRTOS が独自のヒープ管理（heap_1～heap_5）を提供する理由が  
この問題一覧から理解できます。

### 3.1.5 Options for Dynamic Memory Allocation

FreeRTOS の初期バージョンでは、  **メモリプール方式**（memory pool allocation）が使用されていました。

メモリプール方式とは：
- 異なるサイズのメモリブロックをコンパイル時にあらかじめ確保しておき    
- メモリ割り当て関数が、その中からブロックを返す方式    

リアルタイムシステムでは一般的な手法ですが、  **非常に小さな組み込みシステムでは RAM を非効率に使用してしまう**ため、  
多くのサポート問い合わせが発生しました。  
その結果、FreeRTOS ではメモリプール方式は廃止されました。

FreeRTOS は現在、  **動的メモリ割り当てを「ポータブル層（移植層）」の一部として扱っています。**
理由：
1. 組み込みシステムによって、   **動的メモリ割り当ての要件やタイミング要件が大きく異なる**ため    
2. 単一のアルゴリズムでは、すべての用途に対応できないため    
3. メモリ割り当てをコアコードから切り離すことで、   **アプリケーション側で独自の実装を提供しやすくなる**    

FreeRTOS は標準 C の malloc/free を使用せず、次の関数を呼びます：
- `pvPortMalloc()` … malloc() の代わり    
- `vPortFree()` … free() の代わり   

それぞれ、標準 C ライブラリの関数と **同じプロトタイプ**を持っています。

これらの関数は **public（公開）関数** なので、  アプリケーション側から直接呼び出すこともできます。

FreeRTOS は、次の 5 種類の pvPortMalloc()/vPortFree() 実装例を提供しています：
- heap_1.c    
- heap_2.c    
- heap_3.c    
- heap_4.c    
- heap_5.c

これらはすべて以下のディレクトリにあります：
`FreeRTOS/Source/portable/MemMang`

アプリケーションは：
- これらの実装例のいずれかを使用してもよい
- または独自のメモリ割り当て方式を実装してもよい
## 3.2 Example Memory Allocation Schemes

### 3.2.1 Heap_1
### 3.2.2 Heap_2
### 3.2.3 Heap_3

### 3.2.4 Heap_4

### 3.2.5 Heap_5

### 3.2.6 Initialising heap_5: The vPortDefineHeapRegions() API Function

## 3.3 Heap Related Utility Functions and Macros

### 3.3.1 Defining the Heap Start Address

`heap_1`、`heap_2`、`heap_4` は、  **configTOTAL_HEAP_SIZE で指定されたサイズの静的配列**からメモリを割り当てます。  
この節では、これら 3 つの方式をまとめて **heap_n** と呼びます。

場合によっては、ヒープを **特定のメモリアドレス**に配置したいことがあります。
理由の例：
- 動的に作成されるタスクのスタックはヒープから割り当てられる    
- スタックは **高速な内部メモリ** に置きたいが   ヒープが **低速な外部メモリ** にあると性能が低下する   
（※ 下の「Placing Task Stacks in Fast Memory」も参照）

そのような場合に備えて、  **configAPPLICATION_ALLOCATED_HEAP** というコンパイル時設定が用意されています。

この設定を有効にすると：
- 本来 heap_n.c の中で宣言されるヒープ用配列を**アプリケーション側で宣言できるようになる**   
これにより、アプリケーション開発者は **ヒープの開始アドレスを自由に指定できる** ようになります。

設定：
- `configAPPLICATION_ALLOCATED_HEAP = 1`  または    **未定義のまま**
この場合、FreeRTOS を使用するアプリケーション側は必ず：

`uint8_t ucHeap[ configTOTAL_HEAP_SIZE ];`

という配列を定義しなければなりません。
この配列はヒープ領域として FreeRTOS に使用されます。

変数を特定のメモリアドレスに配置するための構文は、  **使用するコンパイラによって異なります**。  
そのため、実際の指定方法は各コンパイラのマニュアルを参照する必要があります。

以下は代表例として、2 種類のコンパイラの例を示しています。
- Listing 3.5 — GCC でのヒープ配置方法
	GCC では、次のような構文を使用して 配列を **.my_heap というメモリセクションに配置**できます。
	`uint8_t ucHeap[ configTOTAL_HEAP_SIZE ] __attribute__ ( ( section( ".my_heap" ) ) );`
	（この配列を heap_4 のヒープとして使用可能）
- Listing 3.6 — IAR でのヒープ配置方法
	IAR では、次の構文を使って  配列を **絶対アドレス 0x20000000** に配置します。
	`uint8_t ucHeap[ configTOTAL_HEAP_SIZE ] @ 0x20000000;`
	（こちらも heap_4 のヒープ領域として使用される）

### 3.3.2 The xPortGetFreeHeapSize() API Function

`xPortGetFreeHeapSize()` API 関数は、  **呼び出し時点でヒープに残っている未使用バイト数** を返します。

ただし、**ヒープの断片化（fragmentation）に関する情報は返しません。**
また、**heap_3 では xPortGetFreeHeapSize() は実装されていません。**

 `xPortGetFreeHeapSize()` 
 プロトタイプ
	`size_t xPortGetFreeHeapSize( void );`
 戻り値（Return value）
`xPortGetFreeHeapSize()` は、  **呼び出し時点でヒープに未割り当てのバイト数** を返します。

### 3.3.3 The xPortGetMinimumEverFreeHeapSize() API Function

### 3.3.4 The vPortGetHeapStats() API Function

### 3.3.5 Collecting Per-task Heap Usage Statistics

### 3.3.6 Malloc Failed Hook Functions

## 3.4 Using Static Memory Allocation


# 4. Task Management

## 4.1 Introduction
### 4.1.1 Scope

この章では次の内容を扱います：
-  **FreeRTOS がタスクに処理時間を割り当てる方法**
- **FreeRTOS が任意の時点でどのタスクを実行するか選択する仕組み**
- **タスクの相対的な優先度がシステム動作にどのように影響するか**
- **タスクが取り得る状態（ステート）**


また、この章では次の内容についても説明します：
- **タスクの実装方法**    
- **タスクのインスタンス（1つ以上）を作成する方法**    
- **タスクパラメータの使い方**    
- **すでに作成されたタスクの優先度を変更する方法**    
- **タスクを削除する方法**    
- **タスクを使って周期処理を実装する方法**  
    （ソフトウェアタイマーを使用する方法については後の章で説明）    
- **アイドルタスク（idle task）がいつ実行され、どのように活用できるか**    

この章で紹介する概念は、  FreeRTOS の使い方や FreeRTOS アプリケーションの動作を理解するうえで **基本となるもの** です。
そのため、**この章は本書の中で最も詳細に説明されている章** となっています。

### 4,2 Task Functions

タスクは **C 関数として実装**されます。  
タスクは、リスト 4.1 に示される **期待される関数プロトタイプ** に従わなければなりません。

そのプロトタイプは、
- **引数に `void*` 型のポインタを受け取り**    
- **戻り値は `void`**    
という形式です。

```
void vATaskFunction( void * pvParameters );
```

Listing 4.1 The task function prototype

各タスクは、それ自体が **小さな独立したプログラム** といえます。

- タスクには **エントリポイント（開始地点）** があり、    
- 通常は **無限ループ内で永遠に実行**され、    
- **終了（exit）しません。**    

リスト 4.2 には、典型的なタスクの構造が示されています。

FreeRTOS タスクは、  そのタスクを実装している関数から **戻る（return）ことが許されていません**。
- `return` 文を含めてはいけない    
- 関数の末尾まで実行されて抜けてしまってはいけない    
もしタスクが不要になった場合は、  
リスト 4.2 に示されているように **明示的にタスクを削除**する必要があります。

単一のタスク関数定義から、**任意の数のタスクを作成することができます**。  
作成されたそれぞれのタスクは独立した **実行インスタンス** を持ちます。
各タスクインスタンスは：
- **専用のスタック領域を持ち**    
- そのタスク内で定義された自動変数（スタック変数）の **独立したコピー** を保持します

```
void vATaskFunction( void * pvParameters )
{
/*
* Stack-allocated variables can be declared normally when inside a function.
* Each instance of a task created using this example function will have its
* own separate instance of lStackVariable allocated on the task's stack.
*/
long lStackVariable = 0;
/*
* In contrast to stack allocated variables, variables declared with the
`static`
* keyword are allocated to a specific location in memory by the linker.
* This means that all tasks calling vATaskFunction will share the same
* instance of lStaticVariable.
*/
static long lStaticVariable = 0;
for( ;; )
{
/* The code to implement the task functionality will go here. */
}
/*
* If the task implementation ever exits the above loop, then the task
* must be deleted before reaching the end of its implementing function.
* When NULL is passed as a parameter to the vTaskDelete() API function,
* this indicates that the task to be deleted is the calling (this) task.
*/
vTaskDelete( NULL );
}
```
### 4.3 Top Level Task States

アプリケーションは、複数のタスクから構成される場合があります。  
しかし、アプリケーションを実行しているプロセッサが **単一コア** の場合、  任意の時点で **実行できるタスクは 1 つだけ** です。
このことから、タスクは基本的に次の 2 つの状態のいずれかに存在することになります：
- **Running（実行中）**    
- **Not Running（非実行）**    

まずはこのシンプルなモデルから説明します。  その後の節で、Not Running 状態の細かなサブ状態について解説します。

タスクが **Running 状態** であるとは、  **プロセッサがそのタスクのコードを実際に実行している** 状態を意味します。

タスクが **Not Running 状態** のとき：
- タスクの実行は一時停止されており    
- タスクの状態（レジスタ等）は保存され    
- スケジューラが再びそのタスクを Running 状態にするまで待機する    

タスクが再開されるときは：
- **Running 状態を離れる直前に実行しようとしていた命令から**  
- **正確に処理が再開されます。**

これは FreeRTOS の **コンテキストスイッチ（文脈切り替え）** によって実現されます。

タスクが **Not Running（非実行）状態 → Running（実行中）状態** に遷移すると、  
そのタスクは **“switched in（スイッチイン）された”** または **“swapped in（スワップイン）された”**  
と言います。

逆に、タスクが **Running 状態 → Not Running 状態** に遷移すると、  **“switched out（スイッチアウト）された”** または **“swapped out（スワップアウト）された”**  
と言います。

FreeRTOS において、  **タスクをスイッチイン / スイッチアウトできるのはスケジューラ（scheduler）だけ**です。
つまり：
- どのタスクを実行させるか    
- どのタスクを実行停止するか    

は **すべて FreeRTOS スケジューラが決定します**。
## 4.4 Task Creation

FreeRTOS では、次の **6つの API 関数**を使用してタスクを作成できます：
- `xTaskCreate()`    
- `xTaskCreateStatic()`    
- `xTaskCreateRestricted()`    
- `xTaskCreateRestrictedStatic()`    
- `xTaskCreateAffinitySet()`    
- `xTaskCreateStaticAffinitySet()`    

各タスクは次の **2種類の RAM 領域**を必要とします：
1. **Task Control Block (TCB)** を保存する領域    
2. **スタック領域**    

`Static` が名前に含まれる API は：
- TCB とスタックのための RAM を    
- **アプリケーション側で事前に割り当てたメモリ（静的メモリ）**として    
- 関数のパラメータに渡す方式をとります。    

`Static` が付かない API は：
- 必要な RAM（TCB とスタック）を    
- **システムヒープから動的に割り当てる** 方式です。    

一部の FreeRTOS ポートでは、タスクを **restricted mode（制限モード／非特権モード）** で動作させることができます。
- `Restricted` が付く API  
    → **メモリアクセスが制限されたタスク（unprivileged task）** を作成    
- `Restricted` が付かない API  
    → **privileged mode（特権モード）** で実行され、  
    システム全体のメモリマップにアクセス可能    
	※ これは MPU（Memory Protection Unit）を持つ Cortex-M などで利用されます。

SMP をサポートする FreeRTOS ポートでは、  **複数の CPU コア上でタスクを並列に実行**できます。

`Affinity` が付く API 関数では：
- タスクをどの CPU コアで実行させるか    
- **実行コアの固定（CPU affinity）** を指定できます。    

FreeRTOS のタスク作成 API は柔軟である反面、  
**非常に複雑**です。

そのため、この文書のサンプルコードでは  **`xTaskCreate()`（最もシンプルな関数）** を使用する例がほとんどです。

#### 4.4.1 The xTaskCreate() API Function

リスト 4.3 には **xTaskCreate() API 関数のプロトタイプ** が示されています。  
一方、**xTaskCreateStatic()** には 2 つの追加パラメータがあり、  
それぞれ次のメモリ領域を指します：

1. **タスクのデータ構造（TCB: Task Control Block）を保持するために事前割り当てされたメモリ**    
2. **タスクのスタックとして使用するために事前割り当てされたメモリ**    

セクション **2.5「データ型とコーディングスタイルガイド」** では、  
FreeRTOS で使用されるデータ型や命名規則について説明されています。

```
BaseType_t xTaskCreate( TaskFunction_t pvTaskCode,
                         const char * const pcName,
                         const configSTACK_DEPTH_TYPE uxStackDepth,
                         void *pvParameters,
                         UBaseType_t uxPriority,
                         TaskHandle_t *pxCreatedTask
                       );
```

xTaskCreate() Parameters and return value:

- pvTaskCode
	タスクは **終了しない C 関数**であり、通常は **無限ループ**として実装されます。  
	`pvTaskCode` パラメータは、  
	**タスクを実装している関数へのポインタ（実質的にはその関数名）** を指定します。

- pcName
	タスクの **説明的な名前** です。  
	FreeRTOS 本体はこの名前を利用しませんが、**デバッグ時の識別用**に役立ちます。  
	（人間がハンドル値で識別するより文字列で識別する方がはるかに分かりやすいため）
	
	タスク名の最大長は、アプリケーションが定義する  
	`configMAX_TASK_NAME_LEN` によって決まります（NULL 終端を含む）。  
	もしそれより長い文字列を渡した場合、**文字列は切り詰められます**。

- usStackDepth
	タスク用に割り当てられるスタックサイズを指定します。
	- `xTaskCreate()` を使用する場合：  
	    → FreeRTOS がヒープから **動的にスタックメモリを確保**します
    - `xTaskCreateStatic()` を使用する場合：  
	    → **事前割り当てしたメモリ** をスタックとして使用できます（静的確保）
    
	 重要な注意点：
	`usStackDepth` は **バイト数ではなく「ワード数」** です。
	例：  
	CPU が 32bit スタックの場合  
	ワードサイズ = 4バイト  
	`usStackDepth = 128`  
	→ 確保されるスタック領域は **128 × 4 = 512 バイト**

- configSTACK_DEPTH_TYPE
	`configSTACK_DEPTH_TYPE` はスタックサイズを保持するデータ型を指定できます。
	- 未定義の場合 → **uint16_t（16bit）** が使用される
    - もしスタックサイズ * スタック幅 が **65535（16bit上限）を超える場合は、  
	    unsigned long や size_t を指定する必要があります**
	    （FreeRTOSConfig.h 内で `#define configSTACK_DEPTH_TYPE size_t` などと定義）
	スタックサイズの決め方については **13.3章 Stack Overflow** で解説されています。

- pvParameters
	タスクを実装する関数は、__void_ 型の引数を 1つ受け取ります_*。  
	`pvParameters` は、その引数としてタスクに渡される値です。

- uxPriority
	タスクの優先度を指定します。
	- **0 が最も低い優先度**
    - **configMAX_PRIORITIES - 1 が最も高い優先度**
    `configMAX_PRIORITIES` はユーザー定義の定数で、4.5節で説明されています。
	もし `uxPriority` に `configMAX_PRIORITIES - 1` より大きい値を指定した場合、  
	優先度は **自動的に最大値に丸められます（上限にキャップされる）**。

- pxCreatedTask
	作成されたタスクの **ハンドルを格納するためのポインタ** を指定します。
	取得したタスクハンドルは次の用途に使えます：
	- タスクの優先度を変更する
    - タスクを削除する
    - その他タスク制御 API に渡す
    ハンドルが不要な場合は **NULL を指定できます**。

xTaskCreate() / xTaskCreateStatic() が返す値は次の 2 種類です：
- pdPASS
	タスクが正常に作成された。
- pdFAIL
	ヒープメモリが不足していてタスクを作成できなかった。  
	（ヒープ管理については第3章を参照）

## 4.5 Task Priorities

## 4.6 Time Measurement and the Tick Interrupt

4.12 節「スケジューリングアルゴリズム」で、  オプション機能である **タイムスライシング（time slicing）** が説明されています。

これまでの例ではタイムスライシングが使用されており、  出力結果にもその動作が反映されていました。
例では：
- 2つのタスクは **同じ優先度** で作成され    
- **常に実行可能（Ready）状態** であったため    

スケジューラは両タスクに CPU を公平に割り当て、それぞれが **1タイムスライスずつ実行しては切り替わる** という動作になりました。

図 4.3 において t1 と t2 の間が **1つのタイムスライス** に相当します。

スケジューラは **各タイムスライスの終わり**に実行され、  次に Running 状態にするタスクを決定します。
この処理は **tick 割り込み**（periodic interrupt）を使って行われます。

tick 割り込みの周波数は、  **configTICK_RATE_HZ** によって設定されます。

例：  
configTICK_RATE_HZ = 100 (Hz) → 1タイムスライス 
つまり：
- **タイムスライス = 1 tick period**  

タイムスライスの終わりだけがタスク切り替えのタイミングではない。
スケジューラは以下のタイミングでも即座にタスクを切り替える：
- 現在実行中のタスクが **Blocked 状態に入ったとき**   
- 割り込みにより **より高優先度のタスクが Ready になったとき**    

図 4.4 では、図 4.3 を拡張し **スケジューラの実行タイミング**も示しています。

- 最上段の線がスケジューラの実行タイミング    
- 細い矢印が    
    - Running タスク → tick 割り込み        
    - tick 割り込み → 次に Running になるタスク への遷移を表す        

configTICK_RATE_HZ の最適値はアプリケーションによるが、  一般的には **100Hz** がよく使われます。

FreeRTOS の API では **時間は tick 単位で指定**します。  
そのためミリ秒表示を tick に変換するために：

`pdMS_TO_TICKS()`

マクロを使用します。
注意：
- configTICK_RATE_HZ > 1000 の場合（1kHz以上）は使用できない  
    → tick 精度が1ms未満となり計算が成立しないため    

```
/* 
 * pdMS_TO_TICKS() はミリ秒を tick 数に変換する。
 * 下記は 200ms に相当する tick 数を xTimeInTicks に代入する例。
 */ 
 TickType_t xTimeInTicks = pdMS_TO_TICKS( 200 );`
```

これを使うことで、tick 周波数を変更しても アプリケーション内の時間指定を変更する必要がありません。

tick count とは：
- スケジューラ開始後から現在までに起きた tick 割り込みの総数    

このカウンタはオーバーフローする場合があるが、  FreeRTOS は時間計算を内部で正しく処理するため  ユーザがオーバーフローを意識する必要はありません。

これまでの例ではタスクは同じ優先度だった。  
ここでは **異なる優先度のタスクを作るとどうなるか** を示す。
Listing 4.11 のコードでは：
- Task 1 → priority = **1**    
- Task 2 → priority = **2（高い方）**    

これらは同じ関数 vTaskFunction を利用し、  単に文字列を出力し、空ループでウェイトしている。

```
static const char * pcTextForTask1 = "Task 1 is running";
static const char * pcTextForTask2 = "Task 2 is running";
int main( void ) {
     /* 優先度1のタスク作成 */
     xTaskCreate( vTaskFunction,
                  "Task 1",
                  1000,
                  ( void * ) pcTextForTask1,
                  1,
                   NULL );      
    /* 優先度2のタスク作成（より高い） */
    xTaskCreate( vTaskFunction,
                      "Task 2",
                       1000,
                       ( void * ) pcTextForTask2,
                       2,
                       NULL );
                       
      /* スケジューラ開始 */
      vTaskStartScheduler();
      
      return 0; /* ここには到達しない */ 
}
```
---

```
Task 2 is running
Task 2 is running
Task 2 is running
...
```

Task 1 のメッセージは **一度も表示されない**。
理由：
- スケジューラは **実行可能なタスクの中で最も高い優先度を必ず選ぶ**    
- Task 2（優先度2）は常に Ready 状態（無限ループしているだけ）    
- よって Task 1（優先度1）は **一度も Running 状態になれない**    

この状態を **starvation（飢餓状態）** と呼ぶ。

図 4.6 では、優先度が高い Task 2 が CPU を独占し、  
Task 1 が一切実行されないスケジューリングパターンが表示されている。
## 4.7 Expanding the Not Running State

これまでの例で作成したタスクは、  常に何らかの処理を行っており、  **一度も「待つ必要がある状態」になりませんでした**。

そして、待たないということは、  **常に Running 状態になれる** ということです。

このような「**連続処理型（continuous processing）タスク**」は、  実際のアプリケーションでは **ほとんど役に立ちません**。

理由：
- 連続処理タスクを **最下位（最も低い）優先度**でしか作れない    
- もしそれより高い優先度で作れば、   **下位優先度のタスクは永遠に実行されなくなる（飢餓状態）**

連続処理型のタスクを実用的にするには、  それらを **イベント駆動型（event-driven）タスク** に書き換える必要があります。
イベント駆動タスクとは：
- **イベントが発生して初めて処理を行う**    
- イベントが発生するまでは Running 状態に入れない（= 何もしない）    

スケジューラは：
- **実行可能なタスクの中で最も優先度が高いもの**  を常に選ぶ。

しかし、高優先度タスクがイベント待ちのため Running になれなければ：
- その間は **より低優先度のタスクが実行される**    

つまり、タスクをイベント駆動型にすることで：
- すべてのタスクに処理時間が回るようになる    
- 高優先度タスクが下位タスクを **完全に飢餓状態にしてしまう問題** を防げる    
- タスクを **異なる優先度で安全に作成できる**

### 4.7.1 The Blocked State

タスクがイベントを待っているとき、  そのタスクは **「Blocked（ブロック）」状態** にあると言います。  
Blocked は **Not Running（非実行）」のサブ状態**です。

タスクが Blocked 状態に入る理由は、次の **2種類のイベント待ち**に分類できます。

 1. 時間関連イベント（Temporal events）
	これは **指定時間の経過や、特定の絶対時刻の到達** を待つ場合です。
	例：
	- タスクが「10ミリ秒待つ」ために Blocked 状態に入る  
	    → 時間が来るまで実行されない
    
 2. 同期イベント（Synchronization events）
	これは **他のタスクや割り込みによって発生するイベント**を待つ場合です。
	例：
	- キューにデータが届くのを待つ
    - セマフォを待つ
    - イベントビットがセットされるのを待つ
	同期イベントは非常に幅広い種類を含みます。
	FreeRTOS では、次の機能が同期イベントを生成できます：
	- キュー（queue）
    - バイナリセマフォ
    - カウンティングセマフォ
    - ミューテックス
    - 再帰ミューテックス
    - イベントグループ
    - ストリームバッファ
    - メッセージバッファ
    - **タスク通知（direct to task notification）**
	   これらの多くは後の章で説明されます。

タスクは「イベントを待つ + タイムアウトを設定する」ことで、  **2種類のイベントを同時に待つ**動作ができます。
例：
- 「最大 10ms の間、キューにデータが届くのを待つ」
この場合：
- 10ms の間にデータが届けば Blocked 状態から復帰    
- 10ms 経過しても届かなければタイムアウトで復帰    

どちらが先に起きても Blocked 状態を抜けます。

### 4.7.2 The Suspended State

### 4.7.3 The Ready State

### 4.7.4 Completing the State Transition Diagram

### 4.7.5 The vTaskDelayUntil() API Function

直前の説明のとおり、**vTaskDelay()** の引数は：
- 「タスクが vTaskDelay() を呼んでから、  
- Blocked 状態を抜けるまでに発生する tick 割り込みの数」

を指定します。

つまり、**待ち時間は「呼び出した時点」からの相対時間**です。

これに対して **vTaskDelayUntil()** の引数は：
- **タスクが Ready 状態へ遷移すべき “絶対的な tick カウント値” を指定する**

という点が決定的に異なります。

vTaskDelayUntil() は、タスクを **正確な周期で実行したい** 場合に使用します。
- vTaskDelay() → 待ち時間は「呼び出した瞬間からの相対時間」    
- vTaskDelayUntil() → 待ち時間は「スケジュール上の絶対時刻」    

よって、  **タスクが一定の頻度（一定周期）で正確に実行される必要がある場合は、  必ず vTaskDelayUntil() を使用すべきです。**

例えばタスクが以下の動作をする場合：
1. 何らかの処理（処理時間 5ms）    
2. vTaskDelay(10) を実行    

理論上は「10msごとに処理される」はずですが、  処理時間が加算されるため実際には：
- 5ms（処理）    
- 10ms（待機）    
- 5ms    
- 10ms  
- …    
	→ **周期は 15ms になってしまう（ドリフトする）**

vTaskDelayUntil() を使用すると：
- 「次に起きるべき tick 時刻」を計算し    
- そこで正確に Blocked から Ready に戻す    

よって処理時間に関係なく周期が一定になります。

![[Pasted image 20251212175811.png]]

Listing 4.14 vTaskDelayUntil() API function prototype

#### vTaskDelayUntil() のパラメータ説明
- **pxPreviousWakeTime**
	このパラメータ名は、vTaskDelayUntil() を **一定周期で実行されるタスク** に使うことを想定して付けられています。
	`pxPreviousWakeTime` が保持する値は、  **タスクが直前に Blocked 状態から復帰した時の tick 時刻** です。
    vTaskDelayUntil() は、この値を基準にして    **次にタスクが Blocked から抜けるべき絶対時刻** を計算します。
    `pxPreviousWakeTime` が指す変数は、   **vTaskDelayUntil() の内部で自動的に更新されます。**
    
	アプリケーション側で書き換える必要はありませんが、  
	**初回使用前に必ず現在の tick count で初期化する必要があります。**
	初期化例は Listing 4.15 に示されています。

-  **xTimeIncrement**
	このパラメータ名も、  タスクが **一定周期（固定された周波数）** で動作することを前提にしています。
	`xTimeIncrement` には **周期（実行間隔）を tick 単位で指定**します。
	 ミリ秒で周期を指定したい場合は`pdMS_TO_TICKS()`を使って tick 数に変換できます。
	（例：100ms → pdMS_TO_TICKS(100)）

| パラメータ                  | 意味              | アプリ側の操作                                        |
| ---------------------- | --------------- | ---------------------------------------------- |
| **pxPreviousWakeTime** | タスク前回実行時刻（tick） | 初回だけ「現在の tick」で初期化。以降は vTaskDelayUntil() が自動更新 |
| **xTimeIncrement**     | 周期（tick 単位）     | ミリ秒なら pdMS_TO_TICKS() を使って指定                   |

◎ 例：周期 100ms のタスク

```
TickType_t xLastWakeTime;
const TickType_t xInterval = pdMS_TO_TICKS( 100 );

/* 初期化（現在の tick 時刻をセット） */
xLastWakeTime = xTaskGetTickCount();

for( ;; ) {     
	/* 周期実行 */
	vTaskDelayUntil( &xLastWakeTime, xInterval );      /* 実行したい処理を書く */ }`
```

このタスクは、処理時間にかかわらず **正確に100ms周期**で動作します。

図 4.11 は、**例 4.6（Example 4.6）** によって生成される出力を示しています。  
そして図 4.12 は、その出力で観察される挙動を説明するための **実行シーケンス** を示しています。


図 4.11 は、例 4.6（Example 4.6）によって生成される出力を示しています。  
そして図 4.12 は、その出力で観察される動作を説明するための実行シーケンスを示しています。

## 4.8 The Idle Task and the Idle Task Hook

Example 4.4 で作成したタスクは、  ほとんどの時間を **Blocked 状態** で過ごします。
Blocked 状態のタスクは **実行不能（Not Runnable）** であるため、  スケジューラがそれらを Running 状態として選択することはできません。

FreeRTOS では必ず、**Running 状態に遷移できるタスクが最低 1つ存在する必要があります**。
そのために、  **vTaskStartScheduler()** が呼ばれた時点で、  スケジューラは自動的に **Idle Task（アイドルタスク）** を生成します。

Idle タスクは単純なループを繰り返すだけのタスクで、  最初の例に登場した「常に実行可能なタスク」と同様、  **常に Running 状態に入ることができます。**

 - 注釈 1
	Idle タスクは **優先度 0（最下位）** で動作します。
	これは：
	- Idle タスクがアプリケーションタスクの実行を妨げないようにするため
    - 優先度 0 にアプリケーションタスクを作りたい場合は、それと共存できるようにするため
    
アプリケーション側で **優先度 0 のタスク** を自由に作成できます。
FreeRTOSConfig.h の設定：
`configIDLE_SHOULD_YIELD`
を使うと、Idle タスクが  
**同じ優先度（0）のアプリケーションタスクより CPU を取りすぎないように** 調整できます。  
（詳細は 4.12 Scheduling Algorithms で説明）

Idle タスクは最も低い優先度なので、**より高い優先度のタスクが Ready になった瞬間にスケジューラが切り替える**

これが **プリエンプション（preemption）** です。
図 4.9 の tn の時刻では：
- Idle タスクが Running 状態だったが    
- Task 2 が Blocked → Ready に変わった瞬間に    
- Idle タスクは即座に **swapped out（スイッチアウト）** され    
- Task 2 が **swapped in（スイッチイン）** される    

プリエンプションは **自動的** に行われ、  プリエンプトされたタスク（Idle タスク）はこれを知りません。

もしタスクが **vTaskDelete() を使って自分自身を削除する場合**は、 **Idle タスクが CPU 時間を奪われないようにすることが非常に重要です。**
理由：
- 自分自身を削除したタスクの後始末（TCBやスタックの解放）は   **Idle タスクの役割** だから    

Idle タスクが starvation（飢餓状態）になると、  削除されたタスクのメモリが解放されず、  深刻なリソースリークや動作不良につながります。

### 4.8.1 Idle Task Hook Functions

FreeRTOS では **Idle Hook（アイドルフック）** と呼ばれる仕組みを使うことで、  Idle タスクにアプリケーション固有の処理を追加できます。

Idle Hook とは：
	**Idle タスクのループが 1 回まわるたびに自動的に呼び出される関数**
です。

Idle Hook は次のような用途でよく使われます：
 1. **低優先度のバックグラウンド処理を行う**
	バックグラウンド処理や連続的に行う軽い処理を行いたいときに便利です。
	- わざわざ専用のタスクを作る必要がない
    - RAM 使用量（TCB + スタック）が増えない
    → **軽量な周期処理**に最適。
 2. **CPU の空き時間（余剰処理能力）を測定する**
	Idle タスクが実行されるのは：
	 **すべての高優先度タスクが実行できないときだけ**
	そのため Idle Hook 内で処理時間を測定すれば：
	- Idle タスクにどれだけ CPU が割り当てられたか
	- → システムの **空き処理能力（CPU マージン）**
    が把握できます。
	例：CPU 使用率の計算など。
3. **CPU を低消費電力モードに移行させる**
	Idle タスクにはアプリケーションの処理がないときに CPU に入るため、Idle Hook 内で MCU を低電力モードに入れる
	という使い方ができます。
	ただし：
	- 省電力効果は **tickless idle（ティックレスアイドル）** ほど高くはない
	- それでも簡単に電力節約を実現できる   

FreeRTOSConfig.h に以下を追加します：
`#define configUSE_IDLE_HOOK 1`

そして以下の関数を定義します：
```
void vApplicationIdleHook( void ) {
     /* Idle タスク内で実行したい処理を書く */ 
}
```

### 4.8.2 Limitations on the Implementation of Idle Task Hook Functions

## 4.9 Changing the Priority of a Task

## 4.10 Deleting a Task

## 4.11 Thread Local Storage and Reentrancy

Thread Local Storage（TLS）は、  **各タスクの Task Control Block（TCB）内に、アプリケーションが任意のデータを保存できる仕組み**です。

つまり：
- **タスクごとに独立した“専用の変数”を持てる仕組み**
です。
これは、多くの RTOS や POSIX システムでも一般的な概念です。

TLS は主に以下のような目的で利用されます。
- 非再入可能（non-reentrant）関数のためのデータ保存
	通常、非RTOS環境では「グローバル変数」に保存されるデータを、  
	RTOS 環境では **TLS に保存してタスクごとに独立させる** 目的で使用されます。

 再入可能（reentrant）関数とは？
- 複数のタスク（スレッド）から同時に呼んでも    
- 副作用やデータ破壊を起こさず    
- 完全に安全に動作できる関数    

非再入可能（non-reentrant）関数とは？
内部で以下のようなものを使用する関数です：
- グローバル変数    
- static ローカル変数    
- 共有バッファ    

そのため **複数タスクから同時に呼ぶと危険** です。

通常はクリティカルセクションで保護する必要がありますが…
非再入関数を守るためにクリティカルセクションを多用すると：
- 割り込み禁止時間が増える   
- スケジューリング精度が下がる    
- リアルタイム性が低下する    

これを避けるために、TLS を使って：
 **タスクごとに独立したデータ領域を持たせるほうが、RTOS では有利**
になります。

TLS の代表例は：

> **ISO C 標準の global 変数 `errno`**

です。

`errno` は、標準Cライブラリや POSIX システムで使用される  
**拡張エラーコードを格納するグローバル変数**です。

以下のような標準関数で使用されます：
- `strtof`   
- `strtol`    
- その他の標準ライブラリ関数    

これらは非再入関数であるため、  マルチスレッド環境では `errno` を **TLS によってタスクごとに分離する**必要があります。

| 項目                        | 説明                             |
| ------------------------- | ------------------------------ |
| TLS（Thread Local Storage） | タスクごとに独立したデータ領域を持てる仕組み         |
| 目的                        | 非再入関数が使うデータをタスクごとに分離する         |
| 利点                        | クリティカルセクションを減らし、RTOS の性能を維持できる |
| 代表例                       | C 標準ライブラリの `errno`             |
### 4.11.1 C Runtime Thread Local Storage Implementations

多くの組み込み向け libc 実装は、  **非再入（non-reentrant）関数がマルチスレッド環境で正しく動作できるようにするための API** を提供しています。

FreeRTOS は、広く使われている 2 つのオープンソース C ランタイムライブラリ  **newlib** と **picolibc** の再入性（reentrancy）API をサポートしています。

これらの **事前構築済みの C ランタイム用 Thread Local Storage（TLS）実装** は、  プロジェクトの FreeRTOSConfig.h に以下のマクロを定義することで有効化できます。

- **configUSE_NEWLIB_REENTRANT**（newlib を使用する場合）    
- **configUSE_PICOLIBC_TLS**（picolibc を使用する場合）    

これらのマクロを有効にすると、FreeRTOS がタスクごとに TLS 領域を確保し、  newlib/picolibc が必要とする「タスク固有の errno、標準ライブラリ状態」などを  自動的に提供してくれます。

特に **newlib の `_reent` 構造体** の扱いは複雑で手作業でサポートするのは困難なため、  `configUSE_NEWLIB_REENTRANT` を 1 にするのが推奨されます。

### 4.11.2 Custom C Runtime Thread Local Storage

アプリケーション開発者は、FreeRTOSConfig.h ファイル内で以下のマクロを定義することで、 独自の Thread Local Storage（TLS）を実装できます。

- **configUSE_C_RUNTIME_TLS_SUPPORT** を 1 に定義すると、   C ランタイム用 Thread Local Storage サポートが有効になります。    
- **configTLS_BLOCK_TYPE**   C ランタイム Thread Local Storage データを保存するために使用する **C の型** を定義します。    
- **configINIT_TLS_BLOCK**     C ランタイム Thread Local Storage ブロックを **初期化するときに実行される C コード**を定義します。    
- **configSET_TLS_BLOCK**      新しいタスクが **スイッチインされる際に実行される C コード**を定義します。    
- **configDEINIT_TLS_BLOCK**     C ランタイム Thread Local Storage ブロックを **解放（デイニシャライズ）するときに実行される C コード**を定義します。    

この仕組みは以下のような場合に必要になります：
- newlib や picolibc のようなライブラリを使わず  **アプリケーション側で独自の TLS 管理を実装したい場合**    
- `errno` のようなスレッド固有データを自前で管理したい場合    
- C ランタイム以外のライブラリで TLS を必要とする場合    

FreeRTOS は TCB（Task Control Block）内に TLS 用の領域を確保できるため、  上記マクロを使うことで自由に TLS の初期化・切り替え・解放処理を追加できます。

### 4.11.3 Application Thread Local Storage

C ランタイム用 Thread Local Storage に加えて、 アプリケーション開発者は **アプリケーション固有のポインタセット** を  タスクコントロールブロック（TCB）内に追加することもできます。

この機能は、プロジェクトの FreeRTOSConfig.h にて  `configNUM_THREAD_LOCAL_STORAGE_POINTERS` を **0 以外の値に設定**することで有効になります。

`Listing 4.30` に示す関数 **vTaskSetThreadLocalStoragePointer** と  **pvTaskGetThreadLocalStoragePointer** を使用すると、  実行時にそれぞれ TLS ポインタの値を **設定**および **取得**できます。

### Thread Local Storage Pointer API の関数プロトタイプ

```
void * pvTaskGetThreadLocalStoragePointer( TaskHandle_t xTaskToQuery,
                                           BaseType_t xIndex );

void vTaskSetThreadLocalStoragePointer( TaskHandle_t xTaskToSet,
                                         BaseType_t xIndex,                                          
                                         void * pvValue );`

```

FreeRTOS の TLS 機能は次のように使えます：
- **タスクごとに独立したデータ構造を持たせたい**    
- **ドライバ/ミドルウェアがタスクごとに状態を保持したい**    
- **グローバル変数をなくしたい**    
- **オブジェクト指向ライクにタスクごとの “メンバ変数” を持たせたい**    

`configNUM_THREAD_LOCAL_STORAGE_POINTERS` に 5 を設定した場合、  各タスクには __5 個の void_ スロット_* が割り当てられます。
例：タスクごとにログバッファを割り当てたい場合などに利用できます。

## 4.12 Scheduling Algorithms

# 5 Queue Management

## 5.1 Introduction

「キュー（Queue）」は、  **タスク間（task-to-task）、タスクから割り込み（task-to-interrupt）、割り込みからタスク（interrupt-to-task）**  
への通信メカニズムを提供します。
### 5.1.1 Scope

この章では次の内容を扱います：
- **キューの作成方法**    
- **キューが内部データをどのように管理するか**    
- **キューへデータを送信する方法**    
- **キューからデータを受信する方法**    
- **キューでブロックするとはどういうことか**    
- **複数のキューを待ち受け（ブロックし）ながら扱う方法**    
- **キュー内のデータを上書きする方法**    
- **キューをクリア（空に）する方法**    
- **キューの読み書きにおけるタスク優先度の影響**    

なお、この章で扱うのは **タスク間（task-to-task）通信のみ** です。  
**タスク ↔ 割り込み** の通信については第7章で説明します。
## 5.2 Characteristics of a Queue

# 6 Software Timer Management

## 6.1 Chapter Introduction and Scope

ソフトウェアタイマーは、**将来の決められた時刻** または **一定周期（固定周期）** で、ある関数を実行するために使用されます。  
このとき実行される関数を **ソフトウェアタイマーのコールバック関数** と呼びます。

ソフトウェアタイマーは FreeRTOS カーネルによって実装・管理されます。  
ハードウェアのサポートを必要とせず、ハードウェアタイマーやハードウェアカウンタとは無関係です。

FreeRTOS の「必要なときだけ動作させる」という設計哲学に沿い、  **ソフトウェアタイマーのコールバック関数が実行されるとき以外は、  ソフトウェアタイマーは CPU 時間を一切消費しません。**

ソフトウェアタイマー機能はオプションです。  
ソフトウェアタイマーを利用するには次の手順を実行します：
 1. FreeRTOS ソースファイル
	`FreeRTOS/Source/timers.c`  
	をプロジェクトにビルド対象として追加する。
 2. FreeRTOSConfig.h に以下の定数を定義する：
	- **configUSE_TIMERS**
		 値を `1` に設定するとソフトウェアタイマー機能が有効になる。
    - **configTIMER_TASK_PRIORITY**
		タイマーサービスタスクの優先度を  
	    `0 ～ (configMAX_PRIORITIES - 1)` の範囲で設定する。
    -  **configTIMER_QUEUE_LENGTH**
		 タイマーコマンドキューが保持できる  
	    **未処理コマンド数の最大値** を設定する。
    - **configTIMER_TASK_STACK_DEPTH**
	 タイマーサービスタスクに割り当てるスタックサイズを  **ワード数（bytes ではなく words）** で設定する。

### 6.1.1 Scope