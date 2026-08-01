---
type: summary
source_path: "raw/articles/Scaling Managed Agents_ Decoupling the brain from the hands.md"
source_kind: blog
title: "Scaling Managed Agents: Decoupling the brain from the hands"
authors: [Lance Martin, Gabe Cemaj, Michael Cohen]
year: 2026
venue: "Anthropic Engineering Blog"
ingested: 2026-08-01
tags: [agent-frameworks, agent-loop, context-engineering, agent-safety-and-guardrails, agent-observability, model-context-protocol, harness, meta-harness, sandbox, session]
translation: "[[translations/2026-managed-agents]]"
---

# Managed Agents — ハーネスが陳腐化し続ける前提を、インターフェースで受け止める（Anthropic, 2026）

> 原典: [[translations/2026-managed-agents]] ・ `raw/articles/Scaling Managed Agents_ Decoupling the brain from the hands.md`
> 著者・年・媒体: Lance Martin, Gabe Cemaj, Michael Cohen / 2026-04-08 / Anthropic Engineering Blog

## 一言まとめ

エージェントを **session（イベントログ）/ harness＝brain（Claude を回すループ）/ sandbox＝hands（実行環境）** の 3 つの差し替え可能なインターフェースに仮想化した、ホスト型サービス Managed Agents の設計記録。OS がハードウェアを process・file に仮想化して「まだ書かれていないプログラム」に備えたのと同じ発想で、**ハーネスが陳腐化し続けるという前提そのものを設計に織り込んだ**。

## 背景と問題意識

Anthropic の実務連作（[[summaries/2024-building-effective-agents]] → [[summaries/2025-multi-agent-research-system]] → [[summaries/2025-effective-harnesses]] → [[summaries/2026-harness-design]]）を貫く主題は、**ハーネスの各部品は「Claude が単独ではできないこと」についての仮定を符号化している**というものだった。4 本目はそこから「新モデルが出たら 1 部品ずつ外して検証する」という**縮小の方法論**を導いた。

本作はその次の一手である。前作までが「ハーネスをどう作り、どう剥がすか」だったのに対し、本作の問いは **「ハーネスが入れ替わり続ける前提で、その下に何を敷くか」**。冒頭で挙げられる例は wiki に既にある逸話と同じもの——Sonnet 4.5 はコンテキスト限界が近づくと仕事を早々に畳む（context anxiety）ので context reset を足したが、Opus 4.5 ではその挙動が消えており、リセットは死荷重になった。同じ教訓を、個別のハーネスを直す話ではなく**インターフェース設計の話**に読み替えたのが本作である。

> 補足: context reset が不要になった時期について、本記事は「Opus 4.5 で挙動が消えていた」と書く。[[summaries/2026-harness-design]] 側の記述（Opus 4.5 で大幅減、4.6 で撤去）と整合し、矛盾ではない。

## 提案手法 — 3 つのインターフェースへの仮想化

<figure>

![](../../raw/assets/2026-managed-agents/architecture-overview.png)

<figcaption>図1（再掲）: Managed Agents の全体構成。中央の Harness をハブに、Session（左）・Sandbox（右）・Tools + Resources / MCP（上）・Orchestration（下）がすべてインターフェース越しに接続される。出典: 本記事。</figcaption>
</figure>

- **session** — 起きたことすべての append-only なイベントログ。**ハーネスの外側**に永続化される。
- **harness（brain）** — Claude を呼び、そのツール呼び出しを配線するループ。**ステートレス**で使い捨て。
- **sandbox（hands）** — コードを実行しファイルを編集する環境。`execute(name, input) → string` という 1 本の口だけを持つ。

記事が公開したインターフェース仕様（図の表を転記）は、散文に出てこない構成要素まで含んでおり、この設計の骨格がよく分かる。

| コンポーネント | インターフェース | 何がそれを満たすか |
| --- | --- | --- |
| Session | `getSession(session_id) -> (Session, Event[])` / `getEvents(session_id) -> PendingEvent[]` / `emitEvent(id, event)` | 順に消費でき冪等な追記を受ける append-only ログなら何でも（Postgres, SQLite, インメモリ配列） |
| Orchestration | `wake(session_id) -> void` | ID で関数を呼び失敗時に再試行できるスケジューラなら何でも（cron, キュー, while ループ） |
| Harness | `yield Effect<T> -> EffectResult<T>` | エフェクトを yield し進捗を Session へ追記するループなら何でも |
| Sandbox | `provision({resources})` / `execute(name, input) -> String` | 一度設定して何度も呼べる実行器なら何でも（ローカルプロセス, リモートコンテナ） |
| Resources | `[{source_ref, mount_path}]` | 参照で取得できる永続ストアなら何でも（Filestore, GCS, git リモート, S3） |
| Tools | `{name, description, input_schema}` | 名前と入力の形で書ける能力なら何でも（MCP サーバー, カスタムツール） |

