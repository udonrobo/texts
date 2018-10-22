# コーディングガイド
これは設計とコーディングの為のガイドラインです。
`coding_rule.md`と違い必ず守る必要は無いですし、これを絶対的なものとはしないでください。

# 一つの状態は一つの変数
```c
bool switch1, switch2;
if (switch1 && switch2) {

}
else if (switch1 && !switch2) {

}
else if (!switch1 && switch2) {

}
else {

}
```
人類は一般に頭が悪いので複数の変数で表現された状態を管理できない。
```c
enum State{
  GoAhead,
  RightTurn,
  LeftTurn,
  Stop
}

enum State state;

if (switch1 && state2) {
  state = GoAhead;
}
else if (switch1 && !switch2) {
  state = RightTurn;
}
else if (switch2 && !switch1) {
  state = LeftTurn;
}
else {
  state = Stop;
}

switch (state) {
  case GoAhead:
    break;
  case RightTurn:
    break;
  case LeftTurn:
    break;
  case Stop:
    break;
}
```
一度しか使わないなら最初のコードにコメントを追加してもいいと思うだろうが、状態に依存する処理が増えるにつれバグを埋め込むようになる。

# 結合を疎に
コードやデータが互いに強く依存しあっている状態を密結合、
依存度が低い状態を粗結合という。

```c
void driveVerticalTire(uint8_t controllerInfo[]) {
  if (controllerInfo[3] & _BV(2) != 0) {
    digitalWrite(D7, HIGH);
    digitalWrite(D8, LOW);
    analogWrite(D9, 200);
  }
  else {
    digitalWrite(D7, LOW);
    digitalWrite(D8, LOW);
  }
}
```
上のコードは送られてくるデータの形式に強く依存しており、
コントローラのデータ形式が変更された場合容易にバグを発生させることが予想される。

```c
enum class DriveState {
  Drive,
  Stop
};

void driveVerticalTire(DriveState state) {
  if (state == DriveState::Drive) {
    digitalWrite(D7, HIGH);
    digitalWrite(D8, LOW);
    analogWrite(D9, 200);
  }
  else {
    digitalWrite(D7, LOW);
    digitalWrite(D8, LOW);
  }
}
```
引数にはコントローラからのデータではなく期待される動作の状態を渡すように変更した。
これでコントローラのデータ形式が変更されてもdriveVerticalTireは変更する必要が無くなる。
変更する箇所が減ればその分バグを生みにくくなる。
# コードの再利用
```c
enum class DriveState {
  Drive,
  Stop
};

void driveVerticalTire(DriveState state) {
  if (state == DriveState::Drive) {
    digitalWrite(D7, HIGH);
    digitalWrite(D8, LOW);
    analogWrite(D9, 200);
  }
  else {
    digitalWrite(D7, LOW);
    digitalWrite(D8, LOW);
  }
}
```
上記のコードは基板の配線を直書きしているため基板を変更することも、他のアクチュエータに使う事も出来ない。

```c
struct Motor {
  uint8_t ina, inb, pwm;
}

void drive(struct Motor motor, DriveState state) {
  if (state == DriveState::Drive) {
    digitalWrite(motor.ina, HIGH);
    digitalWrite(motor.inb, LOW);
    analogWrite(motor.pwm, 200);
  }
  else {
    digitalWrite(motor.ina, LOW);
    digitalWrite(motor.pwm, LOW);
  } 
}

void driveVerticalTire(DriveState state) {
  struct Motor motor = { D7, D8, D9 };
  drive(motor, state);
}
```
この様に変更するとdriveの部分は他のアクチュエータにも使い回せて嬉しさがある。
# 値に名前を付ける
```c
delay(12);
```
上のコードだけでは意味が分からない。書いた本人も書いてからしばらくは覚えているだろうけどそのうち忘れる。

```c
SENSOR_INIT_TIME = 8;
SENSOR_INIT_MARGIN = 4;

delay(SENSOR_INIT_TIME + SENSOR_INIT_MARGIN);
```

これならひと目で意味が分かる。

# true/falseに注意する
true/falseは意外と意味が分かりづらい。
```c
bool x = getFrontSwitchState();
```
単純にHIGHならtrueなのか?これがプルアップされている場合は?
常時onで何かあるとoffになる場合は?
```
enum class SwitchState {
  Open,
  Close
};

SwitchState x = getFrontSwitchState();
```

これならスイッチの開閉状態だと確実に分かる。
boolのままにする場合は
```c
bool x = isFrontSwitchClosed();
```
などとする。
# コメントに頼りすぎない
なるべくならコメントは書かずとも容易に意味を取れるようなコードを書くべき。
C++とコメントでは言語が違うので切り替えながら読むと疲れる。
ただ実際にはそんな事は無理なのでコメントを書く。
授業では一行に一つコメントを書けと指示が出る場合もあったりするが明らかに過剰。
# スコープを意識する
スコープというのは大雑把に言うと関数、変数の見える範囲。
スコープの範囲外からはそのスコープに属す変数を触れないのでスコープの外は直接は考えなくて良くなる。
スコープが狭いなら変数名は簡潔さを優先していいが、スコープが広い場合は分かりやすさを優先すべき。
```c
LEDState frontRightLED, frontLeftLED;

void swapFrontLEDState() {
  auto tmp = frontLeftLED;
  frontLeftLED = frontRightLED;
  frontRightLED = tmp;
  // 以下省略
}
```
グローバル変数はどこからでも参照できるので分かりづらい名前だとバグの原因となる。
逆に関数内の一時的な変数はスコープが小さく変数が何を意味するかを忘れる前に変数の見える範囲が終わるのでバグを起こしにくい。
スコープが広いと面倒になるので変数のスコープを小さく保っておけば楽。
staticなどもスコープを狭めるのに役立つ。
# 一関数一機能
一つの関数で複数の機能を実現しようとしない。

