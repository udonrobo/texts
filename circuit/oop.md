# オブジェクト指向プログラミング
オブジェクト指向プログラミング(Object Oriented Programming)とは、
オブジェクト間の相互の作用としてシステムの振る舞いを捉えるパラダイムです。
# オブジェクト指向について
ここではクラスベースのオブジェクト指向について述べます。
オブジェクトとは、結合の強いデータと関数を纏めたものです。
「クラス」がオブジェクトの型で、「インスタンス」がオブジェクトの実体です。
# 注意
オブジェクト指向を使わなくてもチーム開発は出来ます。
関数、グローバル変数を纏めたいだけならnamespaceを使ってください。
この文書だけでなくC++の参考書等も十分に読み、コードを書いてください。
テンプレートや過去のコードをコピペして使うのはそのコピペ元を十分理解し、
それがベターな方法だと判断出来た場合にしてください。
### インスタンスとクラス
C++などの静的な型システムを持つプログラミングで多く採用される。
クラスがインスタンスの型、インスタンスがクラスの実体となる。
クラスが`int`でインスタンスが`-4`みたいな関係を持つ。
オブジェクトの型がクラス、オブジェクトの実体がインスタンス。
### メッセージング
オブジェクト間の相互のメソッド呼び出しによってシステムの振る舞いを記述する。
この時のオブジェクトとはデータとメソッド(関数のようなもの)が一体となった値。
### カプセル化
副作用や状態を示すフラグ等を外部から見えないようにオブジェクトの内部に隠蔽すること。
外部との関わりをなるべくメソッドを介して行うことで、
オブジェクト外と粗結合にして見通しをよく出来たりする。
### 委譲
オブジェクトがメンバ変数として他のオブジェクトを抱え、
自分のメソッドの処理をメンバとして抱えた他のオブジェクトに丸投げする。
### 多態性
JavaScript等の動的に型付けがされる言語だと問題にならないが、
静的に型を付ける言語だと単純な実装ではオブジェクトの動作を変更するにはクラス、
つまり型を変更する事になり扱いづらい。
そこで継承により親クラスのメンバ変数、メソッドを子クラスが全て持つようにすることで、
子クラスを親クラスとしても扱うことが出来て柔軟性が高まる。
子クラスは親クラスとメソッド、メンバ変数の型が揃えばメソッドの中身を入れ替えてしまっても(型システム上は)問題が無い。
ライブラリを設計するのでないなら考えなくて良い。
## 機能とデータを纏める
```c
enum Direction {
	Right,
	Left
}

void drive(int ina, int inb, int pwm, enum Direction dir, uint8_t power) {
	// 実装は省略
}
//----ここより仕様側----

int motor1InA = D10;
int motor1InB = D9;
int motor1Pwm = D8;

void loop() {
	drive(motor1InA, motor1InB, motor1Pwm, Right, 127);
}
```
クラス版
```c++
class Motor {
private:
	int ina, inb, pwm;
public:
	Motor(int ina, int inb, int pwm) {
	}

	void drive(enum Direction dir, uint8_t power) {
		//実装は省略
	}
}

//----ここより仕様側----

Motor motor(D10, D9, D8);

void loop() {
	motor.drive(Right, 127);
}
```
このようにデータと機能を一纏めにすることでより使いやすいインターフェースを提供することが出来ます。
また、ina, inb, pwmのように外部に公開する必要の無いデータ、
外部から変更されると困るデータは、private修飾子で外部からの直接的なアクセスを禁止することが出来ます。
同じように、外部に公開する必要の無いヘルパ関数などもprivate修飾子を付けて内部からしか使えないように出来ます。
## 多態
TODO
# この文書について
## マサカリを投げる場合
Issueを立てるかPRを送ってください。マサカリお待ちしています。