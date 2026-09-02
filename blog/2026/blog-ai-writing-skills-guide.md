# 「AIっぽい文章」を減らす：4つの推敲Skill比較と日本語向けサンプルSkill

> 元記事：[Claude Codeの「これAIが書いたでしょ」問題を直すskill 4本](https://note.com/masa_wunder/n/na6e1d0c0d112)

## 1. なぜ推敲Skillを使うのか

生成AIの文章には、抽象的な前置き、根拠のない強調、同じリズムの繰り返しなどが残ることがあります。<br>

低品質なAI生成コンテンツを表す **slop** という言葉は、Merriam-Websterの2025年「Word of the Year」に選ばれました。<br>
同辞書はslopを、通常AIによって大量に作られる低品質なデジタルコンテンツと定義しています。<br>
対象は文章だけでなく、画像や動画なども含みます。<br>

> **💡 用語解説：AI slop**  
> AIで作られたコンテンツのうち、質が低い、量産的、または受け手に価値を与えにくいものを指す俗称です。<br>
>  AIが作ったコンテンツ全般を意味する言葉ではありません。<br>

LLM（大規模言語モデル）は、文脈に続く次のトークンの確率分布を計算して文章を生成します。<br>
ただし、常に「最も確率の高い語」を一つずつ選ぶわけではありません。<br>
生成設定、プロンプト、追加学習なども出力に影響します。<br>
そのため、「次の語を予測する仕組みだから、必ず平均的でAI臭い文章になる」とまでは言えません。<br>

**Claude Code Skill**を使うと、文章編集の観点や手順を再利用できます。<br>
Skillは品質を自動保証するフィルターではなく、Claudeに渡す指示書です。<br>
定型句の検出や冗長な部分の整理を、同じ観点で繰り返せます。<br>

本記事では、公開されている4つのSkillを比較し、選び方と日本語向けの応用例を紹介します。<br>

---

## 2. 4つのSkillをどう選ぶか

今回取り上げるのは、`humanize`、`no-ai-slop`、`humanizer`、`stop-slop`です。<br>
いずれも文章からAIらしいパターンを減らすことを目的としていますが、作者、ルール、出力形式は異なります。<br>

### 2.1 比較表

| Skill | 主な考え方 | 特徴 | 注意点 | 向いている使い方 |
| :--- | :--- | :--- | :--- | :--- |
| **`humanize`** | 句読点、語彙、文長などの数値・スタイル規則を使って書き直す | 7つのhard rulesと、対になるルールベース評価用`ai-check`を収録 | 商用AI検出器の判定改善は保証されない。<br>書き換え幅が大きくなる場合がある | 数値で確認できる編集チェックリストが欲しい |
| **`no-ai-slop`** | 著者の主張と声を残し、必要最小限の編集を行う | EditとDetectを分け、Editでは`What changed`を返す | 根拠の薄い文を削るため、原稿が短くなる場合がある | まず問題箇所だけ確認したい、修正理由も知りたい |
| **`humanizer`** | Wikipediaの「Signs of AI writing」を基に、文章全体を文脈付きで点検する | 現行版は35パターンを掲載し、事実の捏造を禁止している | 単語辞書との単純照合ではない。<br>ルール数が多く、過剰修正に注意 | 定型句だけでなく構成や論調も広く確認したい |
| **`stop-slop`** | 禁止表現、構造上の癖、文のリズムをチェックする | Markdownの指示と参照資料からなる比較的シンプルな構成 | リポジトリ全体を自動走査する専用ツールではない | 軽量な編集チェックリストとして使いたい |

客観的に「最も人間らしい」といえるSkillはありません。<br>文章の目的に合わせ、次の観点で選びます。<br>

1. **原文の声をどこまで残したいか**
2. **監査だけか、実際の書き換えまで任せるか**
3. **変更理由や差分を確認したいか**
4. **日本語向けにルールを調整できるか**

### 2.2 各Skillの見方

#### `humanize`：数値ルールで表面上の癖を減らす

`humanize`には、em dashは300語につき最大1本、原則としてセミコロンを使わない、特定の定型語を避ける、といったhard rulesがあります。<br>
対になる`ai-check`は、9カテゴリの信号を使って文章をルールベースで採点します。<br>

ただし、`ai-check`は独立した第三者検証機関ではなく、同じリポジトリに収録された自己評価用Skillです。<br>
作者の公開ベンチマークでも、書き換え後の文章がGPTZero、Grammarly、Pangramでは99〜100% AIと判定されたと報告されています。<br>
したがって、「商用AI検出器を通過するための確実な手段」として導入するのは適切ではありません。<br>

#### `no-ai-slop`：DetectとEditを使い分ける

`no-ai-slop`は、著者の語彙、リズム、曖昧さ、ユーモアなどを観察し、必要最小限の編集を行う方針です。<br>

- **Detect**：問題のあるパターンと該当箇所を示し、本文は書き換えない
- **Edit**：本文を編集し、最後に`What changed`を付ける

原稿を変更せずに確認したい場合は、Detectから始めます。<br>

#### `humanizer`：35パターンを文脈付きで確認する

`humanizer`はWikipediaの「Signs of AI writing」を基に、誇大表現、曖昧な出典、定型的な結論、過剰な同意などを点検します。<br>

これは「禁止語を見つけたら機械的に削除する辞書」ではありません。<br>
Skill自身も、em dashが一つあるだけではAI生成の証拠にならず、複数のパターンと文脈を確認すべきだと説明しています。<br>

#### `stop-slop`：シンプルな文章ルール集

`stop-slop`は、前置きの定型句、受動態、二項対立、断片的な文などを見直すためのSkillです。<br>
リポジトリ全体のドキュメントやコメントを自動的に一括走査する専用スクリプトではありません。<br>

複数ファイルを確認したい場合は、対象範囲と「検出だけ」「編集も行う」を別途Claudeに指示し、差分を確認してください。<br>

---

## 3. 比較記事の「99語→22語／100語」をどう読むか

4つのSkillを紹介した元のnote記事では、同じ99語の英文を各Skillに渡したところ、`no-ai-slop`は22語、`humanize`は100語になったと報告されています。<br>

この数字は編集方針の違いを示す一例ですが、性能比較のベンチマークではありません。<br>

- 入力は1種類だけ
- 実行回数は各モードにつき基本1回
- 生成モデルや設定によって結果が変わり得る
- 集計は記事筆者のローカルスクリプトによる
- 日本語での結果は公開されていない

つまり、このテストから分かるのは「同じ入力でもSkillの編集方針によって出力が大きく変わること」です。<br>
「常に`no-ai-slop`は短くなり、`humanize`は長さを維持する」と一般化はできません。<br>

---

## 4. 日本語向けサンプルSkill：`stop-slop-ja`

今回紹介した4つのSkillは、主に英語の文章パターンを扱っています。<br>
そこで筆者は、日本語文書向けのルール設計を試すサンプルとして、[`stop-slop-ja`](https://github.com/murcubcc110/agent-skills/blob/main/skills/stop-slop-ja/SKILL.md)を作成し、GitHubで公開しました。<br>

`stop-slop-ja`は、原版の[`stop-slop`](https://github.com/hardikpandya/stop-slop)を参考に、日本語向けに再設計しています。<br>
日本語の前置き、曖昧な強調、機械的な対比、単調なリズムなどを点検します。<br>
単語を一律に禁止するのではなく、文脈、著者の意図、技術的な正確さを優先します。<br>

### 2026年9月2日時点の構成

| 確認項目 | 状況 |
| :--- | :--- |
| Skill形式 | `SKILL.md`とYAML frontmatterを備え、`name`と`description`を定義 |
| 動作モード | Detect、Edit、Draftを用途に応じて選択 |
| 事実保持 | 数値、引用、URL、コード、必要な不確実性を維持するルールを収録 |
| ライセンス | 原版への帰属を示し、MIT Licenseを同梱 |
| 実行コード・外部通信 | なし。<br>`SKILL.md`と`LICENSE`のみ |
| 評価状況 | サンプルとして作成。<br>日本語文書での効果は未検証 |

標準的なAgent Skillとして配置できる構成ですが、実運用済みの製品や第三者評価済みのSkillではありません。<br>
文書品質を保証するものでもないため、最初はGit管理された短い原稿でDetectを使い、指摘内容を人が確認します。<br>

```text
/stop-slop-ja draft_review.md をDetectしてください。
ファイルは変更せず、該当箇所・パターン名・理由・最小限の修正案を返してください。
```

Detectは文章上のパターンを報告する機能です。<br>
AIが執筆したかどうかの判定や、AI検出器の回避には使用しません。<br>

---

## 5. AI検出器との付き合い方

AI検出器は、文章の予測しやすさ、学習済み分類器、複数モデル間の尤度差など、製品や研究によって異なる方法を使います。<br>同じ文章でも検出器ごとに判定が異なることがあります。<br>

OpenAIは、同社が試したAI文章検出器について、重大な判断に使えるほど信頼できず、人間の文章をAI生成と誤判定する問題があったと説明しています。<br>研究面でも、言い換えに対する弱さや、instruction tuningなど「AI生成一般」とは異なる特徴を検出器が拾っている可能性が指摘されています。<br>

Skillの目的は検出器の回避ではなく、文章品質の改善に置きます。<br>

- 主張と根拠が対応している
- 読者が必要な情報に早く到達できる
- 著者の意図や文体が残っている
- 数値、引用、技術的事実が変わっていない
- 変更理由を人が確認できる

AI検出器のスコアだけを、著者判定、評価、懲戒などの根拠にしないでください。<br>

---

## 6. まとめ

導入時は次の4点を押さえます。<br>

1. **Skillは品質保証ツールではなく、再利用可能な編集指示書**です。<br>
2. **4つのSkillは目的が異なり、単純な順位は付けられません**。<br>原文保持、Detect/Edit、変更理由、検証方法で選びます。<br>
3. **単発テストの結果を、Skillの性能差として一般化しません**。<br>
4. **日本語では`stop-slop-ja`を小さく試し、事実と差分を人が確認します**。<br>

まず、Git管理された短い日本語の下書きに`stop-slop-ja`のDetectを実行します。<br>
指摘が自分たちの文書に役立つことを確認してから、Editへ進んでください。<br>

## 参考文献・参照先

- [Merriam-Webster「2025 Word of the Year: Slop」](https://www.merriam-webster.com/wordplay/word-of-the-year)（2025年12月14日公開）
- [Anthropic公式：Extend Claude with skills](https://code.claude.com/docs/en/skills)（2026年9月1日確認）
- [`harshaneel/humanize`](https://github.com/harshaneel/humanize)（公開リポジトリ、2026年9月1日確認）
- [`petergyang/no-ai-slop`](https://github.com/petergyang/no-ai-slop)（公開リポジトリ、2026年9月1日確認）
- [`blader/humanizer`](https://github.com/blader/humanizer)（公開リポジトリ、2026年9月1日確認）
- [`hardikpandya/stop-slop`](https://github.com/hardikpandya/stop-slop)（公開リポジトリ、2026年9月1日確認）
- [`murcubcc110/agent-skills：stop-slop-ja`](https://github.com/murcubcc110/agent-skills/blob/main/skills/stop-slop-ja/SKILL.md)（筆者作成の日本語向けサンプルSkill v4.0.0、2026年9月2日確認）
- [Agent Skills公式仕様](https://github.com/agentskills/agentskills/blob/main/docs/specification.mdx)（`SKILL.md`とfrontmatterの要件）
- [`hardikpandya/stop-slop`のMITライセンス](https://github.com/hardikpandya/stop-slop/blob/main/LICENSE)（原版のライセンスと著作権表示）
- [OpenAI：How can educators respond to students presenting AI-generated content as their own?](https://help.openai.com/en/articles/8313351-how-can-educators-respond-to-students-presenting-ai-generated-content-as-their-own)（2026年9月1日確認）
- [Hans et al., “Spotting LLMs With Binoculars”](https://arxiv.org/abs/2401.12070)（ICML 2024）
- [Xu et al., “Base Models Look Human To AI Detectors”](https://arxiv.org/abs/2605.19516)（2026年5月19日公開）
- [元の比較記事：Claude Codeの「これAIが書いたでしょ」問題を直すskill 4本](https://note.com/masa_wunder/n/na6e1d0c0d112)（2026年8月31日公開。<br>単発テストの出典）
