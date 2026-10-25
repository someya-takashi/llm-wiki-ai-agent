---
type: translation
source_path: "raw/articles/Claude SWE-Bench performance.md"
source_page: "[[summaries/2025-swe-bench-sonnet]]"
original_language: en
translated_to: ja
translated_at: 2026-10-25
---

# Claude の SWE-bench 性能

> 原題: Claude SWE-Bench performance
> 著者: Anthropic（エージェントの最適化と執筆は Erik Schluntz。謝辞より）
> 出典: https://www.anthropic.com/research/swe-bench-sonnet ・ 2025-01-06

> ［訳注］Obsidian Web Clipper のクリップを底本とし、原ページ（2026-10-25 取得）と照合した。本文の欠落はない。次の 2 点を正規化している。(1) 結果の表がクリップでは縦一列に平坦化されていたので、原ページの構造（2 列×4 行）に戻した。(2) コードブロック内のバッククォートと下線に Web Clipper がバックスラッシュを付け加えていた（`` \`python <filename.py>\` `` など）。原ページにはないので除去した。ツール定義内の `\n` は原文の JSON 文字列の一部なのでそのまま残した。原ページの画像 2 枚は装飾イラストなので収録していない。末尾の謝辞は既定どおり訳出しない。

*私たちの最新モデルである改良版 [Claude 3.5 Sonnet](https://www.anthropic.com/news/3-5-models-and-computer-use) は、ソフトウェアエンジニアリングの評価である SWE-bench Verified で 49% を達成し、それまでの最先端モデルの 45% を上回った。本稿では、私たちがこのモデルの周りに構築した「エージェント」を説明する。本稿は、開発者が Claude 3.5 Sonnet から可能な限り最高の性能を引き出す助けとなることを意図している。*

[SWE-bench](https://www.swebench.com/) は、現実世界のソフトウェアエンジニアリングのタスクを完了するモデルの能力を評価する AI 評価ベンチマークである。具体的には、人気のあるオープンソースの Python リポジトリの GitHub イシューを、モデルがどのように解決できるかをテストする。ベンチマークの各タスクについて、AI モデルには、セットアップ済みの Python 環境と、そのイシューが解決される直前のリポジトリのチェックアウト（ローカルの作業コピー）が与えられる。モデルはそのうえで、コードを理解し、変更し、テストしてから、提案する解決策を提出する必要がある。

各解決策は、元の GitHub イシューをクローズしたプルリクエストに含まれる本物の単体テストに照らして採点される。これは、その AI モデルが、PR を書いた元の人間の作者と同じ機能を実現できたかどうかをテストするものである。

SWE-bench は AI モデルを単独で評価するのではなく、「エージェント」システム全体を評価する。この文脈で「エージェント」とは、AI モデルとその周りのソフトウェアの足場（scaffolding）の組み合わせを指す。この足場は、モデルに入るプロンプトの生成、行動をとるためのモデル出力の解析、そしてモデルの前の行動の結果を次のプロンプトに組み込むという相互作用ループの管理を担う。SWE-bench におけるエージェントの性能は、基盤となる AI モデルが同じであっても、この足場によって大きく変わりうる。

大規模言語モデルのコーディング能力を測るベンチマークは他にも多くあるが、SWE-bench はいくつかの理由で人気を集めてきた。

1. 競技やインタビュー形式の問題ではなく、実際のプロジェクトの本物のエンジニアリングタスクを使っている。
2. まだ飽和していない——改善の余地が十分にある。SWE-bench Verified で 50% の完了率を超えたモデルはまだない（ただし改良版 Claude 3.5 Sonnet は、執筆時点で 49% にある）。
3. モデル単体ではなく「エージェント」全体を測る。オープンソースの開発者やスタートアップは、同じモデルの周りの足場を最適化して性能を大きく引き上げることに大きな成功を収めてきた。

元の SWE-bench データセットには、GitHub イシューの外にある追加の文脈（たとえば、返すべき特定のエラーメッセージについて）なしには解けないタスクが一部含まれている点に注意されたい。[SWE-bench-Verified](https://openai.com/index/introducing-swe-bench-verified/) は SWE-bench の 500 問の部分集合で、解けることを確かめるために人間がレビューしたものであり、したがってコーディングエージェントの性能の最も明確な尺度を提供する。本稿で言及するのはこのベンチマークである。

## Achieving state-of-the-art（最先端の達成）

### Tool Using Agent（ツールを使うエージェント）

改良版 Claude 3.5 Sonnet 向けに最適化したエージェントの足場を作る際の私たちの設計哲学は、**言語モデル自身にできるだけ多くの制御を与え、足場は最小限に保つ**ことだった。エージェントは、1 つのプロンプトと、bash コマンドを実行するための Bash Tool と、ファイルやディレクトリを閲覧・編集するための Edit Tool を持つ。モデルが完了したと判断するか、200k のコンテキスト長を超えるまで、サンプリングを続ける。この足場は、問題をどう追求するかについてモデル自身の判断を使わせるものであり、特定のパターンやワークフローにハードコードされることはない。

プロンプトはモデルに推奨されるアプローチの概略を示すが、このタスクにとって長すぎたり詳しすぎたりはしない。モデルは、厳密で離散的な遷移を持つのではなく、ステップからステップへどう移るかを自由に選べる。トークンを気にしないのであれば、長い応答を生成するようモデルに明示的に促すことが役立ちうる。

次のコードは、私たちのエージェントの足場のプロンプトを示す。

```
<uploaded_files>
{location}
</uploaded_files>
I've uploaded a python code repository in the directory {location} (not in /tmp/inputs). Consider the following PR description:

