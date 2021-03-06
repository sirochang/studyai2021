# Section1：勾配消失問題
```
【確認テスト】
連鎖律の原理を使い、dz/dxを求めよ
・z = t^2
・t = x+y

dz/dt = 2t
dt/dx = 1
dz/dx = (dz/dt)*(dt/dx) = (2t)*1 = 2*(x+y)
```

## 全体像
勾配消失問題とは、誤差逆伝播法が下位層に進んでいくにつれて勾配がどんどん緩やかになっていき、勾配降下法による更新では下位層のパラメータはほとんど変わらず、訓練が最適値に収束しなくなることである。例えば活性化関数にシグモイド関数を使用した時を例に考える。

シグモイド関数は0~1の間を緩やかに変化する関数で、信号の強弱を伝えられることからNN普及のきっかけとなったが、大きな値では出力の変化が微小なため（微分値が小さいため）勾配消失問題を引き起こすことがあった。

```
【確認テスト】
シグモイド関数を微分した時、入力値が0の時に最大値をとる。その値として正しいものを選択肢から選べ。
（1）0.15（2）0.25（3）0.35（4）0.45

sig(x)の微分sig'(x)は sig(x)*(1-sig(x)) であり、sig(x)の微分値はx=0の時に最大となる。
sig(0)=0.5なので、sig'(0)=0.5*(1-0.5)=0.2
よって答えは(2)0.25である。
```

## 1-1.活性化関数
様々な派生系はあるものの、現在もっとも使われている活性化関数がReLU（Rectified Linear Unit）関数である。

x>0の範囲ではf(x)=xなので勾配消失問題も回避できることに加え、x<=0の範囲ではf(x)=0なのでスパース化に貢献できる。

## 1-2.初期値の設定方法
重みの初期値をどう与えるべきかについて考える。ここでは活性化関数の出力を確認し、0~1の値を極端な偏りが無いことや各層で出力の分布が異なることが望ましい。

* 標準偏差1のガウス分布：自然界の一般的な分布としてもっともよく用いられる
  * 活性化関数の出力は各層で同じような結果となり、0と1に偏る
* 標準偏差0.01のガウス分布
  * 活性化関数の出力は各層で同じような結果となり、0.5付近に偏る
* Xavierの初期値：前の層のノード数の平方根の逆数の標準偏差を持つ分布
  * 活性化関数の出力は各層で出力の分布が異なり、0~1の値を極端な偏りが無い
  * 活性化関数が線形であることを前提に導かれており、左右対称で中央付近が線形関数としてみなせるsigmoid関数やtanh関数に適している
* Heの初期値：前の層のノード数の平方根の逆数に2の平方根をかけた標準偏差を持つ分布
  * 活性化関数の出力は各層で出力の分布が異なり、0~1の値を極端な偏りが無い
  * ReLU関数はx<=0でf(x)=0なので、Xavierの初期値の考え方＋係数を倍にするような解釈になる

```
【確認テスト】
重みの初期値を0に設定すると、どのような問題が発生するか。

全ての重みの値が均一に更新されるため、多数の重みを持つ意味がなくなる。
```

## 1-3.バッチ正規化
ミニバッチ単位で、入力値のデータの偏りを抑制する手法で、活性化関数に値を渡す前後にバッチ正規化の処理の層を追加する。

```
【確認テスト】
バッチ正規化の効果を挙げる

ミニバッチ単位で入力値のデータの偏りを抑制できるので、学習が安定して高速化が望める。
また初期値にあまり依存しなくなり、過学習を抑制することができる。
```

数式的にはミニバッチ全体から求めた平均と標準偏差を用いて各入力値に標準化を行い、さらにバッチ正規化のオペレーションとしてスケーリングやシフトを施す。

```python
# 【例題】 ミニバッチアルゴリズムのループ内での入力値の与え方

# 0~Nまでをiをbatch_size分増加しながらループ
for i in range(0, N, batch_size):
  i_end = i + batch_size
  # pythonのスライス機能を使う
  batch_x, batch_t = data_x[i:i_end], data_t[i:i_end]
  _update(batch_x, batch_t)
```

## 実装
演習を実行したgoogle colaboratoryのURL：
* 2層のNN構築：https://colab.research.google.com/drive/1MxhvBGocTveFrZO5k7nXqA8aK1FxFBcw?usp=sharing
  * numpyのchoiceとsampleについて実行確認
  * ループが前回選択したデータも再び選択候補に含まれることを確認
* 活性化関数とXavier、Heの初期値：https://colab.research.google.com/drive/1urDwJfDM4ByeL5V1oGWFl4oxV9sItnGX?usp=sharing
  * 活性化関数と初期値の与え方による学習結果の違いについて確認
* バッチ正規化：https://colab.research.google.com/drive/1BF_zduMprXiIa5SM-ZKui54mTuAntwtP?usp=sharing