センサから情報を読み取り、出力を計算し、出力する関数があるならそれを
それぞれの機能に分割する。

人類が同時に複数のことを考えるのは早すぎる。
関数はn行以内というのもこれから来ている。
モニタの解像度が高まると一度に表示できる行数が増えるのが人間の認識能力は何も変わらない。
# 関数に余計な引数を渡さない
関数に渡す引数は多くても3つ程度にする。
パラメータが関連しているのならば構造体やクラスで関連したパラメータを一つに纏める。
# classの使い方
クラスとは関数を纏めるためのものではない。
関数を纏めるだけならnamespaceで良い。というかnamespaceの方が良い。
privateにしたいならunnamed namespaceを使う。
一つのクラスから一つのインスタンスしか作らない場合や、
本来ならそのクラス自身が持つべきデータ(コントローラの場合、送られてきたデータ、オムニホイールの場合モータのピン番号)をメソッドに渡している場合は注意。

# 例
## ステアリング制御部
```c++
#include <stdint.h>
#include <math.h>

template<typename T>
class Vec {
public:
	T x, y;

	Vec(T x, T y): x(x), y(y) {}
	Vec(): x(0), y(0) {}

	// 内積
	T dot(Vec<T> v) {
		return x * v.x + y * v.y;
	}

	// ノルム、ベクトルの大きさ
	float norm() {
		return sqrt(x*x + y*y);
	}

	// スカラ倍
	template<typename S>
	Vec<T> scale(S s) {
		return Vec<T>(x * s, y * s);
	}

	Vec<T> add(Vec<T> v) {
		return Vec<T>(x + v.x, y + v.y);
	}

	float angle() {
		return atan2(y, x);
	}
};

class PolarVec {
public:
	float r, theta;

	PolarVec(float r, float theta): r(r), theta(theta) {}
	PolarVec(): r(0), theta(0) {}

	// 角を0~2πに
	PolarVec normalize() {
		if (theta < 0) {
			auto times = static_cast<int>(ceil(-theta / (2 * M_PI)));
			return PolarVec(r, theta + M_PI * 2 * times);
		}
		else if (theta > M_PI * 2) {
			auto times = static_cast<int>(floor(theta / (2 * M_PI)));
			return PolarVec(r, theta - M_PI * 2 * times);
		}
		else {
			return PolarVec(r, theta);
		}
	}

	// 角を0~πに
	PolarVec constraintHalf() {
		auto normalized = normalize();
		if (normalized.theta > M_PI) {
			return PolarVec(-normalized.r, normalized.theta - M_PI);
		}
		return normalized;
	}
};

class Stear {
public:
	// (x, y)
	Vec<float> mountingVectors[4];
	// (r, θ)
	PolarVec outputs[4];
	PolarVec realOutputs[4];
	float servoAngles[4];

private:
	// 理論値の計算(outputsの更新)
	void updateOutputs(Vec<float> moveVector, int16_t rotate) {
		Vec<float> outputVectors[4]{};
		float outputCalcMax = 0.0;
		// 移動ベクトルと回転ベクトルを加算して回転ベクトルに変換
		for (uint8_t i = 0; i < 4; ++i) {
			auto outputVector = moveVector.add(mountingVectors[i].scale(rotate));
			auto norm = outputVector.norm();
			outputCalcMax = fmax(outputCalcMax, norm);
			outputs[i] = PolarVec(
				norm,
				outputVector.angle()
			);
		}
		// 移動ベクトルの最大値と回転ベクトルの最大値のどちらか大きな方をmaxとして出力を補正
		auto moveNorm = moveVector.norm();
		auto rotateAbs = abs(rotate);
		auto outputMax = fmax(moveNorm, rotateAbs);
		for (uint8_t i = 0; i < 4; ++i) {
			outputs[i] = PolarVec(
				outputs[i].r * outputMax / outputCalcMax,
				outputs[i].theta
			);
		}
	}

	// 実際の出力値の計算
	void updateRealOutputs(Vec<float> moveVector, int16_t rotate) {
		updateOutputs(moveVector, rotate);
		// 出力をサーボの回転角に合わせる。
		for (uint8_t i = 0; i < 4; ++i) {
			realOutputs[i].r = outputs[i].r;
			realOutputs[i].theta = outputs[i].theta - servoAngles[i];
			realOutputs[i] = realOutputs[i].constraintHalf();
		}
	}

	// 出力値の反映(未実装)
	void reflectOutputs() {
		// outputsのxを出力値、yを角度として計算する。
	}

public:
	Stear(float mountingAngles[4], float _servoAngles[4]): mountingVectors{}, outputs{}, servoAngles{} {
		for (uint8_t i = 0; i < 4; ++i) {
			// 取り付け角から取り付けベクトルを計算。
			// 取り付け角へと向かうベクトルと垂直になるベクトルを計算すれば良い
			mountingVectors[i] = Vec<float>(
				-sin(mountingAngles[i]),
				cos(mountingAngles[i])
			);
		}
		for (uint8_t i = 0; i < 4; ++i) {
			servoAngles[i] = _servoAngles[i];
		}
	}

	void drive(Vec<float> moveVector, float rotate) {
		updateRealOutputs(moveVector, rotate);
		reflectOutputs();
	}

	void driveByPolar(PolarVec moveVector, float rotate) {
		drive(Vec<float>(
				moveVector.r * cos(moveVector.theta),
				moveVector.r * sin(moveVector.theta)
			),
			rotate
		);
	}
};
```
