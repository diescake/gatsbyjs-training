---
title: JavaScript で forEach を使うのは最終手段
date: '2018-12-07T22:40:32.169Z'
---

この記事は [JavaScript2 Advent Calendar 2018](https://qiita.com/advent-calendar/2018/javascript_02) の 1 日目の記事です。

こんばんは。@diescake です。

今年は、JavaScript 経験の浅い新人さんや外注さんをリードする立場として、
とにかく幅広いメンバーのコードレビューをする機会に恵まれたのですが、
事ある毎に `Array.prototype.forEach` を利用する人が多かったため、初心者向けに要点を整理してみました。

以下 ES2015 以降のバージョンをサポートするブラウザ、あるいは polyfill を利用していることを前提としています。

# 結論

配列に対して何らかの操作を行う際は、
`filter`, `find`, `map`, `reduce` などのメソッドを利用できないか検討し、
いずれのメソッドでも実現ができない場合の最終手段として `forEach` を選択しましょう。

下記に、いくつかのサンプルコードを例に説明していきます。

# forEach ➔ filter

まず、最初に `Array.prototype.filter` の例です。

```js:forEach
const pomeranians = []

dogs.forEach(dog => {
  if (dog.type === 'pomeranian') {
    pomeranians.push(dog)
  }
})
```

このコードでは、配列 `dogs` のうち特定の条件に合致した要素を全て抜き出しています。

勿論この実装でも正常に動作しますが、
このケースでは下記のように `filter` を利用することでより簡潔に記述できます。

```js:filter
const pomeranians = dogs.filter(dog => {
  return dog.type === 'pomeranian'
})
```

さらに、実際には arrow function の shorthand を利用することでさらにコンパクトになります。
(これ以降の例では shorthand によるコードのみを紹介します)

```js:filter(shorthanded)
const pomeranians = dogs.filter(dog => dog.type === 'pomeranian')
```

さて、`forEach` から `filter` に書き換えたことによって、コードが短く簡潔になったことはわかると思いますが、
それよりも重要なメリットは、

配列のループ処理に `filter` が利用されていることで、このコードを見た人に、
**このループが dogs の subset を抽出している、という意図が瞬間的に伝わる** ということです。

`forEach` は配列をループするという目的と意味しか持たないため、
何のためにループを行っているのか把握するためには、実装の詳細を追う必要がありますが、
`filter` が利用されている場合、コードの詳細を読まずとも、処理全体として特定条件の要素を抜き出す処理を行っているだろうことが伝わります。

コードリーディングにおいて、ループの処理を読み解くのは脳に対する負荷は高くなりがちですが、
前もって処理全体の目的が分かれば、詳細を理解するにあたって大きくコストを削減できます。

ここでは `filter` の例で説明しましたが、それ以外の `find`, `map`, `reduce` についても同様のメリットがあり、
適切なメソッドを選択することで、リーダビリティに優れたコードとなるため重要です。

# forEach ➔ find

続いて `Array.prototype.find` です。

```js:forEach
let myDog

dogs.forEach(dog => {
  if (dog.name === 'ポメラニアス3世') {
    myDog = dog
  }
})
```

このコードでは、配列 `dogs` から、特定の要素を抜き出しています。

また、`forEach` ではループの `break` が実現できない点が気にかかり、
もっと素朴に `for` 文で記述する人も見かけましたが、
いずれもあまり良いコードとはいえません。

```js:for
let myDog

for (let i = 0; i < dogs.length; i++) {
  if (dog.name === 'ポメラニアス3世') {
    myDog = dog
    break
  }
}
```

このケースでは下記のように `find` を利用することでより目的を明確にし、簡潔に記述できます。

```js:find
const myDog = dogs.find(dog => dog.name === 'ポメラニアス3世')
```

ちなみに、`filter` を使うと以下のように書けます。

```js:filter
const myDog = dogs.filter(dog => dog.name === 'ポメラニアス3世')[0]
```

コードの実装量からすると、どちらも大した差はないように見えますが、
先程説明したように、`find` を使うことで、配列から特定の要素 1 つを検索して抜き出すという意図が明確になるため、
`filter` より `find` を利用するのが良いでしょう。

特にこのケースで `filter` の戻り値を配列のまま変数で受けると、
`myDogs[0]` のように index を指定して先頭の要素のみを参照することになりますが、
この場合、配列の先頭は何を意味するのか？、先頭以外の要素はなぜ参照しないのか？といった労力を読み手に課すことになるため、避けたほうが良いでしょう。

# forEach ➔ map

3 番目、`Array.prototype.map` です。

```js:forEach
const dogNames = []

dogs.forEach(dog => {
  dogNames.push(dog.name)
})
```

このコードは、配列 `dogs` の各要素を参照して、別の配列を作り出す例です。
別の構成の配列を作り出すようなケースでは `map` を使うときれいにかけます。

```js:map
const dogNames = dogs.map(dog => dog.name)
```

この例ではあまりにシンプル過ぎてメリットが伝わらないかもしれませんが、
React で JSX を扱う場合は、必ず利用するといっても良いくらい活躍の場は多いです。

```jsx:map
render() {
  return (
    <ul>
      {this.props.dogs.map(dog => <li>{`${dog.name}: ${dog.description}`}</li>)}
    </ul>
  );
};
```

また、下記のように、特定の要素のみを加工して配列に作るというケースはあると思いますが、
`map` だけでは要素の取捨選択を行うことはできないため `filter` と組み合わせて実現することになります。

```js:forEach
const pomeranians = []

dogs.forEach(dog => {
  if (dog.type === 'pomeranian') {
    pomeranians.push({
      id: uuid(),
      name: dog.name,
    })
  }
})
```

```js:filter->map
const animals = dogs.filter(dog => dog.type === 'pomeranian')
  .map(dog => ({
    id: uuid(),
    name: dog.name,
  });
```

`map` で処理する前に `filter` で条件にあった配列を抜き出しています。

```js:map->filter
const animals = dogs.map(dog => {
  if (dog.type !== 'pomeranian') {
    return null;
  }
  return {
    id: uuid(),
    name: dog.name,
  };
}.filter(v => v);
```

上記は逆に `map` で処理してから `filter` する例です。

この例では前者の方が簡潔ですが `filter` する条件が複雑化した場合、
`map` 関数中で適宜 early return した `null` を後からまとめて `filter` で弾く方が見通し良く記述できる場合があります。

ちなみに、配列に対する `.filter(v => v)` という記述で、
配列中の falsy な値を全て除くことができるというのは覚えておいて損はないと思います。

# forEach ➔ reduce

最後に `Array.prototype.reduce` です。

```js:forEach
let total = 0

dogs.forEach(dog => {
  total += dog.price
})
```

このコードは、配列 `dogs` の各要素を参照して、合計値を計算している例です。

実は前 3 つと比較すると `reduce` を使った方が良いです！！とコメントしたケースは少なかったのですが、
こういった配列の合計値を求める処理や、文字列連結を行う際に、
変数宣言を `let` から `const` に書き換えることができるという点が便利ですね。

```js:reduce
const total = dogs.reduce((acc, dog) => acc + dog.price, 0)
```

また、次の項目で紹介しますが、
内部状態を持つインスタンスを生成し、ループ中でインスタンスメソッドを呼び出し、
最終的に内部状態が変更されたインスタンスを戻り値として返すような処理がワンライナーで書くことができるので、ビシッと嵌まるときがあります。

ただ、`reduce` も `forEach` 程ではないですが、比較的万能なので、
乱用すると意図がよくわからないコードになりがちな気はします。

# forEach が妥当なケース

さて、逆に `Array.prototype.forEach` での記述が妥当だと感じた例です。

```js:forEach
dogs.forEach(dog => {
  console.log(dog.name)
})
```

外部のスコープに対して直接関与しない場合は、`forEach` での記述は妥当な気がします。
(すぐに思い浮かびませんが、例外は何かあるかも…)

また、上記例に近くはありますが `fetch` や通知など、非同期処理を呼び出す場合は、
`forEach` でループごとに `await` することができないことと、実際には `Promise.all` を利用して並行処理できるケースが多いと思うので、
この場合は Promise の配列を返すために `map` を利用することになると思います。

```js:map
await Promise.all(dogs.map(async dog => await dog.eat('ペディグリーチャム'));
```

次に、ある内部状態を持つインスタンスに対して、インスタンスメソッドで操作する例です。

```js:forEach
const pomeranian = new Pomeranian()

foods.forEach(food => {
  if (food.type === 'beef') {
    pomeranian.add(food)
  }
})
```

ちなみに、`reduce` で記述すると下記のように書けます。

```js:reduce
const pomeranian = foods.reduce((pomeranian, food) => {
  if (food.type === 'beef') {
    pomeranian.add(food)
  }
  return pomeranian
}, new Pomeranian())
```

ただこれは `forEach` に比べて簡潔になったかというと微妙で、
関数の戻り値としてインスタンスを受けたいケースでない場合は、`forEach` で記述されていても、
好みの範疇で特に問題はないかなと感じます。

# まとめ

というわけで、ちょっと微妙なサンプルもあったと思いますが、`forEach` を書き換える例を紹介しつつ、
なぜ、書き換えた方が良いか？という話をしてみました。

`forEach` を利用すると何でもできてしまうが故に、
1 つのループ中で、複数意図の処理を詰め込み、
`if` 文を乱立させたり、`for` をネストさせて…、と、スパゲッティ化した PR をいくつも見ました。

元々 `forEach` は戻り値を持たないので、意味のある処理を実現しようとすると外部スコープに対する操作が前提となり、
ともすると、あれもこれもと、やり放題になってしまうのかなと感じています。

また、実際にウェブアプリを実装する際には、
lodash, immutable.js, Rambda などの colletion を操作するライブラリを利用するのは有効ですが、
bundle したファイルサイズにも影響してきますので、無理がない場合は vanilla で書くことは良いユーザ体験に繋がると思います。

最後に、ほぼパフォーマンスについては言及しませんでしたが、
最も素朴に `for` 文で記述する方が恐らく速く(要出展)、リーダビリティとトレードオフになるとは思います。

ただし、`for` が速いとはいえ、この書き換えによって、パフォーマンスに大きく差異がでるケースは稀だと思っていて、
チューニングフェーズで、実測して明らかにボトルネックであることが判明してから、書き換えを検討するという方向性が良いと思います。

(この書換えによって、パフォーマンスに大きく差異がでる程のループ処理を JavaScript 上で行う必要がある時点でサービス全体の設計に問題がある気が…)

さて、明日は @ttokutake さんです。 （╹◡╹）
