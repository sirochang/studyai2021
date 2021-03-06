# 概要
「機械学習」について章ごとに要点をまとめる。

# 序章
機械学習モデリングプロセスは下記の通りで、実業務であれば前半の1~3にコストがかかる。
1. 問題設定：機械学習を使う必要があるか、ルールベースを用いて解決できないかなど
2. データ選定：機械学習のモデルを作成する上で適したデータになっているか、データに偏りやバイアスがかかっていないかなど
3. データの前処理：欠損値や異常値などを除外したり修正する
4. 機械学習モデルの選定→教師あり学習と教師なし学習でそれぞれ代表的な選定方法がある
5. モデルの学習（パラメータの推定）→教師あり学習と教師なし学習でそれぞれ代表的な推定方法がある
6. モデルの評価→教師あり学習の代表的な評価方法がある、教師なし学習は一般的にはモデルの評価は行わない

* 教師あり学習
  * 回帰：線形回帰モデル、非線形回帰モデル（、SVM）
  * 分類：ロジスティック回帰、k近傍法、SVM
* 教師なし学習
  * クラスタリング：k-means
  * 次元圧縮など：主成分分析

# 第1章 線形回帰モデル
* 直線（2次元）の方程式： y = A * x + B
* 平面（3次元）の方程式： z = A * x + B * y + C
* 超平面（n次元）の方程式： y = a_0 * x_0 + a_1 * x_1 + a_2 * x_2 ... a_n-1 * x_n-1  （ただしx_0 = 1）

a_0 ~ a_n-1をまとめたベクトルを**a**、x_0 ~ x_n-1をまとめたベクトルを**x**とすると
n次元の方程式は**a**と**x**の内積をとった形で表される。

回帰問題とは「ある入力(離散あるいは連続値)から出力(連続値)を予測する問題」である。

回帰を使ってランキング予測のようなこともできるがあまりオススメはできない（**Vapnikの原理**；ある問題を解くとき、その問題よりも難しい問題を途中の段階で解いてはならない）

入力をm次元のベクトルとする（パラメータがm次元）。また、教師あり学習なのでn個のxとyのペアが与えられる。

超平面の方程式は1つの方程式だったが、線形回帰モデルではn個の連立方程式を解くことを考える。

連立方程式の解き方からわかるように、一般的に n>m+1 でないと解くことはできない（パラメータ数よりデータ数を多く用意する必要がある）
```
y(n,1;次元) = X(n,m+1;次元) * w(m+1,1;次元) + ε(n,1;次元）
```

データとモデル出力の二乗誤差の和である平均二乗誤差を最小するパラメータを算出する。（最小二乗法）
ただし最小二乗法は外れ値に影響を受けやすいなど欠点もあるので注意する。

行列関連の公式としてmatrix cookbookが参考になる。

## 実装
演習を実行したgoogle colaboratoryのURL：https://colab.research.google.com/drive/1GetzrGgiYbQIH7FpsLVGKfmWMXlbh8IK?usp=sharing

# 第2章 非線形回帰モデル
複雑なモデルを考える場合は線形回帰モデルではなく非線形回帰モデルで考える。

## 基底展開法
基底関数と呼ばれる既知の非線形関数とパラメータベクトルの線型結合を用い、未知のパラメータは線形回帰モデルと同様に最小二乗法や最尤法により推定する。
* 非線形関数により、説明変数を全て新たな空間に写像する
```
y_i = ω_0 + ω_1 * φ_i(x_i1, x_i2, ..., x_im) + ... + ω_n * φ_n(x_i1, x_i2, ..., x_im) + ε_i
```
* 写像した説明変数とパラメータベクトル同士は**線形結合**となる

yの推定量の式は線形回帰モデルの時とほぼ同じように、説明変数の行列を非線形関数の計画行列で表せる。

## 未学習と過学習
* 未学習：学習データに対して、十分小さな誤差が得られないモデル
* 過学習：小さな誤差は得られたけど、テスト集合誤差との差が大きいモデル

過学習の対策として、学習データの数を増やしたり不要な基底関数を削除したりする方法もあるが、実現できないこともある。別の方法として正則化項（罰則化項）を設ける方法を考える。

重みが無制限であれば、より複雑なモデルを学習できるが過学習に陥りやすい。そのため、重みに制限を設けて、その制限下でモデルを学習させる。
* L2ノルム（Ridge推定量）：縮小推定、パラメータを0に近づけるよう推定
* L1ノルム（Lasso推定量）：スパース推定、いくつかのパラメータを0に推定

## 汎化性能
学習データだけではなく、未知のデータに対する予測性能。学習データとは別に用意した検証データを用いて検証する。

バイアスとバリアンスはトレードオフであり、未学習にならないようある程度は複雑なモデルを考えなければいけないが、過学習にならないように注意する。

汎化性能を検証する方法としてホールドアウト法があるが、大量の学習データが必要であり、かつ検証データを選択する際に特徴的なデータが偏らないよう注意する必要がある。別の方法としてクロスバリデーション（交差検証）がある。

