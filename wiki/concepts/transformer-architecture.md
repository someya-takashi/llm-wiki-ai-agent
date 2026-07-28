---
type: concept
aliases: [トランスフォーマー, decoder-only, attention, linear attention, DeltaNet, Mamba, MLA, MoE, Mixture of Experts]
tags: [transformer-architecture, llm-foundations]
related:
  - "[[llm-inference-optimization]]"
  - "[[agent-memory]]"
  - "[[context-engineering]]"
  - "[[test-time-compute]]"
  - "[[reinforcement-learning-from-human-feedback]]"
summaries:
  - "[[summaries/2026-gpt2-to-kimi3]]"
  - "[[summaries/2025-deepseek-r1]]"
  - "[[summaries/2025-kimi-k2]]"
  - "[[summaries/2023-moe-explained]]"
  - "[[summaries/2026-kimi-k2.5]]"
  - "[[summaries/2026-gemma-4]]"
  - "[[summaries/2026-deepseek-v4]]"
  - "[[summaries/2021-switch-transformers]]"
updated: 2026-07-29
---

# Transformer Architecture（トランスフォーマーアーキテクチャ）

現代の LLM（Large Language Model, 大規模言語モデル）の土台となるニューラルネットワーク構造と、その発展の系譜を扱うページ。エージェントの視点では、アーキテクチャは**コンテキストウィンドウの上限・推論のコストとレイテンシ・長い trajectory での記憶の持ち方**を物理的に規定する層であり、[[context-engineering]] や [[llm-inference-optimization]] の制約の出どころにあたる。

## 基本構造 — decoder-only

GPT-2 以来の標準形は **decoder-only**（デコーダのみ）構成である（[[summaries/2026-gpt2-to-kimi3]] が実装コード付きで解説）: トークン埋め込み＋位置埋め込み → 同一構造のブロック × N 層 → 最終正規化 → LM head（隠れ状態を語彙のロジットへ写像）。各ブロックは **attention**（系列内の他トークンから情報を集める）と **MLP**（位置ごとの変換）を、**残差接続**（入力に出力を足し込む）で包んだもの。GPT-2 は 12 層・12 ヘッド・埋め込み 768 で 124M パラメータ、Kimi K3（2026）は 2.8T——**7 年で 22,580 倍**にスケールしたが、この骨格自体はほぼ変わっていない。

自己回帰生成では、過去トークンの key/value を **KV cache** に保存して再計算を避ける。この cache は系列長 O(N) で成長し、長コンテキストではメモリ帯域のボトルネックになる——以降のアーキテクチャ発展の大半は、この「**推論時の記憶をどう持つか**」への応答である → [[llm-inference-optimization]]。

## attention の系譜 — 固定容量メモリと追い出しポリシー

[[summaries/2026-gpt2-to-kimi3]] の整理に従うと、attention 変種の系譜は「固定容量の連想メモリには追い出しポリシーが要る」という一本の線で読める:

