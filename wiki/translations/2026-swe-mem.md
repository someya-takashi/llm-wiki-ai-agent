---
type: translation
source_path: "raw/papers/SWE-MeM- Learning Adaptive Memory Management for Long-Horizon Coding Agents.pdf"
source_page: "[[summaries/2026-swe-mem]]"
original_language: en
translated_to: ja
translated_at: 2026-10-28
---

# SWE-MeM: 長期的なコーディングエージェントのための適応的な記憶管理の学習

> 原題: SWE-MeM: Learning Adaptive Memory Management for Long-Horizon Coding Agents
> 著者: Shuzheng Gao¹, Wenhao Zeng², Zhaojian Yu³, Jianqiao Wangni⁴, Chaozheng Wang¹, Kai Cai⁴, Shilin He⁴†, Michael R. Lyu¹（¹The Chinese University of Hong Kong（香港中文大学）、²Shanghai Jiao Tong University（上海交通大学）、³Tsinghua University（清華大学）、⁴ByteDance。† 連絡先著者）
> 出典: arXiv:2606.28434 v1（2026-06-26 投稿、PDF の日付は 2026-06-30）。連絡先著者: Shilin He

> 訳注（底本）: PDF を底本にし、本文は pdftotext で抽出したテキストとページ画像で確認しながら訳した。図 1〜4 は PDF のベクタ図をキャプション位置を境界にレンダリングした。式・表・Algorithm 1 はページ画像から組み直した。プロンプトと事例の軌跡は原文のまま収録した。参考文献は訳していない。
>
> 訳注（原典内の食い違い）: 原典の中で次の食い違いを確認した。本文は原文どおりに訳した。
>
> - 図 1(c) では SWE-MeM 30B の点が解決率 60.5〜60.6 付近に描かれている。表 1・表 2 の値は 60.2 である。同じ図の他の手法の点（51.6、53.8、55.0、57.8、58.0）は表の値と一致する。
> - 図 4(b)（30B モデル）には、本文に数値が出てこない結果も描かれている。記憶ツールを外して 256K の文脈で動かした Vanilla OpenHands は 53.6、Vanilla SFT は 58.2 である。4.2 節の本文が挙げる数値は 4B モデルのもの（43.4% → 37.6%、40.8% → 41.6%）だけである。
> - 1 節は「より少ない対話ラウンド数で既存の記憶管理手法を上回る」と述べる。表 2 で SWE-MeM が少ないのは相対ステップ使用量（123.5% 対 151.3%・194.9%）である。平均ステップ数の絶対値では、Threshold-Compression Agent [41] の 74.9 が SWE-MeM SFT+RL の 77.0 より少ない（ベースモデルは異なる）。4.1 節の記述（ステップ数の増加が Context Folding よりはるかに小さい）は表 2 と整合する。
> - 付録 A.1 が参照する行番号は Algorithm 1 の行番号と合わない。Algorithm 1 では、予算圧迫の分岐が 8〜12 行目、能動的な分岐が 13〜16 行目、プロプライエタリモデルの呼び出しが 11 行目と 15 行目、文脈の更新が 19 行目、ベースモデルによる続行が 21〜22 行目である。本文はそれぞれを「8〜10 行目」「11〜12 行目」「9〜12 行目」「14 行目」「15〜16 行目」としている。
> - 記号が節によって異なる。圧縮ツールの引数は、Algorithm 1 では $(r_{\text{start}}, r_{\text{end}}, c, g)$、2.1 節と付録 A.2 では $(s, e, c, f)$ と書かれている。$\tau$ は、Algorithm 1 と式 (6) では軌跡を、2.3.2 節では残りの文脈予算の閾値を表す。
> - 付録 B.2 は「$\tau=0.05$。軌跡合成で用いた低温度（low-temperature）の設定に合わせた」と書く。本文で $\tau$ が定義されているのは 2.3.2 節の残り予算の閾値だけで、軌跡合成の温度はどこにも書かれていない（RL の rollout の温度は 1.0）。0.05 は、Algorithm 1 で $p_t$ が 1 に達する残り予算 5% と一致する。閾値の値を指している可能性があるが、原文どおり「低温度」と訳した。
> - 表 4 の「Avg ratio (%) 15.50」は、圧縮後と圧縮前の平均トークン数の比（1,496.52 / 9,657.93 ≈ 15.5%）に一致する。2.2.2 節の定義（選択範囲から取り除かれたトークンの割合）に直すと、平均の削減率は約 84.5% になる。これは、同節が「重要な情報を捨てうる」としてフィルタリングする 80% 超の基準を平均値が上回ることを意味する。表 4 の比率がどちらの定義によるかは説明されていない。
> - 付録 A.2 は「$(k-1)$ 番目の記憶イベント」に添字 $k$（$s_k, e_k, c_k, f_k$ と $C'_{t_k}$）を当てている。ターン $t_k$ のイベントは $k$ 番目にあたるので、番号が 1 つずれているように読める。
> - Split Quality のプロンプトでは、条件 1 の名前が `no_bad_assistant_content` なのに、JSON スキーマのキーは `no_bad_unmasked_assistant_content` になっている。
> - 付録 C.2 の事例 B は、Memory Management 2 の説明で「Summary 1」と書いている。直前の Memory Management 1 を指すと読める。
> - 図 3 の中の「Curriculum Leanring」は「Curriculum Learning」の誤記である。

## Abstract（要旨）

長期的な（long-horizon）ソフトウェアエンジニアリングエージェントは、限られた文脈予算（context budget）のもとで、長くノイズの多い対話履歴を管理しなければならないことが多い。
既存の記憶管理（memory management）手法は、典型的には静的な圧縮（compression）ワークフローに依存するか、圧縮のタイミングと粒度に硬直した制約を課す。
さらに、これらのアプローチは、トークン使用量を削減しつつ性能を向上させるために記憶管理とイシュー解決の能力を同時に最適化する、ということができていない。
我々は、ソフトウェアエンジニアリングエージェントにおける能動的（proactive）かつオンデマンドな記憶管理のための訓練フレームワークである SWE-MeM を提示する。
SWE-MeM は、軌跡の状態、タスクの進捗、残りの文脈予算に基づいて、いつ、何を、どのように圧縮するかをエージェントが決定できる柔軟な記憶ツールを提供する。
我々は、合成した能動的な記憶管理の軌跡と Memory-aware GRPO によってエージェントを訓練する。Memory-aware GRPO は、記憶を考慮した軌跡分割とステップ単位のクレジット割り当て（credit assignment）を通じて、記憶管理とイシュー解決を同時に最適化する。
SWE-Bench Verified において、SWE-MeM は 4B と 30B のモデルでそれぞれ 43.4% と 60.2% の解決率（resolve rate）を達成し、性能と効率の両面で既存の記憶管理のベースラインを上回る。

連絡先（Correspondence）: Shilin He, heshilin@bytedance.com
日付（Date）: 2026 年 6 月 30 日

## 1 Introduction（はじめに）

長期的なソフトウェアエンジニアリングのタスクを解くことは、実世界の開発ワークフローにエージェントを展開するうえで、依然として重要な課題である。
既存のエージェントの多くは ReAct 型の追記のみ（append-only）のパラダイム [48, 56, 57] に従っており、そこでは対話履歴が時間とともにモデルの文脈へ継続的に蓄積されていく。
ソフトウェアエンジニアリングのタスクでは、多数の中間的な行動と長い環境フィードバックがすぐに積み重なって長くノイズの多い軌跡となり、しばしば LLM の文脈ウィンドウ（context window）を超える [2, 31]。
履歴がウィンドウ内に収まる場合でも、エージェントは、長くノイズの多い軌跡からタスクに関連する文脈情報を効果的に活用するという課題に依然として直面する [7, 8]。
これらの課題により、長期的なコーディングエージェントにとって効果的な記憶管理が不可欠となる。

これらの課題に対処するため、最近の研究は、閾値ベースの要約、サブタスク単位の折り畳み（folding）、学習された圧縮ツールを通じた記憶管理を探究してきた [25, 41, 52]。
これらの手法は、閾値に達した時点で履歴全体を要約する [52]、完了したサブタスクを折り畳む [41]、あるいは直近の対話を保持しつつそれより前の会話ターンを圧縮する [25] ことによって、文脈長を削減する。

<figure>

![](../../raw/assets/2026-swe-mem/fig1.png)

<figcaption>図1: (a) と (b) におけるツールの柔軟性の問題の図解と、(c) における性能とコストの比較（右下ほど良い）。SWE-MeM は、他の手法と比べてトークン使用量とステップ使用量を比較的低く保ちながら、最も高い解決率を達成している。</figcaption>
</figure>

しかし、これらの手法は柔軟性と最適化の目的の両面で依然として限界がある。
第一に、固定された圧縮ルールは、ソフトウェアエンジニアリングの軌跡における情報の価値の不均一さに適応できない。図 1 の (a) と (b) に示すように、長い実行ログや無関係なファイルは典型的にはどの時点でも安全に圧縮できるが、軌跡の早い段階で調べたコードは後の推論にとって不可欠であり続けることがあり、したがって圧縮に適さない場合がある。
第二に、既存の手法は典型的に、タスク解決の効率ではなく文脈の削減を最適化する。
図 1(c) に示すように、既存の手法はエージェントのピーク文脈長を大幅に削減できるものの、総トークン使用量や対話ステップ数を劇的に増加させ、全体的な効率を悪化させる。

これらの問題に対処するため、我々は、長期的なソフトウェアエンジニアリングエージェントに能動的かつオンデマンドな記憶管理を備えさせる訓練フレームワークである SWE-MeM を提案する。
SWE-MeM は、現在の軌跡の状態に基づいて、いつ圧縮するか、何を圧縮するか、選択した内容をどのように要約するかをモデルが決定できる柔軟な記憶ツールを提供する。
我々は、能動的な記憶ツールの呼び出しを含むタスク解決の軌跡を合成することで、この能力を訓練する。そこでは、圧縮のタイミングと対象が、現在の軌跡の状態と残りのサブタスクに応じて選ばれる。
さらに、効果的なツール使用、特に能動的な圧縮の場面でのツール使用を教えるために、カリキュラム学習（curriculum learning）を適用する。
最後に、記憶を考慮した軌跡分割とステップ単位のクレジット割り当てを通じてイシュー解決と記憶管理を同時に改善する、Memory-aware GRPO を導入する。
SWE-Bench Verified において、SWE-MeM は Qwen3-4B-Instruct と Qwen3-Coder-30B-A3B でそれぞれ 43.4% と 60.2% の解決率を達成する。
32K の文脈予算のもとで、SWE-MeM はより高い解決率とより少ない対話ラウンド数で、既存の記憶管理手法を上回る。
まとめると、本研究の貢献は以下のとおりである。

- 現在の軌跡の状態に基づいて、いつ圧縮するか、何を圧縮するか、選択した文脈をどのように要約するかをエージェントが決定できる、適応的な記憶ツールを導入する。
- 記憶管理の軌跡合成、カリキュラム・ファインチューニング、Memory-aware GRPO を組み合わせ、記憶に関する意思決定とタスク解決の性能を同時に最適化する訓練フレームワークを提案する。
- SWE-Bench Verified での実験により、SWE-MeM が 32K の文脈予算のもとで、4B モデルで 43.4%、30B モデルで 60.2% の解決率を達成し、性能と効率の両面で既存の記憶管理手法を上回ることを示す。

## 2 Proposed Approach（提案手法）

<figure>

![](../../raw/assets/2026-swe-mem/fig2.png)

<figcaption>図2: 異なる記憶管理手法の比較。圧縮なし（No Compression）のモードでは、文脈が際限なく増大する。閾値トリガー（Threshold Trigger）型の手法は、予算に達した後にしか圧縮しない。固定スコープ（Fixed Scope）型の手法は、硬直した境界やウィンドウに依存する。これに対し、SWE-MeM は柔軟な文脈の範囲に対して、能動的かつオンデマンドな記憶管理を行う。</figcaption>
</figure>

### 2.1 Memory Management Tool Design（記憶管理ツールの設計）

我々は、固定された圧縮ワークフローに従うのではなく、いつ圧縮を呼び出すか、どの範囲（span）を圧縮するか、選択した範囲をどのように要約するかをソフトウェアエンジニアリングエージェントが決定できる、柔軟な記憶管理ツールを導入する。
この過程を定式化するため、時間ステップ $t$ における対話履歴を次のように表す。

$$
C_t = \big\langle p, \{a_1, o_1\}, \{a_2, o_2\}, \ldots, \{a_{t-1}, o_{t-1}\} \big\rangle, \tag{1}
$$

ここで、$p$ は初期の問題文であり、$a_i$ はステップ $i$ におけるエージェントの行動（テキストによる応答、推論の過程、ツール呼び出しを含む）を表し、$o_i$ は対応する環境の観測を表す。観測には、エージェントに進捗と文脈予算を知らせるため、現在のステップ番号や残りの文脈長などのメタデータが付加される。

エージェントは、圧縮が必要だと判断すると、このツール `compress(analysis, start_step_number, end_step_number, content , remaining_work)` を呼び出して、能動的に文脈を管理できる。
ここで、`analysis` は、現在の進捗と未解決のサブタスクに対するエージェントの評価を記録する。
引数 `start_step_number` $s$ と `end_step_number` $e$ は、圧縮する範囲を指定する。
選択される範囲には通常、長いログ、失敗した探索的な試行、もはや古くなった中間的な観測など、後続の推論にとってあまり必要でない生の詳細が含まれる。
`content` 引数 $c$ は、選択された範囲を置き換える要約であり、不要な詳細を取り除くことで元の情報をより高いレベルの抽象へと蒸留する。
最後に、`remaining_work` $f$ は残りのサブタスクを記録し、タスクの連続性を保つために軌跡の末尾に追記される。
エージェントは、これらすべての引数をこの順序で、1 回の LLM 呼び出しの中で生成する。
図 2 に示すように、結果として得られる圧縮操作は次のとおりである。

$$
\begin{aligned}
C_t \xrightarrow{\ \mathcal{M}\ } C_t' = \big\langle\, & p, \{a_1, o_1\}, \ldots, \{a_{s-1}, o_{s-1}\}, \\
& \{c\}, \{a_{e+1}, o_{e+1}\}, \ldots, \\
& \{a_{t-1}, o_{t-1}\}, \{f\} \big\rangle.
\end{aligned} \tag{2}
$$

ここで、$\{c\}$ と $\{f\}$ は軌跡に挿入される記憶エントリである。
この操作は、時間的な順序を保ちながら文脈長を削減する。
圧縮された要約自体も後のツール呼び出しで選択できるため、エージェントは軌跡が伸びるにつれて内容を段階的に圧縮していくことができる。

### 2.2 Curriculum Fine-Tuning with Synthesized Memory Trajectories（合成した記憶軌跡によるカリキュラム・ファインチューニング）

記憶ツールを使うモデルの能力を立ち上げる（bootstrap）ため、我々は、図 3(a) に示すように、タスク解決の軌跡に記憶管理の行動を付け加える合成ワークフローと、能動的な記憶行動を学習するためのカリキュラム訓練手法を開発する。

<figure>

![](../../raw/assets/2026-swe-mem/fig3.png)

<figcaption>図3: SWE-MeM の訓練パイプラインの概要。</figcaption>
</figure>

#### 2.2.1 Trajectory Synthesis Workflow（軌跡合成ワークフロー）

合成ワークフローは、付録 A の Algorithm 1 に詳述するように、タスク解決と記憶管理を単一の軌跡生成の過程に統合する。
各タスクについて、rollout は、より強力な教師モデルではなく、後でファインチューニングされるベースモデルによって生成される。
プロプライエタリモデルは、能動的トリガーの判定と記憶ツールの引数の合成にのみ使われ、タスク解決の行動は生成しない。

図 3(a) に示すように、我々は、各ステップの開始時に記憶管理の行動をトリガーすべきかどうかを確認する文脈モニタ（context monitor）を導入する。
既存のエージェントの文脈圧縮の設計 [41, 52, 55] に着想を得て、このモニタは、異なる軌跡の状態を反映する 2 種類のトリガー、すなわち文脈予算の圧迫（context budget pressure）と能動的な記憶管理（proactive memory management）を考慮する。

**文脈予算の圧迫（Context budget pressure）** は、確率的なスケジューラによって、差し迫った文脈の枯渇に対処する。
トリガーの確率は、残りの文脈予算が 20% を下回ると非ゼロになり、予算が縮むにつれて単調に増加し、予算が 5% しか残っていないときに 1 に達する。
このソフトなスケジュールは、単一の硬い閾値への依存を避け、合成される軌跡の多様性を高める。

