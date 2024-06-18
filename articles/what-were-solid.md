---
title: 可換的なソフトウェア設計
emoji: 🤷‍♀️
type: tech
topics: [アーキテクチャ, クリーンアーキテクチャ, オニオンアーキテクチャ, SOLID原則]
published: false
---

# あれこれ読んで結局いつも通り実装している方々，いかがお過ごしでしょうか？

タイトルの通りです．

結局「クリーンアーキテクチャ」だ，「SOLID原則だ」，といいはします（ワタシも散々言った訳です）が，これらの可換的な設計は結局何がうれしくて，どのようなタイミングでほしいのかについて言語化できればと思います．

前段で述べた過去2作はワタシの個人開発での経験がメインだった（だからこそ，あれだけ読まれたことにかなりゾッとしている）のですが，この記事はワタシが実際にインターン生として週4で働いて得た経験も踏まえながら整理していくつもりです．過去2作よりかは現実見ながら整理していくつもりです．そのため，過去，ワタシは「オニオンアーキテクチャ」，「SOLID原則」というテーマで技術記事を書いて，それぞれワタシとしてはかなり満足のいく記事を書けたと思っているのですが，今回の読者対象はより幅広い層を対象に広げられたらと思います．

# 過去に書いた記事を整理する
今回話の主題としてそもそもオニオンアーキテクチャとは？，SOLID原則とは？見ていくために，振り返っていきます．
執筆した当時ほどの語彙力があるかどうかは怪しいですが，少なくとも「この記事ではあの言葉をこういう認識をしている」というニュアンスが伝わればと思います．

## クリーンアーキテクチャとオニオンアーキテクチャ

もう，最近は聞かなくなったような気がします（ワタシ自身がソフトウェア設計から身を一歩置いているというのはあるけれども）が，結局のところ，単にSOLID原則を満たすための設計方法の一つという認識をワタシはしています．

SOLID原則についてもいえることですが，コンセプトは **「可換的」** であり，個々のオブジェクトの依存関係をインターフェイスなどを用いて弱い依存関係とすることで依存先を気にせずにオブジェクト内部の問題に焦点を向けるためのもの，という認識です．適切に実装することができれば，何らかの都合で個々のオブジェクトを更新する場面が出てきても部品を交換するだけで済む反面，実行手順としては複雑になりがちなので結果として実行速度の低下を招いたり，オブジェクト数が増えることによる管理コストが増えたりします．

可換的，という視点で見るとインフラ構成レベルではマイクロサービスアーキテクチャというのもあります．両者はメリットもデメリットもよく似ています．それぞれのマイクロサービスはHTTPなりgRPCなりで構成されたAPIの仕様書にしたがって各マイクロサービスで扱う問題の大きさを小さくすることができる反面，規模が大きくなるとリソースも増えてくるため，コストもかさみ各マイクロサービス間の通信速度もボトルネックになりがちです．

## リスコフの置換原則と依存関係逆転則

なんだかんだ，SOLID原則の説明も前章で書いてしまったので，ここではその中でも「可換的」に大きく関与するリスコフの置換原則と依存関係逆転則についてピックアップします．

結論から言うと，リスコフの置換原則ではオブジェクト実装時に規定された仕組みに対応させることを，依存関係逆転則ではオブジェクトが直接対象としていないオブジェクトの概念についてより抽象的な概念を用いて操作することをあげています．

リスコフの置換原則における「規定された仕組み」は様々なものが考えられます．分かりやすいものでいえば，GoのインターフェイスやRustのトレイトなどで定義された，「関数名，引数，戻り値のセット」です．Goの `io.Writer` インターフェイスは `Write([]byte) (int, error)` 関数を実装するすべてのオブジェクトに適用することができますが，つまり `Write` という名前で，`[]byte` という引数を1個とり，戻り値が `int`, `error` であることを求めているわけです．

依存関係逆転則はオブジェクトを利用しようとする際に，オブジェクトを直接呼び出すのではなく，より抽象度の高い概念から呼び出すことを提案しています．Goの例でいえば，標準出力である `os.Stdout` を直接呼び出すのではなく， `io.Writer` として呼び出すことを求めています．こうすることで，ファイル出力 `os.File` ，メモリバッファ `byte.Buffer` などを用いて実行したい場面でもそのまま利用することができるのです．

# まあ実際問題として

## 可換的とは言うけれど交換したい場面がない
たぶん，クリーンアーキテクチャに難色を示す方の4割くらい（複数回答なら8割くらい）がこの疑問に行き着く気がしますし，ワタシも実際そう思いました．

### 依存ライブラリに後方互換性がある
一番幸せなケースです．ライブラリとしてはずっと使われ続けたいわけですし，依存しているライブラリがそう簡単に互換性を崩すはずはない，とする楽観的な思考です．でも，機能を追加こそすれ，削除したり替えたりという場面は少ないんじゃないかと思います（Pythonをはじめとして互換性を破壊することはもちろんあるけれど）．需要さえあれば，頻繁にアーキテクチャが変わっているGPUでさえ，GPUドライバで対応させることでアプリケーションに対して後方互換性を保ち続けているのです．

