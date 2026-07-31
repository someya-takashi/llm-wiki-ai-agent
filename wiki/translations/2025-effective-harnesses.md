---
type: translation
source_path: "raw/articles/Effective harnesses for long-running agents.md"
source_page: "[[summaries/2025-effective-harnesses]]"
original_language: en
translated_to: ja
translated_at: 2026-07-31
---

# 長時間走行エージェントのための効果的なハーネス

> 原題: Effective harnesses for long-running agents
> 著者: Justin Young（Anthropic）
> 出典: Anthropic Engineering Blog（https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents）

> **訳注**: 本訳は Obsidian Web Clipper によるクリップを底本とし、原ページと照合して以下を補った。(1) クリップで本文が欠落していた**脚注 1 件**を原ページから復元し `[^fn1]` として収録。(2) 本文中の GIF 画像（Puppeteer によるテストのスクリーンショット）を Next.js の変換 URL から素の CDN URL でローカル保存した。(3) 表タイトル「Agent failure modes and solutions」の平文化をキャプションとして整形した。除外: サイトロゴ（chrome）、Acknowledgements（謝辞。運用ルールに従い除外）。プロンプト指示文・JSON・セッション例のコード部分は原文のまま収録する。

AI エージェントの能力が高まるにつれ、開発者は、数時間、時には数日にわたる作業を要する複雑なタスクをエージェントに任せることが増えている。しかし、複数のコンテキストウィンドウをまたいでエージェントに一貫した前進をさせることは、依然として未解決の問題である。

長時間走行エージェントの核心的な課題は、**離散的なセッションで作業しなければならず、新しいセッションはそれぞれ、以前に何があったかの記憶なしで始まる**ことである。シフト勤務のエンジニアで運営されるソフトウェアプロジェクトを想像してほしい。ただし、新しく来るエンジニアは前のシフトで何が起きたかの記憶を一切持っていない。コンテキストウィンドウは有限であり、複雑なプロジェクトの大半は単一のウィンドウ内では完了できないため、エージェントにはコーディングセッション間のギャップを橋渡しする方法が必要になる。