**能動的な記憶管理（Proactive memory management）** は、文脈予算が危機的になる前に圧縮が有用となる状況を捉える。
我々は 3 つのよくあるケースを考える。*サブタスクの完了（subtask completion）* は、あるサブタスクが完了し、その結論だけを保持すればよい場合である。*低い情報密度（low-information density）* は、直近のラウンドが冗長なログ、無関係なファイルの探索、その他の価値の低い内容で占められている場合である。*焦点の劣化（focus degradation）* は、長くノイズの多い文脈がエージェントの注意を中核のイシューからそらしているように見え、圧縮によってよりクリーンな作業文脈を回復できる場合である。
これらのケースは LLM-as-a-judge の仕組みで検出される。詳細なプロンプトは付録 A.3 に示す。

トリガーが発火すると、プロプライエタリモデルを呼び出して記憶ツールの引数を合成し、式 2 に従って軌跡を再編成する。
合成ループは、エージェントがイシューを解決するか、最大ステップ数に達するまで続く。

#### 2.2.2 Trajectory Quality Filtering（軌跡の品質フィルタリング）

まず、テストケースに合格した軌跡のみを残すために、棄却サンプリング（rejection sampling）を行う。
次に、訓練の前に、残した軌跡のノイズを減らすためのルールベースのデータクリーニングのパイプラインを適用する。
局所的な欠陥が検出されたときに軌跡全体を破棄するのではなく、訓練中に該当するアシスタントのメッセージをマスクし、残りの有効なステップが学習に寄与できるようにする。

我々のフィルタリング基準は、圧縮の行動と非圧縮の行動の両方を対象とする。
圧縮の行動については、3 種類の欠陥をフィルタリングする。
**(1) 不適切な削減率（Inappropriate reduction ratio）** は、攻撃的すぎるか保守的すぎる要約を指す。
削減率を、選択された範囲から取り除かれたトークンの割合と定義する。80% を超える比率は重要な情報を捨ててしまう可能性があり、一方で 20% 未満の比率は実用上の文脈の節約がほとんどない。
**(2) 短すぎる圧縮範囲（Overly short compression ranges）** は、選択された範囲が短すぎて意味のある文脈の削減が得られず、圧縮のオーバーヘッドが無駄になる場合に生じる。
**(3) 範囲の不整合（Range misalignment）** は、要約が宣言されたステップ範囲に忠実に対応しておらず、範囲内のステップの情報を欠落させているか、範囲外の内容を取り込んでいる場合に生じる。
非圧縮の行動については、**(4) 低品質な応答（low-quality responses）** が、空の出力、繰り返し、途中での打ち切り、または無効なツール呼び出しを含む場合にマスクされる。

#### 2.2.3 Curriculum Learning for Proactive Memory Management（能動的な記憶管理のためのカリキュラム学習）

能動的な記憶管理は、軌跡の状態についての意味的な判断に依存するため、予算ベースの圧縮よりも学習が難しい。
我々の合成した軌跡では、予算ベースの圧縮のほうが出現頻度も高く、より明確な文脈予算のパターンに従う。
そこで我々は、2 段階のカリキュラム学習の戦略を採用し、記憶ツールを使うようにモデルを段階的に訓練する。その際、特に能動的な記憶行動に重点を置く。

第 1 段階では、フィルタリング済みの合成軌跡の全体でモデルを訓練し、記憶管理ツールの基本的な形式と使用パターンを学習させる。
第 2 段階では、より的を絞った訓練セットを構築する。
まず、能動的な記憶管理の行動を含む軌跡を抽出し、ルーブリックに基づくフィルタリングを適用して高品質な事例を残す。
ルーブリックは、`analysis` 引数が現在の進捗と残りのサブタスクについて妥当な評価を与えているかどうか、そして圧縮された内容がこの分析と整合しているかどうかを確認する。
詳細なルーブリックのプロンプトは付録 A.3 に示す。

破滅的忘却（catastrophic forgetting）を避けるため、同数の問題もサンプリングし、予算ベースの圧縮を含むが能動的な記憶管理の行動は含まない軌跡を収集する。
この部分集合には、別の LLM-as-a-judge のルーブリック群を適用し、エージェントが圧縮された情報を後続のステップで効果的に使い、記憶にすでに保存されている情報を冗長に再取得することを避け、確立された解決の道筋に沿って一貫した進捗を保っている軌跡を残す。
最後に、残した 2 つの部分集合を統合して第 2 段階のファインチューニングに用い、予算ベースの圧縮と一般的なタスク解決の能力を保ちながら、能動的な記憶行動を強化する。

### 2.3 Reinforcement Learning with Memory-aware GRPO（Memory-aware GRPO による強化学習）

記憶ツールの使用とタスク解決の性能をさらに改善するため、我々は Memory-aware GRPO を提案する。これは、圧縮によってもたらされる状態の変化を考慮しながら、強化学習（reinforcement learning, RL）でモデルを訓練するものである。

#### 2.3.1 Memory-aware Trajectory Splitting and Loss Aggregation（記憶を考慮した軌跡分割と損失の集約）

標準的なエージェントの訓練とは異なり、記憶管理は圧縮のたびに文脈の状態を変化させ、したがって後続のすべての行動に影響する。
推論時の状態との一貫性を保つため、図 3(b) に示すように、圧縮ステップで軌跡にチェックポイントを置き、各 rollout を部分軌跡（sub-trajectory）に分割する。
各部分軌跡は、元の圧縮されていない履歴からではなく、エージェントがオンラインで観測するであろう圧縮済みのプレフィックスそのものから始まる。
訓練と推論の一貫性を保つため、教師ありファインチューニング（supervised fine-tuning, SFT）の際にも同じ分割方式を適用する。

記憶管理は、軌跡の長さに大きなばらつきをもたらしうる。
長い rollout には、非効率な記憶操作が含まれていたり、圧縮された記憶にすでに保存されている結論を再利用できていなかったりすることが多く、それが冗長な行動につながる。
長さのばらつきによる最適化のバイアスを減らすため、損失を 2 段階で集約する。まず、分割後の各 rollout の中で有効なトークン単位の目的関数をすべて平均し、次に、得られた rollout 単位の損失をバッチ全体で平均する。

$$
\mathcal{L} = \frac{1}{S}\sum_{i=1}^{S} \bar{\ell}_i, \qquad \bar{\ell}_i = \frac{1}{N_i}\sum_{k=1}^{K_i}\sum_{t=1}^{T_{i,k}} \ell_{i,k,t}. \tag{3}
$$

ここで、$\ell_{i,k,t}$ は rollout $i$ の部分軌跡 $k$ におけるトークン $t$ の損失を表し、$N_i = \sum_{k=1}^{K_i} T_{i,k}$ は rollout $i$ の有効なトークン数、$S$ はバッチ内の rollout サンプルの数である。
この集約により、各 rollout が最終的な損失に等しく寄与するようになり、極端に長い軌跡や部分軌跡が過大な重みを持つことを防ぐ。

#### 2.3.2 Step-level Mask for Credit Assignment（クレジット割り当てのためのステップ単位のマスク）

標準的な GRPO は、最終的なイシュー解決の結果のみを用いてクレジットを割り当てる。そのため、失敗した軌跡の中の適切な中間行動を誤って罰したり、成功した軌跡の中の問題のある行動に報酬を与えたりしうる。
そこで我々は、複数軌跡の GRPO（multi-trajectory GRPO）の上にステップ単位のクレジットマスクを導入する。
ステップ単位のマスクは、対応するアシスタントのステップに属するすべてのトークンにブロードキャストされる。

$$
\begin{aligned}
\mathcal{J}_{\text{MGRPO}} = \mathbb{E}_i \Bigg[ & \frac{1}{M_i}\sum_{k=1}^{K_i}\sum_{t=1}^{T_{i,k}} m_{i,k,t} \times \min\Big( r_{i,k,t}\hat{A}_i, \\
& \operatorname{clip}(r_{i,k,t}, 1-\epsilon, 1+\epsilon)\hat{A}_i \Big) \Bigg],
\end{aligned} \tag{4}
$$

ここで、$m_{i,k,t} \in \{0, 1\}$ はトークン $t$ が勾配に寄与するかどうかを制御し、$M_i = \sum_{k=1}^{K_i}\sum_{t=1}^{T_{i,k}} m_{i,k,t}$ は rollout $i$ の中のマスクされていない有効なトークンの数である。
クレジット割り当てを改善するため、以下のマスキング戦略を設計する。

**記憶管理の行動の品質（Memory Management Action Quality）.** 成功した軌跡の中の記憶管理の行動について、ルールベースのチェックを適用し、不適切な削減率、短すぎる圧縮範囲、範囲の不整合をもつステップをマスクする。

**遅すぎる記憶管理（Late Memory Management）.** 成功した軌跡では、残りの文脈予算が閾値 $\tau$ を下回った後になって初めて実行された記憶管理の行動もマスクする。
これらの行動は現在の軌跡では失敗を引き起こさないかもしれないが、記憶管理を遅らせており、将来の類似した状態でオーバーフローのリスクを高める可能性がある。

**オーバーフローによる失敗（Overflow Failure）.** 軌跡が記憶管理ツールを呼び出さないまま長さの上限を超えた場合、ペナルティの割り当てには最後のオーバーフローした部分軌跡のみを残す。
この部分軌跡の中で、ペナルティは、残りの予算が閾値 $\tau$ を下回っているにもかかわらずエージェントが記憶管理を呼び出せていないステップにのみ割り当てられる。

**無効なツールの使用（Invalid Tool Usage）.** 成功した軌跡について、形式や引数が誤った呼び出しなどの無効なツール呼び出しや、繰り返しの内容を含むステップをマスクし、望ましくない行動パターンをモデルが学習することを防ぐ。

## 3 Experiment（実験）

### 3.1 Datasets（データセット）

訓練データとして、SWE-ReBench [1] と SWE-Gym [29] から軌跡を構築する。
SFT では、タスク解決の rollout は Qwen3-Coder-30B-A3B によって生成され、GPT-5.1 は能動的トリガーの判定と記憶ツールの引数の合成にのみ使われる。
RL では、これら 2 つのデータセットから、サンプリングした rollout がすべて成功またはすべて失敗した問題を取り除くことでフィルタリング済みの訓練グループを構築し、結果に自明でないばらつきのあるグループを残す。
評価には主に SWE-Bench Verified [21] を使い、解決率を主要な指標として報告する。
データセットの構築と評価設定の詳細は付録 B.1 に示す。

**表1**: SWE-Bench Verified における性能の比較。（訳注: 原表で緑色に強調されている行は SWE-MeM SFT+RL の 2 行である。）

| Method/Model（手法／モデル） | Base Model（ベースモデル） | Base Scaffold（ベースの scaffold） | Length Limitation（長さの上限） | Resolve Rate（解決率） |
|---|---|---|---|---|
| **<10B Models** | | | | |
| *ReAct Agent* | | | | |
| Qwen3-4B-Instruct [43] | – | OpenHands | 128k | 7.0 |
| Qwen3-4B-Instruct [43] | – | OpenHands | 32k | 5.2 |
| SWE-Gym-7B [29] | Qwen2.5-Coder-7B | OpenHands | 32k | 10.6 |
| SWE-Mirror-7B [46] | Qwen2.5-Coder-7B | MOpenHands | 128k | 22.8 |
| SWE-Dev-7B [45] | Qwen2.5-Coder-7B | OpenHands | 160k | 23.4 |
| SWE-Master-4B [37] | Qwen3-4B-Instruct | OpenHands | 128k | 33.4 |
| Klear-Agent-8B [47] | Qwen3-8B | Mini-SWE-agent-plus | 64k | 39.4 |
| SWE-Lego-8B [42] | Qwen3-8B | OpenHands | 128k | 42.2 |
| *SWE-MeM* | | | | |
| SWE-MeM SFT | Qwen3-4B-Instruct | OpenHands | 32k | 41.6 |
| SWE-MeM SFT+RL | Qwen3-4B-Instruct | OpenHands | 32k | 43.4 |
| **∼30B Models** | | | | |
| *ReAct Agent* | | | | |
| Qwen3-Coder-30B-A3B [43] | – | OpenHands | 256k | 51.6 |
| Qwen3-Coder-30B-A3B [43] | – | OpenHands | 32k | 38.6 |
| Seed-OSS-36B [3] | – | OpenHands | 327K | 55.2 |
| SWE-Gym-32B [29] | Qwen2.5-Coder-32B | OpenHands | 32k | 20.6 |
| SWE-Dev-32B [45] | Qwen2.5-Coder-32B | OpenHands | 160k | 36.6 |
| DeepSWE-32B [44] | Qwen3-32B | R2E-Gym | 128k | 43.2 |
| SWE-Mirror-32B [46] | Qwen2.5-Coder-32B | MOpenHands | 128k | 52.2 |
| SWE-Lego-32B [42] | Qwen3-32B | OpenHands | 128k | 52.6 |
| *Summary Agent* | | | | |
| Threshold-Compression Agent [25] | Qwen2.5-Coder-32B | OpenHands | 64k | 53.8 |
| Threshold-Compression Agent [41] | Seed-OSS-36B | OpenHands | 32k | 55.0 |
| *Folding Agent* | | | | |
| SWE-Compressor (150 step) [25] | Qwen2.5-Coder-32B | OpenHands | 64k | 54.8 |
| SWE-Compressor [25] | Qwen2.5-Coder-32B | OpenHands | 64k | 57.8 |
| Context Folding [41] | Seed-OSS-36B | OpenHands | 32k | 58.0 |
| *SWE-MeM* | | | | |
| SWE-MeM Workflow-only | Qwen3-Coder-30B-A3B | OpenHands | 32k | 58.4 |
| SWE-MeM SFT (150 step) | Qwen3-Coder-30B-A3B | OpenHands | 32k | 57.2 |
| SWE-MeM SFT | Qwen3-Coder-30B-A3B | OpenHands | 32k | 58.8 |
| SWE-MeM SFT+RL | Qwen3-Coder-30B-A3B | OpenHands | 32k | 60.2 |

### 3.2 Baselines（ベースライン）

我々の手法を、3 つのカテゴリのベースラインと比較する。

(1) **記憶管理なしの ReAct エージェント（ReAct agents without memory management）.** これらのベースラインをモデル規模でグループ分けし、10B 未満のモデルと約 30B パラメータのモデルを含める。
これらのエージェントは標準的な ReAct 型のフレームワークに従い、報告されている文脈上限のもとで、明示的な記憶管理なしに動作する。

(2) **既存の記憶管理のアプローチ（Existing memory management approaches）.** 文脈の閾値に達した後に文脈を圧縮する要約ベースのエージェントに加えて、SWE-Compressor [25] と Context Folding [41] を含む折り畳みベースのエージェントと比較する。

(3) **我々の手法の変種（Our method variants）.** いくつかの構成を報告する。
Workflow-only は、提案する記憶管理のワークフローを推論時に未調整のベースモデルへ適用するもので、圧縮の判断と引数はプロプライエタリモデルが与える。
SFT と RL の変種は訓練済みのモデルであり、異なるステップ上限のもとで評価する。
これらの設定では、推論時にワークフローやプロプライエタリモデルによる支援は一切使わない。

**表2**: SWE-Bench Verified における、記憶管理のベースラインとの効率の比較。相対使用量（Relative usage）は、トークン使用量とステップ数を、ReAct エージェントを用いた対応するベースモデルのものと比較した値である。（訳注: 原表で緑色に強調されている行は SWE-MeM SFT+RL である。）

| Method（手法） | Resolve Rate（解決率） | Token Usage（トークン使用量） | Relative Token Usage（相対トークン使用量） | Average Step（平均ステップ数） | Relative Step Usage（相対ステップ使用量） |
|---|---|---|---|---|---|
| *Summary Agent* | | | | | |
| Threshold-Compression Agent [25] | 53.8 | 5.18M | 203.9% | – | – |
| Threshold-Compression Agent [41] | 55.0 | – | – | 74.9 | 151.3% |
| *Folding Agent* | | | | | |
| SWE-Compressor (150 step) | 54.8 | 1.89M | 96.4% | – | – |
| SWE-Compressor | 57.8 | 2.75M | 108.3% | – | – |
| Context Folding | 58.0 | – | – | 96.5 | 194.9% |
| *SWE-MeM* | | | | | |
| SWE-MeM SFT (150 step) | 57.2 | 0.86M | 89.0% | 73.8 | 119.6% |
| SWE-MeM SFT | 58.8 | 0.91M | 94.7% | 77.7 | 125.9% |
| SWE-MeM SFT+RL | 60.2 | 0.91M | 94.7% | 77.0 | 123.5% |

