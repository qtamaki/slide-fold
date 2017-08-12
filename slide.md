name: inverse
layout: true
class: left, middle, inverse
---
# fold沼について
[@qtamaki](https://twitter.com/qtamaki)

[株式会社チェンジ・ザ・ワールド](http://ctws.jp)

---
## Scalaのfold

[scala.collection.immutable.List](http://www.scala-lang.org/api/2.12.3/scala/collection/immutable/List.html) なんかについているメソッド

↓使い方

```
 List(1,2,3).fold(0)(_+_)
```
---
## 定義

foldメソッドは、だいたいここで定義されている。

* [scala.collection.TraversableOnce](http://www.scala-lang.org/api/2.12.3/scala/collection/TraversableOnce.html)
  * → [scala.collection.GenTraversableOnce](http://www.scala-lang.org/api/2.12.3/scala/collection/GenTraversableOnce.html)

シグニチャ

```
def fold[A1 >: (K, V)](z: A1)(op: (A1, A1) ⇒ A1): A1
```

既知のサブクラスは220個ほど。標準ライブラリのcollectionは、全部foldできる。

---
## その他の定義


[検索](http://www.scala-lang.org/api/2.12.3/scala/collection/GenTraversableOnce.html?search=fold)して調べる。

```
$('.entity a').map((i,n) => console.log(n.innerText))
```

以下のクラスは、GenTraversableOnceを継承しておらず、独自にfoldを定義している。

* [Either](http://www.scala-lang.org/api/2.12.3/scala/util/Either.html)
* [Option](http://www.scala-lang.org/api/2.12.3/scala/Option.html)
* [Try](http://www.scala-lang.org/api/2.12.3/scala/util/Try.html)
* [Future(object)](http://www.scala-lang.org/api/2.12.3/scala/concurrent/Future#.html)(Deprecated)

※FutureのfoldはDeprecatedで、objectに定義されていて、TraversableOnce[Future[T]]を畳み込むだけなので無視

---
## TraversableOnce.fold

もう一度、シグニチャ。

```
def fold[A1 >: (K, V)](z: A1)(op: (A1, A1) ⇒ A1): A1
```

なお、foldLeftのシグニチャ。

```
def foldLeft[B](z: B)(op: (B, A) ⇒ B): B
```

### 違い

* foldは、並列計算を意識しており、結合則を満たす必要がある(a->b->cを(a->b)->cでもa->(b->c)でも計算できる必要がある)
* その為、型を変更できない(A->B)

---
## 型を変更できない(A->B)

例えば、foldLeftであれば、下記のようにreverseを実装できる

```
List(1,2,3,4).foldLeft(List.empty[Int])((a,b) => b::a)
```

foldでは出来ない！(List[Int]の場合、Intの処理しか出来ない)

結論: foldLeft/foldRightの方が柔軟性が高く便利(であるが、並列化可能なため、foldの要求を満たす計算はfoldで実装しておいたほうが良い)

---
## Option.fold

とにかく多用します(当社比)。

```
User.find(1).fold("unknown user"){u => u.userName}
```

Option/Try/Eitherという、失敗系のコンテナから値をスムーズに取り出すのに使います。

※ とある方面ではコンテナを失敗系・状態系などと分けたりします。なんと、Listも失敗系に分類されるので、今回登場するコンテナは全て失敗系です。ズコー。

---
## Option map f getOrElse isEmpty

[scaladoc](http://www.scala-lang.org/api/2.12.3/scala/Option.html)には下記のような注釈があります。

```
This is equivalent to scala.Option map f getOrElse isEmpty
```
これと同義だということです。

カッコが省略されてますね。

```
Option.map(f).getOrElse(isEmpty)
```

となります。

mapメソッドは、AからBへコンテナの値を変換するメソッドです。

getOrElseは、OptionがNoneだった場合に返す値を定義しています。

---
先程の例で言うと、これでも問題ないということです。

```
User.find(1).fold("unknown user"){u => u.userName}
↓
User.find(1).map(u => u.userName).getOrElse("unknown user")
```

失敗する可能性のある計算には、OptionやEither, Tryなんかを使いますが、最終的には別の型(Resultとか)で返すというパターンになります。

失敗系の計算を連結するには、map/flatMapが定番ですが、最終的に値が取り出せません。

```
find(x).map(hoge).map(fuga).map(foo).map(bar)... // どこまでもOption型の計算
```

値を取り出すための方法の１つがfoldという事になります。

foldの場合、先にデフォルトの値を書き、getOrElseはmapの後に書くことになります。単に好みの問題ですが、(u => u.userName)の部分は、実際には数行になることが多いので、ソースの見通し上foldが好まれるのだと思います。

---
## Either.fold

Eitherのfoldは、結果がLeftだった時の処理、Rightだった時の処理と2つの関数を取ります。

```
def fold[C](fa: (A) ⇒ C, fb: (B) ⇒ C): C
```

正直、matchで書くのと大差ないので、matchでいいんじゃ？

```
hoge match {
  case Left(x) => ...
  case Right(x) => ...
}
```

---
## Try.fold

Tryのfoldは、結果がFailureだった時の処理、Successだった時の処理と２つの関数を取ります。

```
def fold[U](fa: (Throwable) ⇒ U, fb: (T) ⇒ U): U
```

正直、match(ry

---
## まとめると

* Scalaのfoldは、TraversableOnce系とそれ以外に別れる
* TraversableOnce系のfoldは結合則を満たす必要があるが、並列化可能
  * そのため計算結果を別の型に出来ない。foldLeft/foldRightの方が柔軟性が高いが用途に応じて選択すると良い
* それ以外のfoldは、失敗時と成功時の値や処理を振り分けつつ最終的な値を取り出すのに便利
  * getOrElseやmatchを使うかは好み(だと思う)

---
## おまけ

Haskellの場合、foldは、Foldableという型クラスになっている。

```
class Foldable (t :: * -> *) where
  fold :: Monoid m => t m -> m
  foldMap :: Monoid m => (a -> m) -> t a -> m
  foldr :: (a -> b -> b) -> b -> t a -> b
  foldl :: (b -> a -> b) -> b -> t a -> b
```

コンテナの中身がMonoidである必要があり、mappendで自動的に畳み込まれる。

Scalaのfoldと同じくMonoidである必要がある(つまり結合則を満たす)。