- **softmax attention**（2017〜）: 全トークン対で類似度を取る完全検索。表現力は最強だが、状態（KV cache）が O(N) 成長。
- **linear attention**（2020, Katharopoulos ら）: ELU+1 のような特徴写像を q, k に別々に適用して積を再結合可能にし、K・V の蓄積を**固定 D×D 状態**へ畳む。デコードは O(1) になるが、特徴写像は softmax の劣化近似であり、**加算だけの書き込みは容量超過で干渉する**（Schlag の Fast Weight Programmers が指摘した overcapacity 問題）。
- **DeltaNet**（delta rule）: 新しい key で古い値を読み出し、**差分だけを書く**選択的上書き。干渉を能動的に解消する。一般化 Householder 遷移行列への再パラメータ化により、チャンク単位の並列訓練が可能（チャンク C=64/128 がテンソルコアに合う）。
- **Gated DeltaNet**（Mamba-2 のゲートとの統合）: データ依存のスカラー α で状態全体を減衰させる**忘却ゲート**を追加（α=1 で純 delta rule、α=0 で全消去）。ただし減衰は一様。
- **KDA（Kimi Delta Attention）**: 減衰を**チャネルごと**に学習する細粒度ゲーティング。
- **ハイブリッド**（Kimi Linear / K3）: 固定状態の再帰系（KDA）と完全 softmax 検索（**MLA** = Multi-head Latent Attention, 潜在圧縮つきの完全 attention）を**層単位でインターリーブ**する（K3 は KDA 3 : MLA 1）。固定状態は必然的に情報を捨てるため、周期的な完全検索で取りこぼしを回収する分業である。
- **間引きと共有**（Gemma 系）: 状態を固定サイズに畳む代わりに、**softmax attention のまま KV cache を削る**別路線もある。Gemma 4（[[summaries/2026-gemma-4]]）は (1) local:global 比 5:1（大半の層を sliding window ローカル attention にし、全系列を見るグローバル層を間引く）、(2) グローバル層で **values = keys**（key を value として再利用し保存テンソルを半減）、(3) **p-RoPE**（位置回転を次元の一部 p=0.25 のみに適用）の 3 点でグローバル KV cache を実質 37.5% 削減した。完全検索の表現力を保ったままメモリを削る、エッジ・オンデバイス志向の解である。E2B/E4B の **per-layer embeddings**（層ごとの埋め込みを外部化し、総 5B/8B のうち実効 2.3B/4.5B だけを実効計算に使う）も同系の「容量と実効コストの分離」にあたる。
- **圧縮してから選ぶ**（DeepSeek-V4）: 第三の路線が **KV の学習圧縮＋スパース選択**の二段構えである（[[summaries/2026-deepseek-v4]], 2026）。**CSA**（Compressed Sparse Attention）は KV を m=4 トークンごとに 1 エントリへ学習された重み付き和で圧縮した上で、軽量な **Lightning Indexer** が各クエリに top-k（512〜1024）個の圧縮エントリだけを選ぶ。**HCA**（Heavily Compressed Attention）は m′=128 の激圧縮＋密 attention で、両者を層で交互配置する。圧縮では拾えない局所依存は直近 128 トークンの sliding window ブランチが補い、attention sink（合計 attention を 1 未満に調整できる学習ロジット）・部分 RoPE・QKV の RMSNorm を併用する。結果は劇的で、**1M コンテキストの KV cache は一般的な BF16 GQA8 構成の約 2%**、推論 FLOPs は前世代 V3.2 の 27%——linear 化が「状態を固定サイズに畳む」なら、この路線は「**可変長のまま解像度を落とし、読む場所を選ぶ**」ことで 100 万トークンを実用化した。1.6T/284B の 2 サイズで実証されている。

この系譜は [[agent-memory]] と美しく同型である——追記だけの記憶が干渉し（A-Mem 以前の生ログ蓄積）、選択的上書き（A-Mem の memory evolution）や退避・要約（MemGPT）が要る、という問題を、アーキテクチャは**重み・状態の内部で**解いている。

## MoE — 条件付き容量

**MoE**（Mixture-of-Experts）は、MLP を多数の「エキスパート」に分割し、学習された**ルータ**がトークンごとに一部だけを起動する疎な構造。総パラメータを増やしながら 1 トークンあたりの計算を抑える「条件付き容量」の手法である。系譜は古く、1991 年の Adaptive Mixture of Local Experts（エキスパート＋ゲートの原型）→ Shazeer 2017 のスパース化（Noisy Top-k Gating, 137B LSTM）→ GShard（transformer への適用・expert capacity）→ Switch Transformers（**top-1 で十分**という簡素化・1.6T/2048 エキスパート・事前学習 4 倍速）→ Mixtral 8x7B（オープン MoE の実用化）と発展した（[[summaries/2023-moe-explained]] が定番の入門整理）。

実用化を決めた転換点が **Switch Transformers**（[[summaries/2021-switch-transformers]], 2021）で、その貢献は機構の追加でなく**削除**にある。「ルータの学習には k≥2 のエキスパート比較が要る」という通念（Shazeer 2017）に反して **top-1** で品質を保てることを実証し（ゲート値を出力に掛ければ微分可能性は残る）、補助損失も単一の負荷分散損失 $\alpha N\sum_i f_i P_i$（$\alpha=10^{-2}$）へ簡素化した。安定化の実装レシピ——**selective precision**（ルータ関数の内部のみ float32。dispatch/combine は bfloat16 へ戻すので通信は軽いまま）・**0.1x の小初期化**（実行間分散 0.68→0.01）・**expert dropout**（微調整時にエキスパート層のみ 0.4）——はこの論文が確立したもので、同一計算資源で T5-Base 比 7 倍の事前学習高速化と、初の 1.6T モデル（Switch-C。訓練不安定ゼロ。むしろ FLOPs が 10 倍の Switch-XXL の方が不安定＝**不安定性はパラメータ数でなく FLOPs/トークンに相関**という観察も）に到達した。否定的結果の公開も価値が高い: attention への Switch 適用は bfloat16 で発散、あふれトークンを第 2 候補へ回す No-Token-Left-Behind は**効かない**（学習されたトークン・エキスパートの関連を崩すと逆効果）——「単純化が勝つ」という本論文の主題の裏づけである。