「何がそれを満たすか」の列がすべて「〜なら何でも」で書かれているのが要点で、**実装ではなく形だけを固定する**という設計思想がそのまま表れている。

### pets vs cattle — なぜ分離したのか

出発点は全部を 1 コンテナに同居させる構成だった。これは **pet（ペット）** を飼う——名前を付けて手ずから世話をし、失うわけにいかない個体を抱える——ことであり、対義語の **cattle（家畜, 交換可能で使い捨てられる個体）** と対比される、インフラ運用の古い教訓に引っかかった。具体的な症状は 3 つ:

1. **コンテナが死ぬと session が消える**。
2. **障害を局在化できない**（→ [[agent-observability]]）。唯一の覗き窓が WebSocket のイベントストリームで、ハーネスのバグも・パケットの喪失も・コンテナのオフライン化も**すべて同じに見えた**。中を見るにはシェルで入るしかないが、そのコンテナには顧客データも入っていたため、実質的にデバッグ能力が無かった。
3. **顧客の VPC に繋げない**。ハーネスが「作業対象は自分と同じコンテナにある」と想定していたため、顧客はネットワークをピアリングするか自前でハーネスを動かすかを迫られた。

分離後は両方が cattle になる。コンテナが死ねばハーネスはそれを**ツール呼び出しのエラーとして受け取り Claude に返す**（Claude が再試行を決めれば `provision({resources})` で作り直す）。ハーネスが死んでも、session がその外にあるので `wake(sessionId)` → `getSession(id)` で最後のイベントから再開できる。**ハーネスの中に、クラッシュを生き延びる必要のあるものが 1 つも無い**——これが設計の要である。

### session は context window ではない

長期タスクではコンテキストウィンドウがあふれる。標準的な対処（compaction＝要約圧縮、memory ツールへの書き出し、context trimming＝古いツール結果や思考ブロックの選択的除去）は**どれも「何を残すか」の不可逆な決定を伴う**。そして**どのトークンが将来のターンで必要になるかは事前に分からない**。

本記事の解は、session を**コンテキストウィンドウの外に生きるコンテキスト・オブジェクト**として扱うことである。`getEvents()` でイベントストリームの**位置指定スライス**を取れる——最後に読んだところから再開する、ある瞬間の数イベント前まで巻き戻して前段を見る、ある行動の前の文脈を読み直す、といった使い方ができる。

<figure>

![](../../raw/assets/2026-managed-agents/session-events.png)

<figcaption>図3（再掲）: Harness から Session へ `getEvents` を要求し、Events が返る。返ったイベントはハーネス内で変換されてから Claude のコンテキストへ入る。出典: 本記事。</figcaption>
</figure>

重要なのは**関心の分離**である。session は「永続的で問い合わせ可能」であることだけを保証し、取得したイベントをどう変換してコンテキストに積むか（プロンプトキャッシュのヒット率を上げる整列、その他のコンテキスト設計）は**ハーネス側の裁量**に残す。理由は明快で、**将来のモデルにどんなコンテキスト設計が要るかは予測できない**から。ここでも「予測できないものは固定しない」という一貫した態度が取られている。

なお本記事は、この「コンテキストをウィンドウ外のオブジェクトとして持ち、コードで絞り込む」着想の先行研究として **Recursive Language Models**（Zhang・Kraska・Khattab, arXiv:2512.24601）を挙げている（REPL 内のオブジェクトを LLM がコードで filter/slice する形）。

### セキュリティ — 仮定を狭めるのではなく、到達不能にする

密結合時代の脆弱性の連鎖が率直に書かれている: Claude が生成した未信頼コードと認証情報が**同じコンテナ**にいたため、prompt injection（外部入力に紛れ込ませた指示でエージェントを乗っ取る攻撃）は「自分の環境を読ませる」だけでよかった。トークンを取られれば、攻撃者は**無制限の新しい session を起こして作業を委任できる**。