### 3.3 Experiment Details（実験の詳細）

ベースモデルとして Qwen3-Coder-30B-A3B-Instruct [43] と Qwen3-4B-Instruct-2507 [43] を使い、ベースの scaffold として OpenHands [48] を採用し、データ収集、SFT、RL を通じて文脈上限を 32,768 トークンに設定する。
記憶管理のベースラインについては、括弧内で異なるステップ予算が明示されていない限り、すべての手法で最大 500 ステップを用いる。
ハイパーパラメータとフレームワークの設定の詳細は付録 B.2 に示す。

### 3.4 Main Results（主要な結果）

表 1 は、SWE-MeM が 32K というコンパクトな文脈上限を使いながら、4B と 30B の両方の規模でベースライン手法を一貫して上回ることを示している。

4B の設定では、SWE-MeM は SFT で 41.6% の解決率を達成し、RL でさらに 43.4% に向上して、この規模帯のすべての ReAct エージェントのベースラインを上回る。
30B の設定では、Workflow-only の構成がモデルの訓練なしですでに 58.4% の解決率を達成し、Qwen3-Coder-30B-A3B を用いた ReAct のベースラインを大きく上回る。
この結果は、効果的な記憶管理が、タスクに関連する情報を保持し、冗長な文脈による注意散漫を減らすことで、長期的なソフトウェアエンジニアリングエージェントを大幅に改善しうることを示している。

訓練後、SWE-MeM はこの能力を内在化し、推論時に外部の LLM にもはや依存しない。
SWE-MeM は SFT 後に 58.8% に達し、RL 後にさらに 60.2% へと向上する。
既存の記憶管理のベースラインと比べても、SWE-MeM は一貫した向上を示す。
例えば、それぞれのベースモデルに対して、Context Folding は Seed-OSS-36B を 55.2% から 58.0% に改善するのに対し、SWE-MeM は Qwen3-Coder-30B-A3B を 51.6% から SFT 後に 58.8%、RL 後に 60.2% へと改善する。
これらの結果は、SWE-MeM が従来の記憶管理のアプローチよりも大きな性能向上をもたらすことを示唆している。

## 4 Further Analysis（さらなる分析）

### 4.1 Efficiency Analysis（効率の分析）

表 2 は、SWE-MeM の効率を既存の記憶管理のベースラインと比較している。
SWE-MeM は、これらの手法の中で報告されている最も少ないトークン使用量を保ちながら、最良のタスク性能を達成する。
また、SWE-MeM は対応するベースの ReAct エージェントよりも少ないトークンしか使わないのに対し、競合する記憶管理のベースラインの多くは総トークン使用量を増加させる。
対話ステップ数の観点では、SWE-MeM がもたらす増加は Context Folding のそれよりもはるかに小さい。これは、選択的な圧縮が、過剰な再探索を誘発することなく冗長な文脈を削減できることを示唆している。

<figure>

![](../../raw/assets/2026-swe-mem/fig4.png)

<figcaption>図4: アブレーション研究。(a) Qwen3-4B-Instruct、(b) Qwen3-Coder-30B-A3B。</figcaption>
</figure>

### 4.2 Ablation Study（アブレーション研究）

**エージェントの設計の分析（Analysis of agent design）.** 能動的な呼び出しを閾値ベースの呼び出しに置き換えること、および選択的な範囲選択を軌跡全体の要約に置き換えることによって、2 つの主要な設計上の選択をアブレーションする。
元の 4B モデルは弱すぎて安定した比較ができないため、この分析は 30B モデルでのみ行う。
図 4 に示すように、これらの構成要素を置き換えると、解決率はそれぞれ 57.4% と 57.6% に低下する。
これらの結果は、能動的なトリガーと選択的な圧縮の両方が最終的な性能にとって重要であることを示している。

**カリキュラム SFT の分析（Analysis of curriculum SFT）.** 図 4 に示すように、カリキュラム SFT は、第 1 段階の訓練のみを用いるバニラ SFT を一貫して上回る。
具体的には、4B モデルで性能が 40.8% から 41.6% に向上する。
さらに、第 2 段階の訓練の後、能動的なツール呼び出しの割合は、4B モデルで 2.3% から 16.1% に、30B モデルで 4.1% から 14.7% に増加する。
これらの結果は、カリキュラムの設計が、モデルがより信頼できる能動的な記憶管理の行動を獲得するのに役立つことを示唆している。

**Memory-aware GRPO の分析（Analysis of memory-aware GRPO）.** 図 4 はまた、Memory-aware GRPO が両方のモデル規模でバニラ GRPO を一貫して上回ることも示している。これは、損失の集約とステップ単位のクレジット割り当てが長期的な RL の訓練にとって重要であることを示している。

**記憶管理ツールの分析（Analysis of memory management tool）.** さらに、訓練済みモデルを、バニラの OpenHands の scaffold と、はるかに大きい 256K の文脈ウィンドウで動かすことで、記憶ツールの役割を評価する。
記憶ツールを取り除くと、より大きな文脈予算があるにもかかわらず、モデルの性能は大幅に低下する。
例えば 4B モデルでは、解決率が 43.4% から 37.6% に低下する。
これは、記憶管理が文脈のオーバーフローを防ぐ以上のことをしていることを示唆している。すなわち、記憶管理はよりクリーンな作業文脈を提供し、モデルがタスクに関連する情報をより効果的に使うのを助ける。

### 4.3 Performance on SWE-Bench Multilingual and SWE-Bench Pro（SWE-Bench Multilingual と SWE-Bench Pro での性能）

さらに、2 つの難しいベンチマークで SWE-MeM を評価する。複数のプログラミング言語を対象とする SWE-Bench Multilingual と、より長期的なタスクに焦点を当てた SWE-Bench Pro である。
表 3 に示すように、SWE-MeM は、ベースモデルが用いる 256K の文脈の代わりにより小さい 32K の文脈上限を用いているにもかかわらず、両方のベンチマークで対応するベースモデルを一貫して上回る。
SWE-Bench Multilingual では、SWE-MeM は解決率を、4B モデルで 7.3% から 19.0% に、30B モデルで 35.3% から 40.7% に改善する。
SWE-Bench Pro では、解決率を、4B モデルで 2.6% から 15.2% に、30B モデルで 28.9% から 31.7% に改善する。
全体として、これらの結果は、学習された記憶管理の行動がプログラミング言語をまたいで転移し、より長期的なソフトウェアエンジニアリングのタスクで明確な向上をもたらすことを示唆している。

**表3**: SWE-bench Multilingual と SWE-bench Pro における実験結果。

| Model（モデル） | Length（長さ） | SWE-bench Multilingual | SWE-bench Pro |
|---|---|---|---|
| Qwen3-4B-Instruct | 256k | 7.3 | 2.6 |
| + SWE-MeM SFT+RL | 32k | 19.0 | 15.2 |
| Qwen3-Coder-30B-A3B | 256k | 35.3 | 28.9 |
| + SWE-MeM SFT+RL | 32k | 40.7 | 31.7 |

## 5 Related Work（関連研究）

### 5.1 Software Engineering Agents（ソフトウェアエンジニアリングエージェント）

ソフトウェアエンジニアリングエージェントは、原子的なコード知能の能力 [9, 11, 12, 16] から、自律的なリポジトリレベルのタスク解決 [22, 48, 54, 56] へと至る進展の上に築かれている。
初期の能力志向の研究は、コード生成 [4, 14, 19, 23, 34, 49]、プログラム修復 [30, 53]、コード要約 [61]、コード翻訳 [27, 32]、テスト生成 [6, 13] を研究した。
その後の研究は、SWE-Agent、OpenHands、Agentless [48, 54, 56] を含む、リポジトリレベルの問題解決のためのエージェント scaffold を探究した。
最近の進展は、より豊かな環境とベンチマーク [1, 5, 17, 24, 29, 40]、合成データの生成 [38, 46]、訓練または事後訓練（post-training）のレシピ [18, 37, 44, 51]、報酬モデルや検証器（verifier）のモデリング [36] を通じて、このエコシステムをさらに広げてきた。
本研究は、長期的なタスクの間にエージェントが文脈をどのように管理するよう学習すべきかに焦点を当てることで、この系統の研究を補完する。

### 5.2 Memory Management（記憶管理）

数百ラウンドにわたる長期的なタスクを遂行するエージェントにとって、長い対話の中で文脈を管理することは決定的に重要である。
初期の文脈圧縮のアプローチには、LLMLingua [20] のようなトークン単位の枝刈り手法や、DietCode [63] のようなコードに特化した最適化手法がある。
より最近の研究は、文脈の折り畳みとルーティング、適応的な枝刈り、軌跡の削減、ウェブエージェントとコーディングエージェントのための訓練可能な文脈管理の方策によって、この方向性を急速に広げてきた [10, 15, 26, 35, 39, 50, 58, 60, 62]。
例えば、AgentDiet は軌跡の削減を通じて無駄な履歴の文脈を減らし [55]、Context Folding は完了したサブタスクを要約して、関連する行動の記録をコンパクトなサブタスク単位の要約へと折り畳む [41]。一方 SWE-Compressor は、文脈の維持をエージェントの意思決定過程に統合された呼び出し可能なツールへと格上げし [25]、受動的または外部からトリガーされる圧縮戦略に頼るのではなく、適切なマイルストーンで SWE エージェントが文脈を圧縮できるようにする。
固定された圧縮ルールや外部で定義されたスコープに依存する手法と比べて、SWE-MeM は、現在の軌跡の状態のもとで、いつ圧縮するか、どの範囲を圧縮するか、それをどのように書き直すかを決定するようにエージェントを訓練する。

## 6 Conclusion（結論）

我々は、能動的かつオンデマンドな記憶管理を備えた長期的な SWE エージェントを訓練するためのフレームワークである SWE-MeM を提示した。
柔軟な記憶ツール、カリキュラム・ファインチューニングを伴う軌跡合成、Memory-aware GRPO を組み合わせることで、SWE-MeM は SWE-Bench Verified での性能を改善し、我々の 4B と 30B の設定でそれぞれ 43.4% と 60.2% の解決率に達し、性能と効率の両面で既存の記憶管理のベースラインを上回った。

> 訳注: 参考文献（References, [1]〜[63]）は訳していない。

## Appendix（付録）

## A Method Details（手法の詳細）

### A.1 Detailed Trajectory Synthesis Procedure（軌跡合成の詳細な手順）

Algorithm 1 は、記憶管理の軌跡の構築過程の詳細を記述している。

**ステップ 1: rollout の初期化（Step 1: rollout initialization）.** Algorithm 1 の 1〜3 行目に示すように、タスクのプロンプト、文脈予算、ベースモデル、プロプライエタリモデルで会話を初期化する。
ベースモデルは通常のエージェントの行動を生成するために使われ、一方プロプライエタリモデルは、合成による記憶管理の教師信号が必要な場合にのみ使われる。
この分離により、記憶管理のみに焦点を当てた訓練が SWE エージェントの能力をどの程度改善できるかを調べることができる。

**ステップ 2: 予算を考慮した監視（Step 2: budget-aware monitoring）.** 各反復で、4〜7 行目は残りのトークン予算を計算し、圧縮の状態を初期化する。
次に 8〜12 行目で、現在の軌跡が圧縮をトリガーすべきかどうかを決定する。
8〜10 行目の予算圧迫の分岐は、残り予算のヒューリスティック $p_t$ を使い、文脈ウィンドウが飽和するにつれて圧縮の確率を高める。
11〜12 行目の能動的な分岐は、Context Monitor Prompt で記述される文脈監視の判定器に対応する。
実際には、すべてのステップでプロプライエタリモデルに問い合わせることはしない。そうすると不必要にコストがかかり、またほとんどの状態は能動的な圧縮に適さないからである。
さらに、*サブタスクの終了（subtask end）* のようなパターンが常に有用な圧縮の機会をもたらすとは限らない。そのため、SFT のコールドスタートの目的は、あらゆる可能なタイミングで圧縮を強制することではなく、モデルを基本的な能動的記憶の行動に触れさせることだけである。
具体的には、各ステップの後に 10% の確率でのみモニタに問い合わせ、残りの 90% のステップでは能動的な圧縮を完全にスキップする。
さらに、直近の 5 ステップ以内に何らかの圧縮の行動が発生していた場合もモニタをスキップする。このような短い間隔で必要な条件が再び満たされる可能性は低いからである。

**ステップ 3: 圧縮の行動の生成（Step 3: generating the compression action）.** 圧縮がトリガーされると、9〜12 行目でプロプライエタリモデルを呼び出し、実際の記憶操作を合成する。
生成される行動は Compression Generation Prompt に従う。このプロンプトはモデルに、まず現在の進捗を振り返り、次に有効な圧縮範囲を選択し、最後に圧縮された内容を具体的な残作業の計画とともに生成するよう求める。
このプロンプトはまた、以前に圧縮された範囲との不正な重なりを避けるよう選択範囲を制約し、後のラウンドでまだ必要となる情報を保持するようモデルに促す。

**ステップ 4: 軌跡の更新と rollout の継続（Step 4: trajectory update and continued rollout）.** 圧縮の行動が合成された後、14 行目で式 (2) に従って文脈を更新する。すなわち、選択された元の範囲が圧縮された内容で置き換えられ、残作業の要約が軌跡の末尾に追記される。
圧縮がトリガーされない場合、15〜16 行目で単にベースモデルに環境との対話を続けさせる。

**Algorithm 1**: Trajectory Synthesis with Dynamic Memory Management（動的な記憶管理を伴う軌跡合成）

```text
 1: Input: Task prompt p, max iterations T, context budget B, local trajectory model M_local, Proprietary model M_prop
 2: Output: Trajectory with memory operations τ = {(c_t, a_t, o_t)}_{t=1}^{T}
 3: Initialize conversation history C ← [system, user(p)]
 4: while t < T and task not finished do
 5:     t ← t + 1, compute remaining budget b_t ← B − TokenCount(C)
 6:     NeedCompression ← False
 7:     (r_start, r_end, c, g) ← (∅, ∅, ∅, ∅)
 8:     // Category (1): Context Budget Pressure
 9:     p_t ← max(0, min(1, (0.20B − b_t)/(0.15B)))
10:     if u ∼ Uniform(0, 1) < p_t then
11:         (r_start, r_end, c, g) ← M_prop.BuildCompression(C)
12:         NeedCompression ← True
13:     else if NeedProactiveCompression(C) then
14:         // Category (2): Proactive Memory Management
15:         (r_start, r_end, c, g) ← M_prop.BuildProactiveCompression(C)
16:         NeedCompression ← True
17:     end if
18:     if NeedCompression then
19:         Update C following Eq. (2)
20:     else
21:         a_t ← M_local(C) {Generate agent action}
22:         Execute a_t, observe o_t, update C and record τ
23:     end if
24: end while
25: return τ
```

### A.2 Detailed Description of Trajectory Splitting Process（軌跡分割の過程の詳細な説明）

記憶管理の行動がターン $\{t_1, \ldots, t_{K-1}\}$ でトリガーされ、$0 = t_0 < t_1 < \cdots < t_{K-1} < t_K = T$ であるとする。
$(k-1)$ 番目の記憶イベントについて、$(s_k, e_k, c_k, f_k)$ を、選択された圧縮範囲と生成された要約の内容とする。ここでは元の範囲 $[s_k, e_k]$ のみが置き換えられ、この範囲の外にある、それより前と後の文脈はすべて保持される。
式 (2) に従うと、このイベントの後の圧縮された文脈は、次のように直接書ける。

$$
\begin{aligned}
C'_{t_k} = \big\langle\, & p, \{a_1, o_1\}, \ldots, \{a_{s_k-1}, o_{s_k-1}\}, \\
& \{c_k\}, \{a_{e_k+1}, o_{e_k+1}\}, \ldots, \\
& \{a_{t_k-1}, o_{t_k-1}\}, \{f_k\} \big\rangle.
\end{aligned} \tag{5}
$$

したがって、圧縮後の文脈は、プレフィックス全体の単一の要約ではなく、保持された元のラウンド、1 つの圧縮された範囲、追記された残作業の項目からなる混合した軌跡である。

次に、各セグメントがデプロイ時に利用可能となるであろう圧縮済みの文脈そのものから始まるように、圧縮の境界で rollout 全体を分割する。

$$
\begin{aligned}
\tau^{(k)} = \big\langle\, & C'_{t_{k-1}}, \{a_{t_{k-1}+1}, o_{t_{k-1}+1}\}, \ldots, \\
& \{a_{t_k}, o_{t_k}\} \big\rangle, \qquad k = 1, \ldots, K.
\end{aligned} \tag{6}
$$

