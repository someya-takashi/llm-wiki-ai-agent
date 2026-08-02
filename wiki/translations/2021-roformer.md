---
type: translation
source_path: raw/papers/RoFormer_ Enhanced Transformer with Rotary Position Embedding.md
source_page: "[[summaries/2021-roformer]]"
original_language: en
translated_to: ja
translated_at: 2026-08-03
---

# RoFormer: 回転位置埋め込みによる強化トランスフォーマ

> 原題: RoFormer: Enhanced Transformer with Rotary Position Embedding
> 著者: Jianlin Su, Yu Lu, Shengfeng Pan, Ahmed Murtadha, Bo Wen, Yunfeng Liu
> 所属: Zhuiyi Technology Co., Ltd., Shenzhen（追一科技、深圳）
> 出典: arXiv:2104.09864（ar5iv 版）/ 後に Neurocomputing 568 (2024) 127063 として出版

> **訳注（クリップの状態と復元）**
> - 底本は ar5iv 版の Web Clipper クリップ。**本文・図表キャプション・数式には欠落がなく**、Figure 1〜3・Table 1〜5 はすべて残っていた。
> - **画像 4 枚中 1 枚が欠落**していた（`x4`）。**Figure 3 は 2 パネル構成**（左: BERT と RoFormer の訓練損失、右: RoPE の有無による PerFormer の訓練損失）で、その右パネルが失われていた。本 wiki で繰り返し出ている「多パネル図の 2 枚目が落ちる」型である。原ページから復元した。
> - **脚注 2 件の本文が欠落**していた（`<sup>1</sup>` `<sup>2</sup>` のマーカーのみ残存）。原ページから復元して該当箇所に挿入した。
> - ar5iv は複数行にまたがる整列数式を `\displaystyle` 付きの独立した数式ブロックに分解して出力する。読みにくいので、**原典どおりの整列形（`aligned` 環境）に戻して**掲載した。式の内容には手を加えていない。
> - 原典自体に含まれる表記の揺れ・誤植は、本文をそのまま訳したうえで該当箇所に訳注として記した（原典の性質であり、クリップ不良ではない）。
> - 参考文献一覧は既定どおり訳出しない。式番号は ar5iv 版の付番に従う。

---

###### Abstract（要旨）

位置符号化（position encoding）は近年、トランスフォーマアーキテクチャにおいて有効であることが示されている。それは、系列の異なる位置にある要素間の依存関係をモデル化するための価値ある教師信号を可能にする。本論文では、まずトランスフォーマベースの言語モデルの学習過程に位置情報を統合するさまざまな手法を検討する。次に、位置情報を効果的に活用するための新しい手法である **Rotary Position Embedding（RoPE, 回転位置埋め込み）** を提案する。具体的には、提案する RoPE は絶対位置を回転行列で符号化すると同時に、明示的な相対位置の依存関係を self-attention の定式化に組み込む。注目すべきことに、RoPE は価値ある性質を可能にする。すなわち、系列長の柔軟性、相対距離の増大にともなうトークン間依存の減衰、そして線形 self-attention に相対位置符号化を装備する能力である。

*キーワード* 事前学習済み言語モデル $\cdot$ 位置情報の符号化 $\cdot$ 事前学習 $\cdot$ 自然言語処理

## 1 Introduction（はじめに）

単語の系列的な順序は自然言語理解にとって非常に価値がある。再帰型ニューラルネットワーク（RNN）にもとづくモデルは、時間方向に沿って隠れ状態を再帰的に計算することでトークンの順序を符号化する。畳み込みニューラルネットワーク（CNN）にもとづくモデル [^1] は一般に位置に無関係（position-agnostic）だと考えられてきたが、最近の研究 [^2] は、よく使われるパディング操作が暗黙的に位置情報を学習しうることを示している。近年、トランスフォーマ [^3] の上に構築された事前学習済み言語モデル（PLM）が、文脈表現学習 [^4]、機械翻訳 [^3]、言語モデリング [^5] をはじめとするさまざまな自然言語処理（NLP）タスクで最先端の性能を達成している。RNN や CNN ベースのモデルと異なり、PLM は self-attention 機構を用いて、与えられたコーパスの文脈表現を意味的に捉える。その結果として、PLM は RNN に対して並列化の点で顕著な改善を達成し、CNN と比べてより長いトークン間関係のモデリング能力を向上させる<sup>1</sup>。

> 訳注（脚注 1、原ページより復元）: 複数の CNN 層を積み重ねればより長いトークン内関係を捉えることもできるが、ここでは単層の設定のみを考える。

注目すべきことに、現在の PLM の self-attention アーキテクチャは位置に無関係であることが示されている [^6]。この主張を受けて、位置情報を学習過程に符号化するさまざまなアプローチが提案されてきた。一方では、あらかじめ定められた関数によって生成される絶対位置符号化 [^3] が文脈表現に加算され、他方では学習可能な絶対位置符号化 [^1] [^4] [^7] [^8] [^5] [^9] が用いられた。もう一方では、先行研究 [^10] [^11] [^12] [^13] [^14] [^15] [^16] [^17] [^18] が相対位置符号化に注目しており、これは典型的には相対位置情報を attention 機構の中に符号化する。これらのアプローチに加えて、[^19] の著者らは Neural ODE [^20] の観点から位置符号化の依存関係をモデル化することを提案し、[^21] の著者らは複素空間で位置情報をモデル化することを提案した。これらのアプローチが有効であるにもかかわらず、**それらは一般に位置情報を文脈表現に加算しており、そのために線形 self-attention アーキテクチャには適さない**。

本論文では、位置情報を PLM の学習過程に活用するための新しい手法である **Rotary Position Embedding（RoPE）** を導入する。具体的には、RoPE は絶対位置を回転行列で符号化すると同時に、明示的な相対位置の依存関係を self-attention の定式化に組み込む。提案する RoPE が既存手法より優先されるのは、系列長の柔軟性、相対距離の増大にともなうトークン間依存の減衰、線形 self-attention に相対位置符号化を装備する能力といった価値ある性質を通じてである。さまざまな長文分類ベンチマークデータセットでの実験結果は、回転位置埋め込みを備えた強化トランスフォーマ、すなわち **RoFormer** がベースラインの代替手法と比べてより良い性能を与えうることを示し、提案する RoPE の有効性を実証している。

要するに、我々の貢献は次の三つである。