MoE の本質は**パラメータ数と計算量の分離**にある——Mixtral 8x7B は「メモリ 47B・推論 FLOPs 12B 相当」（attention 等は共有されるため 8×7B にはならない）。訓練の急所は**負荷分散**: 放置するとルータは少数の人気エキスパートへ自己強化的に収束するため、auxiliary loss（均等な重要度）・router z-loss（ゲートの大きなロジットへのペナルティ）・expert capacity（あふれの制御）の 3 点セットで抑える。**ルータは指数関数を含むため高精度必須**（エキスパートは bfloat16 でも、ルーティングは full precision——selective precision）。エキスパートの専門化は「言語」のような大きな単位ではなく**トークン群・浅い概念**の単位で起こり（句読点・固有名詞のエキスパート等）、疎モデルは過学習しやすい一方で **instruction tuning からは密モデル以上の恩恵**を受ける。Kimi K3 の実例（[[summaries/2026-gpt2-to-kimi3]]）: 898 エキスパートのうち**共有 2 つ**が全トークンを処理し、残り 896 から**各トークンに 16** をルータが選ぶ。さらにエキスパートを圧縮された**潜在空間**で動かして FLOPs をほぼ半減させる。DeepSeek-V3/R1（[[summaries/2025-deepseek-r1]] の基盤モデル）も大規模 MoE の代表例である。

「どれだけ疎にするか」には経験則がある。Kimi K2（[[summaries/2025-kimi-k2]], 1.04T/活性 32B）は**スパース性スケーリング則**——活性化パラメータ（FLOPs）を固定したまま総エキスパート数を増やすほど loss が下がる——を実測し、同じ検証 loss 到達に必要な FLOPs がスパース性 48 でスパース性 8 比 1.69 倍削減されることを示して、384 エキスパート・スパース性 48 を採用した。もうひとつの設計判断が示唆的である: **attention ヘッドを 128→64 に半減**した——128k 系列でヘッド倍増は推論 FLOPs +83% に対し loss 改善 0.5〜1.2% しかなく、**エージェント用途の長コンテキスト推論効率が事前学習品質より優先された**。アーキテクチャがエージェントのワークロードから逆算され始めた実例である → [[llm-inference-optimization]]。

## マルチモーダル拡張 — 視覚をいつ・どう繋ぐか

テキスト transformer に視覚を加える標準構成は「**視覚エンコーダ（ViT）→ プロジェクタ → LLM**」の 3 部品で、設計論点は (1) 視覚エンコーダの作り、(2) 視覚トークンをいつ混ぜるか、(3) 訓練インフラへの載せ方、に分かれる。Kimi K2.5（[[summaries/2026-kimi-k2.5]]）が 3 点すべてに具体的な設計判断を公開している:

- **MoonViT-3D**: SigLIP（対照学習系の視覚エンコーダ）初期化のネイティブ解像度 ViT。NaViT の patch packing——可変解像度の画像をパッチ列として 1 次元系列に詰め、分割・タイル化なしで扱う——を**時間方向へ一般化**し、連続 4 フレームを時空間ボリュームとして同一エンコーダで処理する（時間プーリングで 4 倍圧縮）。画像と動画で**完全に重みを共有**するため、静止画で得た知識がそのまま動画へ転移し、2,000 フレーム級の長時間動画理解を特化モジュールなしで達成した。
- **early fusion**: 視覚を「後付け」する従来の定石（訓練後半に視覚 50% など）に対し、総トークン予算固定の対照実験で**最初から低比率（視覚 10%）で混ぜる方が全指標で優る**ことを示した。late fusion はテキスト性能の一時劣化（dip-and-recover）を起こす——確立済みの言語表現空間に視覚トークンが突然入るドメインシフトが原因とされ、「モダリティは器の設計段階から混ぜる」方向の実証になっている。
- **DEP（Decoupled Encoder Process）**: Pipeline Parallelism では視覚エンコーダが Stage-0 に同居し、画像枚数・解像度の変動が負荷不均衡を生む。DEP は視覚エンコーダが「forward の始点・backward の終点」であるという計算グラフ上の位置を利用し、**全 GPU に複製して独立に前計算 → テキストと同じ並列化でバックボーン訓練 → 再計算して backward** の 3 段に分離。テキスト用に最適化済みの並列化戦略をそのまま再利用でき、マルチモーダル訓練効率はテキスト比 90%（→ [[llm-inference-optimization]] の「実装が実効性能を決める」と同じ構図）。

事後学習側の対応物（zero-vision SFT・視覚 RL がテキストも改善する双方向転移）は [[reinforcement-learning-from-human-feedback]] を参照。

対極の設計が **encoder-free** である。Gemma 4 12B（[[summaries/2026-gemma-4]]）は視覚・音声エンコーダを持たずにスクラッチから訓練される: 視覚は生の 48×48 RGB パッチを**単一の行列積（35M）**で埋め込みへ射影（550M ViT を置換。2D 座標埋め込みのみで空間認識を維持）、音声は 305M Conformer を完全廃棄して 40ms チャンクの生ベクトルを直接射影する。専用音声エンコーダなしでエンコーダ持ちの下位モデルと同等以上の ASR 性能を出しており、「エンコーダを鍛えて共有する」（K2.5 の MoonViT-3D）か「捨てて LLM 本体に吸収させる」（Gemma 4 12B）か——**モダリティ処理のパラメータをどこに置くか**が、2026 年時点のマルチモーダル設計の明確な分岐点になっている。