注目すべきは対処の選び方である。スコープを狭めたトークンを渡す（narrow scoping）のは自明な緩和策だが、それは**「限定トークンで Claude に何ができないか」という仮定を符号化する**にすぎない——そして Claude は賢くなり続ける。つまり**この記事の主題（仮定は陳腐化する）がセキュリティにも適用される**。だから採ったのは構造的な修正、すなわち **sandbox からトークンに到達できなくする**ことだった。2 パターン:

- **リソースに同梱** — Git は sandbox の初期化時にアクセストークンで clone し、ローカルの git リモートに配線しておく。以後 `push` / `pull` は**エージェントがトークンに触れることなく**動く。
- **sandbox 外の vault** — MCP（Model Context Protocol, ツールやデータソースをモデルに接続する標準プロトコル）ツールは、OAuth トークンを金庫に置き、Claude は**専用プロキシ**経由で呼ぶ。プロキシは session に紐づくトークンを受け取り、対応する認証情報を vault から引いて外部サービスを叩く。**ハーネスすら認証情報を知らされない**。

### many brains, many hands

hands はどの brain にも結合していないので、**brain 同士が hands を受け渡せる**。ハーネスは自分が繋がっている sandbox が「コンテナなのか、電話なのか、ポケモンのエミュレータなのか」を知らない。

<figure>

![](../../raw/assets/2026-managed-agents/many-brains-many-hands.png)

<figcaption>図4（再掲）: 複数の Harness と複数の Sandbox が、固定的に結びつかない形で相互に接続される。出典: 本記事。</figcaption>
</figure>

ここで面白いのは**能力の順序**である。多数の実行環境を扱い「どこへ仕事を送るか」を決めるのは、単一シェルで動くより難しい認知タスクなので、当初は**モデルにそれができなかったから**単一コンテナにした。ところが知能が伸びると、今度は単一コンテナのほうが制約になった（そのコンテナが落ちると、brain が手を伸ばしていた全 hand の状態が消える）。**モデル能力の向上が、アーキテクチャの制約を反転させた**具体例である。

## 実験結果と知見

数値は 1 つだけだが、性格がはっきりしている。コンテナを**必要になったときだけ** brain がツール呼び出しで確保する構成にした結果:

- **p50 の TTFT が約 60% 低下、p95 が 90% 以上低下**。

分離前は、たとえ sandbox に一度も触れない session であっても、リポジトリの clone・プロセス起動・保留イベントの取得という**コンテナ準備を全額前払い**していた。分離後はオーケストレーション層が session からイベントを引いた瞬間に推論を開始できる。

> **注意**: ここでの TTFT（Time To First Token, 最初の応答トークンまでの時間）は、[[llm-inference-optimization]] で扱う**推論側の TTFT（prefill の計算時間）とは別物**である。本記事が縮めたのは「session が仕事を受け付けてから最初のトークンが出るまで」であり、その大部分は**コンテナ確保の待ち時間**だった。同じ略称だが測っている区間が違うので、数値を横並びに比較してはいけない。p95 の改善幅が p50 より大きいのも、テール側にコンテナ確保の遅い個体が集中していたことの表れと読める。

## 限界・批判的視点

- **自社ホスト型サービスの設計記録であり、第三者による検証はない**。アーキテクチャの妥当性は語られているが、比較対象となる代替設計での実測はない。
- **TTFT の測定条件が書かれていない**。ワークロードの構成比（sandbox を使う session の割合）に強く依存する数値のはずで、「sandbox を必要としない session が多い」ほど改善幅は大きく出る。自分の環境に外挿するには、まずその比率を測る必要がある。
- **「多くの hands を使い分けられる」はモデル能力への依存が強い**。記事自身が「初期モデルにはできなかった」と認めており、これは**本記事が警告するまさにその種の仮定**である。逆向きの陳腐化（将来のモデルがもっと少ないインターフェースで済ませられる）も起こりうる。
- **vault + プロキシは運用者が Anthropic 側にいる前提**。自前でこの構成を組む場合に何が必要か（プロキシの信頼境界、vault の運用、session トークンの失効設計）は書かれていない。構造的な修正という方針は真似られるが、実装はそのまま持ち出せない。
- **session が単一障害点に見える**。ハーネスも sandbox も cattle になった一方、session は「永続的であること」を保証する存在として残る。その可用性・整合性・保持期間の設計には触れられていない。
- **append-only ログのコスト**が論じられていない。長期タスクでイベントが際限なく積み上がったとき、`getEvents()` の絞り込みをどう賢くするかは結局ハーネス側の課題として残る——つまり**コンテキスト設計の難しさは消えたのではなく、場所を移した**。

