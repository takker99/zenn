---
title: "ScrapboxでVim key bindingsを作ろうと試行錯誤する話"
emoji: "🦔"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["vim", "scrapbox", "javascript"]
published: true
---

本記事は[Vim2 Advent Calendar 2020](https://qiita.com/advent-calendar/2020/vim2)の 18 日目の記事です。

- 17 日目：[君はまだ Vim の真の美しさを知らない - Qiita](https://qiita.com/psyashes/items/1e1716a59a0dc22ea204) by [psyashes](https://qiita.com/psyashes)さん
- 19 日目：[僕がVim色に染まるまで](https://qiita.com/ryosk7/items/1e2679f9dbe5692f6da1) by [ryosk7](https://qiita.com/ryosk7)さん

# Topic

Scrapbox で vim key bindings を使えるようにする UserScript を紹介するつもりでしたが、アドカレ締切までに完成しませんでした……。
代わりに開発する過程で直面した課題を解説します。

# はじめに

Vim を特徴づけているものの一つに Vim key bindings があります。 [14 日目](https://zenn.dev/hasu_83/articles/3dcf927b43f689365788)の記事でも言及されているように一度使い始めたら最早この key bindings なしには生きられなくなります。

いや生きられなくなるのは誇張しすぎですね。ですが少なくとも、他の software や web service を使うと不満を覚えるような身体になってしまいます。

- 「Vim なら diw ですんだ」
- 「Vim ならマウス触らずに操作できた」
- 「Vim ならカーソルキーまで手を伸ばさず済んだのに」
- 「Vim なら(ry」

このような無茶な欲望を叶えるべく、一部の services では有志の手によって vim key bindings を使えるようにする拡張機能が開発されています。

- Excel: [VimExcel](https://www.slideshare.net/XlsVim/vimexcel-90834259)
- Visual Studio Code: [VsCodeVim](https://github.com/VSCodeVim/Vim)
- web browser: [Vimium](https://github.com/philc/vimium), [VimVixen](https://github.com/ueokande/vim-vixen)

しかし星の数ほどある softwares や web services に比べれば、vim key bindings を使える service はほんのわずかでしょう。

そのようなまだ vim key bindings が作られていない web service の一つに[Scrapbox](https://scrapbox.io)があります。
簡単に説明すると思考を練り、情報を手動でつなげ、communication を取るための強力な tool です。とても気軽に output できる tool なので、もっと多くの人に使ってもらいたいと思っていますが、残念ながら vim key bindings がありません。Emacs key bindings はあるのに。
この欠点のせいで使われないというのは非常に残念なことです。とはいえ[公式で vim key bindings は実装しなさそう](https://scrapbox.io/forum-jp/Vim%E5%85%A5%E5%8A%9B%E3%83%A2%E3%83%BC%E3%83%89%E3%81%8C%E6%AC%B2%E3%81%97%E3%81%84%E3%81%A7%E3%81%99)なので、公式の力は頼れません。**ならば自分で作ってしまおう**。

幸いにも、scrapbox には UserScript/UserCSS という、user が自由に program code を追加して拡張できる機能が備わっています。これらを使えば、公式が実装しなくても user 側で vim key bindings を実装できるはずです！

---

そんなこんなで意気込んで開発を開始したものの、Topic で触れたとおりアドカレ当日までに完成できませんでした[^1]……ちくせう。

[^1]: 原因は時間を全然考えていなかったことと大規模な program を作った経験がないことだと考えています。前者はお察しですね。

今の所使えるのは、本当に一部の機能しか実装していない[alpha 版](https://scrapbox.io/takker/ScrapVim-lite-2)のみです。まだ人に勧められる状態ではないですが、もし興味のある方がいらっしゃれば使ってみて下さい。以下の code を自分のページに貼り付ければ使えます[^2]

```js:script.js
import { ScrapVim } from "/api/code/takker/ScrapVim-lite-2/script.js";
const scrapVim = new ScrapVim();
scrapVim.start();
```

理想だとこのあと使い方をメインに説明するつもりでした。しかし完成していない以上それも出来ないので、代わりに開発の経緯と、そこで得られた知見、課題を書いていこうと思います

[^2]: 実際の code に興味がある方は、[このへん](https://scrapbox.io/takker/ScrapVim-lite-3)のリンクから飛んでいただくと良いかもしれません。他者が読めるようには書いていませんので、そこだけご承知下さい

# 開発の流れ

だいたいこんな感じで開発しています

1. 動けばいいだけの code を作る
2. コマンドの実装方法を模索する
3. 設計を組み立てる

prototype を作って勝手を調べてから本格的な設計を考えるという流れで作っています。今 3.にいるところです。
最初からこの流れに沿うよう意識して開発したわけではなく、試行錯誤した過程が結果的にこの流れと同じになりました。

今回の開発では今までに 3 つ prototype を作成しました

## コマンドの実装方法を模索する段階

[/villagepump/Vim の Normal mode を実装してみる](https://scrapbox.io/villagepump/VimのNormal_mode_を実装してみる)

一番最初に書いたコードです。とにかく動く事を最優先に作っていました。
いくつか機能を足して、browser を reload して試して、バグを直して reload して試して、……の繰り返しです

[ScrapVim-lite](https://scrapbox.io/takker/ScrapVim-lite)

最初のと同様、具体的な command を実装する事を最優先に作っています。
本筋とは関係ないですが、ここでキー入力イベントの処理方法を変更しました。
最初の prototype では簡単にキーバインドを追加できる[Mousetrap](https://scrapbox.io/takker/Mousetrap)という library を使っていました。しかし、2 文字以上のコマンドを使うと、最後の文字以外が editor にそのまま入力されてしまう問題が発生することに途中で気づきました。
例えば`yy`と入力すると、行 yank が実行される前に`y`が editor に入力されてしまいます。
どうしようもないので、Web API の`addeventListener`を直接使うように変更しています

## program 設計を考える段階

[ScrapVim-lite-2](https://scrapbox.io/takker/ScrapVim-lite-2)

ESModule によるファイル分割と class 化をメインに行っています。
一応機能単位を考えてファイル分割をしていますが、機能で分割するより、コードが長くなりすぎて見にくくなったので分割したという動機の方が大きかった気がします。
class を使って機能を分割してはいますが、まだ重複しているコードがちらほらあります

[ScrapVim-lite-3](https://scrapbox.io/takker/ScrapVim-lite-3) (開発中)

アドカレまでに間に合わなかったやつです。つらい。
ScrapVim-lite-3 の設計があんまりよくないように思えたので、1 からえいやと作り直しました
ここから、でできるだけ粗結合になるように機能を分割させています

# 実装方法

キーボード入力代行やテキストの座標計算などは Vim と全然関係ない部分なので割愛します。
代わりに Vim key bind から command を解釈する処理を実装する試行錯誤の過程を解説します。

## switch を使った実装

scrapVim-lite-2 まで最初は動けばいいの精神で、switch を使って key binings ごとにコマンドを対応させていました

```js
 onNormalMode(keymap) {
     this._log('Analyze commands as a one of the normal mode.');
     switch (keymap) {
         case 'h':
             move.left({cursor: this.cursorInfo});
             break;
         case 'j':
             move.down();
             break;
         case 'k':
             move.up();
             break;
         case 'l':
             move.right({cursor: this.cursorInfo});
             break;

             // 略

         case 'i':
             insert.before({onInsert: () => this._onInsert()});
             break;
         case 'a':
             insert.after({onInsert: () => this._onInsert()});
             break;
         case 'I':
             insert.startOfLine({onInsert: () => this._onInsert(), cursor: this.cursorInfo});
             break;
         case 'A':
             insert.endOfLine({onInsert: () => this._onInsert()});
             break;
         case 'o':
             insert.newLineBelow({onInsert: () => this._onInsert()});
             break;
         case 'O':
             insert.newLineAbove({onInsert: () => this._onInsert()});
             break;
         case 'x':
             edit.cut();
             break;
         case 'd':
             if (this.stack.last !== 'd') return;
             edit.deleteLine();
             break;
         case 'D':
             edit.deleteForEnd();
             break;

             // 略
         default:
             break;
     }
     this.stack.flush();
 }
```

はい。見るからにダメコードですね。このままではユーザがコマンドを定義できませんし、何より 2 文字以上のコマンドに対応するのが不可能です。

流石にこんなコードをそのまま放置するのは嫌気が差したので、key bindings をまともに解析して対応するコマンドを決定する処理を設計することにしました。

## 候補 1: コマンドと key bindings とのペアを作る

switch の回避は出来ますが柔軟なコマンドの指定が出来ないので即却下しました
ここが vim key bindings と他の key bindings/ keyboard shortcut とで決定的に異なる点です。
他の service の shortcut key は、キーとコマンドとが一対一に対応しています。なので候補 1 の方法で十分です。
一方 vim key bindings は一定の文法に従って柔軟に変化させられます
`d`を例に取ってみると、`["x][count]d{motion}`のように 3 つの文字列をとることができ、それぞれ決まったグループの文字列のみ入れられるようになっています
全ての文字列を予め列挙しておくことが無謀である以上、一対一のペアで対応することは不可能です

## 候補 2: 状態遷移を使う

register や motion などのグループ単位なら、key bindings と操作の一対一対応が成立しているので、そのグループ単位で解析するという案です

図式するとこんなかんじです
![](http://www.plantuml.com/plantuml/svg/RP112iCW44NtESMGPS4hbDoZxKBYa0ZLaN5SzFQLo5GgxlJvdFzros9PIdWlftS8699ym67UsIVn59V7xGM6_N6AkGFZuRCW_rFQgKGPM4AsGeCv4GDTCRM68AnwxHalTGMRTRncW-cHoYQvpPWSw09CILgfmx5NcpBIhbTMdxCq_jjk65tzqoy0#.svg)
_normal mode の状態遷移図[^3]_
初期状態 (図の黒丸)は register, 繰り返し指定, operator, motion のにいずれかに該当する key を受け付けます
その後例えば`"`が入力されたら`register`に移行し、さらに`a`が押されたら register`"a`を確定し、`operator count`, `operator`のいずれかに該当する key を受け付ける状態に遷移します
二重丸の状態にまで遷移したら、解析したコマンドを実行します

[^3]: text object は複雑になるので図から省いています。もしかしたら後で修正するかも

おそらくここまでは自然に思いつくことだと思います。ただいざ実装しようと設計を詰めていったときに、解析しているコマンドをどこに持たせるか等でだいぶ詰まりました。
最終的には状態を全て関数にして、解析中のコマンドと入力された key を受け取って次の状態を表す関数を返すような形式にしました

書いていて今更ですが、既存の構文解析処理を真似て作ったほうがどう考えても早いですね……視野が狭くなっていたのか、他の program を参考にするという発想は思いつきませんでした。

# 今後の改善点

技術周りでは、まず既存の構文解析の手法を学ぶが挙げられます。key bindings が一定の文法に従っているのなら、既存の構文解析手法を適用するのが手っ取り早いです。何も自分で車輪の再発明をする必要なんてなかったんだ……

それから他の vim emulator の source code を見て実装を学ぶことも改善点です。今更になって[VSCodeVim/Vim](https://github.com/VSCodeVim/Vim)を見れば良いことに気づいたりしました。`diw`など text object なども考慮した構文解析を行っているようなので参考になりそうです。

開発手法に関しては、program が大規模になっても、簡単に小さくテストできるような仕組みが必要だと感じました。
[ScrapVim-lite-2](https://scrapbox.io/takker/ScrapVim-lite-2)など以前の prototyping では、全てのコードが完成するまでテストはおろかバグ取りすら出来ませんでした。これでは完成するまで致命的なミスに気づけず、かなりの時間を無駄にすることになります。[ScrapVim-lite-3](https://scrapbox.io/takker/ScrapVim-lite-3)では機能ごとになるべく独立させて作っているので、後はテストを簡単にできる環境を作る必要があります。

---

最後までお読みいただきありがとうございました。
1時間近く遅刻した上に怪しい日本語になってしまい申し訳ありません。今回が自分にとって初のインターネット投稿記事になりますが、初心者などということに甘んじたくはないので、マサカリをじゃんじゃん投げていただけると幸いです。
また記事執筆のアドバイスをくださったVim-jp Slackの皆様、本当にありがとうございました。おかげでなんとか記事に仕立て上げることが出来ました。この場を借りて感謝申し上げます。