<pr_description>
{pr_description}
</pr_description>

Can you help me implement the necessary changes to the repository so that the requirements specified in the <pr_description> are met?
I've already taken care of all changes to any of the test files described in the <pr_description>. This means you DON'T have to modify the testing logic or any of the tests in any way!

Your task is to make the minimal changes to non-tests files in the {location} directory to ensure the <pr_description> is satisfied.

Follow these steps to resolve the issue:
1. As a first step, it might be a good idea to explore the repo to familiarize yourself with its structure.
2. Create a script to reproduce the error and execute it with `python <filename.py>` using the BashTool, to confirm the error
3. Edit the sourcecode of the repo to resolve the issue
4. Rerun your reproduce script and confirm that the error is fixed!
5. Think about edgecases and make sure your fix handles them as well

Your thinking should be thorough and so it's fine if it's very long.
```

モデルの 1 つ目のツールは Bash コマンドを実行する。スキーマは単純で、環境で実行するコマンドだけを受け取る。しかし、ツールの説明文（description）のほうがより重みを持つ。そこには、入力のエスケープ、インターネットアクセスがないこと、コマンドをバックグラウンドで実行する方法など、モデル向けのより詳細な指示が含まれている。

次に、Bash Tool の仕様を示す。

```
{
   "name": "bash",
   "description": "Run commands in a bash shell\n
* When invoking this tool, the contents of the \"command\" parameter does NOT need to be XML-escaped.\n
* You don't have access to the internet via this tool.\n
* You do have access to a mirror of common linux and python packages via apt and pip.\n
* State is persistent across command calls and discussions with the user.\n
* To inspect a particular line range of a file, e.g. lines 10-25, try 'sed -n 10,25p /path/to/the/file'.\n
* Please avoid commands that may produce a very large amount of output.\n
* Please run long lived commands in the background, e.g. 'sleep 10 &' or start a server in the background.",
   "input_schema": {
       "type": "object",
       "properties": {
           "command": {
               "type": "string",
               "description": "The bash command to run."
           }
       },
       "required": ["command"]
   }
}
```

モデルの 2 つ目のツール（Edit Tool）ははるかに複雑で、ファイルの閲覧・作成・編集のためにモデルが必要とするすべてを含む。ここでも、ツールの説明文には、ツールの使い方についてのモデル向けの詳細な情報が含まれている。

私たちは、多種多様なエージェント的タスクにわたって、これらのツールの説明文と仕様に多くの労力を注いだ。モデルが仕様を誤解しうるあらゆる点や、ツールを使う際に起こりうる落とし穴を洗い出すためにそれらをテストし、そうした問題を先回りして防ぐよう説明文を編集した。**人間向けのツールインターフェースの設計に大きな注意が払われているのと同じように、モデル向けのツールインターフェースの設計にもはるかに多くの注意が向けられるべきだ**と私たちは考えている。

次のコードは、私たちの Edit Tool の説明文を示す。

```
{
   "name": "str_replace_editor",
   "description": "Custom editing tool for viewing, creating and editing files\n
* State is persistent across command calls and discussions with the user\n
* If `path` is a file, `view` displays the result of applying `cat -n`. If `path` is a directory, `view` lists non-hidden files and directories up to 2 levels deep\n
* The `create` command cannot be used if the specified `path` already exists as a file\n
* If a `command` generates a long output, it will be truncated and marked with `<response clipped>` \n
* The `undo_edit` command will revert the last edit made to the file at `path`\n
\n
Notes for using the `str_replace` command:\n
* The `old_str` parameter should match EXACTLY one or more consecutive lines from the original file. Be mindful of whitespaces!\n
* If the `old_str` parameter is not unique in the file, the replacement will not be performed. Make sure to include enough context in `old_str` to make it unique\n
* The `new_str` parameter should contain the edited lines that should replace the `old_str`",
...
```

性能を改善した方法の 1 つは、ツールを「誤りにくくする（error-proof）」ことだった。たとえば、エージェントがルートディレクトリの外へ移動したあと、モデルが相対ファイルパスを取り違えることがあった。これを防ぐため、私たちは単純に、ツールが常に絶対パスを要求するようにした。

既存ファイルへの編集を指定するいくつかの異なる戦略を実験し、最も高い信頼性が得られたのは文字列置換だった。これは、モデルが、指定したファイル内で `new_str` に置き換える `old_str` を指定するものである。置換は `old_str` がちょうど 1 か所だけ一致した場合にのみ行われる。一致がそれより多いか少ない場合、モデルには再試行のための適切なエラーメッセージが示される。

Edit Tool の仕様を以下に示す。

```
...
   "input_schema": {
       "type": "object",
       "properties": {
           "command": {
               "type": "string",
               "enum": ["view", "create", "str_replace", "insert", "undo_edit"],
               "description": "The commands to run. Allowed options are: `view`, `create`, `str_replace`, `insert`, `undo_edit`."
           },
           "file_text": {
               "description": "Required parameter of `create` command, with the content of the file to be created.",
               "type": "string"
           },
           "insert_line": {
               "description": "Required parameter of `insert` command. The `new_str` will be inserted AFTER the line `insert_line` of `path`.",
               "type": "integer"
           },
           "new_str": {
               "description": "Required parameter of `str_replace` command containing the new string. Required parameter of `insert` command containing the string to insert.",
               "type": "string"
           },
           "old_str": {
               "description": "Required parameter of `str_replace` command containing the string in `path` to replace.",
               "type": "string"
           },
           "path": {
               "description": "Absolute path to file or directory, e.g. `/repo/file.py` or `/repo`.",
               "type": "string"
           },
           "view_range": {
               "description": "Optional parameter of `view` command when `path` points to a file. If none is given, the full file is shown. If provided, the file will be shown in the indicated line number range, e.g. [11, 12] will show lines 11 and 12. Indexing at 1 to start. Setting `[start_line, -1]` shows all lines from `start_line` to the end of the file.",
               "items": {
                   "type": "integer"
               },
               "type": "array"
           }
       },
       "required": ["command", "path"]
   }
}
```

## Results（結果）

全般に、改良版 Claude 3.5 Sonnet は、私たちの以前のモデルや[それまでの最先端](https://solverai.com/)モデルよりも高い推論・コーディング・数学の能力を示している。また、改善されたエージェント的能力も示している——ツールと足場は、そうした改善された能力を最大限に活かす助けとなる。

**表**: 私たちのいくつかのモデルの SWE-bench Verified のスコア。すべてこのエージェントの足場を使用。

| モデル（Model） | SWE-bench Verified スコア |
|---|---|
| **Claude 3.5 Sonnet (new)** | **49%** |
| Previous SOTA | 45% |
| Claude 3.5 Sonnet (old) | 33% |
| Claude 3 Opus | 22% |

## Examples of agent behavior（エージェントの振る舞いの例）

ベンチマークの実行には、エージェントのコードの土台として [SWE-Agent](https://swe-agent.com/) フレームワークを用いた。以下のログでは、エージェントのテキスト出力・ツール呼び出し・ツールの応答を THOUGHT・ACTION・OBSERVATION として表示しているが、モデルを固定の順序に制約しているわけではない。

以下のコードブロックは、Sonnet 3.5 が SWE-bench の問題を解く典型的な事例を順にたどるものである。

この最初のブロックでは、モデルに与えられた初期プロンプトの一部を見ることができ、`{pr_description}` には SWE-bench のタスクの実際の値が入っている。重要なのは、このタスクにイシューを再現する手順が含まれていることで、これがモデルにとって調査の貴重な出発点になる。

```
<uploaded_files>
/repo
</uploaded_files>
I've uploaded a python code repository in the directory /repo (not in /tmp/inputs). Consider the following PR description:

<pr_description>
linear_model.RidgeClassifierCV's Parameter store_cv_values issue
#### Description
Parameter store_cv_values error on sklearn.linear_model.RidgeClassifierCV

#### Steps/Code to Reproduce
import numpy as np
from sklearn import linear_model as lm

#test database
n = 100
x = np.random.randn(n, 30)
y = np.random.normal(size = n)

rr = lm.RidgeClassifierCV(alphas = np.arange(0.1, 1000, 0.1), normalize = True, 
                                         store_cv_values = True).fit(x, y)

#### Expected Results
Expected to get the usual ridge regression model output, keeping the cross validation predictions as attribute.

#### Actual Results
TypeError: __init__() got an unexpected keyword argument 'store_cv_values'

lm.RidgeClassifierCV actually has no parameter store_cv_values, even though some attributes depends on it.
```

モデルは応答し、まず Edit Tool を使ってリポジトリの構造を閲覧する。THOUGHT と ACTION の下にモデルのテキスト出力とツール呼び出しの引数が、OBSERVATION の下にツールの出力の一部が見える。

```
THOUGHT:
I'll help you implement the changes to satisfy the PR description. Let's follow the steps you outlined.

1. First, let's explore the repo structure:
ACTION (str_replace_editor):
command:
view

path:
/repo