同値的に、rollout 全体は $\tau \xrightarrow{\ \mathcal{M}\ } \{\tau^{(1)}, \tau^{(2)}, \ldots, \tau^{(K)}\}$ と分解される。
ここで $C'_{t_{k-1}}$ は、直前の記憶イベントで選択された元の範囲の外にある圧縮されていない文脈をすでにすべて保持しているので、各部分軌跡 $\tau^{(k)}$ は、エージェントがオンラインで実際に観測するのと同じ、部分的に圧縮された情報状態に条件付けられる。
これにより、モデルの訓練と推論の状態の一貫性が保たれる。

### A.3 Prompts（プロンプト）

***Context Monitor Prompt*（文脈モニタのプロンプト）** — 圧縮をトリガーすべきかどうかを判断するために、文脈モニタが使うプロンプト。

```text
You will be provided with a complete trajectory of an AI agent resolving a software bug. Your task is to determine whether compression is needed and identify the compression type.

**Input Trajectory:**
<trajectory>
{text_conversation}
</trajectory>

Please evaluate whether the trajectory meets ANY of the following conditions for compression:

**Condition 1: Subtask Completion with Redundant Details**
- The last action marks the **completion** of a whole phase rather than an intermediate step or the **start** of a new phase. The next logical action would be to proceed to a subsequent phase
- All errors encountered during this phase have been resolved and validated
- **CRITICAL EXCLUSION**: If the last step shows the agent **entering** a new phase without completing substantive work, this does NOT qualify as subtask completion
- Do NOT choose `subtask_end` if the last 1-3 rounds show unresolved blocker evidence, active debugging of bugs, or unvalidated regressions
- **CRITICAL: Low overall information density**:
  - The completed subtask is lengthy but **overall information density is low**
  - **Only key conclusions or final states** (e.g., "bug found in file X", "all tests pass") will be referenced in subsequent steps
  - **Intermediate details are NOT needed for future actions**: Future steps only need the "what was accomplished" rather than the "how it was accomplished"

**Condition 2: Low Information Density with No Future Utility**
Evaluate recent actions (only consider last 1-5 rounds, don't consider earlier rounds) for lengthy outputs with low information value that will NOT be needed in subsequent steps:
- **Successful file modification**: →HIGH density (don't compress)
- **Early exploration phase**: Examining files/directories →relatively HIGH density (they are likely to be referenced in future steps, so they should be carefully decided whether to compress)
- **Untested test cases that reproduce the bug or examine the fix** →HIGH density (don't compress)
- **Script execution or test results**:
  - Execution succeeded with verbose output unlikely to be referenced again →LOW density
  - Execution failed with useful error messages needed for debugging →HIGH density (don't compress)
  - Execution failed with configuration/usage errors already resolved →LOW density
- **Lengthy failed execution with simple failure reason** that won't inform future decisions →LOW density
- **Long thinking process** only the conclusion is useful →LOW density
- **Key criterion**: The content should be compressible because its information density is low and future actions won't need to refer to this step's detailed content
- **REQUIRED: Standalone/isolated content**: Recent rounds must contain self-contained, one-off tasks completely unrelated to subsequent workflow and future tasks. If the content is part of an ongoing workflow or may be referenced in future steps, do NOT compress regardless of information density.
- **CRITICAL: Consecutive low-density requirement**: Only compress when the last 3-5 rounds (counting backwards from the most recent round) are ALL low density. If any round within this range is high density or potentially useful for future steps, do NOT compress at all. Prefer compressing 3-5 consecutive low-density rounds together. Don't just compress for only one or two rounds unless they are too long.

**Condition 3: Focus Degradations Requiring Agent Alert**
**ONLY evaluate this condition AFTER a patch has been generated.** Before patch generation, ignore this condition entirely.
Use `critical_issue` conservatively:
- Do NOT use `critical_issue` only because a problem just appeared; the agent may fix it in the next few steps.
- Do NOT use `critical_issue` only because tests have not been run yet; the agent may still test later.
- Do NOT use `critical_issue` based on generic risk wording without concrete, recent evidence.
- Use `critical_issue` when the same core issue remains unresolved across multiple recent attempts, or when repeated attempts still fail to fix it.
- Use `critical_issue` when the patch itself shows clear correctness risk (e.g., wrong edit location, broken boundary conditions, contradictory logic, or likely regression) and the agent is not correcting it.
- Use `critical_issue` when the agent is moving to final review / summarization / submission while key risks are still unresolved.
- Do NOT repeat the same `critical_issue` in consecutive decisions unless new evidence appears.
- Require concrete evidence: include specific failing test/error/traceback or explicit wrong behavior in recent rounds.
- Require persistence: the core issue should remain unresolved across at least two recent attempts (or one attempt plus clear prior failed attempt evidence).
- If the same issue was already flagged and there is no new evidence in recent rounds, return `should_compress: false`.

Compress with special alert if ANY of the following serious problems exist:

**1. Patch Quality Issues with High Correctness Risk**
The generated patch itself contains clear flaws that pose significant correctness risks:
- Wrong edit location or scope (modifying unrelated code sections)
- Broken boundary conditions or edge case handling
- Contradictory logic or likely regressions
- **AND** the agent shows no awareness of these critical flaws or is not attempting to correct them

**2. Unresolved Core Issues After Multiple Attempts**
The same fundamental problem persists across multiple iterations:
- The agent has made repeated attempts but the core issue remains unresolved
- Validation or tests continue to fail for the same underlying reason
- The problem is not merely "not fixed yet" but shows a pattern of ineffective approaches

**3. Premature Finalization with Unaddressed Risks**
The agent is moving toward completion while critical risks remain:
- Entering final review, summarization, or submission phase
- While the patch has unresolved correctness concerns that were identified in previous steps, or tests may pass (or haven't been run) but there are untested edge cases, likely impact on related code paths, or the test coverage is clearly inadequate for the scope of changes


**IMPORTANT**: If Condition 3 applies, you should still compress and use type `"critical_issue"` to alert the agent about these problems.

**Output Format:**

<analysis>
[Your detailed analysis evaluating all conditions and determining which (if any) applies]
[If compression is needed, explain which condition triggered it]
[DO NOT specify the exact round numbers - this will be determined in the next step based on detailed reflection]
</analysis>

<decision>
[Your decision in JSON format]

**Important**: When compression is needed, write the reason field in first-person perspective describing why you want to compress the content, based on the compression type. Describe naturally what happened without following a fixed template.

If compression is needed:
{{
    "should_compress": true,
    "type": "subtask_end" | "low_density" | "critical_issue",
    "reason": "[First-person description of what you just completed and why it can be compressed]"
}}

If no compression is needed:
{{
    "should_compress": false,
    "type": "none",
    "reason": "[Brief explanation why compression is not needed]"
}}
</decision>
```

> 訳注（要旨）: ソフトウェアのバグを解決する AI エージェントの軌跡全体を与え、圧縮が必要か、必要ならどの種類かを判定させる。条件 1（`subtask_end`）は、フェーズ（段階）が完了し、途中の詳細は今後不要で結論だけが参照される場合である。条件 2（`low_density`）は、直近 3〜5 ラウンドがすべて今後参照されない低情報密度の内容である場合である。条件 3（`critical_issue`）はパッチ生成後にだけ評価し、同じ中核の問題が複数回の試行でも未解決である、パッチに明確な正しさのリスクがある、リスクを残したまま最終確認・提出に進もうとしている、といった状況が具体的な証拠とともに見られる場合で、エージェントへの警告を兼ねて圧縮する。出力は `<analysis>` と、`should_compress`・`type`・一人称の `reason` を持つ JSON の `<decision>` である。

***Compression Generation Prompt*（圧縮生成のプロンプト）** — 圧縮された内容と残作業の計画を生成するために使うプロンプト。

```text
You are an AI agent working on resolving a software bug. You need to reflect on your debugging progress and compress portions of your conversation history to maintain context efficiency while preserving critical information.

{compression_context}

## Context

Here is your complete debugging trajectory so far:
<trajectory>
{text_conversation}
</trajectory>

## Your Tasks

Complete the following tasks in order:

### Task 1: Comprehensive Reflection and Compression Planning

Write a natural, flowing reflection about your debugging process–think of it as your internal thoughts. This should be a cohesive narrative (not bullet points or a list of answers) that weaves together:

**Part 1: Current Status Review**

- Briefly summarize what you have accomplished in the current trajectory, referencing specific round numbers for key milestones. The summarization should be very concise and focus on the key achievements. Do not describe too much details about each part.
- Taking a critical perspective, carefully review the current solution to identify whether it contains any of the following problems: incorrect issue understanding, incorrected issue location (Truly identify the root problem, or is it just a superficial error influenced by the current test case? Is there any verification?), previously failing tests that were left unresolved before moving forward, tests that failed to run and were simply skipped or abandoned, fixes that do not represent the minimal necessary changes, patches that modify aspects beyond the issue description (such as inconsistent boundary handling that may introduce regressions), or after modification not write more tests to verify various edge cases (such as empty or none input). Make your own independent judgment about each problem type–this is not a reading comprehension task where you summarize what have been found in the trajectory; you need to actively detect these issues based on your own analysis.

**Part 2: Future Task Planning and Information Requirements**

- What are your pending tasks and what specific actions are you planning to take in the next few steps? (Note that you cannot engage in any communication with users and should only focus on resolving this issue without any interaction with users.)
- **Information Requirements Analysis**: Before selecting compression ranges, explicitly think through: If you were to hand off these pending tasks to another person who can only see the compressed trajectory, what specific information would they absolutely need to complete each action successfully? For example:
  - If writing a patch: They need the original file content, the exact location of the bug, and the root cause explanation
  - If running tests: They need to understand what each test case is validating and why
  - If implementing a fix: They need the context of how the code is used and what edge cases to consider
  Naturally articulate these information requirements in your reflection before moving to compression selection.

**Part 3: Compression Range Selection**

Based on the information requirements for your future actions, identify which range of rounds should be compressed. Your selection should balance the positioning of task-relevant content with compression efficiency, prioritizing ranges with a lower proportion of information that will be needed for pending tasks. However, this doesn't mean you can't include such content–if your selected range contains information that will be needed later, that's acceptable, as you can choose to preserve it verbatim. Your selected range must satisfy the following Mandatory Requirements and carefully consider the Selection Priorities:

**Mandatory Requirements:**
- You cannot select the entire trajectory range from start to end
- First list existing compressed ranges only from previous user compression messages (content in <compressed_content>, specifically {existing_ranges_text}). Do NOT treat merged round titles inside compressed text (e.g., "Rounds 4-7 (Merged)") as existing compressed ranges. The new range must be either disjoint, or a full combination that fully covers any existing compressed ranges. Do not choose identical ranges or partial overlaps.
  - Allowed: existing [11-20], [21-30] -> new [11-30] (combine two adjacent ranges), or existing [11-20], [21-30] -> new [21-35] (combine compressed content with uncompressed content)
  - Not allowed: existing [11-20] -> new [15-25] (overlap), or new [11-20] (identical)
- The issue description cannot be compressed, so the starting round number must be at least 1
- The compression range should cover approximately 30% of the total trajectory token length. Avoid selecting too few rounds at once (generally no fewer than 5 rounds)

**Selection Priorities:**
- Try not to compress the last few rounds that are still in progress
- Prioritize compressing earlier rounds over later ones when possible
- Prioritize selecting ranges containing:
  1. Lengthy code execution outputs with minimal relevance to subsequent steps
  2. Large file openings or directory listings where only limited information is utilized later
  3. Consecutive rounds completing a phase where intermediate results have no bearing on subsequent processes
  4. Steps from many rounds earlier that have little impact on the most recent round
  5. Multiple adjacent existing compressed segments that can be combined

**Part 4: Content Importance Assessment**

For your selected range, assess the importance level of each piece of content:

- **Need preserve verbatim**: Content that might be referenced by future steps, such as test cases revealing the bug, error traces showing root cause, key code snippets related to the bug, the files you modified
- **Summarize while retaining details**: Content with limited ongoing relevance, such as long grep or list results, repeated trials, errors that are ultimately resolved and methods that you tried but didn't work
- **Can compress heavily with key findings/lessons**: Content unlikely to be referenced again, such as long passed logs, script execution method, or useless execution outputs

In general, you should verbatim preserve about 30% of the content in your selected range. Make sure all information relevant to future steps is preserved.

Naturally integrate these assessments into your reflection, explaining which content needs to be preserved, and which can be summarized or heavily compressed.

At the end of your reflection, naturally indicate which rounds you will compress (you don't need to use a fixed format–just make it clear which range you've selected).

### Task 2: Generate Compressed Content

Compress your selected range following these guidelines:

**1. Structure for each round:**

<example>
**Round X** or **Rounds X-Y (Merged)** (Phase 1 optional) –[Brief description]

- **Action:**
  [What was done and why–focus on purpose and outcome]

- **Results/Findings & Relevance:**
  [Important discoveries or results (either preserve verbatim or summarize), with brief note on why this information is/isn't still needed.]
</example>

- You can merge multiple adjacent rounds together to describe their content. If the range to be compressed includes content that has already been compressed, you can further expand the scope of the existing merges and appropriately reduce the description of unimportant details.
- **CRITICAL: Phase annotations is optional, but must strictly follow the Original Conversation's phase declarations:** When rounds are annotated with phase numbers (e.g., "**Rounds 1-5** (Phase 1)"), these phase labels MUST come from explicit phase declarations in the Original Conversation. If the original text does not explicitly mention phases, do NOT add phase annotations. Phase numbers should logically increment (e.g., Phase 1, Phase 2, Phase 3) and should not appear out of order.


**2. Apply compression based on importance level:**
- **Need preserve verbatim**: Preserve verbatim with full context
- **Summarize while retaining details**: Summarize results and keep key details that are relevant to future steps
- **Can compress heavily with key findings**: Key results/findings or merge with adjacent rounds

**3. Content-specific handling:**
- **Code files**: Keep essential and problem-related sections, omit boilerplate code or code comments containing lengthy but unnecessary explanations
- **Test results**: One sentence for all passing tests; for failures that are still relevant, keep key error message
- **Error tracebacks**: Keep entry point and actual error line, omit system library paths and framework internal calls
- **Repetitive Content like list or grep results**: Summarize directory contents instead of listing every file, such as "Under `/testbed/src/docx/opc/` there are: part.py, parts/, phys_pkg.py, pkgreader.py, pkgwriter.py. Under `/testbed/src/docx/oxml/` there are: __pycache__, coreprops.py, document.py, drawing.py, exceptions.py"

**4. Always exclude:**
- Content outside your selected range
- Meta information in `<NOTE>` tags
- Explanatory text about the compression process in user message

**5. Maintain continuity:**
- No placeholders like "...rest remains the same..."
- Ensure compressed content flows naturally
- Preserve enough information so future rounds remain understandable

### Task 3: Progress Summary and Forward Planning

Based on your reflection in Task 1, extract and write a concise summary that includes:

**Analysis of Past Steps:**
- Briefly state the Phases that you have completed in one sentence
- Clearly identify if there are any mistakes, oversights, or problems in your previous actions that haven't been corrected yet
- If exists, Be specific about what went wrong and why
- If no significant issues were found, state this clearly

**Future Action Plan:**
- List the specific next steps you plan to take and where in the trajectory the information needed to complete these tasks can be found (with round numbers)
- The plan here must be precise and cannot be vague. For example, you cannot write what can optionally be done, nor can you simply say "run more tests" - you must specify which cases still need to be tested.

This summary should be clear, actionable, and distilled from your reflection–focusing on concrete problems and solutions without repeating all the details from the reflection.

## Output Format

<reflection>
[Your natural narrative reflection that includes: your accomplishments with round numbers, assessment of your process, pending tasks, compression range selection with rationale, and importance assessment of content within the selected range. End by naturally stating which rounds you will compress.]
</reflection>

<start_round_number>
[integer]
</start_round_number>

<end_round_number>
[integer]
</end_round_number>

<compressed_content>
[Your compressed content for the selected range only]
</compressed_content>

<progress_summary>
[Your concise summary of issues with past steps and future action plan, extracted from your reflection above]
</progress_summary>
```

