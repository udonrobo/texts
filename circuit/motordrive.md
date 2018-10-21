# モータードライバとは
マイコンからの電気信号でモーターを制御する回路。
マイコンから流せるのはせいぜい5 V、100 mAとかなので24 V、20 A等を要求するモーターは当然動かせない。

# 制御方式
## PWM
ONとOFFを高速で切り替えて電圧の平均値を変化させることで出力を制御する方式。
少電流ならば可変抵抗を直列に繋ぐのでも良いが、
大電流となると可変抵抗部にもそれなりの電圧と相当の電流が流れることになり損失が非常に大きくなる。
抵抗の場合は損失は全て熱となるので最悪燃える。
PWMだとON時は抵抗値がほぼ0の為電圧が制御部に殆どかからず損失が少なく、OFF時は抵抗値が非常に大きいので電流が小さく損失が少ない。
## Hブリッジ
Hの縦棒の4箇所にスイッチ、中央の横棒にモーターを配置する回路。
上部を正極に、下部をGNDに接続する。
対角の位置のスイッチだけをONにすると正極からGNDにモーターを通して電流が流れ、逆の対角を取ると逆向きに電流が流れる。
上部だけ、もしくは下部だけをONにするとモーターと導線のみの閉回路となり、モーターがダイナモになるのでブレーキが掛かる。
素子は殆どMOSFETが使われる。BJTやリレーはスイッチング速度が遅く、耐電流も低い。
## 形式
素子によって幾つか種類があり、それぞれ利点と欠点がある。
最も高性能なのはNchフルブリッジ
### Pch-Nch混合
NchMOSFETはゲート電位がソース電位よりも高く、
PchMOSFETはゲート電位がソース電位より低くなればいいのでハイサイドにPch、ローサイドにNchを使う。
欠点としてはPchMOSFETは一般的にNchMOSFETより性能が低いのでNchMOSFETのポテンシャルを活かせなくなること。
### リレー-Nch混合
あまり見たことがない。ローサイドをNchMOSFETでPWMでスイッチングさせ、ハイサイドはリレーを使う。
リレーは壊れやすく接点が溶ける場合もある上かなり重い。設計班の迷惑だしあんまり利点が無い。
### Nchフルブリッジ
Pch-Nch混合で述べたようにNchはゲート電位がソース電位より高くしなければならないので、
どうにかしてモータの電源より高電圧を出せる電源を用意する。
モータドライバの回路内にチャージポンプを入れるブートストラップ方式と外部にチャージポンプを用意する方法の2種類がある。
ブートストラップ方式を使って自作する場合、ハイサイドとローサイドをドライブ出来るゲートドライバがIR(現在はInfineon)などから出ているのでそれを使うと良い。
ブートストラップ方式の場合はPWM周波数を100%にすると充電が追いつかなくなって止まる。
外部から電圧を供給すればその問題はなくなる。
NHKのルールでは大体電源24Vまで回路内電圧最大48Vまでになっている。

## 信号形式
PWM, INA(IN1), INB(IN2)で制御する方式とPWM, DIRで制御する方式がある。
### 3線式
PWMで比率を変え、INA, INBでブレーキ、正転、逆転、フリーを切り替える。
#### 利点
ブレーキにPWMが適用できる。0ならフリー、255ならフルブレーキ。
ただしこれはモータードライバの設計によるので、実際にそうなっているかは分からない。
今部室にあるCNCで製造するタイプのモータードライバにはフリー機能はない。
#### 欠点
線が多い。LAP制御で使うには論理回路を外付けで組む必要がある。
### 2線式
PWMで比率を変え、DIRで方向を指定する。ブレーキはPWMを0にする。
#### 利点
線が少ない。DIRとPWMを逆に繋ぐとLAP方式の制御になる。
#### 欠点
フリーが無い。

# 既成品
|名前|URL|価格|特徴|
|:--:|:-:|:--:|:--:|
| Cytron GROVE互換DCモータドライバ| https://www.switch-science.com/catalog/3606/ | 1283 | 安くて高機能|
| BTS7960 | https://www.aliexpress.com/premium/HiLetgo-BTS7960.html?SearchText=HiLetgo+BTS7960&d=y&tc=ppc&initiative_id=SB_20181020201420&origin=y&catId=0&isViewCP=y | $5 | とにかく安くてヒートシンク付き|