## 実装・運用上の示唆

- **真似る価値があるもの**: 「状態をループの外に出す」——ハーネスをステートレスにしてイベントログを外部に置けば、ハーネスは落ちてよいものになる。これはフレームワーク選択に依存しない設計原則である。
- **失敗をツール呼び出しのエラーとして扱う**。コンテナ障害を特別なインフラ例外にせず、モデルが見て再試行を判断できる普通のツール結果に落とし込むと、回復の制御をモデル側に返せる。
- **認証情報は「狭める」より「届かなくする」**。narrow scoping は攻撃者の能力についての仮定を含む。到達不能化は含まない。→ [[agent-safety-and-guardrails]]
- **不可逆な圧縮の前に、復元可能な保存を用意する**。compaction や trimming を否定するのではなく、**捨てた先が残っている**状態を先に作る。これは [[summaries/2025-manus-context-engineering]] の「復元可能（restorable）な圧縮」と同じ結論に、別の経路から到達している。
- **観測できない構成を作らない**。「障害の種類が全部同じに見える」状態は設計の失敗として扱ってよい。分離は可用性のためだけでなく、**障害を局在化できるようにするため**でもある。→ [[agent-observability]]

## 用語と略称

- **LLM** = Large Language Model（大規模言語モデル）
- **TTFT** = Time To First Token（仕事を受け付けてから最初の応答トークンが出るまでの時間。本記事では主にコンテナ確保の待ち時間を指す）
- **p50 / p95** = 分布の 50 パーセンタイル（中央値）／95 パーセンタイル（遅い方から 5% の境目。テールの遅さを表す）
- **harness（ハーネス）** = ループ・ツール・コンテキスト管理・プロンプトを束ねて LLM を動かす実行基盤
- **meta-harness** = 本記事の用法では「**多様なハーネスを載せられる汎用インターフェース層**」。同名だが [[summaries/2026-meta-harness]] の用法（ハーネスを探索で最適化する外側ループ）とは指す方向が逆なので注意（→ [[agent-frameworks]] に両義を対置した）
- **session** = 起きたことすべての append-only なイベントログ
- **sandbox（サンドボックス）** = コード実行を隔離環境に閉じ込める仕組み。本記事では実行環境そのものを指す
- **pets vs cattle** = 手ずから世話をする交換不能な個体（ペット）と、使い捨て可能で交換自在な個体（家畜）の対比。インフラ運用の比喩
- **prompt injection** = 外部入力に埋め込まれた指示でエージェントを乗っ取る攻撃
- **MCP** = Model Context Protocol（ツールやデータソースをモデルに接続する標準プロトコル）
- **VPC** = Virtual Private Cloud（クラウド上に切り出された専有ネットワーク）
- **compaction** = 履歴をその場で要約して圧縮すること
- **context trimming** = 古いツール結果や思考ブロックを選択的に取り除くこと
- **REPL** = Read-Eval-Print Loop（対話的にコードを実行する環境）
- **OAuth** = 認証情報そのものを渡さずにアクセス権を委譲する標準的な仕組み
- **idempotent（冪等）** = 同じ操作を複数回行っても結果が 1 回のときと変わらない性質

## 関連ページ

- [[agent-frameworks]] — 本記事の主な受け皿。連作 5 本目としての位置づけと、meta-harness の語義の書き分け
- [[context-engineering]] — session ≠ context window、不可逆な取捨選択の問題
- [[agent-safety-and-guardrails]] — 認証情報の隔離と、仮定を符号化する緩和 vs 構造的修正
- [[agent-observability]] — 障害を局在化できなかった実例と、session ログ＝永続トレース
- [[model-context-protocol]] — MCP をプロキシ＋vault 越しに呼ぶ実装パターン
- [[multi-agent-systems]] — many brains / many hands と hands の受け渡し
- [[agent-loop]] — ループの実行場所を session / harness / sandbox に分解する見方
- [[summaries/2026-harness-design]] — 直前の 4 本目（「作る→剥がす」の縮小の方法論）
- [[summaries/2026-meta-harness]] — 同名だが別方向の meta-harness（探索による自動設計）
- [[summaries/2025-manus-context-engineering]] — 「復元可能な圧縮」に別経路で到達した先例
