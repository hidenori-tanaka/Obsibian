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