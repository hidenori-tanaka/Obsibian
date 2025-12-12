## 4. Task Management
### 4.1 Introduction
#### 4.1.1 Scope

この章では次の内容を扱います：

### 🧵 **FreeRTOS がタスクに処理時間を割り当てる方法**

### 🧠 **FreeRTOS が任意の時点でどのタスクを実行するか選択する仕組み**

### ⚖ **タスクの相対的な優先度がシステム動作にどのように影響するか**

### 🔄 **タスクが取り得る状態（ステート）**

---

また、この章では次の内容についても説明します：

- **タスクの実装方法**
    
- **タスクのインスタンス（1つ以上）を作成する方法**
    
- **タスクパラメータの使い方**
    
- **すでに作成されたタスクの優先度を変更する方法**
    
- **タスクを削除する方法**
    
- **タスクを使って周期処理を実装する方法**  
    （ソフトウェアタイマーを使用する方法については後の章で説明）
    
- **アイドルタスク（idle task）がいつ実行され、どのように活用できるか**
    

---

この章で紹介する概念は、  
FreeRTOS の使い方や FreeRTOS アプリケーションの動作を理解するうえで **基本となるもの** です。

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

---

### ⚠ FreeRTOS タスクは決して関数から戻ってはいけない

FreeRTOS タスクは、  
そのタスクを実装している関数から **戻る（return）ことが許されていません**。

- `return` 文を含めてはいけない
    
- 関数の末尾まで実行されて抜けてしまってはいけない
    

もしタスクが不要になった場合は、  
リスト 4.2 に示されているように **明示的にタスクを削除**する必要があります。

---

### 🧵 1つのタスク関数から複数の実行インスタンスを生成できる

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

---

## 戻り値（Return values）

xTaskCreate() / xTaskCreateStatic() が返す値は次の 2 種類です：

### ✔ pdPASS

タスクが正常に作成された。

### ✘ pdFAIL

ヒープメモリが不足していてタスクを作成できなかった。  
（ヒープ管理については第3章を参照）