> 訳注（要旨）: エージェント自身の立場で、3 つのタスクを順に行わせる。タスク 1 は物語調の振り返りで、進捗の批判的な点検、今後のタスクとその遂行に必要な情報の洗い出し、圧縮範囲の選択（軌跡全体は選べない、既存の圧縮範囲と部分的に重ねない、問題文は圧縮しない、軌跡全体のトークンの約 30%・概ね 5 ラウンド以上を対象にする）、範囲内の内容の重要度の評価を含める。タスク 2 は、ラウンドごとに「Action」と「Results/Findings & Relevance」の形式で圧縮内容を書くことで、範囲内の約 30% は逐語的に残す。タスク 3 は、過去のステップの問題点と、ラウンド番号付きの具体的な次の行動計画の要約である。出力は `<reflection>`、`<start_round_number>`、`<end_round_number>`、`<compressed_content>`、`<progress_summary>` の各タグで返させる。

***Stage-2 Filtering Prompt: Compression Quality*（第 2 段階のフィルタリングのプロンプト: 圧縮の品質）** — 合成された圧縮のサンプルを残すべきかどうかを判定するための、第 2 段階のフィルタリングで使うルーブリックのプロンプト。

```text
You are judging whether a context-compression trajectory is a good training sample.

Only use the full trajectory below as evidence.

Judge these five conditions as true or false:
1. reflection_identifies_process_problem_and_future_fix_plan:
   The reflection must identify real problems, weaknesses, mistakes, risks, or gaps in the current problem-solving process itself, and it must propose how the later work should change because of that. If the reflection only says progress is fine, a phase is complete, tests passed, or that it wants to compress to save context, then this condition must be false.
2. source_selection_is_good:
   The selected compressed range is worth compressing. It should not be too short in actual content volume. A small round span is still acceptable if the source itself is long or dense, such as long logs or detailed debugging content.
3. compressed_content_preserves_source_critical_information:
   The produced compressed content preserves the source-critical information for this specific source range. Judge this relative to the actual source content: for logs, preserving the key signals and conclusions may be enough; for analysis, implementation, or verification rounds, the summary should preserve the technically important facts needed for later continuation.
4. compression_plan_matches_actual_compressed_content:
   This must be judged strictly. The pre-tool analysis, the claimed compression range, and the actual compressed content must point to the same material. If the analysis says it will compress one part but the result actually summarizes another part, or if the summary mixes in out-of-range material, or if it omits the very material it said it would compress, then this condition must be false.
5. future_plan_is_specific_and_actionable:
   The future plan in the final parameter/progress-summary style content must be concrete and actionable, not generic phase language. It should name specific next checks or actions, such as running tests for a particular file/function/bug scenario, checking a specific behavior, or verifying a specific risk. Generic statements like 'continue verification', 'move to the next phase', or 'keep working' are not enough.

Return strict JSON only with this schema:
{
  "reflection_identifies_process_problem_and_future_fix_plan": true,
  "source_selection_is_good": true,
  "compressed_content_preserves_source_critical_information": true,
  "compression_plan_matches_actual_compressed_content": true,
  "future_plan_is_specific_and_actionable": true,
  "failed_conditions": ["..."],
  "reason": "brief explanation"
}

If any condition is not satisfied, set it to false and name it in failed_conditions.
Do not output scores.

Full trajectory:
{sample.full_trajectory_rendered or '[EMPTY]'}
```

> 訳注（要旨）: 圧縮を含む軌跡が良い訓練サンプルかどうかを、5 つの条件の真偽で判定させる。条件は、振り返りが問題解決の過程の実際の問題とその後の修正方針を指摘しているか、選んだ範囲が圧縮に値する分量か、圧縮内容が範囲内の重要な情報を保っているか、ツール呼び出し前の分析・宣言した範囲・実際の圧縮内容が同じ素材を指しているか、今後の計画が具体的で実行可能か、である。厳密な JSON だけを返させ、点数は出力させない。

***Stage-2 Filtering Prompt: Split Quality*（第 2 段階のフィルタリングのプロンプト: 分割の品質）** — 圧縮後の軌跡の品質を判定するための、第 2 段階のフィルタリングで使うルーブリックのプロンプト。

```text
You are judging whether this trajectory is a good training sample.

Judge these conditions as true or false:
1. no_bad_assistant_content:
   Assistant messages should NOT be bad content such as obvious repeated output, repetitive low-value text, malformed garbage, bizarre or corrupted characters, broken formatting artifacts, or other clearly abnormal content that should not be learned.
2. post_reflection_actions_follow_future_plan:
   After the reflection/compression user message, the later assistant actions should substantially follow the future plan implied by that reflection and compressed summary. It is okay if the agent adapts slightly, but it should not ignore the future plan and jump to unrelated work.
3. no_unnecessary_repeated_actions_after_compression:
   After the reflection/compression user message, there should NOT be clearly unnecessary repeated actions caused by forgetting compressed context. For example, if the earlier trajectory already established some file contents, logs, test results, or conclusions, the later assistant should not redundantly redo the same file viewing, same test runs, or same checks without a clear new reason.

Return strict JSON only with this schema:
{
  "trajectory_type": "with_compressed_user_message",
  "no_bad_unmasked_assistant_content": true,
  "post_reflection_actions_follow_future_plan": true,
  "no_unnecessary_repeated_actions_after_compression": true,
  "failed_conditions": ["..."],
  "reason": "brief explanation"
}

If any condition is not satisfied, set it to false and include it in failed_conditions.
Do not output scores.

Full trajectory:
{sample.rendered_trajectory or '[EMPTY]'}
```

> 訳注（要旨）: 圧縮後の軌跡が良い訓練サンプルかどうかを、3 つの条件の真偽で判定させる。条件は、アシスタントの発話に繰り返しや崩れた文字などの学習させるべきでない内容がないか、振り返り・圧縮のメッセージの後の行動がその将来計画に概ね従っているか、圧縮された文脈を忘れたことによる不要な繰り返し（同じファイルの閲覧や同じテストの実行など）がないか、である。厳密な JSON だけを返させる。

## B Experiment Details（実験の詳細）

### B.1 Dataset Details（データセットの詳細）

**訓練データ（Training Data）.** SFT と RL のデータを、SWE-ReBench [1] と SWE-Gym [29] から構築する。
SFT のデータ合成では、ベースモデルとして Qwen3-Coder-30B-A3B-Instruct を、プロプライエタリな合成モデルとして GPT-5.1 を使い、問題ごとに 3 回の rollout を行い、2.2.2 節で述べたフィルタリングのパイプラインを適用する。
完全に正解した問題については、単純すぎる事例に過大な重みがかからないよう、軌跡を 1 つだけ残す。
得られた SFT データの統計を表 4 にまとめる。

RL については、同じ 2 つのソースから、フィルタリングした自明でない部分集合をさらに構築する。
まず正解パッチによる評価（ground-truth evaluation）を実行し、参照パッチが対応するテストケースに失敗する事例を取り除く。
次に best-of-n による検証を適用して中間的な合格率をもつ事例を残し、RL の訓練セットが、自明に易しいサンプルと完全に手に負えないサンプルの両方を避けるようにする。
この手順により、2,194 件の訓練事例が得られる。

**評価データ（Evaluation Data）.** SWE-Bench-Verified [21] で評価する。
また、SWE-bench 形式の評価を複数のプログラミング言語のリポジトリに拡張した SWE-bench Multilingual と、より難しい長期的なイシュー解決のタスクを重視する SWE-bench Pro でも評価する。両ベンチマークとも、ベンチマークの全セットを用いる。
評価環境では、意図しないリポジトリ履歴からの漏洩を避けるため、将来のコミットを取り除く。
主要な評価指標として、イシューの解決率を報告する。

### B.2 Training Details（訓練の詳細）

すべての実験で、ベースモデルとして Qwen3-Coder-30B-A3B-Instruct と Qwen3-4B-Instruct-2507 を使い、エージェントの scaffold として OpenHands を採用し、データ収集、SFT、RL を通じて文脈上限を 32,768 トークンに設定する。

**教師ありファインチューニング（Supervised Fine-Tuning）.** SFT モデルは VeOmni フレームワーク [28] で訓練し、グローバルバッチサイズ 128、学習率 $2 \times 10^{-5}$、コサイン型の学習率スケジュール、3 エポックの訓練を用いる。
次に、フィルタリングした能動的記憶の部分集合に対して、学習率 $2 \times 10^{-6}$ で 1 エポックの第 2 段階のカリキュラム調整を行う。
Qwen3-4B については、Qwen3-Coder-30B-A3B-Instruct で収集した訓練データをそのまま再利用する。

**表4**: SFT データセットの統計。

| Statistic（統計量） | |
|---|---|
| *Data size（データの規模）* | |
| Sample number（サンプル数） | 8,728 |
| Unique problem number（重複を除いた問題数） | 5,867 |
| Total sub-trajectory number（部分軌跡の総数） | 27,088 |
| Avg sub-trajectories per sample（サンプルあたりの平均部分軌跡数） | 3.10 |
| *Compression（圧縮）* | |
| Avg tokens before（圧縮前の平均トークン数） | 9,657.93 |
| Avg tokens after（圧縮後の平均トークン数） | 1,496.52 |
| Avg ratio (%)（平均比率） | 15.50 |

**強化学習（Reinforcement Learning）.** RL モデルは Verl フレームワーク [33] で訓練し、バッチサイズ 64、事例あたり 8 個の rollout サンプル、学習率 $2 \times 10^{-6}$、$\tau=0.05$ を用いる。$\tau$ の値は、軌跡合成の際に用いた低温度（low-temperature）の設定に合わせた。
クリッピングの設定は DAPO [59] に従って clip low を 0.2、clip high を 0.27 とし、rollout の温度を 1.0 に設定する。
また、RL 中の訓練と推論の不一致を緩和するために R3 Rollout Routing Replay を有効にし、これは 30B の RL 訓練の設定でのみ適用する。
訓練コストを抑えるため、RL 中の最大対話ステップ数を 150 に設定する。
この最大ステップ数の上限を超えたエラーサンプルは、訓練中にフィルタリングされ、マスクされる。
RL の目的関数では、エントロピー正則化や KL ダイバージェンスのペナルティは用いない。

## C Case Study（事例研究）

本節では、記憶管理ツールに関するいくつかの事例を示す。
まず実際の記憶管理の呼び出しの例をいくつか示し、次に記憶管理の行動を含む完全な軌跡をいくつか示す。

### C.1 Memory Management Action Examples（記憶管理の行動の例）

実際の記憶管理の行動を 4 つ示す。
各ボックスには、圧縮の直前のアシスタントによる短い根拠の説明と、完全なツール呼び出しが含まれる。
これら 4 つの行動の例は、次の小節の軌跡の事例とは重複しない。

***Example 1: Context-Budget Pressure*（例 1: 文脈予算の圧迫）.** インスタンス: `django__django-11551-0`。ステップ: 38。

```text
[analysis]
So far in this trajectory I've gone from initial orientation to a concrete code change, but the fix still isn't behaving as the issue description's "correct behaviour" example promises. In round 1 I restated the problem and constraints. Rounds 2--7 (already compressed) covered repo inspection and confirming this is a Django source tree with tests and no `manage.py`. In rounds 8--14 I located `_check_list_display_item` in `django/contrib/admin/checks.py`, inspected the current implementation, and then used `git show` to compare it with the historical commit that broke the behaviour. By round 22--25 I had a small reproduction script (`simple_reproduce.py`) that demonstrated `admin.E108` being raised for a `MockPositionField`-like descriptor. In rounds 30--34 I added more debug scripts (`debug_reproduce.py`, `debug_reproduce2.py`) to understand how `hasattr`, `getattr`, and `_meta.get_field` behave for the descriptor. Finally in rounds 37--38 I reworked `_check_list_display_item` again, aligning it more closely with the "correct behaviour" snippet from the issue description.

Looking critically at the current solution, there are several issues. First, my understanding of the root cause is broadly correct--the bug stems from the 28490-fix that introduced a `hasattr(model, item)` guard that blocks `_meta.get_field(item)` for instance-only descriptors--but I haven't yet achieved the desired behaviour. The latest `_check_list_display_item` (round 38) now tries `_meta.get_field(item)` first, falls back to `getattr(model, item)` on `FieldDoesNotExist`, and only errors if that fails. However, my debug script in round 30 showed that `getattr(Thing, 'order')` raises `AttributeError: PositionField can only be accessed on model instances`, which means my fix still ends up hitting the E108 path instead of treating this as a valid callable/attribute. That indicates my logic conflates "no field at all" with "invalid list_display entry"; the correct behaviour should treat anything callable or attribute-like as valid, regardless of whether `_meta.get_field` finds a real Django field. So Phase 5 is incomplete--my fix is still wrong relative to the issue's expectations.

Second, I haven't run any of Django's own tests yet; only my standalone scripts have been executed. That means I have no assurance about regressions in the broader admin checks or anywhere else that `_check_list_display_item` is used. There are no skipped or abandoned failing tests; I simply haven't invoked the test runner at all, which is an oversight relative to good practice. Third, my patch may be slightly less minimal than ideal because it changes the control flow around `FieldDoesNotExist` and `AttributeError` in a way that might not perfectly mirror the historical behaviour, but it is still localized to `_check_list_display_item`. The more serious problem is that I haven't yet validated against the expected behaviour table in the issue description; I've only checked that E108 still appears for the descriptor case.

Looking ahead, my pending tasks are clear. First, I need to refine `_check_list_display_item` so that it truly follows the "correct behaviour" logic described in the issue: treat any callable or attribute-like item as valid, and only treat it as an error when neither `_meta.get_field(item)` nor `getattr(model, item)` succeeds and yields something callable/attribute-like. That likely means treating the "field found by `_meta.get_field`" and "field found by `getattr(model, item)`" as two distinct success paths, not conflating them. Second, I must re-run `simple_reproduce.py` to confirm that E108 no longer appears for the descriptor case, and adjust my logic if it still does. Third, I need to run Django's own tests--specifically `python -m pytest tests/admin_checks/tests.py` and ideally some additional admin-related modules--to ensure there are no regressions in existing admin checks. Only after those tests pass will I be confident the fix is correct.

If I were handing these tasks to someone else who only saw a compressed trajectory, they would absolutely need a few key pieces of information. To adjust `_check_list_display_item`, they need to know where it lives (`django/contrib/admin/checks.py`), its current implementation (round 38), and the historical diff from commit 47016adbf5 (round 14) showing how it was changed and why. They also need the precise semantics of the bug: that a descriptor like `MockPositionField` (rounds 30--34) is not a Django field and thus `_meta.get_field('order')` raises `FieldDoesNotExist`, but `getattr(Thing, 'order')` raises an `AttributeError` with a specific message. They need my debug observations that `hasattr(Thing, 'order')` is False and `hasattr(instance, 'order')` is True, plus instance access works (`instance.order` returns 1). That's essential context for designing a correct fix. For running tests, they need to know how tests are executed in this repo (no `manage.py`, use `python -m pytest`), and which test module is most relevant (`tests/admin_checks/tests.py`). For implementing the patch properly, they need the issue description's "correct behaviour" function (quoted in round 28) and the expectations table; those encode how different combinations of `hasattr(model, item)` and `_meta.get_field(item)` should be interpreted.

Given those information needs, I should compress earlier exploratory rounds that don't carry unique insights, while keeping verbatim the key code snippets, diffs, and reproductions. The user has already compressed rounds 2--7; per the rules I can't partially overlap that range, but I can extend around it. Rounds 8--14 are important: they contain locating `_check_list_display_item`, viewing its code, and showing the historical diff from commit 47016adbf5. Those I should preserve with enough detail to reconstruct what changed. Rounds 22--25 are also important because they introduce and run `simple_reproduce.py`, confirming the E108 error. Rounds 30--34 contain crucial debug outputs about how `MockPositionField` behaves--hasattr/ggetattr/meta/getattr interactions--which are necessary for understanding why the current fix fails. By contrast, rounds 16--21 are mostly directory browsing and an abandoned attempt to create a huge settings dict; they can be heavily compressed. Similarly, some of the intermediate thinking steps (e.g., round 26's meta-analysis) are useful but can be summarized rather than preserved verbatim.

To balance context and efficiency, I'll compress rounds 16--21 into a concise merged summary. These rounds include viewing parts of `tests/admin_checks/tests.py`, some grep output for list_display tests, and an overly verbose attempt at creating a settings configuration in `reproduce_issue.py`. The only truly important content there is that I inspected `tests/admin_checks/tests.py` and saw how list_display checks are structured; the rest (the huge INSTALLED_APPS list) is not needed later. So within 16--21: the fact that tests exist and where list_display checks live should be summarized; the long settings configuration dump can be heavily compressed; there's nothing that needs to be preserved verbatim. That range is disjoint from the already-compressed [2--7], doesn't touch the issue description or initial understanding in round 1, and focuses on early exploration rather than core bug logic.

So I will compress rounds 16 through 21 into a merged summary, keeping only the key findings about test structure and discarding the noisy settings snippet.

[start_round_number]
16

[end_round_number]
21

[content]
**Rounds 16--21 (Merged)** -- Inspecting admin checks tests and first failed reproduction attempt

- **Action:**
  - Listed contents of `tests/admin_checks/` and opened `tests/admin_checks/tests.py` to understand existing admin checks tests.
  - Searched within that file for `list_display` usage.
  - Attempted to create a standalone reproduction script (`reproduce_issue.py`) with a huge `INSTALLED_APPS` configuration to mimic Django's settings.

- **Results/Findings & Relevance:**
  - Confirmed that `tests/admin_checks/tests.py` contains various admin checks tests, including ones around list_display and list_editable.
  - Identified how list_display-related tests are structured (e.g., checking for E125 when fields are non-editable), which may be relevant for ensuring no regressions after modifying `_check_list_display_item`.
  - The initial attempt at `reproduce_issue.py` produced an extremely verbose `INSTALLED_APPS` list with many repeated postgres entries; this approach was abandoned in favour of simpler, more focused scripts (`simple_reproduce.py`) created later.
  - These rounds mainly established where admin checks tests live; no unique technical insights beyond confirming test structure were needed later. The long settings configuration snippet is not required for future steps and can be safely omitted.

[remaining_work]
I've completed Phase 1 (understanding the issue), Phase 3 (locating relevant code and history), Phase 4 (creating reproduction/debug scripts), and part of Phase 6 (implementing a fix) in `_check_list_display_item`. The main outstanding issue is that my current implementation still returns E108 for a descriptor-based field (like `MockPositionField`) because it treats any AttributeError from `getattr(model, item)` as an invalid list_display entry, instead of recognizing that such descriptors are acceptable when accessed via instances. I also haven't run Django's own tests at all, so regressions are unverified.

Next steps are concrete: (1) refine `_check_list_display_item` in `django/contrib/admin/checks.py` so that it follows the "correct behaviour" logic from the issue description: treat any callable or attribute-like item as valid, and only error when both `_meta.get_field(item)` and `getattr(model, item)` fail; this should be implemented around lines 718--749 as shown in round 38. (2) Re-run `python simple_reproduce.py` (from rounds 22--25) to confirm that E108 no longer appears for descriptor-based list_display entries such as `'order'`. (3) Run at least `python -m pytest tests/admin_checks/tests.py` to validate existing admin checks still pass with the new logic; additional admin-related test modules may be run if time allows. The information needed for these tasks--current implementation and diff (rounds 14 and 38), behaviour of `MockPositionField` (rounds 30--34), and test locations (rounds 16--21)--is preserved in the trajectory.
```