### そもそも交換する機会がない
こう言ってしまっては元も子もないですが，可換性があるからと言って，例えばデータベースドライバが置換できるといわれても，そもそもデータベースを移行する機会やメリットなんてそんなにないという話です．

### 負の遺産になって結局最初から作り直す
さて，規模が大きくなって管理の行き届かないプロダクトコードは最後，負の遺産となることが想定されます．そうなると最後は可換性などすべて無視して一から作り直すことになるのです．そうなっては苦労して構築したアプリケーション設計も水の泡となり，一から構築するコストだけがかさみ始めるのです．

## 高い設計レベルを求められる
設計していくにしたがって，それなりに高い設計レベルを求められます．言い換えれば，設計者，実装者ともに高レベルの人間が求められるわけですが，そのような人間を **継続的に** 見つけて充てるためにコストを割く必要があるのです．

---

この章における結論ですが，可換的な設計というのはコストがかさんだり，意味を成す場面が少なかったりとそれなりにデメリットも多く，本質的に **「必要がなければ実装しなくていいもの」** だと今のワタシは考えています．

# インターフェイスという言語機能を使用する場面
さて，ここまでの話は，可換的な設計というのは必要がなければ実装しなくていいものである，という話題でした．しかしながら，現実にGoでは `io.Reader`， `io.Writer` インターフェイスは実装側も参照側も多くのものが存在しますし， `error` インターフェイスに至っては，それを使っていないプロダクトを見たことがありません．なぜこれらは頻繁に用いられるのでしょう．

:::message
標準ライブラリとして，言語機能として実装されているから，と言われてしまうとたしかにその通りで何も言えなくなってしまうので，もう少しお付き合いください．
:::

## 期待されているものがとても小さい
これはSOLID原則におけるインターフェイス分離の原則，あるいは単一責任の原則とも通じるものがありますが，今あげた3個のGoインターフェイス型に共通して1個の関数しか規定していません．

これによって実装側は敷居が低くなるので多くの実装が存在すると考えられます．参照側も，実装が多いために，インターフェイスとして対応させた方が守備範囲が広くなります．また，一見関数を1個しか実装していないので使い勝手が悪いようにも見えますが，これらのインターフェイスを用いて実装した関数が `util` パッケージにあったり，ラップしてより使いやすくした実装が `bufio` パッケージにあったりなど，標準ライブラリだけでも十分扱いやすくなっています．

対して，「データベースに指示を出すためのインターフェイス」というのはどうでしょうか．一つのインターフェイスにまとめることは汎用性が下がるためインターフェイスとして実装する意味がなくなりますし，個別のインターフェイスとして扱う場合であってもどの程度の数を実装するのか，という問題に衝突します．

:::message
機能の分離が行えていればワタシとしては何でもいいので，インターフェイスの定義コストが気にならないのであれば，この点に注目したとき前者はともかく，後者にはまだ一考の価値があることをここに記しておきます．
:::

## 多くの実装パターンがあることを「想定されている」
これは特に `io.Writer`, `io.Reader` について言えることですが，求めていることは指定したバイトデータを書き込む，または読み込む関数があることだけです．そのため，どのように読み込むのかや書き込んだデータをどうするかなどの規定が一切ありません．そのために多くの実装が存在します．言い換えれば，「`io.Writer` と `io.Reader` が想定している実装の幅が広い」ということです．

`error` 型に関しても，「`Error()` という関数で，エラーメッセージが文字列で取得できる」事しか定義していません．実体がエラーメッセージそのものであろうが，ハードウェアドライバのエラーコードであろうが，内部に`error` 型を内包したKey-Value Pairのリッチなオブジェクトなのかなどに興味はなく，最終的にエラーメッセージが出ればそれでよいのです．

対して，「データベースに指示を出すためのインターフェイス」というのはどうでしょうか．「データベースに」と明確に指示していることからもその幅の狭さを感じます．これが例えば「（どこからでもいいので）データをとってくる」インターフェイスであればまた状況は変わったかもしれません．けれど，求められている最初に求めたインターフェイスに多くの実装パターンは期待できないでしょう．

## 変数としての関数や型を期待している

実際問題として， `io.Writer` や `io.Reader` を用いるパターンって「`io.Reader` でバイト列取得できるオブジェクトを拾って`json.Reader` で解析する」「ログの出力先を `io.Writer` 型のオブジェクトとして指定する」が一番多いと思います．特に後者の例だとイメージしやすいのですが，ログの出力先がファイルなのか標準出力なのか，というのはコーディング時には決定できず，コマンド実行時の引数などで変化したりします．インターフェイスを用いたい場面というのは総じて，実行時までどの型を用いるかわからなかったりします．

対して，「データベースに指示を出すためのインターフェイス」は，複数のデータベースを持つようなニーズでもない限り，おそらくビルド時にはすでにどのようなデータベースを使うかは決まっているでしょうしインターフェイスを用いる理由がないのです．