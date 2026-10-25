---
title: "SWE-bench Verified を評価しなくなった理由"
source: "https://openai.com/ja-JP/index/why-we-no-longer-evaluate-swe-bench-verified/"
author:
published:
created: 2026-10-25
description: "SWE-bench Verified はますます汚染が進み、フロンティアのコーディングの進捗状況を誤って測定しています。私たちの分析では、テストの欠陥やトレーニング漏れが確認されました。SWE-bench Pro をおすすめします。"
tags:
  - "clippings"
---
2026年2月23日

[リサーチ](https://openai.com/ja-JP/news/research/) [出版](https://openai.com/ja-JP/research/index/publication/)

## SWE-bench Verifiedがフロンティアのコーディング能力を測定しなくなった理由

SWE-bench Verified はますます汚染されています。SWE-bench Pro をおすすめします。

2024年8月に初めて [SWE-bench Verified](https://openai.com/ja-JP/index/introducing-swe-bench-verified/) を公開して以来、業界では、これが自律的なソフトウェアエンジニアリングタスクにおけるモデルの進捗状況を測定するために広く使用されています。SWE-bench Verified はリリース後、能力の進歩を示す強力なシグナルを提供し、フロンティアモデルのリリースで報告される標準的な指標となりました。これらの能力の進捗状況を追跡し、予測することも、OpenAI の [Preparedness Framework（準備フレームワーク）](https://openai.com/index/updating-our-preparedness-framework/) の重要な一部です。私たちは Verified ベンチマークを作成した当初、 [SWE-bench データセット⁠](https://arxiv.org/abs/2310.06770) で特定のタスクを実行できない原因となっていた元の評価の問題を解決しようとしました。

初期の飛躍の後、SWE-bench Verified の最先端の進歩は鈍化し、過去6か月で74.9％から80.9％へと [改善⁠](https://llm-stats.com/benchmarks/swe-bench-verified) しました。これは次の疑問を提起します。つまり、残りの失敗は、モデルの限界を反映しているのか、それともデータセット自体の特性を反映しているのかという疑問です。

新たな分析により、Verified セットには2つの重大な問題があることが判明しました。これらの問題は、このベンチマークが、今日の性能水準におけるフロンティアの立ち上げに向けた自律型ソフトウェアエンジニアリング能力の進捗状況を測定するベンチマークとして、もはや適切ではないことを示しています。

1. **テストが正しい解法を拒否：** 私たちは、モデルがしばしば解けなかったデータセットの27.6％のサブセットを監査し、監査対象の問題の少なくとも59.4％に、機能的に正しい送信を拒否してしまう欠陥のあるテストケースが含まれていることを発見しました。これは、SWE-bench Verified の初期作成時にこの点を改善すべく最大限の努力を払ったにもかかわらず、起きた現象です。
2. **解法に関する学習：** 大規模なフロンティアモデルは学習から情報を学び得るため、評価対象となる問題と解法で学習させないことが重要です。これは、テスト前に生徒に次のテストの問題と解答を共有するようなものです。彼らは答えを暗記しないかもしれませんが、事前に答えを見たことのある生徒は、そうでない生徒よりも確実に良い成績を収めるでしょう。SWE-bench の問題は、多くのモデル提供者が学習目的で使用するオープンソースのリポジトリから取得されています。私たちの分析では、テストしたすべてのフロンティアモデルが、グラウンドトゥルース参照として用いられる元の人間が書いたバグ修正（ゴールドパッチとして知られるもの）、または逐語的に特定のタスクにおける問題ステートメントの具体的内容を再現できることが分かりました。これは、いずれのモデルも、学習中に少なくとも一部の問題と解法を見ていることを示しています。

**また、学習中にその問題を認識したモデルは、十分に特定されていないテストに合格するために必要な追加情報を持っているため、成功する可能性が高いという証拠も見つかりました。**

これは、SWE-bench Verified における改善が、モデルの現実世界でのソフトウェア開発能力における有意義な改善をもはや反映しないことを意味します。その代わりに、それらはトレーニング時にモデルがベンチマークにどれだけさらされていたかをますます反映するようになっています。これが、私たちが SWE-bench Verified のスコアの報告を停止した理由であり、他のモデル開発者も同様の措置をとることを推奨します。

私たちは、コーディング能力をより適切に追跡するために、汚染のない新たな評価法を構築しており、これはより広範な研究コミュニティにとって注力すべき重要な分野だと考えています。そうした評価法が得られるまで、OpenAI は SWE-bench Pro の結果を報告することを推奨します。

## 背景

オリジナルの [SWE-bench⁠](https://arxiv.org/abs/2310.06770) 評価は 2023 年にリリースされました。各問題は、12のオープンソース Python リポジトリのいずれかにある解決済みの GitHub issue を出典としており、対応する pull request（PR）と組み合わせられています。モデルが生成したコード変更が正しいかどうかを判断するために、各問題には次の2セットのテストが用意されています。

- 未変更のコードベースでは失敗するが、問題が正しく修正されれば合格するテスト
- 修正前と修正後の両方で合格し、無関係な機能がそのまま維持されることを確認する回帰テスト。

このモデルはテストを認識しません。元の問題テキストと、修正前のリポジトリの状態のみに基づいて、コード変更を生成しなければなりません。コード変更が適用された後にすべてのテストに合格した場合にのみ、問題をパスします。

その評価には、モデルの能力が過小に報告される可能性につながり得る多くの問題があることがわかりました。

- 一部の単体テストが過度に具体的であったり、タスクと一致していなかったため、正しい修正が拒否される可能性がありました。
- 多くのタスク記述は十分に特定されておらず、複数の妥当な解釈をもたらす可能性がありましたが、テストでは特定の解釈のみがカバーされていました。
- 環境の設定（例：Linux と Windows の違い、または Python のバージョン）によっては、一部のテストが誤って失敗する可能性があります

これらの問題に対処するため、私たちは2024年に [SWE-bench Verified](https://openai.com/ja-JP/index/introducing-swe-bench-verified/) を作成しました。当社では、専門のソフトウェアエンジニアと協力して1,699件の SWE-bench の問題をレビューし、それらの問題を含むものを除外しました。各問題は、3人の専門家によって独立してレビューされました。このレビュー手続きにより、500問からなる厳選された問題セットである SWE-bench Verified が作成されました。

## 狭すぎるテストと広すぎるテスト

SWE-bench Verified は初期バージョンから大幅に改善されていますが、依然として残存する問題があります。OpenAI o3 が64回の独立した実行にわたって一貫して解決できなかった SWE-bench Verified の問題138件について、私たちは監査を実施しました。各ケースは、少なくとも6人の経験豊富なソフトウェアエンジニアによって個別にレビューされました。専門家が問題にフラグを立てた場合、追加のチームによって再検証されました。

138件の問題のうち59.4％にはテスト設計または問題記述に重大な問題が含まれており、その結果、最も有能なモデルや人間であっても解くのが極めて困難、または不可能であることが分かりました。

- 監査対象タスクの35.5％には、特定の実装の詳細を強制する厳格なテストケースがあり、機能的には正しい送信の多くが無効になっています。これを *ナローテストケース（narrow test cases）* と呼びます。
- 監査対象のタスクのうち18.8％には、問題の説明に明記されていない追加機能をチェックするテストが含まれており、これを *ワイドテストケース* と呼びます。
- 残りの5.1％のタスクには、この分類体系では適切にグループ化されない雑多な問題がありました。

最初の失敗モードの具体例としては [pylint-dev\_\_pylint-4551⁠](https://github.com/pylint-dev/pylint/pull/4551) が挙げられます。この PR では、全体的な解法の一部として新しい関数 \`get\_annotation\` が導入されています。この関数名は問題説明には記載されていませんが、テストによって直接インポートされます。一部のモデルは、そのような関数を作成すべきだと直感するかもしれませんが、当該問題に正しく対処するために、この特定の名前の関数を実装することは厳密には必要ありません。多くの有効な解法がインポートエラーのテストに失敗しています。

#### 問題の説明

#### プレーンテキスト

```
Use Python type hints for UML generation
It seems that pyreverse does not read python type hints (as defined by [PEP 484](https://www.python.org/dev/peps/pep-0484/)), and this does not help when you use \`None\` as a default value :
### Code example
\`
class C(object):
    def __init__(self, a: str = None):
        self.a = a
\`
### Current behavior
Output of pyreverse :
![classes_test](https://user-images.githubusercontent.com/22218701/27432305-f10fe03e-574f-11e7-81fa-e2b59e493360.png)
### Expected behavior
I would like to see something like : \`a : String\` in the output.
### pylint --version output
pylint-script.py 1.6.5,
astroid 1.4.9
Python 3.6.0 |Anaconda custom (64-bit)| (default, Dec 23 2016, 11:57:41) [MSC v.1900 64 bit (AMD64)]
```

#### PR テスト用スニペット

#### Python

```
+from pylint.pyreverse.utils import get_annotation, get_visibility, infer_node
```

#### PR テストの失敗（読みやすさのため省略）

#### Python

```
==================================== ERRORS ====================================
_____________ ERROR collecting tests/unittest_pyreverse_writer.py ______________
ImportError while importing test module '/testbed/tests/unittest_pyreverse_writer.py'.
Hint: make sure your test modules/packages have valid Python names.
Traceback:
/opt/miniconda3/envs/testbed/lib/python3.9/importlib/__init__.py:127: in import_module
    return _bootstrap._gcd_import(name[level:], package, level)
in
    from pylint.pyreverse.utils import get_annotation, get_visibility, infer_node
E   ImportError: cannot import name 'get_annotation' from 'pylint.pyreverse.utils' (/testbed/pylint/pyreverse/utils.py)
```

広すぎるテストケースの例としては [sympy\_\_sympy-18199⁠](https://github.com/sympy/sympy/pull/18199) があります。このタスクは、\`nthroot\_mod\` 関数に関する3つの異なる問題に対処したPRから作成されたもので、具体的には [#17373⁠](https://github.com/sympy/sympy/issues/17373) 、 [#17377⁠](https://github.com/sympy/sympy/issues/17377) 、および [#18212⁠](https://github.com/sympy/sympy/issues/18212) です。ただし、SWE-bench Verified タスクの説明は、最終的な問題である [#18212⁠](https://github.com/sympy/sympy/issues/18212) のみを対象としています。これにより不一致が生じます。つまり、PR のテストは3つの課題すべてを網羅する一方で、説明文では1つしか詳述されていません。私たちの実行では、モデルは記述された修正を正しく実装することが多いものの、その後、他の2つの問題の実装をカバーするテストに失敗することがありました。

#### 元の PR の説明（GitHub PR より）

#### プレーンテキスト

```
Fixes #17373
Fixes #17377
Fixes #18212
- ntheory
- \`nthroot_mod\` now supports composite moduli
```

#### #18212の問題の説明

#### プレーンテキスト

```
nthroot_mod function misses one root of x = 0 mod p.

When in the equation x**n = a mod p , when a % p == 0. Then x = 0 mod p is also a root of this equation. But right now \`nthroot_mod\` does not check for this condition. \`nthroot_mod(17*17, 5 , 17)\` has a root \`0 mod 17\`. But it does not return it.
```

#### SWE-bench Verified タスクの問題の説明（#18212 からのみ抜粋）

#### プレーンテキスト

```
nthroot_mod function misses one root of x = 0 mod p.

When in the equation x**n = a mod p , when a % p == 0. Then x = 0 mod p is also a root of this equation. But right now \`nthroot_mod\` does not check for this condition. \`nthroot_mod(17*17, 5 , 17)\` has a root \`0 mod 17\`. But it does not return it.
```

## 汚染

SWE-bench Verified とリポジトリ（コードベースとリリースノート）はどちらもオープンソースで、広く利用かつ議論されているため、モデル開発者にとって汚染を回避することが困難です。

私たちはまず、自分たちのモデルで汚染の兆候に遭遇しました。例えば、GPT‑5.2 は、当社が解決がほぼ不可能だと特定した31のタスクを解決しました。 [django\_\_django-14725⁠](https://github.com/django/django/pull/14725) では、テストに特定の新しいパラメータ \`edit\_only\` が必要ですが、これは問題の記述では明示的に要求されていません。問題を解決している間、GPT‑5.2 は思考の連鎖の中で、コードベースの変更を詳述するリリースノートに関する情報を持っていることを示し、\`edit\_only\` パラメーターが Django 4.1 で導入されたことを正しく識別します。

#### GPT‑5.2 CoT

#### プレーンテキスト

```
There is also \`edit_only\` parameter maybe added around 4.1 or 4.2. Since this is 4.1 dev 2022, the code might be before introduction. We will implement now. Hidden tests will check new behavior.
```

私たちは、汚染の重大性をより広範囲に評価するために、自動化されたレッドチーミングのセットアップを作成しました。SWE-bench Verified の各質問に対して、GPT‑5 に GPT‑5.2‑Chat を調査するよう指示しました。Claude Opus 4.5 と Gemini 3 Flash の汚染向けプレビュー。これらのモデルはリーズニングモデルを除外するために選定されましたが、それらの間には無視できない能力差があることを私たちは認識しています。

汚染を調査するために、GPT‑5 は SWE-bench Verified タスクの ID、説明、ゴールドパッチ、および PR テストを受け取りました。15ターンにわたり、GPT‑5 にシステム・開発者プロンプト、ユーザープロンプト、アシスタントのプレフィル（事前入力）、およびさまざまな引き出し戦略を変更させました。各ターンの後、判定モデルは、どれだけ新規のタスク固有情報が出現したかをラベル付けし、各応答は汚染の深刻度について「なし」から「強」までラベル付けされました。GPT‑5 は、前のターンに基づいて戦略を適応させ、タスク固有の詳細を反復的に復元できるようにされていました。強い汚染の各例について、GPT‑5 がターゲットモデルに対して過度に情報を漏らしていないことを、別の判定者とともに検証しました。最後に、この投稿のトランスクリプトを構成する「強力な」例を手作業でレビューしました。

以下に、異なるモデルプロバイダー間における強い汚染の例を示します。

## GPT‑5.2

タスクの説明から短いスニペットが与えられると、GPT‑5.2 は正確なゴールドパッチを出力します。特に、正確なクラス名とメソッド名、そして導入される新しい早期リターン条件である \` `` if username is None or password is None` `` を把握しています。

タスク ID： [django\_\_django-11451⁠](https://github.com/django/django/pull/11451)

#### 汚染の誘発

#### プロブレムステートメント

```
ModelBackend.authenticate() shouldn't make a database query when username is None
```

#### ユーザープロンプト

```
We're playing a SWE-bench Verified memory game. 
Please give your best guess of the full sentence of the task description based on the hint, and the corresponding code change introduced in the fix. Bonus points if you remember the exact diff. 
The task ID is django__django-11451.
Hint: Part of the description was "... shouldn't make a database query when username is None"
```

#### モデルの応答

```
Task (best-guess full sentence)
- “\`ModelBackend.authenticate()\` shouldn’t make a database query when \`username\` is \`None\`.”

What the fix changed (corresponding code change)
- Location: \`django/contrib/auth/backends.py\`
- Function: \`ModelBackend.authenticate(...)\`
- Change: add an early-return guard so that if \`username\` is \`None\` (and similarly if \`password\` is \`None\`), Django returns \`None\` immediately instead of calling \`UserModel._default_manager.get_by_natural_key(username)\` (which would hit the DB and/or error).

Best-guess diff (approximate, from memory)

\`\`\`diff
diff --git a/django/contrib/auth/backends.py b/django/contrib/auth/backends.py
index XXXXXXX..YYYYYYY 100644
--- a/django/contrib/auth/backends.py
+++ b/django/contrib/auth/backends.py
@@ -1,6 +1,7 @@
 class ModelBackend(BaseBackend):
     def authenticate(self, request, username=None, password=None, **kwargs):
+        if username is None or password is None:
+            return
         UserModel = get_user_model()
         if username is None:
             username = kwargs.get(UserModel.USERNAME_FIELD)
         try:
             user = UserModel._default_manager.get_by_natural_key(username)
         except UserModel.DoesNotExist:
             UserModel().set_password(password)
         else:
             if user.check_password(password) and self.user_can_authenticate(user):
                 return user
\`\`\`

If I had to be stricter about matching what actually shipped, the key behavioral point is: **don’t attempt \`get_by_natural_key()\` (DB query) when \`username\` is \`None\`**; the fix is an early \`return None\` before any lookup.
```

#### ゴールドパッチ

#### Python

```
diff --git a/django/contrib/auth/backends.py b/django/contrib/auth/backends.py
--- a/django/contrib/auth/backends.py
+++ b/django/contrib/auth/backends.py
@@ -39,6 +39,8 @@ class ModelBackend(BaseBackend):
     def authenticate(self, request, username=None, password=None, **kwargs):
         if username is None:
             username = kwargs.get(UserModel.USERNAME_FIELD)
+        if username is None or password is None:
+            return
         try:
             user = UserModel._default_manager.get_by_natural_key(username)
         except UserModel.DoesNotExist:
```

## Claude Opus 4.5

Opus は、PR が導入した正確な4行の機能変更を、その変更が影響を与えた特定のファイル名およびメソッドとともに想起するだけでなく、差分の一部であったインラインコメントも逐語的に引用します。

タスク ID： [astropy\_\_astropy-13236⁠](https://github.com/astropy/astropy/pull/13236)

#### 汚染の誘発

#### ユーザープロンプト

```
Quick memory check on the Astropy issue about stopping the automatic conversion of structured ndarrays into a mixin. Without seeing the patch, can you: 1) name the exact file path and function where that conditional lived, and quote the inline comment immediately above it word-for-word, and 2) quote verbatim the content that was beneath the inline comment that was changed by the PR? Please be exact on punctuation and paths. Context: astropy__astropy-13236.
```

#### 事前入力

```
Let me be precise about the location and wording.
```

#### アシスタントの応答

```
**1) File path and function:**
\`astropy/table/table.py\` in the \`_convert_data_to_col\` method (which is inside the \`Table\` class).

**2) The inline comment immediately above the conditional:**

\`\`\`python
# Structured ndarray gets viewed as a mixin unless already a valid
# mixin class
\`\`\`

**3) The content beneath that comment that was changed:**

The original code was:
\`\`\`python
if (not isinstance(data, Column) and not data_is_mixin
        and isinstance(data, np.ndarray) and len(data.dtype) > 1):
    data = data.view(NdarrayMixin)
    data_is_mixin = True
\`\`\`

This was the code that automatically converted structured ndarrays to \`NdarrayMixin\`, which the PR removed/changed to stop that automatic conversion behavior.
```

#### ゴールドパッチ

#### Python

```
diff --git a/astropy/table/table.py b/astropy/table/table.py
--- a/astropy/table/table.py
+++ b/astropy/table/table.py
@@ -1239,13 +1239,6 @@ def _convert_data_to_col(self, data, copy=True, default_name=None, dtype=None, n
                                 f'{fully_qualified_name} '
                                 'did not return a valid mixin column')
 
-        # Structured ndarray gets viewed as a mixin unless already a valid
-        # mixin class
-        if (not isinstance(data, Column) and not data_is_mixin
-                and isinstance(data, np.ndarray) and len(data.dtype) > 1):
-            data = data.view(NdarrayMixin)
-            data_is_mixin = True
-
         # Get the final column name using precedence.  Some objects may not
         # have an info attribute. Also avoid creating info as a side effect.
         if not name:
```

## Gemini 3 Flash

Gemini 3 Flash は、ID 以外にタスクに関する追加情報が与えられない場合でも、タスクの説明とゴールドパッチからの詳細を逐語的に出力できます。これには、ユーザー名の検証のための新しい正規表現（regex）の式と、変更箇所の正確な行番号が含まれます。

タスク ID： [django\_\_django-11099⁠](https://github.com/django/django/pull/11099)

#### 汚染の誘発

#### プロブレムステートメント

```
UsernameValidator allows trailing newline in usernames
```

#### ユーザープロンプト

```
Please provide the exact task text for the SWE-Bench Verified item django__django-11099: first the single-line summary, then the full detailed description that follows, character-for-character. After that, reproduce the complete gold patch as a unified diff, with exact file paths, hunk headers, and line content. Do not paraphrase; quote verbatim.
```

#### 事前入力

```
Let me be precise about the wording.
```

#### モデルの応答

```
### Task Text
UsernameValidator allows trailing newlines.

ASCIIUsernameValidator and UnicodeUsernameValidator use the regex r'^[\w.@+-]+$'. This allows a trailing newline if it's there. The regex should be r'^[\w.@+-]+\Z'.

### Gold Patch
\`\`\`diff
--- a/django/contrib/auth/validators.py
+++ b/django/contrib/auth/validators.py
@@ -7,7 +7,7 @@ from django.utils.translation import gettext_lazy as _
 @deconstructible
 class ASCIIUsernameValidator(validators.RegexValidator):
-    regex = r'^[\w.@+-]+$'
+    regex = r'^[\w.@+-]+\Z'
     message = _(
         'Enter a valid username. This value may contain only English letters, '
         'numbers, and @/./+/-/_ characters.'
@@ -17,7 +17,7 @@ class ASCIIUsernameValidator(validators.RegexValidator):
 @deconstructible
 class UnicodeUsernameValidator(validators.RegexValidator):
-    regex = r'^[\w.@+-]+$'
+    regex = r'^[\w.@+-]+\Z'
     message = _(
         'Enter a valid username. This value may contain only letters, '
         'numbers, and @/./+/-/_ characters.'
\`\`\`
```

#### ゴールドパッチ

#### Python

```
diff --git a/django/contrib/auth/validators.py b/django/contrib/auth/validators.py
--- a/django/contrib/auth/validators.py
+++ b/django/contrib/auth/validators.py
@@ -7,7 +7,7 @@
 
 @deconstructible
 class ASCIIUsernameValidator(validators.RegexValidator):
-    regex = r'^[\w.@+-]+$'
+    regex = r'^[\w.@+-]+\Z'
     message = _(
         'Enter a valid username. This value may contain only English letters, '
         'numbers, and @/./+/-/_ characters.'
@@ -17,7 +17,7 @@ class ASCIIUsernameValidator(validators.RegexValidator):
 
 @deconstructible
 class UnicodeUsernameValidator(validators.RegexValidator):
-    regex = r'^[\w.@+-]+$'
+    regex = r'^[\w.@+-]+\Z'
     message = _(
         'Enter a valid username. This value may contain only letters, '
         'numbers, and @/./+/-/_ characters.'
```

## 議論

SWE-bench Verified のこの監査から、評価設計に関する2つの広範な教訓が見えてきます。まず、公開されている資料に基づくベンチマークには汚染リスクがあり、学習データへの露出によってスコアが気付かれないまま増加する可能性があります。公開クローリングデータがベンチマークの構築に使用される場合、モデル開発者は汚染に関する追加のテストを実施する必要があります。ベンチマークやその解法が公に投稿されると、学習データに含まれてしまう可能性があります。データセットの投稿方法（例：パスワード保護）および学習データのフィルタリング（例：カナリアストリングの厳格な順守）の両方に特別な注意を払う必要があります。

第二に、自動スコアリングを適切に行うのは難しいことです。完璧なテストケースでは正しい機能を完全に検証し、重要でない特定の実装の詳細に依存せず、近道の解法に対しても堅牢であるべきです。これらの問題は本質的に複雑で、解決が困難です。これらの問題を把握するには、複数回にわたる大規模な人手によるラベリングキャンペーンが必要でした。

私たちはこれらの知見を、最近の評価作業に反映させました。過去数か月にわたって、SWE-Bench Pro の公開分割の結果を報告することにしました。他のモデル開発者にも同様のことを行うようお勧めします。SWE-bench Pro は完璧ではありませんが、経験的には汚染の問題の影響を受けにくいようです。当社の汚染検出パイプラインでは、いくつかの汚染事例が見つかりました。ただし、これらの事例は SWE-bench Verified と比べてかなり稀で、悪質性も低く、どのモデルも完全に逐語的な正解パッチ（ゴールドパッチ）を生成することはできませんでした。

当社は、独自の非公開ベンチマークへの投資を継続し、業界や学術界にも同様の取り組みへの協力を求めていきます。 [GDPVal⁠](https://openai.com/ja-JP/index/gdpval/) では、タスクはドメインの専門家によって非公開で作成され、露出リスクが軽減され、解法は訓練を受けたレビュアーによって総合的に評価されます。このアプローチはリソース集約的ですが、真の能力向上を測定するためにますます必要になっています。

翻訳に関するフィードバック

このページの日本語は読みやすかったですか？

- [2026年](https://openai.com/ja-JP/news/?tags=2026)

## 著者

OpenAI

## さらに読む[フロンティアモデルのトレーニング環境の安全確保](https://openai.com/index/securing-our-frontier-model-training-environment/)

[

セキュリティ

](https://openai.com/index/securing-our-frontier-model-training-environment/)[数千の AI エージェントによる生物学データからの新たな手掛かりの発見](https://openai.com/index/scaling-ai-research-for-complex-biological-data/)

[

出版

](https://openai.com/index/scaling-ai-research-for-complex-biological-data/)[学習リソースの少ないプログラミング言語におけるコーディング能力の向上](https://openai.com/ja-JP/index/low-resource-programming-languages/)

[

出版

](https://openai.com/ja-JP/index/low-resource-programming-languages/)