モデルの学習・評価に使用するデータを学習データと検証データに分けるのはホールドアウト法と同じだが、クロスバリデーションでは全てのデータを一度は検証データとして扱うのでデータの偏りがあった際も検証できる。
あるモデルに対してクロスバリデーションをn回（n個のグループに分けている）した際、検証データを用いた各回の**精度の平均値**をそのモデルの精度として評価する。

## ハイパーパラメータの探索（グリッドサーチ）
学習モデルに採用するアルゴリズムによっては重みや説明変数の他にハイパーパラメータと呼ばれる値があり、この値によっても精度が変動するのでどの値が良いかを決めなければならない。

ただ、ハイパーパラメータが取りうる実数値を全て探索することは現実的に不可能なので、一定の間隔ごと（指数など）に値を決めて探索する。

アルゴリズムによってはハイパーパラメータの数が複数存在することもあるので、最適な評価を得られるパラメータにチューニングする必要がある。

## 実装
演習を実行したgoogle colaboratoryのURL：https://colab.research.google.com/drive/1CaKmLx71vtYoy06FZRIBmZQ8Xp-a3i68?usp=sharing

修正して実行した点：
* Lassoのパラメータalpha：10000（ほぼ直線） -> 0.001（だいたいRidge推定時と同じ）
* svm.SVRのパラメータgamma：0.1（ほぼ二次関数） -> 1（だいたいRidge推定時と同じ）
* KerasRegressionのパラメータbatch_size：5（val_lossが下がり切らず、真値とそこそこ誤差がある） -> 1（あまり変わらず）

# 第3章 ロジスティック回帰モデル
## 分類問題
まず、分類問題として大きく3つのアプローチがある。
* 識別的アプローチ：データがどのクラスに属するかの確率をモデル化する（ロジスティック回帰など）
* 生成的アプローチ：ベイズの定理を用いてモデル化する。外れ値の検出ができたり、新たにデータを生成することができる
* 識別関数：データがどのクラスに属するかを関数化する（SVMなど）

識別的アプローチは、データがどのクラスに属するかが確率で表されるので、クラス選定の条件の閾値を変えたり、確率だけ出してクラス選定を保留にすることもできる。

## ロジスティック回帰モデルの考え方
学習のベースは線形回帰モデルと同じだが、求めたい値が実数全体ではなく0〜1なので（二値分類）、最後にsigmoid関数を入れて実数全体の出力を0〜１の範囲に絞っている。

回帰モデルのパラメータは最小二乗法により推定したが、**分類のロジスティック回帰モデルのパラメータは最尤法により推定**する。

## パラメータ推定（最尤法）
それぞれの入力データに対する出力データを得る確率分布としてベルヌーイ分布を用いる。

分類モデルでは与えられた入力データから出力データを生成するような尤もらしいパラメータを推定する為に、尤度関数を考える。

それぞれのデータはお互いに独立なので、各ベルヌーイ分布の積が尤度関数となり、最大になるようにパラメータを推定する。

実際に推定する際には、尤度関数の対数のマイナスをかけたものを最小化することを考える。（尤度関数は確率を掛け合わせたものなので値がかなり小さくなるので、対数にすることによって桁落ちを回避できる）

## パラメータ学習法
回帰モデルでは最小二乗法によりパラメータの最適解を計算式で求めることができたが、分類モデルでは最尤法を用いてもパラメータの最適解を計算式で求めることが難しい。

その為、反復学習によって徐々にパラメータを最適解に近づけていく方法を考える。
* 勾配降下法：毎回、入力データ全てを使ってパラメータを更新する
* 確率的勾配降下法：毎回、データを1つずつランダムに選んでパラメータを更新する
* ミニバッチ勾配降下法：毎回、入力データのいくつかを使ってパラメータを更新する

## モデルの評価
データの偏りがあったり、極端なモデル（二値分類だとすると片方のクラスに全て分類する、50%の確率でどちらかに分類するなど）を考えれば正答率は上がるかもしれないが、実際に欲しいモデルとは異なる（正答率が高いから良いモデルとは限らない）。

クラス1をtrue,クラス0をfalseした際、クラス1と正しく分類できた数を**TruePositive**、クラス0を正しく分類できた数を**TrueNegative**としてこれらを出来るだけ増やすように、かつクラス1のデータを誤ってクラス0と分類した数を**FalseNegatibe**、クラス0のデータを誤ってクラス1と分類した数を**FalsePositive**としてこれらを出来るだけ減らすようにモデルを学習しなければならない。

* 適合率（Precision）：見逃しが多くてもより正確な予測をしたい場合に見るべき指標。 TP/(TP+FP)
* 再現率（Recall）：誤りが多少多くても抜け漏れが少ない予測をしたい場合に見るべき指標。 TP/(TP+FN)
* F値：PrecisionとRecallの調和平均