我々は、[Claude Agent SDK](https://platform.claude.com/docs/en/agent-sdk/overview) が多数のコンテキストウィンドウをまたいで効果的に働けるようにする、二部構成の解決策を開発した: 最初の実行で環境をセットアップする **initializer agent** と、毎セッションで漸進的な前進を果たしつつ、次のセッションのために明確な artifact を残すことを課された **coding agent** である。コード例は付属の [quickstart](https://github.com/anthropics/claude-quickstarts/tree/main/autonomous-coding) にある。

## The long-running agent problem（長時間走行エージェントの問題）

Claude Agent SDK は、コーディングをはじめ、モデルがツールを使って文脈を集め、計画し、実行することを要するタスクに長けた、強力で汎用的なエージェントハーネスである。compaction（コンテキストウィンドウを使い尽くさずにタスクを続けられるようにする文脈圧縮）のようなコンテキスト管理機能も備える。理論上は、このセットアップがあれば、エージェントは任意に長い時間、有用な作業を続けられるはずである。

しかし、compaction だけでは不十分である。そのままの状態では、Claude Agent SDK 上でループする Opus 4.5 のようなフロンティアのコーディングモデルでさえ、「[claude.ai](http://claude.ai) のクローンを作って」のような高レベルのプロンプトだけを与えられた場合、複数のコンテキストウィンドウをまたいで本番品質の Web アプリを作り上げることはできない。

Claude の失敗は 2 つのパターンで現れた。第一に、エージェントは一度に多くをやりすぎる傾向——本質的にはアプリを一発で完成させようとする傾向——があった。これはしばしば、実装の途中でモデルが文脈を使い果たし、次のセッションが「機能は半分実装済み・記録なし」という状態から始まる事態を招いた。そうなるとエージェントは何が起きたのかを推測し、基本的なアプリを再び動く状態に戻すことに相当な時間を費やさざるを得ない。これは compaction があっても起きる。compaction が次のエージェントへ完全に明確な指示を渡せるとは限らないからである。

第二の失敗モードは、プロジェクトの後半でよく起きた。いくつかの機能がすでに作られた後、後続のエージェントインスタンスが周囲を見回し、進捗があることを確認して、**仕事は完了したと宣言してしまう**のである。

これは問題を 2 つの部分に分解する。第一に、与えられたプロンプトが要求する*すべて*の機能の土台となる初期環境をセットアップする必要がある。これはエージェントが段階的に、機能ごとに働けるようにする。第二に、各エージェントに、目標へ向けた漸進的な前進を促すと同時に、セッションの最後に環境をクリーンな状態で残すよう促すべきである。「クリーンな状態」とは、main ブランチへのマージにふさわしい種類のコードを意味する: 大きなバグがなく、コードは整然としてよく文書化されており、総じて、開発者が無関係な散らかりをまず片付けることなく、新しい機能の作業を容易に始められる状態である。

社内で実験する際、我々はこの問題に二部構成の解決策で対処した:

1. **Initializer agent**: 一番最初のエージェントセッションは、初期環境のセットアップをモデルに求める特化プロンプトを使う: `init.sh` スクリプト、エージェントたちが行ったことのログを保持する claude-progress.txt ファイル、そして追加されたファイルを示す最初の git コミットである。
2. **Coding agent**: 以降のすべてのセッションは、モデルに漸進的な前進を求め、その後、構造化された更新を残させる。[^fn1]

ここでの鍵となる洞察は、新しいコンテキストウィンドウで開始するエージェントが**作業の状態を素早く理解する方法**を見つけることであり、これは git 履歴と並ぶ claude-progress.txt ファイルによって達成される。これらのプラクティスの着想は、有能なソフトウェアエンジニアが日々行っていることを知っていることから来た。

## Environment management（環境管理）

更新版の [Claude 4 プロンプティングガイド](https://docs.claude.com/en/docs/build-with-claude/prompt-engineering/claude-4-best-practices#multi-context-window-workflows)では、「一番最初のコンテキストウィンドウには別のプロンプトを使う」ハーネス構造を含む、複数コンテキストウィンドウのワークフローのベストプラクティスをいくつか共有した。この「別のプロンプト」は、将来の coding agent が効果的に働くために必要なすべての文脈を備えた環境を、initializer agent がセットアップすることを要求する。ここでは、そのような環境の主要コンポーネントのいくつかをより深く掘り下げる。

### Feature list（機能リスト）

エージェントがアプリを一発で作ろうとしたり、プロジェクトを早まって完了と見なしたりする問題に対処するため、initializer agent に、ユーザーの初期プロンプトを拡張した**機能要件の包括的なファイル**を書くよう促した。claude.ai クローンの例では、これは「ユーザーは新しいチャットを開き、クエリを入力し、Enter を押して、AI の応答を見ることができる」のような **200 超の機能**を意味した。これらの機能はすべて最初「failing」とマークされ、後続の coding agent が完全な機能性とは何かの明確な輪郭を持てるようにした。

```
{
    "category": "functional",
    "description": "New chat button creates a fresh conversation",
    "steps": [
      "Navigate to main interface",
      "Click the 'New Chat' button",
      "Verify a new conversation is created",
      "Check that chat area shows welcome state",
      "Verify conversation appears in sidebar"
    ],
    "passes": false
  }
```

coding agent には、このファイルを **passes フィールドの状態変更によってのみ**編集するよう促し、「テストを削除・編集することは、機能の欠落やバグにつながりうるため容認不可能である（It is unacceptable to remove or edit tests because this could lead to missing or buggy functionality.）」のような強い文言の指示を使う。いくらかの実験を経て、これには JSON を使うことに落ち着いた。**モデルは Markdown ファイルに比べ、JSON ファイルを不適切に変更・上書きしにくい**ためである。

### Incremental progress（漸進的な前進）

この初期環境の足場を与えられた上で、coding agent の次のイテレーションには**一度に 1 機能だけ**に取り組むよう求めた。この漸進的アプローチは、一度に多くをやりすぎるエージェントの傾向への対処として決定的に重要だと分かった。

漸進的に働くようになっても、コード変更の後にモデルが環境をクリーンな状態で残すことは依然として不可欠である。実験では、この挙動を引き出す最良の方法は、**説明的なコミットメッセージで進捗を git にコミットし、進捗ファイルに進捗の要約を書く**ようモデルに求めることだった。これによりモデルは、悪いコード変更を git で差し戻し、コードベースの動く状態を回復できるようになった。

これらのアプローチは効率も高めた。エージェントが「何が起きたのかを推測し、基本的なアプリを再び動かすことに時間を費やす」必要をなくしたからである。

### Testing（テスト）

観察されたもうひとつの大きな失敗モードは、適切なテストなしに機能を完了とマークする Claude の傾向だった。明示的なプロンプトがなければ、Claude はコード変更を行い、単体テストや開発サーバーへの `curl` コマンドでのテストまではするものの、その機能が **end-to-end では動かない**ことを認識し損ねる傾向があった。

Web アプリ構築のケースでは、ブラウザ自動化ツールを使い**人間のユーザーが行うようにすべてのテストを行う**よう明示的に促せば、Claude は機能の end-to-end 検証をおおむねうまくやれた。

<figure>

![](../../raw/assets/2025-effective-harnesses/puppeteer-testing.gif)

<figcaption>Puppeteer MCP サーバーを通じて Claude が claude.ai クローンをテストする際に撮影したスクリーンショット。</figcaption>
</figure>

この種のテストツールを Claude に与えることは性能を劇的に改善した。コードだけからは明らかでないバグを、エージェントが特定して修正できるようになったからである。

いくつかの問題は残る。Claude の視覚とブラウザ自動化ツールの限界により、あらゆる種類のバグを特定するのは難しい。例えば、Claude は Puppeteer MCP を通じてブラウザネイティブの alert モーダルを見ることができず、これらのモーダルに依存する機能は結果としてバグが残りがちだった。

## Getting up to speed（状況把握）

以上のすべてを整えた上で、すべての coding agent には、自分の現在地を把握するための一連のステップを実行するよう促す。かなり基本的だが、それでも有用なものもある:

1. *Run `pwd` to see the directory you're working in. You'll only be able to edit files in this directory.*
2. *Read the git logs and progress files to get up to speed on what was recently worked on.*
3. *Read the features list file and choose the highest-priority feature that's not yet done to work on.*

（訳注: 上記はプロンプト内の指示文のため原文のまま。大意——1. `pwd` で作業ディレクトリを確認する。編集できるのはこのディレクトリ内のみ。2. git ログと進捗ファイルを読んで直近の作業に追いつく。3. 機能リストを読み、未完了の最優先機能を選んで取り組む。）

このアプローチは、コードのテスト方法を毎回考え出さなくてよくなるため、セッションごとに Claude のトークンをいくらか節約する。開発サーバーを起動できる init.sh スクリプトを initializer agent に書かせ、新機能の実装前に基本的な end-to-end テストを一通り実行させることも有効である。

claude.ai クローンのケースでは、これはエージェントが常にローカルの開発サーバーを起動し、Puppeteer MCP を使って新しいチャットを開始し、メッセージを送り、応答を受け取ることを意味した。これにより、アプリが壊れた状態で残されていないかを Claude が素早く特定し、既存のバグを直ちに修正できることが保証された。もしエージェントが代わりに新機能の実装から始めていたら、問題を悪化させていた可能性が高い。

以上を踏まえると、典型的なセッションは次のようなアシスタントメッセージで始まる:

```
[Assistant] I'll start by getting my bearings and understanding the current state of the project.
[Tool Use] <bash - pwd>
[Tool Use] <read - claude-progress.txt>
[Tool Use] <read - feature_list.json>
[Assistant] Let me check the git log to see recent work.
[Tool Use] <bash - git log --oneline -20>
[Assistant] Now let me check if there's an init.sh script to restart the servers.
<Starts the development server>
[Assistant] Excellent! Now let me navigate to the application and verify that some fundamental features are still working.
<Tests basic functionality>
[Assistant] Based on my verification testing, I can see that the fundamental functionality is working well. The core chat features, theme switching, conversation loading, and error handling are all functioning correctly. Now let me review the tests.json file more comprehensively to understand what needs to be implemented next.
<Starts work on a new feature>
```

**表: エージェントの失敗モードと解決策**（原文キャプション: 長時間走行 AI エージェントにおける 4 つの一般的な失敗モードと解決策の要約。）

| 問題 | Initializer Agent の挙動 | Coding Agent の挙動 |
| --- | --- | --- |
| Claude がプロジェクト全体の勝利を早く宣言しすぎる。 | 機能リストファイルをセットアップする: 入力仕様に基づき、end-to-end の機能記述のリストを持つ構造化 JSON ファイルを用意する。 | セッションの最初に機能リストファイルを読む。取り組む機能を 1 つ選ぶ。 |
| Claude がバグや未記録の進捗のある状態で環境を残す。 | 初期の git リポジトリと進捗メモファイルを書く。 | セッションを進捗メモファイルと git コミットログの読み込みから始め、開発サーバーで基本テストを実行して未記録のバグを捕捉する。セッションの最後に git コミットと進捗更新を書く。 |
| Claude が機能を早まって完了とマークする。 | 機能リストファイルをセットアップする。 | すべての機能を自己検証する。注意深いテストの後にのみ機能を「passing」とマークする。 |
| Claude がアプリの実行方法を把握するのに時間を費やす。 | 開発サーバーを起動できる `init.sh` スクリプトを書く。 | セッションを `init.sh` の読み込みから始める。 |

## Future work（今後の課題）

本研究は、モデルが多数のコンテキストウィンドウをまたいで漸進的な前進を果たせるようにする、長時間走行エージェントハーネスにおけるひとつの可能な解決策のセットを実証した。しかし、未解決の問いが残っている。

最も顕著なのは、**単一の汎用 coding agent が文脈を通じて最良の性能を発揮するのか、それともマルチエージェントアーキテクチャによってより良い性能が達成できるのか**が依然として不明なことである。テストエージェント・品質保証エージェント・コードクリーンアップエージェントのような特化エージェントが、ソフトウェア開発ライフサイクル全体のサブタスクをさらにうまくこなせるだろうというのは、理にかなっているように思われる。

加えて、このデモはフルスタック Web アプリ開発に最適化されている。将来の方向のひとつは、これらの知見を他の分野へ一般化することである。これらの教訓の一部または全部は、例えば科学研究や金融モデリングで求められる種類の長時間のエージェントタスクにも適用できる可能性が高い。

---

[^fn1]: これらを別々のエージェントと呼ぶのは、この文脈では初期のユーザープロンプトが異なるからに過ぎない。システムプロンプト・ツールセット・エージェントハーネス全体はそれ以外同一だった。（訳注: クリップで欠落していた脚注を原ページから復元）
