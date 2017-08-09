name: inverse
layout: true
class: center, middle, inverse
---
# foldについて

---
## Scalaのfold

[scala.collection.immutable.List](http://www.scala-lang.org/api/2.12.3/scala/collection/immutable/List.html) なんかについているメソッド

↓使い方

```
 List(1,2,3).fold(0)(_+_)
```
---
## 定義

foldメソッドは、**だいたい**ここで定義されている。

* [scala.collection.TraversableOnce](http://www.scala-lang.org/api/2.12.3/scala/collection/TraversableOnce.html)
  * [scala.collection.GenTraversableOnce](http://www.scala-lang.org/api/2.12.3/scala/collection/GenTraversableOnce.html)

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

* [Future(object)](http://www.scala-lang.org/api/2.12.3/scala/concurrent/Future#.html)
* [Either](http://www.scala-lang.org/api/2.12.3/scala/util/Either.html)
* [Option](http://www.scala-lang.org/api/2.12.3/scala/Option.html)
* [Try](http://www.scala-lang.org/api/2.12.3/scala/util/Try.html)

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

* どちらから畳み込むのか未定義
* 型を変更できない(A->B)

---
## 型を変更できない(A->B)

例えば、foldLeftであれば、下記のようにreverseを実装できる

```
List(1,2,3,4).foldLeft(List.empty[Int])((a,b) => b::a)
```

foldでは出来ない！

結論: foldLeft/foldRightの方が便利。

---
## Option.fold

とにかく多用します(当社比)。

```
User.find(1).fold("unknown user"){u => u.userName}
```

Option/Try/Either/Futureという、失敗系の値コンテナから値をスムーズに取り出すのに使います。

---
## Option map f getOrElse isEmpty

scaladocには下記のような注釈があります。

```
scala.Option map f getOrElse isEmpty
```
これと同義だということです。

カッコが省略されてますね。

```
Option.map(f).getOrElse(isEmpty)
```

となります。

mapメソッドは、AからBへコンテナの値を変換するメソッドです。