## 実装
演習を実行したgoogle colaboratoryのURL：https://colab.research.google.com/drive/1KkVdJsYjPXrWY45h6T9ZNsPITFwAZvU4?usp=sharing

修正して実行した点：
* 2変数から生死を判別するモデルで、2変数とsurvivedを表す図
  * コメントアウトされていたZも表示
  * (x1,y1)~(x2,y2)の直線がZ（sigmoid関数の結果）の境界値を表すことを確認


# 第4章 主成分分析
次元圧縮法の教師なし学習で、多変量データの持つ構造をより少数個の指標に圧縮する。ただし個数を少なくすることによる情報の損失は出来るだけ抑えたい。

例えば1000個の変数が与えられてモデルを学習する際、1000個の変数全てを学習に使用しても良いが、取り除いても学習モデルにはさほど影響しない変数もあり、より効果的な変数（やその組み合わせ）を用いてモデルを学習すれば良い。各変数が持つ情報量を分散の大きさとして捉えると、線形変換後の変数の分散が最大となる射影軸を探索する。

**ラグランジュの未定乗数法**によって制約付き最適化問題を解くと0以上になる固有値と互いに直行する固有ベクトル（主成分）を得ることができる。元のデータの総分散と主成分の総分散は等しくなるので、少ない個数の主成分で元のデータの総分散に近づくように主成分を選択すれば、元のデータより少ない変数でかつ情報の損失も抑えれることになる。

## 実装
演習を実行したgoogle colaboratoryのURL：https://colab.research.google.com/drive/1FOCBbjyGfAErg4_qCQyXD7fJ2seV_XE-?usp=sharing

修正して実行した点：
* 第2主成分まででモデルを学習し検証、検証スコアが93%となることを確認
  * 元々は次元数33で検証スコアが97%なので、次元数をかなり減らしつつスコアもそれほど下がっていないことがわかる

# 第5章 アルゴリズム
## k-means（k-平均法）
クラスタリングの教師なし学習で、与えられたデータをk個のクラスタに分類する問題。

アルゴリズム
1. 各クラスタ中心の初期値を設定する
2. 各データ点に対して、各クラスタ中心との距離を計算し、最も距離が近いクラスタを割り当てる
3. **各クラスタの平均ベクトル（中心）を計算する**
4. 収束するまで2,3 の処理を繰り返す

特徴
* k個のクラスタに分類するので、kの値を変えるとクラスタリング結果も当然変わる
* 初期値依存があり、距離が近い点を初期値として与えてしまうとうまくクラスタリングできない

## k近傍法
分類問題を解くための手法の教師あり学習。

新しいデータを分類する際、最近傍のデータをk個取ってきて、それらが最も多く所属するクラスに分類する。

特徴
* k個の近傍データを使うので、kの値を変えると分類結果も当然変わる
* kを大きくすると、決定境界が滑らかになる

## 実装
演習を実行したgoogle colaboratoryのURL：https://colab.research.google.com/drive/1A_-iCQNLufOhNJoHQ6sgKIG08_r9FMwC?usp=sharing

修正して実行した点：
* sklearnのみでの確認ですが、cancerデータを用いて KNeighborsClassifierで学習や識別を実行（k-meansはサンプル通り）

# 第6章 サポートベクターマシーン
## 概要
分類問題を解くための手法の教師あり学習。（回帰問題も解くことはできる）

線形判別関数と最も近いデータ点との距離をマージンと呼び、マージンが最大となる線形判別関数を求める（重みと切片）を求める。

## 分類器の解の求め方
目的関数が二次で、制約条件が線形である二次計画問題である。この問題を普通に解こうとすると変数の数が多く最適化に時間がかかるので、等価表現である**ラグランジュ双対問題**を考える。

KKT条件により、マージン上にあるデータは予測に影響を与えることになり、これをサポートベクトルと呼ぶ。

## ソフトマージン
線形分離できない時、ある程度誤差を許容して分類を試みることができるが、誤差に対してはペナルティを与えて制約条件にも加える。

マージンの大きさと許容誤差はトレードオフになり、パラメータを変動すると判別関数も変わる。

## 非線形化
ソフトマージンを考慮しても線形分離できない場合は、非線形な分離を考える。

与えられたデータの次元より高次元な空間に写像してやれば分離は可能になるが、計算コストも増えてしまう。

そこで**カーネルトリック**と呼ばれる手法を用い、高次元ベクトルの内積をスカラー関数で表現することにより、特徴空間が高次元でも計算を抑えることができる。

RBFカーネルやガウシアンカーネルを用いることが多い。

## 実装
演習を実行したgoogle colaboratoryのURL：https://colab.research.google.com/drive/1ZnVADmzfzNdPTyot-1tFaM6bqyGMWZCv?usp=sharing

修正して実行した点：
* マージンと決定境界を可視化後、サポートベクトルの数値を出力し、図の点（黒丸囲み）と一致することを確認
