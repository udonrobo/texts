# ファイル分割のすゝめ

複雑なプログラムを開発するようになるとソーステキストが長くなり，一つのソースファイルで作成していると全体を見通すのが難しくなってくる。
また，一部だけを修正した際に全体をコンパイルし直さなければならないので，コンパイルに時間がかかるようになる。
従って，プログラムを分割して別々に開発すれば効率が良い。  
C++では別々のcppファイルをそれぞれコンパイルし，最後にそれらをリンクすることによって一つのプログラムを生成することができる**分割コンパイル**という手法が可能であり，Arduinoでも，分割コンパイルのようにcppファイルを複数作成しソースを分割することができる。  
ファイルを分割すれば，複数人でプログラムを開発することも可能になり，プログラムの整備性も上がる。一方，変数や関数がファイルを跨ぐので管理が必要である。


# 分割方法
実際の分割方法について説明する。
## e.g.

```arduino
//file1.cpp
#include<arduino.h>

void printHello(){
    Serial.println("hello");
}
```

```arduino
//file2.ino
void printHello();

void setup(){
    Serial.begin(9600);
}

void loop(){

    printHello();
}
```

```
[Serial Monitor]
hello
```

このように，`file1.cpp`で定義した関数を`file2.ino`で使用することができる。だが，この場合`file2.ino`で`file1.cpp`の関数を宣言しなければならない。関数が少ない場合はこれでも構わないが，多くなる場合は結局プログラムが見にくくなる。そこで，関数を宣言するためのヘッダファイルを用意しそれをインクルードすると良い。

## e.g.

```arduino
//file1.cpp
#include<arduino.h>

void printHello(){
    Serial.println("hello");
}
```

```arduino
//sub.h
#pragma once

void printHello();
        ･
        ･
        ･
 //関数を書いていく
```

```arduino
//file2.ino
#include"sub.h"

void setup(){
    Serial.begin(9600);
}

void loop(){

    printHello();
}
```

```
[Serial Monitor]
hello
```

# extern記憶クラス指定子
関数のファイル分割法について説明したが，変数についても同様に他のファイルにある変数を参照することができる。

## e.g.

```arduino
//file1.cpp
#include<arduino.h>

int x = 10;
```

```arduino
//file2.ino

extern int x;

void setup(){
    Serial.begin(9600);
}

void loop(){

    Serial.print("x=");
    Serial.println(x);
}
```
```
[SerialMonitor]
x=10
```
`extern`を付けると大域変数の宣言とみなされ，そこではメモリを割り当てずに外部にある変数を参照する。よって，`file1.cpp`と`file2.ino`にある変数xは同じメモリ領域を指す。また，関数のファイル分割と同様に宣言用のヘッダを作成しそこで宣言することが好ましい。  
変数をそのファイル内だけで使用したい場合は，`static`を付けることにより内部リンケージとなり，外部からは参照されない。

# グローバルインスタンス
クラスは，インスタンスを生成したファイルでしかそれを使用することができないが，他のファイルでそのインスタンスを使用したい場合はヘッダファイル内でグローバルインスタンスとして宣言することで使用することができる。

## e.g.

```C++
//greeting.h
#pragma once
#include<arduino.h>

class Greeting{

    public:

    void printHello(){
        Serial.println("hello");
    }
};
extern Greeting myGreeting;
```

```arduino
#include"greeting.h"

void setup(){
    Serial.begin(9600);
}

void loop(){
    myGreeting.printHello();
}
```

```
[Serial Monitor]
hello
```
このように`extern`を付けることによってどのcppファイルからもインスタンス`myGreeting`を使用することができる。

# まとめ
ファイル分割は便利

# 発行日
2017/07/14