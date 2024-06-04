# トランプのカードはEntityかValueObjectか

Created: 2024年5月20日 1:34
Updated: 2024年5月20日 2:15
Published_Time: 2024年5月20日
Slug: entity-vs-value-object
Tags: Design
Published: Yes
URL: https://blog.shaba.dev/posts/entity-vs-value-object

## ふと

3〜4年前、前職でDDDの勉強会に参加していた時、「トランプのカードをプログラムで設計する時、EntityなのかValueObjectなのか」という話が上がった。

その時は時間もなく、なーなーになって結局議論しないままになっていた。。

当時はペーペーエンジニアだったのもあり、なんとなく解を思い浮かべたものの根拠のないものだったが、数年経った今ふとこの議題を思い出し、自分の中でまとまったので（成長した）、書き出してみる。

## Entity VS ValueObject

エンティティかバリューオブジェクトかは、要は以下の3点を持つか持たないかという話になる。

1. ミュータブルかイミュータブルか
2. 状態を持つか持たないか（`1.`に含まれるけど一応分ける）
3. 振る舞いを持つか持たないか

全部持たなければ`ValueObject`、一つでも持てば `Entity`となる。

トランプは4種類のマークと1 ~ 13の数値の組み合わせに、JOKER2枚。

これらが不変であることは既知の事実なので、`1.`はイミュータブルで良い。

```tsx
class Card {
  private _mark: "spade"|"clab"|"heart"|"diya"|"joker";
  private _number: ?number;
}

class NomalCard extends Card {
  private _mark: "spade"|"clab"|"heart"|"diya";
  private _number: number;  
}

class JokerCard extends Card {
  private _mark: "joker";
  private _number: null;
}
```

`JOKER` は数値を持たないので、他のカードとは別のクラスとして用意した方が良さそう。

で、問題は`2.`と`3.`

### トランプカードは状態を持つか

考えられる状態は

- 裏か表か
- どこに居るか
    - 山札、場、手札、捨て場、etc…

くらいかな？

```tsx
class Card {
  private _mark: "spade"|"clab"|"heart"|"diya"|"joker";
  private _number: ?number;
  private _isOpen: boolean;
  
  public function isOpen(): boolean {
    return this._isOpen;
  }
}

```

### トランプカードは振る舞いを持つか

こちらもパッと思いつく振る舞いといえばこれくらいかな？

- 引数のカードと同じマークか
- 引数のカードとの数値の比較
- 表裏を変更する

```tsx
class Card {
  private _mark: "spade"|"clab"|"heart"|"diya"|"joker";
  private _number: ?number;
  private _isOpen: boolean;
  
  public function sameMark(card: Card): boolean {
    return this._mark === card.getMark();
  }
}
```

## 遊び方によってカードの意味も置き場も変わる

例えばババ抜きだと、全ての有効なカードは手札にあるので、そもそもカードの裏表は区別する必要がないし、マークの比較も大小を比較する必要もない。

また、他の遊び方では好きな数値になれるJOKERは、ババ抜きではなんの役割も持たない。

一方で七並べだと、マークの区別や数値の評価も厳格である必要がある。

要は、トランプカードは遊び方によって役割や置ける場所がガラッと変わる。

なのでトランプに振る舞いや状態を持たせるのではなく、それぞれの遊びクラスを用意して管理する方が自然だと思われる。

そして多分それは`Entity`というよりは`DomainServiceクラス`や、特にプロパティも持たなくて良さそうなので`Utilクラス`になると思う。

また、`Deck`, `Hand`, `Field`, `Dump`といった置き場所のクラスも用意しておくと良さそう。

所属しているカード、表のカードは何か、カードの増減、といった振る舞いを持たせておける。

```tsx
interface OldLadyService { // ババ抜き。サービスクラスとした例
  public function match(card1: Card, card2: Card): boolean;
  public function isWin(hand: Hand): boolean  
}

class Hand {
  private _cards: Array<Card>;
  public function add(card: Card): void; // _cardsを変更するのではなく、_cardsを更新したHandをnewして返しても良さそう
}
```

## 結論

ということで、個人的にはトランプのカードはValueObjectで定義すると思う。

当時は話についていくだけで精一杯だった内容が、良し悪しは置いておいて自分の中でちゃんと答えを出せて、ちょっとスッキリ。