OBSERVATION:
Here's the files and directories up to 2 levels deep in /repo, excluding hidden items:
/repo
/repo/PULL_REQUEST_TEMPLATE.md
/repo/ISSUE_TEMPLATE.md
/repo/sklearn
/repo/sklearn/decomposition
/repo/sklearn/kernel_ridge.py
/repo/sklearn/model_selection
/repo/sklearn/covariance
/repo/sklearn/cluster
/repo/sklearn/base.py
/repo/sklearn/dummy.py
/repo/sklearn/naive_bayes.py
...
```

リポジトリの構造をよりよく理解したところで、モデルは Edit Tool を使って新しいスクリプトを作成する。これはイシューを再現し、修正をテストするために使うものである。

```
THOUGHT:
Based on the repository structure, we'll need to modify the Ridge Classifier in the `sklearn/linear_model` directory.

Let's create a script to reproduce the error:
ACTION (str_replace_editor):
command:
create

path:
/repo/reproduce_error.py

file_text:
import numpy as np
from sklearn import linear_model as lm

#test database
n = 100
x = np.random.randn(n, 30)
y = np.random.normal(size = n)

rr = lm.RidgeClassifierCV(alphas=np.arange(0.1, 1000, 0.1), normalize=True, 
                         store_cv_values=True).fit(x, y)

OBSERVATION:
File created successfully at: /repo/reproduce_error.py
```

続いてモデルは Bash Tool を使って自分が書いたスクリプトを実行し、タスクのイシューの再現に成功する。

```
THOUGHT:
Let's verify that we can reproduce the error:
ACTION (bash):
command:
python3 /repo/reproduce_error.py

OBSERVATION:
Traceback (most recent call last):
  File "/repo/reproduce_error.py", line 10, in <module>
    store_cv_values=True).fit(x, y)
TypeError: __init__() got an unexpected keyword argument 'store_cv_values'
```

ここから先、モデルは Edit Tool を使ってリポジトリのソースコードを変更し、スクリプトを再実行して、その変更でイシューが解決したかどうかを確かめる。

```
THOUGHT:
The error is reproducible. Looking at the code, I can see that the issue is that while the `RidgeClassifierCV` class inherits from `_BaseRidgeCV`, it fails to pass the `store_cv_values` parameter from its constructor through to the base class constructor.

