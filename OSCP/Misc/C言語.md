# 基礎知識

## アドレス・ポインタについて

- `&hoge`：アドレス演算子で、メモリ上のアドレスを返す
-  ポインタ(`*`)：アドレスを示すもの
	- 定義した関数の戻り値アドレスの場合、ポインタ型の関数とする
```c
int static_value = 10;

int *get_static_value_addr()
{
	return &static_value;
}
```

- 配列を関数に渡す＝ポインタを渡す（ポインタ型は使っていないが）
```c
strcpy(buffer, argv[1])
```
- bufferは実は配列の先頭へのポインタがわたされている　

```c
int main(int argc, char *argv[]) {
    char buffer[64];
    strcpy(buffer, argv[1]);
    return 0;
}
```
- `int argc`：引数の個数（argument count）
- `char *argv[]`：`char*`型の配列
	- 各要素は`char*`（＝文字列へのポインタ）

## 構造体

関連するデータを1つにまとめたいときに使う。

たとえば1人の人間の情報を1つにまとめて管理する場合：
```c
struct Person {
    char name[20];
    int age;
    float height;
};
```

- 各要素（変数）は メンバ（member） と呼ぶ
    - `name`
    - `age`
    - `height`

### メンバにアクセスする方法

方法①：構造体変数そのものからアクセス
```c
struct structure s;
s.b = 1; //sのbメンバにアクセスして値を導入
```

方法②：構造体へのポインタからアクセス（構造体へのアドレスを指している`*p`）
```c
struct structure *p;
p->b = 1; //pが指す構造体のbメンバに1を代入
```

## 文字列の取り扱いについて

- 最後に必ず終端文字が追加される必要がある
```c
char src[] = {'A', 'B', 'C', 0x00, 'D'};
printf("%s", src); // 出力は "ABC" まで。'D' は出力されない
```

- printfなどは自動で終端文字を追加する
```c
printf("Hello world.")
```
	↓
```
Hello world.\0
```

---

# メモリの操作

## malloc

- メモリを確保する（※memsetでメモリを初期化しないと使えない）
- 返り値は確保したバッファの先頭アドレスのポインタ
```c
int buffer_size = 1000;
char *pointer = malloc(buffer_size);
```

## memset

- メモリの初期化
```c
# memset(確保メモリのポインタ, 初期化値, サイズ)
memset(pointer, 0x41, buffer_size);
```