> 訳注（要旨）: Django の admin のチェック（`_check_list_display_item`）のバグを修正している途中、ステップ 38 で文脈予算の圧迫によって圧縮した例である。`[analysis]` では、修正がまだイシューの期待する挙動になっていないこと、Django 自身のテストを一度も実行していないことを認め、今後の作業に必要な情報（コードの場所、履歴上の差分、デバッグでの観察）を洗い出す。そのうえで、独自の知見を含まないラウンド 16〜21（テストファイルの閲覧と、冗長な設定を書いて放棄した再現スクリプト）を選んで要約し、`[remaining_work]` に具体的な次の 3 手順を記している。

***Example 2: Low-Density Exploration*（例 2: 低密度の探索）.** インスタンス: `django__django-15382-0`。ステップ: 40。

```text
[analysis]
Up to this point, I've mostly been in the exploratory and reproduction phases rather than actually fixing anything. Early on, in rounds 2--3, I inspected the repo structure and README to understand how Django tests are normally run. In rounds 6--11 I experimented with various ways to invoke the test suite (pytest, `python -m django test`, `tests/runtests.py`) and discovered that pytest isn't available and Django's normal test runner needs settings, so I switched to using `tests/runtests.py` as the canonical entry point. In rounds 12--15 I located the relevant code for `Exists` and `EmptyResultSet` via `grep`, opening `django/db/models/expressions.py`, `django/db/models/sql/query.py`, and `django/db/models/sql/where.py`, which gave me a good sense of where subqueries and WHERE clauses are handled. Rounds 16--21 were focused on reading the `Exists` and `Subquery` implementations and the query compiler's `as_sql` logic, including the `EmptyResultSet` handling in the WHERE clause compilation. In rounds 22--31 I dug into the compiler's WHERE handling and existing tests around `EmptyResultSet` and `Exists`, confirming how empty/Nothing nodes are treated. Finally, in rounds 33--39 I created a standalone reproduction script `reproduce_issue.py`, iterated on its settings, and confirmed the bug: a query with `~Exists(MyModel.objects.none())` raises `EmptyResultSet` when the compiler tries to render the WHERE clause.

Looking critically at what I've done so far, there are a few things to note. The issue understanding is broadly correct: the problem is that a combination of a negated `Exists` over an empty subquery and another filter (like `name='test'`) causes Django to raise `EmptyResultSet` during query compilation, instead of generating a valid SQL with a WHERE clause (or, at minimum, returning an empty result without blowing up). I've confirmed this behavior with a minimal script in round 38--39, so the root cause is not just inferred from tests; it's observed. However, I haven't yet pinpointed the exact line or branch that decides to raise `EmptyResultSet` in this particular case; I've identified the relevant areas (WHERE node compilation, `Exists.as_sql`, and `Subquery.as_sql`) but not yet derived the minimal change that will fix it. There were also several small dead ends: trying pytest directly (round 10), trying the normal Django test command without settings (round 8), and overcomplicating the reproduction script settings (rounds 33--37) before simplifying them. Those missteps were harmless but somewhat verbose and not strictly necessary for the final fix.

In terms of minimality, I haven't touched any library code yet, so there's no risk so far of over-broad patches or regressions. The only code changes are in the reproduction script, which is not part of the library and is allowed. I also haven't run the project's actual test suite yet (only via the reproduction script), so there is an outstanding step to confirm that whatever fix I implement doesn't break existing tests. The reproduction script itself is a good tool for manual verification but doesn't replace automated tests.

Looking ahead, the pending tasks are clear. First, I need to perform a more precise root-cause analysis: for the specific scenario `MyModel.objects.filter(~Exists(MyModel.objects.none()), name='test')`, I must trace through how `Exists.as_sql` and `Subquery.as_sql` interact with the WHERE node compilation, and identify exactly where the logic decides that the whole WHERE is "empty" and raises `EmptyResultSet`. That likely involves reading more of `where.py`'s `WhereNode.as_sql` (especially around lines 79--105) and possibly how `Exists` is treated as a boolean expression in that context. Once I understand that, I can propose a minimal change: for example, ensuring that a negated `Exists` over an empty queryset is treated as a known predicate (such as returning `"0 = 1"` for the WHERE clause rather than raising `EmptyResultSet`), so that the outer query remains valid and the result is an empty set rather than an exception. After implementing that fix in `django/db/models/expressions.py` or `django/db/models/sql/where.py`, I need to re-run my reproduction script to verify that `qs.query.as_sql()` no longer raises `EmptyResultSet` and that `list(qs)` returns an empty list. Then I must run targeted tests: at minimum, the existing `ExistsTests` and any tests that mention `EmptyResultSet` or WHERE-node edge cases (e.g., in `tests/queries/tests.py` and `tests/expressions/tests.py`). Depending on where the fix sits, I might also run a broader subset of tests around queries or expressions.

If I were handing these pending tasks to another person who only sees a compressed trajectory, they would absolutely need several pieces of information. For writing the patch, they need to know the exact behavior we're trying to change: that `MyModel.objects.filter(~models.Exists(MyModel.objects.none()), name='test')` currently raises `EmptyResultSet` when calling `qs.query.as_sql()`, as demonstrated in round 38--39. They also need pointers to the core code paths: the definition of `Exists` and `Subquery` in `django/db/models/expressions.py` (rounds 15--19), the query compiler's `as_sql` method and its handling of `where`/`having` using `EmptyResultSet` (rounds 22--23), and the WHERE node logic in `django/db/models/sql/where.py` around lines 70--110 (rounds 26--28). For running tests, they need to know that the canonical test runner is `tests/runtests.py` (round 11) and that pytest is not installed, so they should rely on that script with appropriate module arguments. For implementing the fix safely, they need context on how existing tests around `EmptyResultSet` and WHERE-node handling behave (round 31), so they can ensure they don't break those cases. Finally, they need awareness that the current reproduction script exists at `/testbed/reproduce_issue.py` and what it currently prints (round 39), so they can reuse or adapt it.

Given these information requirements, I should compress a range that is relatively verbose but doesn't contain unique, irreplaceable insights. The early rounds where I inspected the repo structure (round 2), read README (round 3), tried pytest (round 4), attempted Django's test command without settings (round 8), and probed `tests/runtests.py` help establish context but aren't crucial once we know that we use `tests/runtests.py` as the test runner. The most important technical content for remaining work lives in rounds 12--15 (locating `Exists` and `EmptyResultSet`), round 19 (Subquery), rounds 22--23 (compiler's WHERE handling), rounds 26--28 (WHERE node logic), rounds 31--32 (existing tests), and rounds 38--39 (reproduction and error trace). Those should remain largely intact. The rounds where I experimented with pytest and generic Django test invocation are less critical now; they are mostly trial-and-error steps with long command outputs that don't inform the fix.

Within my chosen range, some parts should be preserved verbatim: the error trace from running `reproduce_issue.py` in round 38 is important because it pinpoints that the exception arises in `where.py:99` during WHERE compilation; that exact traceback will be useful when reasoning about how to adjust WHERE handling. The fact that pytest is not installed (round 4) is also mildly important as context for why tests are run via `tests/runtests.py`, but only needs a brief summary. Other content--like directory listings from round 2, generic help output from `--help` commands, and long, repetitive attempts to use different test runners--can be heavily compressed into short descriptions of "attempted X, failed because Y".

Given all of this, I'll compress rounds 2 through 11. That range covers initial environment exploration and test-running attempts; it's not the very start (round 1 has the issue description) nor the very end (rounds 12+ contain core code analysis). It's also large enough to give a meaningful compression benefit while leaving later, more fine-grained technical analysis intact for remaining work.

[start_round_number]
2

[end_round_number]
11

[content]
**Rounds 2--3 (Merged)** -- Initial repo inspection and test instructions

- **Action:**
  Listed the `/testbed` directory tree to understand project layout (saw Django source under `/django`, tests under `/tests`, docs, etc.) and opened `README.rst` to see how Django tests are normally run.

- **Results/Findings & Relevance:**
  Confirmed this is a Django checkout with extensive test suite under `/tests`. README mentions running unit tests via docs but doesn't provide concrete commands. This established that tests are centralized under `/tests`, but no specific test invocation method was determined yet. The detailed directory listing is not needed going forward; only the knowledge that Django's code is under `/django` and tests under `/tests` remains relevant.

**Rounds 4--5 (Merged)** -- Attempted pytest usage

- **Action:**
  Tried running `python -m pytest --version` from `/testbed` to see if pytest was installed and usable.

- **Results/Findings & Relevance:**
  Command failed with "No module named pytest," confirming pytest is not available in this environment. This led to seeking an alternative test runner; the exact error message is mildly relevant as justification for not using pytest later, but the full command output is not needed.

**Rounds 6--8 (Merged)** -- Django test command exploration

- **Action:**
  - Ran `python manage.py --help` (failed because there is no `manage.py`).
  - Attempted `python -m django tests.runtests --help`.
  - Opened `tests/runtests.py` to see how tests are orchestrated.
  - Ran a simple import to confirm Django works (`python -c "import django; print(django.VERSION)"`).

- **Results/Findings & Relevance:**
  Discovered there is no standard Django project layout here (no `manage.py`), so Django's usual test command can't be used directly. Running `tests/runtests.py --help` showed that it sets up Django via `django.apps.apps.populate`, imports settings-related modules, and defines a custom runner; this is exactly how tests are meant to be run here. The Django version printed (`(4, 1, 0, 'alpha', 0)`) confirms we're working against a specific Django version but isn't directly used later. The key takeaway is: use `/testbed/tests/runtests.py` as the main test entry point; detailed help output is not needed.

**Rounds 9--10 (Merged)** -- Attempted direct Django test command and pytest again

- **Action:**
  - Ran `python -m django test --help` (failed due to missing settings).
  - Tried again with pytest on a single test file (`python -m pytest tests/queries/test_query.py ...`) which failed because pytest is unavailable.

- **Results/Findings & Relevance:**
  Confirmed multiple times that:
  - The Django test command requires settings configuration or an environment variable (`DJANGO_SETTINGS_MODULE`) -- not trivial here without a project layout.
  - Pytest is not installed, so it cannot be used for running tests.

  These attempts reinforce that we must rely on `tests/runtests.py`. The repeated stack traces are not needed; only the conclusion matters.

**Round 11** -- Discovering `tests/runtests.py` usage

- **Action:**
  Ran `cd /testbed && python tests/runtests.py --help`.

- **Results/Findings & Relevance:**
  Saw detailed help text for `runtests.py`, including options like `-v`, `--failfast`, `--settings`, `--parallel`, `-k`, and positional module arguments. This confirmed that:
  - Tests can be run via `python tests/runtests.py [options] [module ...]`.
  - There are several optional flags for controlling verbosity, parallelism, etc.

  This information is important as a canonical way to run tests later, but the full help text can be summarized as above without losing essential context.

[remaining_work]
I've completed Phases 1--4 and part of Phase 5: I understood the issue, explored the relevant Django code (`Exists`, `Subquery`, compiler, WHERE handling), confirmed there's no pytest installation, learned that tests must be run via `/testbed/tests/runtests.py`, and created a standalone reproduction script that demonstrates the bug (`EmptyResultSet` raised when compiling a query with `~Exists(MyModel.objects.none())`). There are no incorrect library code changes yet--only exploratory commands and a reproduction script--but I haven't yet found the precise root cause or implemented a fix.

The main outstanding issues are: I haven't yet traced exactly where in `where.py`'s `WhereNode.as_sql` logic it decides to raise `EmptyResultSet` for this specific combination of filters; I also haven't run any of Django's own tests after a fix because no fix has been applied yet. There were some minor inefficiencies earlier (trying pytest, various test runner invocations) but those were corrected by standardizing on `tests/runtests.py`.

Next steps:

1. **Deep root-cause analysis**:
   - Re-examine `django/db/models/sql/where.py`, especially the loop from lines ~79--105 where children are compiled and counts of "full" vs "empty" nodes determine whether to raise or return an empty string (see context from round 27--28).
   - Trace how a filter like `~Exists(MyModel.objects.none())` plus another filter (`name='test'`) is represented in the WHERE node tree and why it ends up raising `EmptyResultSet`.
   - Cross-check this behavior with the existing tests in `tests/queries/tests.py` that exercise WHERE-node empty/full handling (round 31) to ensure any fix preserves those semantics.

2. **Implement a minimal fix**:
   - Based on that analysis, adjust either:
     - `Exists.as_sql` / `Subquery.as_sql` in `django/db/models/expressions.py` (rounds 15--19) so that a negated empty EXISTS behaves like a known false predicate (e.g., `"0 = 1"`) rather than triggering an exception; or
     - The WHERE-node logic in `django/db/models/sql/where.py` so that when all children are effectively "empty" but we're dealing with a negated EXISTS over an empty queryset, we generate an always-false predicate instead of raising `EmptyResultSet`.
   - Ensure the patch is minimal and doesn't change behavior for non-empty EXISTS or other WHERE-node edge cases.

3. **Verification**:
   - Re-run `/testbed/reproduce_issue.py` to confirm that:
     - `qs.query.as_sql()` no longer raises `EmptyResultSet`.
     - A query such as `MyModel.objects.filter(~models.Exists(MyModel.objects.none()), name='test')` yields an empty result when evaluated (`list(qs)` returns an empty list).
   - Run targeted tests via `python tests/runtests.py`:
     - At least all tests in modules that mention WHERE-node behavior or EXISTS (`tests/queries/tests.py`, `tests/expressions/tests.py`, which were inspected around rounds 31--32).
     - Potentially an additional subset like all expression-related tests if needed.

All information needed for these steps--exact reproduction behavior and error trace--is in round 38--39; core code locations are in rounds 15--19, 22--23, and 26--28; and canonical test-running instructions are summarized from rounds 6--11.
```