# Section2：学習率最適化手法
## 全体像
深層学習の目的は学習を通して誤差を最小にするネットワークを作成することで、勾配効果法を利用してパラメータを最適化する。

パラメータの最適化には学習率が大きく関わる。
* 学習率の値が大きい場合
  * 最適値にいつまでもたどり着かず発散してしまう 
* 学習率の値が小さい場合
  * 発散することは無いが、収束するまでに時間がかかってしまう
  * 大域局所最適解に収束しづらくなる

初期の学習率設定方法の方針としては下記の2つである。
* 初期の学習率を大きく設定し、徐々に学習率を小さくしていく
* パラメータ毎に学習率を可変させる

## 2-1~2-4.モメンタム/AdaGrad/RMSProp/Adam

```
【確認テスト】
学習率最適化手法の特徴をまとめよ。

下記の表にまとめる。
```

|  学習率最適化手法  |  概要  | メリット | 備考 |
| ---- | ---- | ---- | ---- |
|  勾配降下法  |  ・誤差をパラメータで微分したものと学習率の積を減算する | - | - |
|  モメンタム  |  ・誤差をパラメータで微分したものと学習率の積を減算する<br>・現在の重みに前回の重みを減算した値と慣性の積を加算する | ・局所最適解にならず大域的最適解となる<br>・谷間についてから最適値にいくまでの時間が早い | - |
|  AdaGrad  |  ・誤差をパラメータで微分したものと、再定義した学習率の積を減算する | ・勾配の緩やかな斜面に対して最適値に近づける | 学習率が徐々に小さくなるので、鞍点問題を引き起こす |
|  RMSProp  |  ・誤差をパラメータで微分したものと、再定義した学習率の積を減算する<br>前回の学習率の変化と誤差微分の割合を変えれる | ・局所最適解にならず大域的最適解となる<br>・ハイパーパラメータの調整が楽 | - |
|  Adam  |  ・モメンタムとRMSProp両方の概要を含む<br>　・モメンタム：過去の勾配の指数関数的減衰平均 | ・モメンタムとRMSPropのメリットを含む | - |

## 実装
演習を実行したgoogle colaboratoryのURL：https://colab.research.google.com/drive/1rCyQEvaW67C9Zbr0vmS3GFRAzzALaKFF?usp=sharing

# Section3：過学習
## 全体像
過学習とは、テスト誤差と訓練誤差で学習曲線が乖離すること（特定の訓練サンプルに対して、特化して学習する）。

原因はパラメータの数が多い、パラメータの値が適切でない、ノードが多いなどが考えられ、つまりネットワークの自由度（層数、ノード数、パラメータの値etc）が高いことである。
正則化とは、ネットワークの自由度を制約することである。これによって過学習を抑制する。

過学習の原因としては重みが大きい値を取ることである。よって解決策としては、誤差に対して正則化項を加算することで、重みが大きい値を取りすぎないように重みを抑制する。

## 3-1.L1正則化、L2正則化
誤差関数にpノルムという正則化項を加える。p=1の場合はl1ノルム、p=2の場合はl2ノルムと呼ぶ。
* l1ノルム：重みの絶対値の和
* l2ノルム：重みのユークリッド距離のようなもの（を2で割る）

```
【確認テスト】
l1正則化を表しているグラフは？

スパース性があることが特徴なので、右側のグラフになる
```
```
【例題チャレンジ】
l2正則化を適用した時に、最終的な勾配の更新式に当てはまるものはどれか

l２ノルムは||param||^2なので、その勾配が誤差の勾配に加えられる。l2ノルムの勾配は2*paramになるが、係数2は正則化の係数に吸収されても変わらないので、paramが正解となる
```
```
【例題チャレンジ】
l１正則化を適用した時に、正則化項の勾配に当てはまるものはどれか

l1ノルムは|param|なので、その勾配が誤差の勾配に加えられる。l1ノルムの勾配は符号関数（sign（param))になる
```


## ドロップアウト
NNのノード数が多いとネットワークの自由度が上がり、過学習に陥りやすい。ドロップアウトでは、NNのノード数をランダムに削除して学習させることである。

データ量を変化させなくても、異なるモデルを学習させていると解釈できるのがメリットである。

## 実装
演習を実行したgoogle colaboratoryのURL：https://colab.research.google.com/drive/1IOP8GF0BufbZ6cfjU6EBSelOAYImzxw9?usp=sharing

* 下記を確認
  * L2正則化は過学習を抑えれている
  * L1正則化はうまく学習できず、Dropoutは学習が遅い
  * L1正則化とDropoutを組み合わせると、過学習を抑えながら学習を早く進めることができる

# Section4：畳み込みニューラルネットワークの概念
## 全体像
CNNでは次元間でつながりのあるデータをうまく捉えることができる。もし全結合層のみを用いたNNでは、次元間の繋がりをう考慮せずに学習することになる。