Let's fix this:
ACTION (str_replace_editor):
command:
str_replace

path:
/repo/sklearn/linear_model/ridge.py

old_str:
    def __init__(self, alphas=(0.1, 1.0, 10.0), fit_intercept=True,
                 normalize=False, scoring=None, cv=None, class_weight=None):
        super(RidgeClassifierCV, self).__init__(
            alphas=alphas, fit_intercept=fit_intercept, normalize=normalize,
            scoring=scoring, cv=cv)
        self.class_weight = class_weight

new_str:
    def __init__(self, alphas=(0.1, 1.0, 10.0), fit_intercept=True,
                 normalize=False, scoring=None, cv=None, class_weight=None,
                 store_cv_values=False):
        super(RidgeClassifierCV, self).__init__(
            alphas=alphas, fit_intercept=fit_intercept, normalize=normalize,
            scoring=scoring, cv=cv, store_cv_values=store_cv_values)
        self.class_weight = class_weight
```

この特定の例では、モデルは 12 ステップ作業してから、提出の準備ができたと判断した。その後タスクのテストは正常に通り、モデルの解決策が問題に対処したことが確認された。モデルが解決策を提出するまでに 100 ターン以上かかったタスクもあれば、モデルがコンテキストを使い果たすまで試み続けたタスクもあった。

改良版 Claude 3.5 Sonnet の試行を旧モデルと比べてレビューすると、改良版 3.5 Sonnet はより頻繁に自己修正する。また、同じ誤りを何度も繰り返して行き詰まるのではなく、いくつかの異なる解決策を試す能力も示している。

## Challenges（課題）

SWE-bench Verified は強力な評価だが、単純な単一ターンの評価よりも実行が複雑でもある。以下は、私たちがこれを使う中で直面した課題の一部であり、他の AI 開発者も遭遇しうる課題である。

1. **所要時間と高いトークンコスト。** 上の例は 12 ステップで成功裏に完了したケースのものである。しかし、成功した実行の多くは、モデルが解決するまでに数百ターンと 10 万トークン超を要した。改良版 Claude 3.5 Sonnet は粘り強い——十分な時間があれば、しばしば問題を回避する道を見つけられるが、それは高くつきうる。
2. **採点。** 失敗したタスクを調べるなかで、モデルは正しく振る舞ったのに、環境のセットアップに問題があったり、インストール用のパッチが 2 回適用されてしまう問題があったりしたケースが見つかった。こうしたシステム上の問題を解決することは、AI エージェントの性能を正確に把握するうえで決定的に重要である。
3. **隠しテスト。** モデルは採点に使われるテストを見られないので、実際にはタスクが失敗しているのに、成功したと「思い込む」ことがよくある。こうした失敗の一部は、モデルが誤った抽象度で問題を解いた（より深いリファクタリングではなく、応急処置を施した）ことによる。他の失敗はやや公平さに欠けると感じられる——問題は解決しているが、元のタスクの単体テストと合致しないのである。
4. **マルチモーダル。** 改良版 Claude 3.5 Sonnet は優れた視覚・マルチモーダル能力を持つにもかかわらず、私たちは、ファイルシステムに保存された、あるいは URL で参照されたファイルをモデルが見る手段を実装しなかった。このことが、特定のタスク（とりわけ Matplotlib のもの）のデバッグを特に難しくし、モデルの幻覚も招きやすくした。ここには開発者が改善できる、手の届きやすい改善余地が間違いなくある——そして SWE-bench は、[マルチモーダルなタスクに焦点を当てた新しい評価](https://www.swebench.com/multimodal.html)を立ち上げた。近いうちに開発者たちが Claude でこの評価のより高いスコアを達成するのを楽しみにしている。

改良版 Claude 3.5 Sonnet は、単純なプロンプトと 2 つの汎用ツールで SWE-bench Verified の 49% を達成し、それまでの最先端（45%）を上回った。新しい Claude 3.5 Sonnet で開発する開発者たちが、私たちがここで最初に示したものを超えて SWE-bench のスコアを改善する、新しくより良い方法をすぐに見つけるだろうと私たちは確信している。