- 相対位置符号化に対する既存のアプローチを調査し、それらの大部分が「位置符号化を文脈表現に加算する」という分解の考え方の上に構築されていることを見出した。我々は、位置情報を PLM の学習過程に活用するための新しい手法、Rotary Position Embedding（RoPE）を導入する。鍵となる考え方は、**文脈表現に回転行列を掛けることで相対位置を符号化する**ことであり、これには明快な理論的解釈がある。
- RoPE の性質を研究し、相対距離が増大するにつれて減衰することを示す。これは自然言語の符号化にとって望ましい性質である。また我々は、先行する相対位置符号化にもとづくアプローチが線形 self-attention と両立しないことを、慎ましく主張する。
- 提案する RoFormer をさまざまな長文ベンチマークデータセットで評価する。我々の実験は、それが一貫して代替手法より良い性能を達成することを示す。事前学習済み言語モデルを用いた実験のいくつかは GitHub で公開されている: [https://github.com/ZhuiyiTechnology/roformer](https://github.com/ZhuiyiTechnology/roformer)。

本論文の残りは次のように構成される。第 2 節で self-attention アーキテクチャにおける位置符号化問題の形式的な記述を確立し、先行研究を振り返る。次に第 3 節で回転位置符号化（RoPE）を記述し、その性質を研究する。第 4 節で実験を報告する。最後に第 5 節で本論文を結論づける。

## 2 Background and Related Work（背景と関連研究）

### 2.1 Preliminary（準備）

$\mathbb{S}_{N}=\{w_{i}\}_{i=1}^{N}$ を $N$ 個の入力トークンからなる系列とし、$w_{i}$ をその $i$ 番目の要素とする。$\mathbb{S}_{N}$ に対応する単語埋め込みを $\mathbb{E}_{N}=\{{\boldsymbol{x}}_{i}\}_{i=1}^{N}$ と表記する。ここで ${\boldsymbol{x}}_{i}\in\mathbb{R}^{d}$ は、位置情報を持たないトークン $w_{i}$ の $d$ 次元の単語埋め込みベクトルである。self-attention はまず位置情報を単語埋め込みに組み込み、それらを query、key、value の表現へと変換する。

$$
\begin{aligned}
{\boldsymbol{q}}_{m}&=f_{q}({\boldsymbol{x}}_{m},m)\\
{\boldsymbol{k}}_{n}&=f_{k}({\boldsymbol{x}}_{n},n)\\
{\boldsymbol{v}}_{n}&=f_{v}({\boldsymbol{x}}_{n},n),
\end{aligned} \tag{1}
$$

ここで ${\boldsymbol{q}}_{m},{\boldsymbol{k}}_{n},{\boldsymbol{v}}_{n}$ はそれぞれ $f_{q},f_{k},f_{v}$ を通じて $m$ 番目・$n$ 番目の位置を組み込む。query と key の値は次に attention の重みの計算に使われ、出力は value 表現に対する重み付き和として計算される。

$$
\begin{aligned}
a_{m,n}&=\frac{\exp(\frac{{\boldsymbol{q}}_{m}^{\intercal}{\boldsymbol{k}}_{n}}{\sqrt{d}})}{\sum_{j=1}^{N}\exp(\frac{{\boldsymbol{q}}_{m}^{\intercal}{\boldsymbol{k}}_{j}}{\sqrt{d}})}\\
\mathbf{o}_{m}&=\sum_{n=1}^{N}a_{m,n}{\boldsymbol{v}}_{n}
\end{aligned} \tag{2}
$$

トランスフォーマベースの位置符号化の既存アプローチは、主に式 (1) を形づくる適切な関数を選ぶことに焦点を当てている。

### 2.2 Absolute position embedding（絶対位置埋め込み）

式 (1) の典型的な選択は次である。

$$
f_{t:t\in\{q,k,v\}}({\boldsymbol{x}}_{i},i):={\boldsymbol{W}}_{t:t\in\{q,k,v\}}({\boldsymbol{x}}_{i}+{\boldsymbol{p}}_{i}), \tag{3}
$$

ここで ${\boldsymbol{p}}_{i}\in\mathbb{R}^{d}$ はトークン ${\boldsymbol{x}}_{i}$ の位置に依存する $d$ 次元ベクトルである。先行研究 [^4] [^7] [^8] [^5] [^9] は学習可能なベクトルの集合 ${\boldsymbol{p}}_{i}\in\{{\boldsymbol{p}}_{t}\}_{t=1}^{L}$ の使用を導入した。ここで $L$ は最大系列長である。[^3] の著者らは、正弦関数を用いて ${\boldsymbol{p}}_{i}$ を生成することを提案した。

$$
\begin{cases}{\boldsymbol{p}}_{i,2t}&=\sin(k/10000^{2t/d})\\
{\boldsymbol{p}}_{i,2t+1}&=\cos(k/10000^{2t/d})\end{cases} \tag{4}
$$

ここで ${\boldsymbol{p}}_{i,2t}$ は $d$ 次元ベクトル ${\boldsymbol{p}}_{i}$ の $2t$ 番目の要素である。次節では、提案する RoPE が正弦関数の観点からこの直観と関連していることを示す。ただし、**位置を文脈表現に直接加算するのではなく、RoPE は正弦関数を掛けることによって相対位置情報を組み込むことを提案する**。

### 2.3 Relative position embedding（相対位置埋め込み）

[^11] の著者らは、式 (1) について次のような異なる設定を適用した。

$$
\begin{aligned}
f_{q}({\boldsymbol{x}}_{m})&:={\boldsymbol{W}}_{q}{\boldsymbol{x}}_{m}\\
f_{k}({\boldsymbol{x}}_{n},n)&:={\boldsymbol{W}}_{k}({\boldsymbol{x}}_{n}+\tilde{{\boldsymbol{p}}}^{k}_{r})\\
f_{v}({\boldsymbol{x}}_{n},n)&:={\boldsymbol{W}}_{v}({\boldsymbol{x}}_{n}+\tilde{{\boldsymbol{p}}}^{v}_{r})
\end{aligned} \tag{5}
$$

ここで $\tilde{{\boldsymbol{p}}}^{k}_{r},\tilde{{\boldsymbol{p}}}^{v}_{r}\in\mathbb{R}^{d}$ は学習可能な相対位置埋め込みである。$r=\operatorname{clip}(m-n,r_{\text{min}},r_{\text{max}})$ は位置 $m$ と $n$ の間の相対距離を表すことに注意されたい。彼らは、**正確な相対位置情報はある距離を超えると有用でない**という仮説のもとで相対距離をクリップした。式 (3) の形を保ったまま、[^13] の著者らは式 (2) の ${\boldsymbol{q}}_{m}^{\intercal}{\boldsymbol{k}}_{n}$ を次のように分解することを提案した。

$$
{\boldsymbol{q}}_{m}^{\intercal}{\boldsymbol{k}}_{n}={\boldsymbol{x}}_{m}^{\intercal}{\boldsymbol{W}}_{q}^{\intercal}{\boldsymbol{W}}_{k}{\boldsymbol{x}}_{n}+{\boldsymbol{x}}_{m}^{\intercal}{\boldsymbol{W}}_{q}^{\intercal}{\boldsymbol{W}}_{k}{\boldsymbol{p}}_{n}+{\boldsymbol{p}}_{m}^{\intercal}{\boldsymbol{W}}_{q}^{\intercal}{\boldsymbol{W}}_{k}{\boldsymbol{x}}_{n}+{\boldsymbol{p}}_{m}^{\intercal}{\boldsymbol{W}}_{q}^{\intercal}{\boldsymbol{W}}_{k}{\boldsymbol{p}}_{n}, \tag{6}
$$

鍵となる考え方は、絶対位置埋め込み ${\boldsymbol{p}}_{n}$ を正弦符号化された相対的な対応物 $\tilde{{\boldsymbol{p}}}_{m-n}$ で置き換え、第 3 項・第 4 項の絶対位置 ${\boldsymbol{p}}_{m}$ を query の位置に依存しない 2 つの学習可能なベクトル $\mathbf{u}$、$\mathbf{v}$ で置き換えることである。さらに ${\boldsymbol{W}}_{k}$ は、内容にもとづく key ベクトル ${\boldsymbol{x}}_{n}$ と位置にもとづく key ベクトル ${\boldsymbol{p}}_{n}$ とで区別され、それぞれ ${\boldsymbol{W}}_{k}$ と $\widetilde{{\boldsymbol{W}}}_{k}$ と表記される。結果として次を得る。

$$
{\boldsymbol{q}}_{m}^{\intercal}{\boldsymbol{k}}_{n}={\boldsymbol{x}}_{m}^{\intercal}{\boldsymbol{W}}_{q}^{\intercal}{\boldsymbol{W}}_{k}{\boldsymbol{x}}_{n}+{\boldsymbol{x}}_{m}^{\intercal}{\boldsymbol{W}}_{q}^{\intercal}\widetilde{{\boldsymbol{W}}}_{k}\tilde{{\boldsymbol{p}}}_{m-n}+\mathbf{u}^{\intercal}{\boldsymbol{W}}_{q}^{\intercal}{\boldsymbol{W}}_{k}{\boldsymbol{x}}_{n}+\mathbf{v}^{\intercal}{\boldsymbol{W}}_{q}^{\intercal}\widetilde{{\boldsymbol{W}}}_{k}\tilde{{\boldsymbol{p}}}_{m-n} \tag{7}
$$

注目すべきは、$f_{v}({\boldsymbol{x}}_{j}):={\boldsymbol{W}}_{v}{\boldsymbol{x}}_{j}$ と設定することで value の項における位置情報が取り除かれている点である。後続の研究 [^15] [^17] [^16] [^18] はこれらの設定に従い、相対位置情報を attention の重みの中にのみ符号化した。しかし [^15] の著者らは式 (6) を次のように書き換えた。

$$
{\boldsymbol{q}}_{m}^{\intercal}{\boldsymbol{k}}_{n}={\boldsymbol{x}}_{m}^{\intercal}{\boldsymbol{W}}_{q}^{\intercal}{\boldsymbol{W}}_{k}{\boldsymbol{x}}_{n}+b_{i,j} \tag{8}
$$

ここで $b_{i,j}$ は学習可能なバイアスである。[^16] の著者らは式 (6) の中央 2 項を調査し、絶対位置と単語の間にはほとんど相関がないことを見出した。[^15] の著者らは、単語の対または位置の対を異なる射影行列でモデル化することを提案した。

$$
{\boldsymbol{q}}_{m}^{\intercal}{\boldsymbol{k}}_{n}={\boldsymbol{x}}_{m}^{\intercal}{\boldsymbol{W}}_{q}^{\intercal}{\boldsymbol{W}}_{k}{\boldsymbol{x}}_{n}+{\boldsymbol{p}}_{m}^{\intercal}\mathbf{U}_{q}^{\intercal}\mathbf{U}_{k}{\boldsymbol{p}}_{n}+b_{i,j} \tag{9}
$$

[^17] の著者らは、2 つのトークンの相対位置は式 (6) の中央 2 項を使ってのみ完全にモデル化できると論じた。その結果、絶対位置埋め込み ${\boldsymbol{p}}_{m}$ と ${\boldsymbol{p}}_{n}$ は単純に相対位置埋め込み $\tilde{{\boldsymbol{p}}}_{m-n}$ で置き換えられた。

$$
{\boldsymbol{q}}_{m}^{\intercal}{\boldsymbol{k}}_{n}={\boldsymbol{x}}_{m}^{\intercal}{\boldsymbol{W}}_{q}^{\intercal}{\boldsymbol{W}}_{k}{\boldsymbol{x}}_{n}+{\boldsymbol{x}}_{m}^{\intercal}{\boldsymbol{W}}_{q}^{\intercal}{\boldsymbol{W}}_{k}\tilde{{\boldsymbol{p}}}_{m-n}+\tilde{{\boldsymbol{p}}}_{m-n}^{\intercal}{\boldsymbol{W}}_{q}^{\intercal}{\boldsymbol{W}}_{k}{\boldsymbol{x}}_{n} \tag{10}
$$

相対位置埋め込みの 4 つの変種の比較 [^9] は、式 (10) に類似した変種が他の 3 つの中で最も効率的であることを示している。一般的に言えば、これらのアプローチはすべて、式 (2) の self-attention の設定のもとで式 (3) の分解にもとづいて式 (6) を修正しようと試みるものであり、これはもともと [^3] で提案されたものである。**それらは共通して、位置情報を文脈表現に直接加算することを導入している**。それとは異なり、我々のアプローチはいくつかの制約のもとで式 (1) から相対位置符号化を**導出する**ことを目指す。次に、導出されたアプローチが文脈表現の**回転**によって相対位置情報を組み込むことで、より解釈しやすいものになることを示す。

## 3 Proposed approach（提案手法）

本節では提案する回転位置埋め込み（RoPE）を議論する。まず第 3.1 節で相対位置符号化の問題を定式化し、次に第 3.2 節で RoPE を導出し、第 3.3 節でその性質を調査する。

### 3.1 Formulation（定式化）

トランスフォーマベースの言語モデリングは通常、self-attention 機構を通じて個々のトークンの位置情報を活用する。式 (2) で観察されるように、${\boldsymbol{q}}_{m}^{\intercal}{\boldsymbol{k}}_{n}$ は典型的には異なる位置にあるトークン間の知識の伝達を可能にする。相対位置情報を組み込むために、我々は query ${\boldsymbol{q}}_{m}$ と key ${\boldsymbol{k}}_{n}$ の内積が、単語埋め込み ${\boldsymbol{x}}_{m}$、${\boldsymbol{x}}_{n}$ とそれらの相対位置 $m-n$ のみを入力変数として取る関数 $g$ によって定式化されることを要求する。言い換えれば、我々は**内積が位置情報を相対的な形でのみ符号化する**ことを望む。

$$
\langle f_{q}({\boldsymbol{x}}_{m},m),f_{k}({\boldsymbol{x}}_{n},n)\rangle=g({\boldsymbol{x}}_{m},{\boldsymbol{x}}_{n},m-n). \tag{11}
$$

究極の目標は、上述の関係に適合するように関数 $f_{q}({\boldsymbol{x}}_{m},m)$ と $f_{k}({\boldsymbol{x}}_{n},n)$ を解くための等価な符号化機構を見つけることである。

### 3.2 Rotary position embedding（回転位置埋め込み）

#### 3.2.1 A 2D case（2 次元の場合）

我々は次元 $d=2$ という単純な場合から始める。この設定のもとで、2 次元平面上のベクトルの幾何学的性質とその複素数表現を利用して、我々の定式化である式 (11) の解が次であることを証明する（詳細は第 3.4.1 節を参照）。

$$
\begin{aligned}
f_{q}({\boldsymbol{x}}_{m},m)&=({\boldsymbol{W}}_{q}{\boldsymbol{x}}_{m})e^{im\theta}\\
f_{k}({\boldsymbol{x}}_{n},n)&=({\boldsymbol{W}}_{k}{\boldsymbol{x}}_{n})e^{in\theta}\\
g({\boldsymbol{x}}_{m},{\boldsymbol{x}}_{n},m-n)&=\operatorname{Re}[({\boldsymbol{W}}_{q}{\boldsymbol{x}}_{m})({\boldsymbol{W}}_{k}{\boldsymbol{x}}_{n})^{*}e^{i(m-n)\theta}]
\end{aligned} \tag{12}
$$

ここで $\operatorname{Re}[\cdot]$ は複素数の実部であり、$({\boldsymbol{W}}_{k}{\boldsymbol{x}}_{n})^{*}$ は $({\boldsymbol{W}}_{k}{\boldsymbol{x}}_{n})$ の複素共役を表す。$\theta\in\mathbb{R}$ はあらかじめ設定された非ゼロの定数である。さらに $f_{\{q,k\}}$ を行列の積として書くことができる。

$$
f_{\{q,k\}}({\boldsymbol{x}}_{m},m)=\left(\begin{array}{cc}\cos{m\theta}&-\sin{m\theta}\\
\sin{m\theta}&\cos{m\theta}\end{array}\right)\left(\begin{array}{cc}W^{(11)}_{\{q,k\}}&W^{(12)}_{\{q,k\}}\\
W^{(21)}_{\{q,k\}}&W^{(22)}_{\{q,k\}}\end{array}\right)\left(\begin{array}{c}x^{(1)}_{m}\\
x^{(2)}_{m}\end{array}\right) \tag{13}
$$

ここで $(x^{(1)}_{m},x^{(2)}_{m})$ は 2 次元座標で表現された ${\boldsymbol{x}}_{m}$ である。同様に $g$ も行列とみなすことができ、これにより 2 次元の場合における第 3.1 節の定式化の解が可能になる。具体的には、相対位置埋め込みの組み込みは直截である——**アフィン変換された単語埋め込みベクトルを、その位置インデックスの倍数の角度だけ回転させるだけでよい**。これが Rotary Position Embedding の背後にある直観を説明している。

#### 3.2.2 General form（一般形）

2 次元における我々の結果を、$d$ が偶数である任意の ${\boldsymbol{x}}_{i}\in\mathbb{R}^{d}$ に一般化するために、我々は $d$ 次元空間を $d/2$ 個の部分空間に分割し、内積の線形性を利用してそれらを結合し、$f_{\{q,k\}}$ を次のように変える。

$$
f_{\{q,k\}}({\boldsymbol{x}}_{m},m)={\boldsymbol{R}}^{d}_{\Theta,m}{\boldsymbol{W}}_{\{q,k\}}{\boldsymbol{x}}_{m} \tag{14}
$$

ここで

$$
{\boldsymbol{R}}^{d}_{\Theta,m}=\begin{pmatrix}\cos{m\theta_{1}}&-\sin{m\theta_{1}}&0&0&\cdots&0&0\\
\sin{m\theta_{1}}&\cos{m\theta_{1}}&0&0&\cdots&0&0\\
0&0&\cos{m\theta_{2}}&-\sin{m\theta_{2}}&\cdots&0&0\\
0&0&\sin{m\theta_{2}}&\cos{m\theta_{2}}&\cdots&0&0\\
\vdots&\vdots&\vdots&\vdots&\ddots&\vdots&\vdots\\
0&0&0&0&\cdots&\cos{m\theta_{d/2}}&-\sin{m\theta_{d/2}}\\
0&0&0&0&\cdots&\sin{m\theta_{d/2}}&\cos{m\theta_{d/2}}\end{pmatrix} \tag{15}
$$

は、あらかじめ定められたパラメータ $\Theta=\{\theta_{i}=10000^{-2(i-1)/d},\ i\in[1,2,...,d/2]\}$ を持つ**回転行列**である。RoPE の図示を Figure 1 に示す。式 (2) の self-attention に我々の RoPE を適用すると、次を得る。

$$
{\boldsymbol{q}}_{m}^{\intercal}{\boldsymbol{k}}_{n}=({\boldsymbol{R}}^{d}_{\Theta,m}{\boldsymbol{W}}_{q}{\boldsymbol{x}}_{m})^{\intercal}({\boldsymbol{R}}^{d}_{\Theta,n}{\boldsymbol{W}}_{k}{\boldsymbol{x}}_{n})={\boldsymbol{x}}^{\intercal}{\boldsymbol{W}}_{q}R^{d}_{\Theta,n-m}{\boldsymbol{W}}_{k}{\boldsymbol{x}}_{n} \tag{16}
$$

ここで ${\boldsymbol{R}}^{d}_{\Theta,n-m}=({\boldsymbol{R}}^{d}_{\Theta,m})^{\intercal}{\boldsymbol{R}}^{d}_{\Theta,n}$ である。${\boldsymbol{R}}^{d}_{\Theta}$ が**直交行列**であることに注意されたい。これが位置情報を符号化する過程における安定性を保証する。加えて、$R^{d}_{\Theta}$ の**疎性**のために、式 (16) のように行列の積を直接適用することは計算効率が良くない。我々は理論的説明の節で別の実現方法を与える。

> 訳注（原典の表記について）: 式 (16) 右辺の先頭は原典では ${\boldsymbol{x}}^{\intercal}$ と書かれているが、左辺および文脈から ${\boldsymbol{x}}_{m}^{\intercal}$ の誤記と読める。また、式 (15) の $\Theta$ は $\theta_{i}=10000^{-2(i-1)/d}$（$i$ は 1 始まり）と定義されているのに対し、第 3.3 節と第 3.4.3 節では $\theta_{i}=10000^{-2i/d}$ と書かれている。添字の起点の違いによる原典内の揺れであり、訳文では原典どおりに残した。

先行研究で採用された位置埋め込み手法、すなわち式 (3)、(4)、(5)、(6)、(7)、(8)、(9)、(10) の**加法的**な性質とは対照的に、我々のアプローチは**乗法的**である。さらに RoPE は、self-attention に適用したときに加法的な位置符号化の展開された定式化の項を変更するのではなく、**回転行列の積を通じて自然に相対位置情報を組み込む**。

<figure>

![](../../raw/assets/2021-roformer/x1.png)

<figcaption>図1: Rotary Position Embedding（RoPE）の実装。</figcaption>
</figure>

### 3.3 Properties of RoPE（RoPE の性質）

##### 長距離減衰（Long-term decay）:

[^3] に従い、我々は $\theta_{i}=10000^{-2i/d}$ と設定する。この設定が長距離減衰の性質をもたらすことは証明できる（詳細は第 3.4.3 節を参照）。すなわち、相対位置が増大すると内積が減衰する。この性質は、**相対距離の長いトークンの対はより弱い結びつきを持つべきである**という直観と一致する。

##### 線形 attention との組み合わせ（RoPE with linear attention）:

self-attention はより一般的な形で書き直すことができる。

$$
\operatorname{Attention}(\mathbf{Q},\mathbf{K},\mathbf{V})_{m}=\frac{\sum_{n=1}^{N}\operatorname{sim}({\boldsymbol{q}}_{m},{\boldsymbol{k}}_{n}){\boldsymbol{v}}_{n}}{\sum_{n=1}^{N}\operatorname{sim}({\boldsymbol{q}}_{m},{\boldsymbol{k}}_{n})}. \tag{17}
$$

元の self-attention は $\operatorname{sim}({\boldsymbol{q}}_{m},{\boldsymbol{k}}_{n})=\exp({\boldsymbol{q}}_{m}^{\intercal}{\boldsymbol{k}}_{n}/\sqrt{d})$ を選ぶ。元の self-attention はすべてのトークン対について query と key の内積を計算しなければならず、これは二次の計算量 $\mathbb{O}(N^{2})$ を持つことに注意されたい。[^22] に従い、線形 attention は式 (17) を次のように再定式化する。

$$
\operatorname{Attention}({\boldsymbol{Q}},{\boldsymbol{K}},{\boldsymbol{V}})_{m}=\frac{\sum_{n=1}^{N}\phi({\boldsymbol{q}}_{m})^{\intercal}\varphi({\boldsymbol{k}}_{n}){\boldsymbol{v}}_{n}}{\sum_{n=1}^{N}\phi({\boldsymbol{q}}_{m})^{\intercal}\varphi({\boldsymbol{k}}_{n})}, \tag{18}
$$

ここで $\phi(\cdot),\varphi(\cdot)$ は通常は非負の関数である。[^22] の著者らは $\phi(x)=\varphi(x)=\operatorname{elu}(x)+1$ を提案し、行列の積の結合法則を利用してまず key と value の間の積を計算した。[^23] では、内積の前に query と key をそれぞれ別々に正規化するために softmax 関数が用いられており、これは $\phi({\boldsymbol{q}}_{i})=\operatorname{softmax}({\boldsymbol{q}}_{i})$ かつ $\phi({\boldsymbol{k}}_{j})=\exp({\boldsymbol{k}}_{j})$ と等価である。線形 attention の詳細については、読者には元の論文を参照することを勧める。本節では RoPE を式 (18) に組み込むことの議論に焦点を当てる。**RoPE は回転によって位置情報を注入し、これは隠れ表現のノルムを変えないままにするので、非負関数の出力に回転行列を掛けることで RoPE を線形 attention と組み合わせることができる**。

$$
\operatorname{Attention}(\mathbf{Q},\mathbf{K},\mathbf{V})_{m}=\frac{\sum_{n=1}^{N}\big{(}{\boldsymbol{R}}^{d}_{\Theta,m}\phi({\boldsymbol{q}}_{m})\big{)}^{\intercal}\big{(}{\boldsymbol{R}}^{d}_{\Theta,n}\varphi({\boldsymbol{k}}_{n})\big{)}{\boldsymbol{v}}_{n}}{\sum_{n=1}^{N}\phi({\boldsymbol{q}}_{m})^{\intercal}\varphi({\boldsymbol{k}}_{n})}. \tag{19}
$$

注目すべきは、ゼロ除算の危険を避けるために我々は**分母を変更しないまま**にしており、分子の総和は負の項を含みうるという点である。式 (19) における各 value ${\boldsymbol{v}}_{i}$ の重みは厳密には確率的に正規化されていないが、それでもこの計算は value の重要度をモデル化しうると、我々は慎ましく主張する。

### 3.4 Theoretical Explanation（理論的説明）

#### 3.4.1 Derivation of RoPE under 2D（2 次元における RoPE の導出）

$d=2$ の場合を考え、query と key に対応する 2 つの単語埋め込みベクトル ${\boldsymbol{x}}_{q}$、${\boldsymbol{x}}_{k}$ と、それらの位置 $m$、$n$ をそれぞれ考える。式 (1) によれば、それらの位置符号化された対応物は次である。

$$
\begin{aligned}
{\boldsymbol{q}}_{m}&=f_{q}({\boldsymbol{x}}_{q},m),\\
{\boldsymbol{k}}_{n}&=f_{k}({\boldsymbol{x}}_{k},n),
\end{aligned} \tag{20}
$$

ここで ${\boldsymbol{q}}_{m}$ と ${\boldsymbol{k}}_{n}$ の添字は符号化された位置情報を示す。$f_{\{q,k\}}$ によって生成されるベクトル間の内積を定義する関数 $g$ が存在すると仮定する。

$$
{\boldsymbol{q}}^{\intercal}_{m}{\boldsymbol{k}}_{n}=\langle f_{q}({\boldsymbol{x}}_{m},m),f_{k}({\boldsymbol{x}}_{n},n)\rangle=g({\boldsymbol{x}}_{m},{\boldsymbol{x}}_{n},n-m), \tag{21}
$$

さらに我々は次の初期条件が満たされることを要求する。

$$
\begin{aligned}
{\boldsymbol{q}}&=f_{q}({\boldsymbol{x}}_{q},0),\\
{\boldsymbol{k}}&=f_{k}({\boldsymbol{x}}_{k},0),
\end{aligned} \tag{22}
$$

これは**位置情報が空のまま符号化されたベクトル**と読むことができる。これらの設定のもとで、我々は $f_{q}$、$f_{k}$ の解を見つけようと試みる。まず 2 次元におけるベクトルの幾何学的意味とその複素数としての対応物を利用して、式 (20) と (21) の関数を次のように分解する。

$$
\begin{aligned}
f_{q}({\boldsymbol{x}}_{q},m)&=R_{q}({\boldsymbol{x}}_{q},m)e^{i\Theta_{q}({\boldsymbol{x}}_{q},m)},\\
f_{k}({\boldsymbol{x}}_{k},n)&=R_{k}({\boldsymbol{x}}_{k},n)e^{i\Theta_{k}({\boldsymbol{x}}_{k},n)},\\
g({\boldsymbol{x}}_{q},{\boldsymbol{x}}_{k},n-m)&=R_{g}({\boldsymbol{x}}_{q},{\boldsymbol{x}}_{k},n-m)e^{i\Theta_{g}({\boldsymbol{x}}_{q},{\boldsymbol{x}}_{k},n-m)},
\end{aligned} \tag{23}
$$

ここで $R_{f}$、$R_{g}$ と $\Theta_{f}$、$\Theta_{g}$ はそれぞれ $f_{\{q,k\}}$ と $g$ の動径成分（radial component）と偏角成分（angular component）である。これらを式 (21) に代入すると、次の関係を得る。

$$
\begin{aligned}
R_{q}({\boldsymbol{x}}_{q},m)R_{k}({\boldsymbol{x}}_{k},n)&=R_{g}({\boldsymbol{x}}_{q},{\boldsymbol{x}}_{k},n-m),\\
\Theta_{k}({\boldsymbol{x}}_{k},n)-\Theta_{q}({\boldsymbol{x}}_{q},m)&=\Theta_{g}({\boldsymbol{x}}_{q},{\boldsymbol{x}}_{k},n-m),
\end{aligned} \tag{24}
$$

対応する初期条件は次である。

$$
\begin{aligned}
{\boldsymbol{q}}&=\|{\boldsymbol{q}}\|e^{i\theta_{q}}=R_{q}({\boldsymbol{x}}_{q},0)e^{i\Theta_{q}({\boldsymbol{x}}_{q},0)},\\
{\boldsymbol{k}}&=\|{\boldsymbol{k}}\|e^{i\theta_{k}}=R_{k}({\boldsymbol{x}}_{k},0)e^{i\Theta_{k}({\boldsymbol{x}}_{k},0)},
\end{aligned} \tag{25}
$$

ここで $\|{\boldsymbol{q}}\|$、$\|{\boldsymbol{k}}\|$ と $\theta_{q}$、$\theta_{k}$ は 2 次元平面上の ${\boldsymbol{q}}$ と ${\boldsymbol{k}}$ の動径部分と偏角部分である。

次に、式 (24) で $m=n$ と置き、式 (25) の初期条件を考慮する。

$$
\begin{aligned}
R_{q}({\boldsymbol{x}}_{q},m)R_{k}({\boldsymbol{x}}_{k},m)&=R_{g}({\boldsymbol{x}}_{q},{\boldsymbol{x}}_{k},0)=R_{q}({\boldsymbol{x}}_{q},0)R_{k}({\boldsymbol{x}}_{k},0)=\|{\boldsymbol{q}}\|\|{\boldsymbol{k}}\|,\\
\Theta_{k}({\boldsymbol{x}}_{k},m)-\Theta_{q}({\boldsymbol{x}}_{q},m)&=\Theta_{g}({\boldsymbol{x}}_{q},{\boldsymbol{x}}_{k},0)=\Theta_{k}({\boldsymbol{x}}_{k},0)-\Theta_{q}({\boldsymbol{x}}_{q},0)=\theta_{k}-\theta_{q}.
\end{aligned} \tag{26}
$$

一方で、式 (26a) から $R_{f}$ の直截な解が次のように形づくられる。

$$
\begin{aligned}
R_{q}({\boldsymbol{x}}_{q},m)&=R_{q}({\boldsymbol{x}}_{q},0)=\|{\boldsymbol{q}}\|\\
R_{k}({\boldsymbol{x}}_{k},n)&=R_{k}({\boldsymbol{x}}_{k},0)=\|{\boldsymbol{k}}\|\\
R_{g}({\boldsymbol{x}}_{q},{\boldsymbol{x}}_{k},n-m)&=R_{g}({\boldsymbol{x}}_{q},{\boldsymbol{x}}_{k},0)=\|{\boldsymbol{q}}\|\|{\boldsymbol{k}}\|
\end{aligned} \tag{27}
$$

これは、**動径関数 $R_{q}$、$R_{k}$、$R_{g}$ が位置情報から独立である**ことを解釈している。他方で、式 (26b) で見てとれるように、$\Theta_{q}({\boldsymbol{x}}_{q},m)-\theta_{q}=\Theta_{k}({\boldsymbol{x}}_{k},m)-\theta_{k}$ は**偏角関数が query と key に依存しない**ことを示している。そこで我々はそれらを $\Theta_{f}:=\Theta_{q}=\Theta_{k}$ と置く。そして $\Theta_{f}({\boldsymbol{x}}_{\{q,k\}},m)-\theta_{\{q,k\}}$ という項は位置 $m$ の関数であり単語埋め込み ${\boldsymbol{x}}_{\{q,k\}}$ から独立なので、これを $\phi(m)$ と表記すると次を得る。

$$
\Theta_{f}({\boldsymbol{x}}_{\{q,k\}},m)=\phi(m)+\theta_{\{q,k\}}, \tag{28}
$$

さらに、式 (24) に $n=m+1$ を代入して上式を考慮すると、次を得る。

$$
\phi(m+1)-\phi(m)=\Theta_{g}({\boldsymbol{x}}_{q},{\boldsymbol{x}}_{k},1)+\theta_{q}-\theta_{k}, \tag{29}
$$

右辺は $m$ に無関係な定数なので、連続する整数を入力とする $\phi(m)$ は**等差数列**をなす。

$$
\phi(m)=m\theta+\gamma, \tag{30}
$$

ここで $\theta,\gamma\in\mathbb{R}$ は定数であり、$\theta$ は非ゼロである。式 (27)、(28)、(29)、(30) からの我々の解をまとめると次になる。

$$
\begin{aligned}
f_{q}({\boldsymbol{x}}_{q},m)&=\|{\boldsymbol{q}}\|e^{i\theta_{q}+m\theta+\gamma}={\boldsymbol{q}}e^{i(m\theta+\gamma)},\\
f_{k}({\boldsymbol{x}}_{k},n)&=\|{\boldsymbol{k}}\|e^{i\theta_{k}+n\theta+\gamma}={\boldsymbol{k}}e^{i(n\theta+\gamma)}.
\end{aligned} \tag{31}
$$

我々は式 (22) の $f_{q}$ と $f_{k}$ に何の制約も課していないので、$f_{q}({\boldsymbol{x}}_{m},0)$ と $f_{k}({\boldsymbol{x}}_{n},0)$ は自由に選ぶ余地が残されていることに注意されたい。我々の結果を式 (3) と比較可能にするために、次のように定義する。

$$
\begin{aligned}
{\boldsymbol{q}}=f_{q}({\boldsymbol{x}}_{m},0)&={\boldsymbol{W}}_{q}{\boldsymbol{x}}_{n},\\
{\boldsymbol{k}}=f_{k}({\boldsymbol{x}}_{n},0)&={\boldsymbol{W}}_{k}{\boldsymbol{x}}_{n}.
\end{aligned} \tag{32}
$$

そして、最終的な解として式 (31) で単純に $\gamma=0$ と設定する。

$$
\begin{aligned}
f_{q}({\boldsymbol{x}}_{m},m)&=({\boldsymbol{W}}_{q}{\boldsymbol{x}}_{m})e^{im\theta},\\
f_{k}({\boldsymbol{x}}_{n},n)&=({\boldsymbol{W}}_{k}{\boldsymbol{x}}_{n})e^{in\theta}.
\end{aligned} \tag{33}
$$

> 訳注（原典の表記について）: 式 (21) では $g$ の第 3 引数が $n-m$ と書かれているが、第 3.1 節の式 (11) では $m-n$ である。また式 (31) の指数部は原典では $e^{i\theta_{q}+m\theta+\gamma}$ と書かれているが、続く等号から $e^{i(\theta_{q}+m\theta+\gamma)}$ の意である。式 (32) 第 1 行の右辺は ${\boldsymbol{W}}_{q}{\boldsymbol{x}}_{n}$ と書かれているが、左辺から ${\boldsymbol{W}}_{q}{\boldsymbol{x}}_{m}$ の誤記と読める。いずれも原典どおりに残した。

#### 3.4.2 Computational efficient realization of rotary matrix multiplication（回転行列の積の計算効率の良い実現）

式 (15) における ${\boldsymbol{R}}^{d}_{\Theta,m}$ の**疎性**を利用すると、$R^{d}_{\Theta}$ と ${\boldsymbol{x}}\in\mathbb{R}^{d}$ の積のより計算効率の良い実現は次である。

$$
{\boldsymbol{R}}^{d}_{\Theta,m}{\boldsymbol{x}}=\begin{pmatrix}x_{1}\\
x_{2}\\
x_{3}\\
x_{4}\\
\vdots\\
x_{d-1}\\
x_{d}\end{pmatrix}\otimes\begin{pmatrix}\cos{m\theta_{1}}\\
\cos{m\theta_{1}}\\
\cos{m\theta_{2}}\\
\cos{m\theta_{2}}\\
\vdots\\
\cos{m\theta_{d/2}}\\
\cos{m\theta_{d/2}}\end{pmatrix}+\begin{pmatrix}-x_{2}\\
x_{1}\\
-x_{4}\\
x_{3}\\
\vdots\\
-x_{d}\\
x_{d-1}\end{pmatrix}\otimes\begin{pmatrix}\sin{m\theta_{1}}\\
\sin{m\theta_{1}}\\
\sin{m\theta_{2}}\\
\sin{m\theta_{2}}\\
\vdots\\
\sin{m\theta_{d/2}}\\
\sin{m\theta_{d/2}}\end{pmatrix} \tag{34}
$$

#### 3.4.3 Long-term decay of RoPE（RoPE の長距離減衰）

ベクトル ${\boldsymbol{q}}={\boldsymbol{W}}_{q}{\boldsymbol{x}}_{m}$ と ${\boldsymbol{k}}={\boldsymbol{W}}_{k}{\boldsymbol{x}}_{n}$ の要素を対にまとめることができ、式 (16) における RoPE の内積は複素数の積として書ける。

$$
({\boldsymbol{R}}^{d}_{\Theta,m}{\boldsymbol{W}}_{q}{\boldsymbol{x}}_{m})^{\intercal}({\boldsymbol{R}}^{d}_{\Theta,n}{\boldsymbol{W}}_{k}{\boldsymbol{x}}_{n})=\operatorname{Re}\bigg{[}\sum_{i=0}^{d/2-1}{\boldsymbol{q}}_{[2i:2i+1]}{\boldsymbol{k}}_{[2i:2i+1]}^{*}e^{i(m-n)\theta_{i}}\bigg{]} \tag{35}
$$

ここで ${\boldsymbol{q}}_{[2i:2i+1]}$ は ${\boldsymbol{q}}$ の $2i$ 番目から $(2i+1)$ 番目までの要素を表す。$h_{i}={\boldsymbol{q}}_{[2i:2i+1]}{\boldsymbol{k}}_{[2i:2i+1]}^{*}$、$S_{j}=\sum_{i=0}^{j-1}e^{i(m-n)\theta_{i}}$ と表記し、$h_{d/2}=0$、$S_{0}=0$ とすると、**アーベル変換（Abel transformation, 総和の部分積分に相当する変形）** を用いて総和を書き換えることができる。

$$
\sum_{i=0}^{d/2-1}{\boldsymbol{q}}_{[2i:2i+1]}{\boldsymbol{k}}_{[2i:2i+1]}^{*}e^{i(m-n)\theta_{i}}=\sum_{i=0}^{d/2-1}h_{i}(S_{i+1}-S_{i})=-\sum_{i=0}^{d/2-1}S_{i+1}(h_{i+1}-h_{i}). \tag{36}
$$

したがって、

$$
\begin{aligned}
\bigg{|}\sum_{i=0}^{d/2-1}{\boldsymbol{q}}_{[2i:2i+1]}{\boldsymbol{k}}_{[2i:2i+1]}^{*}e^{i(m-n)\theta_{i}}\bigg{|}&=\bigg{|}\sum_{i=0}^{d/2-1}S_{i+1}(h_{i+1}-h_{i})\bigg{|}\\
&\leq\sum_{i=0}^{d/2-1}|S_{i+1}||(h_{i+1}-{h_{i}})|\\
&\leq\big{(}\max_{i}|h_{i+1}-h_{i}|\big{)}\sum_{i=0}^{d/2-1}|S_{i+1}|
\end{aligned} \tag{37}
$$

$\theta_{i}=10000^{-2i/d}$ と設定することで、$\frac{1}{d/2}\sum_{i=1}^{d/2}|S_{i}|$ の値が相対距離 $m-n$ の増大とともに減衰することに注意されたい。これは Figure 2 に示すとおりである。

<figure>

![](../../raw/assets/2021-roformer/x2.png)

<figcaption>図2: RoPE の長距離減衰。（訳注: 横軸が相対距離、縦軸が相対的な上界。値は振動しながら全体として単調に減衰していく。）</figcaption>
</figure>

## 4 Experiments and Evaluation（実験と評価）

我々は提案する RoFormer を次のようにさまざまな NLP タスクで評価する。まず第 4.1 節で機械翻訳タスクにおける提案手法の性能を検証する。次に第 4.2 節で、事前学習の段階における我々の RoPE の実装を BERT [^4] と比較する。事前学習済みモデルにもとづき、第 4.3 節で GLUE ベンチマーク [^24] のさまざまな下流タスクにわたる評価をさらに実施する。加えて第 4.4 節で、提案する RoPE を PerFormer [^25] の線形 attention と組み合わせる実験を行う。最後に第 4.5 節で中国語データでの追加テストを含める。すべての実験は **4 基の V100 GPU を積んだクラウドサーバ 2 台**の上で実行された。

### 4.1 Machine Translation（機械翻訳）

我々はまず、系列から系列への言語翻訳タスクにおける RoFormer の性能を示す。

#### 4.1.1 Experimental Settings（実験設定）

我々は標準的な WMT 2014 英独データセット [^26] を選ぶ。これはおよそ 450 万の文対からなる。比較対象はトランスフォーマベースのベースラインの代替手法 [^3] である。

#### 4.1.2 Implementation details（実装の詳細）

我々は RoPE をその学習過程に組み込めるようにするため、ベースラインモデル [^3] の self-attention 層にいくつかの修正を施した。英語から独語への翻訳の設定を、ソースとターゲットを結合した byte pair encoding（BPE） [^27] にもとづく 37k の語彙で再現した。評価時には、最後の 5 つのチェックポイントを平均することで単一のモデルを得る。結果はビームサイズ 4、長さペナルティ 0.6 のビームサーチを用いる。実験は fairseq ツールキット（MIT ライセンス） [^28] の PyTorch で実装した。我々のモデルは Adam オプティマイザで最適化し、$\beta_{1}=0.9$、$\beta_{2}=0.98$、学習率は $1e-7$ から $5e-4$ まで線形に増加させたのち、ステップ数の平方根の逆数に比例して減衰させる。0.1 のラベル平滑化も採用している。最終的な指標としてテストセット上の BLEU [^29] スコアを報告する。

#### 4.1.3 Results（結果）

我々はベースラインモデルと RoFormer を同じ設定のもとで訓練し、結果を Table 1 に報告する。見てとれるように、我々のモデルはベースラインのトランスフォーマと比べてより良い BLEU スコアを与える。

**表1**: 提案する RoFormer は、WMT 2014 英独翻訳タスク [^26] においてベースラインの代替手法 [^3] と比べてより良い BLEU スコアを与える。

| モデル | BLEU |
| --- | --- |
| Transformer-base [^3] | 27.3 |
| RoFormer | 27.5 |

### 4.2 Pre-training Language Modeling（事前学習の言語モデリング）

第 2 の実験は、文脈表現の学習という点での我々の提案の性能を検証することである。これを達成するために、事前学習の段階で BERT の元の正弦位置符号化を我々の RoPE で置き換える。

#### 4.2.1 Experimental Settings（実験設定）

事前学習には Huggingface Datasets ライブラリ（Apache License 2.0）の BookCorpus [^30] と Wikipedia Corpus [^31] を使う。コーパスはさらに 8:2 の比率で訓練集合と検証集合に分割される。評価指標として訓練過程の masked language-modeling（MLM）損失の値を使う。よく知られた BERT [^4] をベースラインモデルとして採用する。実験では bert-base-uncased を使うことに注意されたい。

#### 4.2.2 Implementation details（実装の詳細）

RoFormer については、ベースラインモデルの self-attention ブロックにおける正弦位置符号化を提案する RoPE で置き換え、式 (16) に従って self-attention を実現する。BERT と RoFormer の双方を、バッチサイズ 64、最大系列長 512 で 100k ステップ訓練する。オプティマイザには学習率 1e-5 の AdamW [^32] を使う。

#### 4.2.3 Results（結果）

事前学習中の MLM 損失を Figure 3 の左のプロットに示す。素の BERT と比べて、RoFormer はより速い収束を経験する。

<figure>

![](../../raw/assets/2021-roformer/x3.png)

<figcaption>図3（左）: 言語モデリングの事前学習における RoPE の評価。BERT と RoFormer の訓練損失。（訳注: 横軸は訓練ステップ（K）、縦軸は MLM 損失。）</figcaption>
</figure>

<figure>

![](../../raw/assets/2021-roformer/x4.png)

<figcaption>図3（右）: RoPE のある PerFormer とない PerFormer の訓練損失。（訳注: 横軸は訓練ステップ（K）、縦軸は LM 損失。原典では図3 は左右 2 パネルからなる 1 つの図であり、そのキャプションは「言語モデリングの事前学習における RoPE の評価。左: BERT と RoFormer の訓練損失。右: RoPE のある／ない PerFormer の訓練損失。」である。）</figcaption>
</figure>

### 4.3 Fine-tuning on GLUE tasks（GLUE タスクでのファインチューニング）

これまでの実験と整合するように、我々は事前学習済み RoFormer の重みをさまざまな GLUE タスクにわたってファインチューニングし、下流の NLP タスクにおける汎化能力を評価する。

#### 4.3.1 Experimental Settings（実験設定）

GLUE からいくつかのデータセット、すなわち MRPC [^33]、SST-2 [^34]、QNLI [^35]、STS-B [^36]、QQP [^37]、MNLI [^38] を見る。評価指標として、MRPC と QQP データセットには F1 スコア、STS-B にはスピアマン相関、残りには正解率を使う。

#### 4.3.2 Implementation details（実装の詳細）

上述の各下流タスクを 3 エポック、最大系列長 512、バッチサイズ 32、学習率 2,3,4,5e-5 でファインチューニングするために Huggingface Transformers ライブラリ（Apache License 2.0） [^39] を使う。[^4] に従い、検証集合上で最も良い平均結果を報告する。

**表2**: 下流の GLUE タスクでのファインチューニングによる RoFormer と BERT の比較。

| モデル | MRPC | SST-2 | QNLI | STS-B | QQP | MNLI(m/mm) |
| --- | --- | --- | --- | --- | --- | --- |
| BERT [^4] | 88.9 | 93.5 | 90.5 | 85.8 | 71.2 | 84.6/83.4 |
| RoFormer | 89.5 | 90.7 | 88.0 | 87.0 | 86.4 | 80.2/79.8 |

#### 4.3.3 Results（結果）

ファインチューニングタスクの評価結果を Table 2 に報告する。見てとれるように、RoFormer は 6 つのデータセットのうち 3 つで BERT を有意に上回ることができ、その改善は相当なものである。

### 4.4 Performer with RoPE（RoPE を用いた Performer）

Performer [^25] は代替的な attention 機構である線形 attention を導入する。これは入力の系列長にともなって増大する二次の計算コストを避けるように設計されている。第 3.3 節で議論したように、提案する RoPE は PerFormer モデルに容易に実装でき、self-attention における線形スケールの計算量を保ったまま相対位置符号化を実現できる。我々は言語モデリングの事前学習タスクでその性能を示す。

#### 4.4.1 Implementation details（実装の詳細）

我々は Enwik8 データセット [^40] でテストを実施する。これは英語版 Wikipedia に由来し、英語テキストに加えてマークアップ・特殊文字・他言語のテキストを含む。RoPE を 768 次元・12 ヘッドの 12 層の文字ベース PerFormer に組み込む<sup>2</sup>。RoPE の有効性をより良く示すために、同じ設定、すなわち学習率 1e-4、バッチサイズ 128、固定された最大系列長 1024 などのもとで、RoPE のある場合とない場合の事前学習過程の損失曲線を報告する。

> 訳注（脚注 2、原ページより復元）: この実験では [https://github.com/lucidrains/performer-pytorch](https://github.com/lucidrains/performer-pytorch) のコード（MIT ライセンス）を採用している。

#### 4.4.2 Results（結果）

Figure 3 の右のプロットに示すとおり、Performer に RoPE を代入すると、同じ訓練ステップ数のもとで急速な収束とより低い損失につながる。これらの改善は、線形の計算量と相まって Performer をより魅力的にする。

### 4.5 Evaluation on Chinese Data（中国語データでの評価）

英語データでの実験に加えて、中国語データでの追加結果を示す。長文における RoFormer の性能を検証するため、長さが 512 文字を超える長文書で実験を実施する。

#### 4.5.1 Implementation（実装）

これらの実験では、WoBERT [^41] に対して絶対位置埋め込みを提案する RoPE で置き換えるという修正を施した。中国語における他の事前学習済みトランスフォーマベースのモデル、すなわち BERT [^4]、WoBERT [^41]、NEZHA [^42] との相互比較として、それらのトークン化の水準と位置埋め込みの情報を Table 3 に表としてまとめる。

**表3**: 中国語データにおける我々の RoFormer と他の事前学習済みモデルの相互比較。'abs' と 'rel' はそれぞれ絶対位置埋め込みと相対位置埋め込みを表す。

| モデル | BERT [^4] | WoBERT [^41] | NEZHA [^42] | RoFormer |
| --- | --- | --- | --- | --- |
| トークン化の水準 | 文字 | 単語 | 文字 | 単語 |
| 位置埋め込み | abs. | abs. | rel. | RoPE |

#### 4.5.2 Pre-training（事前学習）

我々は中国語版 Wikipedia・ニュース・フォーラムから収集したおよそ 34GB のデータで RoFormer を事前学習する。事前学習はバッチサイズと最大入力系列長を変えながら複数の段階で実施され、モデルをさまざまな場面に適応させる。Table 4 に示すとおり、RoFormer の正解率は系列長の上界の増大とともに上昇し、これは長文を扱う RoFormer の能力を示している。我々はこれを提案する RoPE の優れた汎化性に帰する。

**表4**: 中国語データセットでの RoFormer の事前学習の戦略。訓練の手順はいくつかの連続した段階に分けられる。各段階では、最大系列長とバッチサイズの特定の組み合わせでモデルを訓練する。

| 段階 | 最大系列長 | バッチサイズ | 訓練ステップ | 損失 | 正解率 |
| --- | --- | --- | --- | --- | --- |
| 1 | 512 | 256 | 200k | 1.73 | 65.0% |
| 2 | 1536 | 256 | 12.5k | 1.61 | 66.8% |
| 3 | 256 | 256 | 120k | 1.75 | 64.6% |
| 4 | 128 | 512 | 80k | 1.83 | 63.4% |
| 5 | 1536 | 256 | 10k | 1.58 | 67.4% |
| 6 | 512 | 512 | 30k | 1.66 | 66.2% |

#### 4.5.3 Downstream Tasks & Dataset（下流タスクとデータセット）

長文、すなわち意味的なテキストマッチングを扱う RoFormer の能力を示すために、Chinese AI and Law 2019 Similar Case Matching（CAIL2019-SCM） [^43] データセットを選ぶ。CAIL2019-SCM は中国の最高人民法院が公開した 8964 個の事件の三つ組を含む。入力の三つ組は (A, B, C) と表記され、3 つの事件の事実記述である。タスクは、あらかじめ定められた類似度の尺度のもとで対 (A, B) が (A, C) よりも近いかどうかを予測することである。既存の手法の大部分は、文書の長さ（すなわち大半が 512 文字を超える）のために CAIL2019-SCM データセットで顕著な性能を発揮できないことに注意されたい。よく知られた 6:2:2 の比率にもとづいて訓練・検証・テスト集合を分割する。

#### 4.5.4 Results（結果）

我々は事前学習済みの RoFormer モデルを、異なる入力長で CAIL2019-SCM に適用する。Table 5 に示すとおり、同じ事前学習データで事前学習された BERT および WoBERT モデルと比較される。512 のような短いテキストの打ち切りでは、RoFormer の結果は WoBERT と同等であり、BERT の実装よりわずかに良い。しかし最大入力テキスト長を 1024 に増やすと、**RoFormer は WoBERT を絶対値で 1.5% 上回る**。

**表5**: CAIL2019-SCM タスクでの実験結果。第 1 列の数字は打ち切りの最大系列長を表す。結果は正解率（パーセント）で示されている。

| モデル | 検証 | テスト |
| --- | --- | --- |
| BERT-512 | 64.13% | 67.77% |
| WoBERT-512 | 64.07% | 68.10% |
| RoFormer-512 | 64.13% | 68.29% |
| RoFormer-1024 | 66.07% | 69.79% |

#### 4.5.5 Limitations of the work（本研究の限界）

我々は理論的な裏づけと有望な実験的正当化を提供しているが、我々の手法は次の事実によって制限されている。

- 我々は相対位置関係を 2 次元の部分空間における回転として数学的に定式化しているにもかかわらず、**なぜそれが他の位置符号化戦略を組み込んだベースラインモデルより速く収束するのかについて、徹底した説明を欠いている**。
- 我々は自分たちのモデルがトークン間の積について長距離減衰という好ましい性質を持つことを証明した（第 3.3 節）——これは既存の位置符号化機構と類似している——が、我々のモデルは同種のモデルより長文において優れた性能を示しており、**この点についても忠実な説明を思いつけていない**。

我々の提案する RoFormer はトランスフォーマベースの基盤の上に構築されており、事前学習の目的のためにハードウェア資源を必要とする。

## 5 Conclusions（結論）

本研究では、トランスフォーマアーキテクチャの性能を高めるために、self-attention の中に明示的な相対位置の依存関係を組み込む新しい位置埋め込み手法を提案した。我々の理論的解析は、相対位置が self-attention におけるベクトルの積を用いて自然に定式化でき、絶対位置情報が回転行列を通じて符号化されることを示している。加えて、提案手法をトランスフォーマに適用したときの有利な性質を数学的に例示した。最後に、英語と中国語の双方のベンチマークデータセットでの実験は、我々の手法が事前学習においてより速い収束を促すことを実証している。実験結果は、提案する RoFormer が長文タスクにおいてより良い性能を達成しうることも示している。