CNNでは特徴を抽出する部分である畳み込み層とプーリング層、そしてそれらをまとめて最終的な出力を導くための全結合層に分かれる。畳み込み層では次元間の繋がりを捉えた特徴を抽出しつつ、複数のフィルタを用意することによって異なる視点でも抽出して学習に用いる。

## 畳み込み層
画像の場合、縦・横・チャンネルの三次元のデータをそのまま学習し、次に伝えることができる。つまり、三次元の空間情報も学習できるような層が畳み込み層である。
入力データと重み（フィルター）の演算結果を出力とするが、この重みが学習対象となる。

### バイアス
入力データと重み（フィルター）の演算結果に対して、要素ごとに一定の値分増減させるもの。このバイアスも学習対象となる。

### パディング
畳み込みの演算をすると元の画像よりも小さい結果が出力されることになり、畳み込み層を重ねる回数が増えるとデータがどんどん小さくなってしまう。これを避けるために入力画像の周りに新たなマスを追加して元のデータと合わせて畳み込み演算を行う。

新たなマスには0を入れたり、隣接するマスと同じ値を入れたりする方法がある。また、新たに追加するマス数は1固定ではなく、2マス以上追加することもできる。

### ストライド
入力データとフィルターの畳み込み演算を進める際に、何マス移動させるかの値。1固定ではなく、2マス以上移動させることもできる。

### チャンネル
例えば入力データが画像の場合は縦横情報に加えてチャンネル情報も存在する（RGBなど）。複数チャンネルの入力データとフィルターの畳み込み演算をする際、入力データのチャンネル数分フィルターも用意し、対応するチャンネル同士で畳み込み演算を行い、その結果を最終的に1チャンネルの情報として特徴を抽出する。

入力データのチャンネル数に合わせてフィルタのチャンネル数を用意するのとは別に、畳み込み層の結果として複数チャンネルのデータを出力する場合は、入力データのチャンネル数 * 出力データのチャンネル数分のフィルタを用意することになる。

## プーリング層
入力データと重みの畳み込み演算のような複雑な計算は行わず、単純に対象領域のMax値や平均値を特徴として抽出する層。

## 出力サイズ
パディングとストライドに対して出力サイズを計算する。
* 入力サイズ：（H、W）
* フィルターサイズ：（FH、FW）
* パディング：P
* ストライド：S
* 出力サイズ
  * OH = {(H + 2P - FH) / S} + 1
  * OW = {(W + 2P - FW) / S} + 1

```
【確認テスト】
入力サイズ(6、6)、フィルターサイズ(2、2）、ストライドとパディングが１の時の出力サイズは？

公式より
{6 + 2*1 - 2) / 1} + 1 = 7
OH、OW共に7
```
```
【確認テスト】
入力サイズ(5、5)、フィルターサイズ(3、3）、ストライドが2、パディングが１の時の出力サイズは？

公式より
{5 + 2*1 - 3) / 2} + 1 = 3
OH、OW共に3
```

## 実装
演習を実行したgoogle colaboratoryのURL：
* 単層のCNN：https://colab.research.google.com/drive/1rhysNAc4Nl1e12-Wn_dwRHfl2dM1VJ3g?usp=sharing
* 二層のCNN：https://colab.research.google.com/drive/1kRFrFDQYoXSV6638DoX_Vt1omtaTOSvj?usp=sharing
  * 100回しか学習していないので、まだ学習は完了していない様子
  * 単層のCNNと比べても、やはり学習に時間がかかりそう（同じ100回目の時点で正答率が1層のCNNより低い）

# Section5：最新のCNN
* LeNet：1998年にヤン・ルカンによってモデルが考えられる。
  * 畳込層→PL層→畳込層→PL層→全結合層→全結合層という構成
* AlexNet：2012年のILSVRCで圧倒的な精度を誇ったモデル
  * 畳込層→PL層→畳込層→PL層→畳込層→畳込層→畳込層→PL層→全結合層→全結合層→全結合層という構成
  * 活性化関数にReLUを用いたり、局所的正規化やDropoutを用いる

単純に層を深くしていくだけでは計算量が大きくなってしまうので、小さなサイズの畳み込みフィルターを差し込んで次元を削減する工夫が用いられた。
* VGG16、VGG19:畳み込み層と全結合層の数の合計が16や19個の構成
* GoogLeNet：inceptionモジュールにより並列計算を可能にしたが、層の数を増やしていくと誤差逆伝播しにくくなり学習がうまくいかなくなる問題があった
* ResNet：skip connectionと呼ばれる層を飛び越えた結合を加えることにより、誤差逆伝播がしやすくなっている。また、様々な形のネットワークのアンサンブル学習にもなっている

## 実装
演習を実行したgoogle colaboratoryのURL：https://colab.research.google.com/drive/1Q3gAtGThgN6JD2bycR0oI-Uk4sVD_5Gx?usp=sharing