## 残差ストリームと深さ方向の検索

各層の入力は「埋め込み＋全先行層出力の等重み和」という**残差ストリーム**であり、層が深くなると古い情報が薄まる（残差の希釈）・後段の層が影響力を得るために出力を肥大させる、という問題を持つ。Kimi K3 の **AttnRes** は、和の各項に学習された重み（query-key ドット積から計算）を掛け、**先行層の出力を選択的に読む**——トークン方向だけでなく**深さ方向にも attention を張る**発想で、+2% のレイテンシで残差希釈の緩和と 1.25 倍の計算優位を得る（12 層ごとの blockwise 適用）。

もう一つの設計軸が**残差ストリームの多重化**である。Hyper-Connections（HC）は残差ストリームを n 倍幅（例: 4 本）に拡張し、層ごとに学習された行列で「どの残差レーンから読み・どう混ぜて書き戻すか」を決める——隠れサイズと独立な補完的スケーリング軸だが、深く積むと数値不安定になる。DeepSeek-V4 の **mHC**（Manifold-Constrained Hyper-Connections, [[summaries/2026-deepseek-v4]]）は、残差混合行列を**二重確率行列の多様体（Birkhoff 多面体）へ Sinkhorn-Knopp 反復で射影**してスペクトルノルム ≤1 の非拡大写像に制約し、「信号が層を経ても増幅されない」保証つきで多重残差を 1.6T 級・61 層で成立させた（実時間オーバーヘッド 6.7%）。AttnRes（読み方を選ぶ）と mHC（レーンを増やして安定に混ぜる）は、「残差接続は無設計の足し算でよい」という前提を壊す同時代の 2 解である。

オプティマイザ側にも系譜がある。**Muon**（勾配行列を Newton-Schulz 反復で直交化してから更新する）は Kimi K2 が MuonClip（QK-Clip 併用）として 1T 級で実証し（→ [[summaries/2025-kimi-k2]]）、DeepSeek-V4 も採用——ただし V4 は attention 側の QKV RMSNorm でロジット爆発を防げるため **QK-Clip を使わない**。1.6T 級で残る loss spike には、**Anticipatory Routing**（MoE のルーティングだけ過去パラメータで先読み計算し、ルーティングと本体の同期更新が作る外れ値の悪循環を断つ。spike 検出時のみ動的起動）と **SwiGLU Clamping**（線形成分 ±10）という「原理は未解明だが有効」と明記された 2 技法で対処している——大規模訓練の安定性が、理論より先に実践知として共有されている現状の記録でもある。

## 設計論点

- **「容量をどこに足すか」**: 本系譜の中心命題。パラメータ増は手段であり、各世代は先行システムの具体的な限界（干渉・一様減衰・残差希釈）に対して、システムが使える形で容量を足してきた。「盲目的なスケーリング」ではない、というのが 22,580 倍の中身である。
- **表現力と状態サイズのトレードオフ**: 完全検索（softmax）⇄ 固定状態（linear 系）の間に、選択的上書き・ゲート・ハイブリッド比率という設計変数の連続体がある。Kimi Linear の「full attention 超え」は自己申告の管理下比較であり、鵜呑みにはできない（→ [[agent-evaluation]] の provider-reported 問題）。
- **エージェントへの含意**: 固定状態ハイブリッドの普及は、長い trajectory のデコード単価とコンテキスト上限を変える。一方で「どの連想が減衰されるか」は学習された選択に委ねられるため、**長期タスクでの記憶の一貫性**という新しい評価軸が生まれる（未解決）。
- **アーキテクチャと能力は別レイヤ**: 推論能力（→ [[reasoning-and-planning]]・[[reinforcement-learning-from-human-feedback]]）は主に事後訓練で作られ、アーキテクチャはその器と経済性を決める。個別モデル（GPT 系, Claude, Gemini, Llama, DeepSeek, Kimi 等）の位置づけは、この器×訓練の組で理解する。

## 関連ページ

- [[llm-inference-optimization]] — KV cache・メモリ帯域・カーネルという「速さ」の側
- [[agent-memory]] — 同型の記憶問題をモデルの外側で解く系譜
- [[context-engineering]] — 有限ウィンドウ制約の使い方の側
- [[test-time-compute]] — アーキテクチャが捌く推論時計算の使い方
- [[summaries/2026-gpt2-to-kimi3]] — 本ページの主要な根拠原典