> 訳注（要旨）: `~Exists(MyModel.objects.none())` を含むクエリのコンパイルで `EmptyResultSet` が送出されるバグについて、ステップ 40 で、環境の確認とテストの実行方法の試行錯誤（pytest がない、`manage.py` がない等）に費やしたラウンド 2〜11 を圧縮した例である。結論（テストは `tests/runtests.py` で実行する）だけを残し、根本原因の追跡、最小限の修正、検証の計画を `[remaining_work]` に書いている。

***Example 3: Subtask-End Consolidation*（例 3: サブタスク終了時の集約）.** インスタンス: `sphinx-doc__sphinx-9281-0`。ステップ: 46。

```text
[analysis]
Looking back over this trajectory, the core debugging work is already complete. Early on, in rounds 2--7, I oriented myself in the repo, confirmed this is Sphinx 4.1.0, and located the autodoc and util modules. In rounds 10--18 I drilled into `sphinx/ext/autodoc/__init__.py` and `sphinx/util/inspect.py`, using `grep` to find where signatures are built and where default values are formatted. By round 18 I had identified `stringify_signature` and its use of `object_description` as the key code path for function signatures. In rounds 22--23 I created a small reproduction script (`test_enum_issue.py`) and demonstrated that `object_description(MyEnum.ValueA)` produced `<MyEnum.ValueA: 10>`, matching the user's reported behavior. In rounds 24--26 I analyzed the root cause and then implemented a minimal fix in `sphinx/util/inspect.py` (round 26) by specialcasing enum attributes in `object_description` to use `ClassName.ValueName` instead of `repr`. From rounds 27--31 I verified that this fix produced clean enum output and that nonenum behavior remained intact. Finally, in rounds 32--45 I ran the relevant Sphinx test suites (e.g., `tests/test_util_inspect.py`, `tests/test_ext_autodoc.py`, `tests/test_ext_autodoc_autofunction.py`) and additional custom scripts to confirm the change behaves correctly and doesn't regress other functionality.

Critically reviewing the solution, the understanding of the issue is correct: the problem is not in autodoc per se but in how default values are rendered in signatures, specifically via `stringify_signature` and `object_description`. The location is also correct: `stringify_signature` calls `object_description(param.default)`, and `object_description` was using `repr()` for everything. The fix is minimal and localized: a small conditional at the top of `object_description` that only changes behavior for enum attributes, using the existing `isenumattribute` helper. I verified behavior with direct calls to `object_description` and `stringify_signature`, and with real Sphinx builds (via simple integration scripts), so the root cause is not just inferred from tests but validated by concrete outputs. There were no failing tests that I ignored; when I hit a small issue in an adhoc integration script (round 36--37, where the temporary module wasn't importable by Sphinx), I recognized it as an environment/configuration problem rather than a regression from the fix and didn't treat it as a blocker for the core change. I also didn't change any test files, per instructions. One could argue that I could have added more edgecase tests for enums (e.g., enums with string values, enums with custom `__repr__`, etc.), but the existing tests plus my custom scripts already cover regular enums, string enums, and the interaction with signatures, which is sufficient for this issue. I don't see evidence of overbroad modifications or behavior changes beyond what the issue describes.

At this point, there are essentially no pending technical tasks for fixing this bug: the patch is in place, the relevant tests have been run and passed, and the behavior matches the issue description. If I were to continue, the only remaining work would be housekeeping: perhaps removing the adhoc helper scripts (`test_enum_issue.py`, `test_comprehensive.py`, `test_final_verification.py`, `test_sphinx_integration.py`) if this were a real PR, or adding a formal unit test under `tests/` specifically for `object_description` and enums. But the user explicitly said tests are already handled, so those tasks are out of scope. If I were handing off to someone else to do such followup work, they would need to know: (1) the exact change made to `sphinx/util/inspect.py` (the new `isenumattribute` branch in `object_description`), (2) the rationale for that change (ugly enum repr in signatures), (3) the exact behavior we want (signature should display `MyEnum.ValueA` instead of `<MyEnum.ValueA: 10>`), and (4) where the relevant tests live and what they validate (`tests/test_util_inspect.py` for inspect utilities, `tests/test_ext_autodoc.py` and related tests for autodoc behavior). For running tests, they'd need to know that `pytest` is the test runner and that the key suites are `tests/test_util_inspect.py` and the autodoc test modules. For implementing any further fix, they'd need the context that `stringify_signature` uses `object_description` for default values, and that `isenumattribute` is already imported and defined in the same module.

Given that there are no substantial pending tasks, the main goal of compression is to keep enough context for someone to understand what was changed and why, while trimming redundant or lowvalue detail. The most important information to preserve verbatim includes: the exact diff/patch to `object_description` (round 26/44), the key code snippet showing how `stringify_signature` uses `object_description` (round 18), and at least one concrete demonstration of the pre and postfix behavior (e.g., the output from `test_enum_issue.py` in round 23 and the final verification script in round 39). These are crucial for understanding the root cause and validating the fix.

On the other hand, there's a lot of content that can be compressed heavily. For example, the initial directory listing and README inspection (rounds 2--3), the repeated `pytest --version` and `sphinx.__version__` checks (rounds 4--5), and generic `grep` searches that just confirmed where autodoc and inspect live (rounds 6, 10--13) are not needed going forward. The multiple test runs that simply report "all tests passed" (rounds 32, 33, 34, 35, 43) can be summarized briefly. Similarly, the creation and execution of helper scripts like `test_comprehensive.py` and `test_final_verification.py` (rounds 28--31, 38--39) are useful for my own validation but don't need to be preserved verbatim; their key findings (enum values rendered cleanly, nonenum values unchanged, signatures correct) can be summarized. The Sphinx integration script (`test_sphinx_integration.py`) in round 36--37 is particularly noisy and not essential; the only important lesson is that Sphinx was able to build successfully, even though that particular script had a module import issue (which was unrelated to the fix).

With that in mind, the best range to compress is a midtolate segment where I'm mostly running tests and confirming behavior after the fix is in place. I should avoid compressing the very last few rounds where I reinspect the final patch (rounds 44--45) and the final verification (round 45), because those are still useful as a concise reference. I also want to keep the initial identification of the relevant code (around rounds 10--18) more intact, since that's the reasoning trail that explains why the fix is where it is. The most compressible region, balancing these constraints, is roughly rounds 28--41: this is where I create multiple adhoc test scripts, run them, and run various pytest invocations, much of which is repetitive confirmation. Within that range, I'll preserve verbatim the essential behavioral observations (e.g., that `object_description(MyEnum.ValueA)` now returns `MyEnum.ValueA` and that function signatures look clean) but I can heavily summarize the creation and execution of each helper script and the detailed stdout of the pytest runs.

So I will compress rounds 28 through 41, summarizing the creation and results of the helper scripts and tests while preserving the key conclusions about enum rendering and nonregression.

[start_round_number]
28

[end_round_number]
41

[content]
**Rounds 28--31 (Merged)** -- Creating and running a comprehensive local verification script

- **Action:**
  I created `test_comprehensive.py` to exercise `object_description` and `stringify_signature` across several scenarios: enums (`MyEnum`, `AnotherEnum`), nonenum types (strings, ints, lists, dicts, sets), and functions with default values. Initially I tried to include `enum.AutoEnum`, which raised an `AttributeError` because `enum.AutoEnum` doesn't exist in this Python version; I then removed that part and reran the script.

- **Results/Findings & Relevance:**
  After fixing the script, running it showed:
  - `object_description(MyEnum.ValueA)` and `object_description(AnotherEnum.X)` both returned clean names like `MyEnum.ValueA` and `AnotherEnum.X`, instead of the verbose `<MyEnum.ValueA: 10>` style.
  - Nonenum values (strings, ints, lists, dicts, sets) were still rendered in their previous formats (e.g., `'test'`, `42`, `[1, 2, 3]`, `{'a': 1}`, `{1, 2, 3}`).
  - Function signatures were clean: for example, `(e: __main__.MyEnum = MyEnum.ValueA) -> None` for the enum default case, and `(s: str = 'default') -> None`, `(i: int = 42) -> None`, `(lst: list = [1, 2, 3]) -> None` for other defaults.
  - `isenumattribute(MyEnum.ValueA)` returned `True`, and `isenumattribute('not an enum')` and `isenumattribute(42)` returned `False`.
  These results confirm that the new enum handling in `object_description` works as intended and that enum detection logic is correct. The detailed printouts are not needed going forward; the key takeaway is that enums are now formatted cleanly and nonenum behavior is unchanged.

**Round 32** -- Running Sphinx's inspect util tests

- **Action:**
  I ran `python -m pytest tests/test_util_inspect.py -v`.

- **Results/Findings & Relevance:**
  All 38 tests in `tests/test_util_inspect.py` passed. This includes tests for `object_description` behavior, `stringify_signature`, and other inspect utilities. The only output was pytest's usual "PASSED" lines and some unrelated deprecation warnings from external libraries. This confirms that the change to `object_description` did not break any existing inspect behavior.

**Rounds 33--35 (Merged)** -- Running autodoc tests

- **Action:**
  I ran:
  - `python -m pytest tests/test_ext_autodoc.py -v -k "test_" | head -30` to get a subset of autodoc tests.
  - `python -m pytest tests/test_ext_autodoc_autofunction.py -v | head -20` to focus on autofunction behavior.
  - `python -m pytest tests/test_ext_autodoc.py::test_format_signature -v` to specifically test signature formatting.

- **Results/Findings & Relevance:**
  All selected tests passed. This indicates that the change to `object_description` has not interfered with autodoc's signature handling. The exact test names and "PASSED" lines are not crucial; what matters is that autodoc's signature formatting tests, which rely on inspect utilities, still pass, reinforcing that the fix is compatible.

**Rounds 36--37 (Merged)** -- Attempted Sphinx integration via temp build

- **Action:**
  I created `test_sphinx_integration.py`, which:
  - Writes a temporary `test_module.py` containing an enum and a function with an enum default.
  - Writes a minimal `conf.py` and `index.rst`.
  - Builds HTML using `Sphinx` (with `buildername='html'`) and attempts to inspect the generated `index.html` for enum representations.

- **Results/Findings & Relevance:**
  Sphinx successfully built the HTML with a warning:
  - `WARNING: autodoc: failed to import module 'test_module'; the following exception was raised: No module named 'test_module'`
  The script then printed the HTML content (first 2000 chars) and tried to search for enum values in the generated HTML. The important lesson is that the Sphinx build itself succeeded (including autodoc), but the test script's approach to importing `test_module` was flawed (the module wasn't on the Python path), so the integration test isn't a reliable signal for the fix. The detailed HTML output is not needed; the key is simply that Sphinx built successfully, indicating the core integration is fine despite the adhoc script's limitations.

**Rounds 38--39 (Merged)** -- Final verification script and confirmation

- **Action:**
  I created `test_final_verification.py`, which:
  - Defines a `MyEnum` enum and `ugly_enum_func(e: MyEnum = MyEnum.ValueA) -> None`.
  - Asserts that `object_description(MyEnum.ValueA)` returns `MyEnum.ValueA` and that `stringify_signature(sig)` contains `MyEnum.ValueA` but not `<MyEnum.ValueA: 10>`.
  - Tests several enum scenarios (`Color.RED`, `Color.GREEN`, `Status.PENDING`, `Status.RUNNING`) to ensure they render as `Color.RED`, etc.
  - Verifies that nonenum values (string, int, list, dict, set) are still rendered in their original formats.

- **Results/Findings & Relevance:**
  Running `python test_final_verification.py` produced:
  - For the exact issue scenario, `MyEnum.ValueA object_description: MyEnum.ValueA` and the function signature `(e: __main__.MyEnum = MyEnum.ValueA) -> None` (with no ugly `<MyEnum.ValueA: 10>`).
  - Enum tests showing `Color.RED -> Color.RED` etc., confirming consistent formatting.
  - Nonenum tests showing `'string'`, `42`, `[1, 2, 3]`, `{'key': 'value'}`, `{1, 2, 3}` as before.
  All assertions passed and the script ended with "ALL TESTS PASSED! The fix is working correctly."
  This script succinctly captures the expected behavior after the fix and is a good summary of the final verification. The exact printout is not critical beyond confirming that the clean enum representation is present and the old ugly format is absent.

**Round 40** -- Attempting a filtered inspect test run

- **Action:**
  I ran `python -m pytest tests/test_util.py -v -k "inspect" --tb=short`.

- **Results/Findings & Relevance:**
  All tests in `tests/test_util.py` were deselected by the `-k "inspect"` filter, so nothing actually ran; only the "selected 8 items / 8 deselected" message appeared. This is not informative for the fix and can be safely ignored.

**Round 41** -- Quick sanity check of enum helpers

- **Action:**
  I ran a small inline Python snippet:
  - Defined `TestEnum(enum.Enum)` with `VALUE = 1`.
  - Imported `object_description` and `isenumattribute` and printed their results for `TestEnum.VALUE`.

- **Results/Findings & Relevance:**
  The output was:
  - `isenumattribute TestEnum.VALUE: True`
  - `object_description TestEnum.VALUE: TestEnum.VALUE`
  - "All basic tests passed!"
  This confirms that the new enum branch in `object_description` works as intended for a simple enum, and that `isenumattribute` correctly identifies enum attributes. This is a concise sanity check and doesn't add new insights beyond what was already confirmed by `test_final_verification.py`.

[remaining_work]
I have completed all the main phases: I understood and restated the problem, explored the repository and located the relevant code paths (`stringify_signature` and `object_description` in `sphinx/util/inspect.py`), created a reproduction script to demonstrate the ugly enum repr, implemented a minimal fix to specialcase enum attributes, and then verified the change with both custom scripts and the existing Sphinx test suites. I also checked integration via a Sphinx build and confirmed that no tests were left failing or skipped for the core functionality.

In reviewing my work, I don't see unresolved mistakes or oversights: the issue understanding is correct (ugly enum repr in function signatures), the fix is applied at the true root cause (default value formatting via `object_description`), and I verified behavior after the change. The only "failure" encountered was an adhoc integration script that failed to import a temporary module, which is an environment issue rather than a regression from the patch and does not affect the code change itself. I did not modify any test files, as required, and the patch is minimal and focused. I also ran the key tests for inspect utilities (`tests/test_util_inspect.py`) and autodoc behavior (`tests/test_ext_autodoc.py`, `tests/test_ext_autodoc_autofunction.py`), all of which passed.

There are no mandatory technical tasks left to complete this issue. If I were to extend the work, the next concrete steps would be:
1. Optionally add a formal unit test under `tests/` that specifically asserts `object_description(MyEnum.ValueA) == "MyEnum.ValueA"` and that `stringify_signature` for a function with an enum default yields the clean representation (this would use the behavior already validated in `test_final_verification.py` from rounds 38--39).
2. Optionally clean up or remove the adhoc helper scripts (`test_enum_issue.py`, `test_comprehensive.py`, `test_final_verification.py`, `test_sphinx_integration.py`) if this were a real PR, relying instead on the official test suite.
The information needed to perform any such followup is: the exact patch to `object_description` (round 26/44), the role of `stringify_signature` in formatting defaults (round 18), and the behavioral expectations validated by the helper scripts (compressed rounds 28--39, especially the outputs showing clean enum rendering and nonregression for other types). No further bugfixing steps are currently necessary; the issue is resolved and wellverified.
```

> 訳注（要旨）: 関数シグネチャの中で Enum の既定値が醜く表示されるバグの修正とテストがほぼ完了したステップ 46 で、修正後の検証スクリプトとテスト実行が続いたラウンド 28〜41 を集約した例である。重要な観察（Enum が `MyEnum.ValueA` のように表示され、Enum 以外の値の表示は変わらない）は残し、詳細な出力は省いている。`[remaining_work]` には、必須の作業は残っておらず、任意の後片付けだけが残ると明記している。

***Example 4: Critical-Issue Refocusing*（例 4: 重大な問題への再集中）.** インスタンス: `matplotlib__matplotlib-20826-0`。ステップ: 65。

```text
[analysis]
So far in this trajectory, I've moved through understanding, exploration, and early verification. In round 1 I restated the issue and constraints. Rounds 2--28 (now already compressed by the user) covered initial repo inspection and locating the key code paths: `Axes.clear`/`Axes.cla` in `lib/matplotlib/axes/_base.py` and `Axis.clear` in `lib/matplotlib/axis.py`, plus the shared-axis machinery using `_sharex`, `_sharey`, `_shared_axes`, and `cbook.Grouper`. In rounds 29--40 I created several small reproduction/debug scripts (`reproduce_issue.py`, `reproduce_issue2.py`, `reproduce_issue3.py`, `test_visual_issue.py`) to observe behavior with `clear()` and shared axes, and confirmed that in this older Matplotlib (1.3.1) the behavior is different from the reported 3.4.2 bug but still shows tick-label issues. In rounds 46--50 and 61--62 I dug deeper into `Axes.cla`, `sharex`, `sharey`, and tick labels, and in rounds 57--60 I ran the project's existing tests, including those around `cla`, shared axes, and autoscaling, all of which currently pass. In rounds 61--62 I wrote `debug_tick_visibility.py` to compare tick label counts and visibility before and after `clear()` on a 2x2 shared grid and saw that after `clear()` all axes incorrectly show xtick labels, whereas before `clear()` the expected pattern is that only the bottom row shows them.

Looking critically at the current state, there are several gaps. First, I haven't implemented any fix yet; the code is still at the exploratory/debugging stage. Second, my understanding of the root cause is still incomplete: I've identified that `Axes.cla` calls `Axis.clear()` on the x/y axes and then re-invokes `sharex`/`sharey`, but I haven't yet demonstrated conclusively how this leads to tick labels being shown when they should be hidden. I have strong evidence that the bug is real and reproducible in this repo (round 62), but I haven't tied that to a specific line in the code that changed between 3.4.1 and 3.4.2. Third, I haven't run the full test suite; I only ran `test_axes.py` with specific `-k` filters, and those all passed, which is good but doesn't guarantee that the specific shared-axis tick-label behavior is covered by tests. Fourth, I haven't yet checked whether any tests have been added specifically for this issue (the user said tests are already updated, but I only ran a subset). Finally, I haven't yet designed and applied a minimal patch--no changes to library code have been made--so all of my analysis is theoretical so far.

Going forward, the pending tasks are clear. I need to (1) pinpoint the exact root cause in the code: likely in `Axes.cla` or in the shared-axis helper logic (e.g., `sharex`/`sharey` or `Grouper`) that controls tick-label visibility; (2) design a minimal fix that restores the 3.4.1 behavior after `clear()` on shared axes, ensuring that labels on "edge" axes (top row, right column) remain hidden; (3) implement that patch in the appropriate non-test file (likely in `axes/_base.py` and/or `axis.py`); (4) re-run the reproduction scripts (`test_tick_visibility.py`, `debug_tick_visibility.py`, and the bug-report-style script I haven't yet created) to verify that the bug is fixed; and (5) run the relevant tests again, especially `lib/matplotlib/tests/test_axes.py` and perhaps any new tests the user added for this issue, plus maybe `test_ticker.py` if they touch tick behavior. I also need to consider edge cases: non-shared axes should still behave as before, and clearing and re-plotting should not affect limits or inversion flags.

If I were to hand these tasks off to another person who only sees a compressed trajectory, they would absolutely need some specific information. To write the patch, they must know the current behavior of `Axes.cla` and `Axis.clear` (rounds 8--15 and 46--48), the semantics of `_sharex`, `_sharey`, `_shared_axes`, and `Grouper` (rounds 16--23 and 63), and the observed behavior from `debug_tick_visibility.py` (round 62) that shows tick labels incorrectly appearing after `clear()` on a 2x2 shared grid. They also need to know that all existing tests in `test_axes.py` pass (rounds 57--60), so any fix must preserve that. For running tests, they need to understand which tests are relevant: the shared-axis tests and the `cla` tests in `test_axes.py` (round 59). For implementing the fix, they need to know the exact reproduction scenario: `plt.subplots(2, 2, sharex=True, sharey=True)` followed by `ax.clear()` and `ax.plot(x, y)` (rounds 54, 61--62). For edge cases, they need awareness that non-shared axes and inversion behavior are already covered by existing tests (round 59).

Given these information needs, I should compress a range that mostly contains exploratory scripts and outputs whose detailed content isn't essential, while preserving the key discoveries and code snippets. The user already compressed rounds 2--28, so that's off-limits for partial overlap. The later rounds (57--65) are still active and I want to keep them as-is for now since they contain test runs and the latest `debug_tick_visibility.py` output. The mid-range rounds 29--40 are heavy on ad-hoc scripts (`reproduce_issue*.py`, `test_visual_issue.py`) and their outputs, which were useful at the time but don't need to be preserved verbatim; the main lessons from those runs are already captured in my reasoning. Rounds 41--45 contain some direct code viewing (`sharex`/`sharey`), but those snippets are short and still relevant, so I prefer to keep them uncompressed. Rounds 46--50 include a large `cla()` dump and `sharex`/`sharey` dumps; I'd like to preserve those verbatim because they're directly relevant to designing the patch. Rounds 51--54 are mostly thought-tool calls and yet another reproduction script (`test_exact_issue.py`); the essential insight is that the bug is reproducible and tick labels behave as described, but the script itself can be summarized. Rounds 56--60 are important test runs and should be kept verbatim.

So the best compression candidate is rounds 29--40. This range doesn't overlap the already-compressed 2--28, covers a substantial chunk of tokens, and is mostly exploratory scripts and outputs. Within 29--40, some content needs to be preserved more carefully: the fact that `reproduce_issue.py` and `reproduce_issue2.py` show tick labels appearing after `clear()` (even if the exact outputs aren't needed), and that `test_visual_issue.py` confirms the behavior difference between "with clear" and "without clear" plots. The detailed printouts of tick label counts and texts can be summarized while retaining the key findings. The exact code of those scripts can be heavily compressed, since I've already internalized their purpose and behavior and can reconstruct them if needed. No critical stack traces or error messages appear in these rounds; they're mostly stdout and file creation.

Therefore, I will compress rounds 29--40 into a merged summary that preserves the important conclusions (bug reproduced in this repo, behavior differs from 3.4.2 but still wrong) and omits the verbose script bodies and console outputs.

[start_round_number]
29

[end_round_number]
40

[content]
**Rounds 29--40 (Merged)** -- Exploratory reproduction scripts and initial observations

- **Action:**
  Created several standalone Python scripts in `/testbed` to reproduce and inspect the `clear()`/shared-axes behavior:
  - `reproduce_issue.py` and `reproduce_issue2.py` (rounds 29--32): used `plt.subplots(2, 2, sharex=True, sharey=True)` with `ax.clear()` followed by `ax.plot(x, y)` and printed/inspected axis visibility and tick label counts.
  - `reproduce_issue3.py` (round 33--34): compared plots with and without `clear()` and printed internal state (`_sharex`, `_sharey`, visibility).
  - `debug_shared_axes.py` (round 37--38): probed tick counts and axis visibility before/after plotting and after `clear()` on a shared 2x2 grid.
  - `test_visual_issue.py` (round 39--40): created simple 1x2 and 2x2 subplot layouts, saved images, and printed tick label texts and counts.

- **Results/Findings & Relevance:**
  - All these scripts confirmed that in this repository (Matplotlib 1.3.1) the behavior is different from the 3.4.2 bug report but still incorrect: after `ax.clear()` on shared axes, tick labels and tick counts behave oddly, and the visual appearance includes unwanted ticks and labels.
  - Detailed console output (exact tick label texts, counts, and paths) is not needed going forward; the key takeaway is that `clear()` on shared axes causes tick labels to appear or change behavior in ways that should not occur.
  - The scripts were useful for initial diagnosis but are not required to be preserved verbatim; their main contribution is the confirmation that the bug is reproducible and that tick label behavior is affected by `clear()` on shared axes.

[remaining_work]
I have completed the early phases: understanding the issue (Phase 1), basic environment and test setup (Phase 2), exploration of relevant code (`Axes.clear`/`Axes.cla`, `Axis.clear`, shared-axis machinery) and creation of several reproduction/debug scripts (Phase 3 and 4), and partial test execution (Phase 2/7) with `pytest` runs on `test_axes.py` focused on shared axes and `cla`. I have also started Phase 5 by analyzing the behavior of `Axes.cla`, `sharex`/`sharey`, and tick labels, and by confirming the bug with `debug_tick_visibility.py` (round 62), which shows that after `ax.clear()` on a 2x2 shared grid, all axes incorrectly display xtick labels while only the bottom row should show them.

There are still unresolved issues: I have not yet pinpointed the exact root cause in the code, have not implemented any fix, and have not run the full test suite or any newly added tests for this issue. I haven't validated that a future patch will not regress other behaviors (like axis inversion or non-shared axes). The main gap is that my analysis is still conceptual; I know that `Axes.cla` calls `Axis.clear()` and then re-invokes `sharex`/`sharey`, but I haven't yet proven how that leads to tick labels not being hidden after `clear()`.

Next, I will: (1) revisit the code in `lib/matplotlib/axes/_base.py` and `lib/matplotlib/axis.py` (notably `Axes.cla`, `Axis.clear`, `sharex`, `sharey`, and tick label visibility logic, as examined in rounds 46--48 and 65) to identify the precise mechanism that should hide tick labels on shared axes after `clear()`; (2) design a minimal patch that restores the 3.4.1 behavior (e.g., by ensuring tick label visibility is reset or re-applied correctly when axes are re-shared in `cla`, or by adjusting how `Axis.clear` interacts with shared axes); (3) implement this patch in the appropriate non-test files (likely `axes/_base.py` and possibly `axis.py`); (4) re-run the key reproduction scripts `test_tick_visibility.py` and `debug_tick_visibility.py` (rounds 47--48 and 61--62) to confirm that after `clear()` on shared axes, tick labels appear only on the correct axes; and (5) re-run the relevant tests in `lib/matplotlib/tests/test_axes.py` (rounds 57--60) and any new tests the user added for this issue, plus consider running `test_ticker.py` if it touches tick behavior, to ensure no regressions.
```

> 訳注（要旨）: 共有軸で `clear()` を呼んだ後に目盛りラベルが誤って表示されるバグについて、ステップ 65 の時点でまだ修正を実装しておらず、根本原因も特定できていないことを批判的に認めた例である。探索的な再現スクリプトが並ぶラウンド 29〜40 を圧縮し、根本原因の特定、最小限のパッチ、再現スクリプトとテストによる検証という具体的な計画に焦点を戻している。

### C.2 Trajectory Examples（軌跡の例）

完全な軌跡の例を示す。
ここで、セグメント（segment）は、記憶管理を挟まずに連続するイシュー解決の操作の列を表す。

***Case A: scikit-learn-14087-0*（事例 A）.** この事例は、再現から診断へ進み、その後で広範な回帰テストへと進む。

- **Segment 1（セグメント 1）.**
  - イシューを理解し、環境が使える状態であることを確認する。
  - `logistic.py` を調べ、関連する `LogisticRegressionCV` の経路を特定する。
  - `refit=False` での失敗を明らかにするため、最初の再現スクリプト（reproducer）とデバッグ用スクリプトを作成する。
- **Memory Management 1（記憶管理 1）.** **ラウンド 5〜23** の、初期の準備、コードの読解、最初の再現スクリプト・デバッグ用スクリプトの試行を圧縮する。
- **Segment 2（セグメント 2）.**
  - 元の失敗する状態に戻し、修正の候補をより注意深く比較する。
  - 二値（binary）と多項（multinomial）の扱いについて、分岐の正確な挙動を追跡する。
  - 実装の周辺に的を絞ったデバッグを追加し、形状／インデックスの前提がどこで破れるかを切り分ける。
- **Memory Management 2（記憶管理 2）.** **ラウンド 24〜33** の詳細な診断作業を圧縮する。これには、分岐の追跡、元の失敗する状態への復帰、失敗する正確な経路の切り分けが含まれる。
- **Segment 3（セグメント 3）.**
  - 主要な修正が元のイシューを解決することを確認する。
  - パッチが狭すぎないことを確かめるため、multinomial と `refit=True` のケースを調べる。
  - 上流のテストスイートのうち、ロジスティック回帰に絞ったテストを実行する。
- **Memory Management 3（記憶管理 3）.** **ラウンド 34〜54** の、的を絞った検証の段階を圧縮する。これには、再現スクリプトの再確認、multinomial／refit の調査、ロジスティック回帰に絞ったテストが含まれる。
- **Segment 4（セグメント 4）.**
  - 複数のシナリオを網羅する、より包括的な検証スクリプトを追加する。
  - より広いロジスティック回帰のテストの部分集合と、イシューそのものの確認を実行する。
  - 回帰テストスイートによって、修正がきれいに一般化することが確認された後に終了する。

***Case B: astropy__astropy-13579-0*（事例 B）.** この事例は、要約の順序が単調でない例を示す。
生のツール呼び出しの順序では、エージェントはまず軌跡の中ほどにある再現のブロックを圧縮し、後で戻ってそれより前の準備のブロックを圧縮し、その後で初めて、後続の診断のブロックを要約する。

- **Segment 1（セグメント 1）.**
  - スライスされた WCS（sliced-WCS）のバグを理解し、Astropy の環境を確認し、関連する `wcsapi` のファイルとテストの場所を特定する。
  - 高レベルの座標ヘルパーを使って、最初の完全な再現を試みる。
  - これらの試みが、不要な座標系（frame）の解決の複雑さを持ち込むことに気づく。
- **Memory Management 1（記憶管理 1）.** 最初のプレフィックスではなく、**ラウンド 16〜25** を圧縮する。
  圧縮される内容は最初の完全な再現の試みであり、`wcs_to_celestial_frame`、`SkyCoord`、関連する座標系の仕組みを経由した失敗した回り道を含む。
- **Segment 2（セグメント 2）.**
  - 低レベルの WCS API を中心に、再現スクリプトを書き直す。
  - スライスされた WCS が、巨大な最初のピクセル座標を返すことを確認する。
  - 現在のデバッグの流れが再現の品質に集中している間、より古いリポジトリ／準備の文脈はそのまま完全に保持する。
- **Memory Management 2（記憶管理 2）.** **ラウンド 1〜15** を圧縮する。したがって、生の範囲は後ろ向きに移動する。
  圧縮される内容は、それより前のイシューの言い直し、環境の確認、ファイルの場所の検索、最初のコード読解であり、これらは Summary 1 の後も圧縮されずに残っていたものである。
- **Segment 3（セグメント 3）.**
  - `sliced_wcs.py` を調べ、内部のスライスの状態を計測するコードを入れる。
  - 落とされた（dropped）world 次元がどのように表現されているかを追跡する。
  - バグを、基盤となる WCS に渡す完全な入力タプルを `world_to_pixel_values` がどのように再構成するか、という点に絞り込む。
- **Memory Management 3（記憶管理 3）.** **ラウンド 26〜45** を圧縮する。
  圧縮される内容は、バグを確認するクリーンな再現スクリプトと、`world_to_pixel_values` の中での落とされたスペクトル次元の扱いをめぐる診断である。
- **Segment 4（セグメント 4）.**
  - パッチを仕上げ、的を絞った WCS のテストを実行する。
  - 元のシナリオと、その近辺のスライスされた WCS の確認のための、最終的な検証スクリプトを追加する。
  - 以前の座標系に関する回り道を再び開くことなく終了する。

***Case C: django__django-12193-0*（事例 C）.** この事例は、段階的な再圧縮（progressive recompression）のコンパクトな例を示す。
2 回目の要約は新しい境界から始まるのではなく、すでに要約された**ラウンド 3〜10** のブロックを再圧縮しつつ、範囲を**ラウンド 3〜21** まで前方に広げる。

- **Segment 1（セグメント 1）.**
  - Django の環境を調べ、通常のテストの入口を試す。
  - `pytest`、`manage.py`、`python -m django.test` が、ここではこのケースを実行する正しい方法ではないことを確認する。
  - 準備の計画を、単独で動く検証スクリプトと直接のコード調査に絞り込む。
- **Memory Management 1（記憶管理 1）.** **ラウンド 3〜10** を圧縮する。
  圧縮される内容は、初期の環境の調査、失敗したテストの入口の試行、そしてこの環境では標準的なテストの入口が利用できないという結論である。
- **Segment 2（セグメント 2）.**
  - `SplitArrayField` と関連するウィジェットのコードの場所を特定する。
  - ウィジェットの文脈（context）の経路と、その近くのチェックボックスの扱いを読み通す。
  - それ以前の準備で得た知見を、Boolean フィールドの描画の経路に結びつける。
- **Memory Management 2（記憶管理 2）.** **ラウンド 3〜21** を圧縮する。すでに要約されたラウンド 3〜10 のブロックをさらに圧縮し、範囲を広げて、後の `SplitArrayField` のコード調査とウィジェットの診断までを覆う。
- **Segment 3（セグメント 3）.**
  - 最終的な検証スクリプトによって、チェックボックスの属性の挙動が期待される描画のロジックと一致することが確認された後に終了する。
  - 以前の環境調査のステップを別個の文脈として再び開くことなく終了する。
