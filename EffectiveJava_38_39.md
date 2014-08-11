title: ffective Java
output: ej38-39.html
controls: true

--

# Effective Java
## 38, 39

--

### ここからの内容

* 項目38. パラメータの正当性を検査する
* 項目39. 必要な場合には、防御的にコピーする

--

### 項目38. パラメータの正当性を検査する
* パラメータとして渡される値に関しての制約の例
  * インデックス値が負であってはいけない
  * オブジェクト参照がnullであってはいけない...
* これらの制約について
  * 制約は明確に文書化する
  * メソッド本体のはじめにチェックし、  
  制約を強制する

--

### チェックしないことによる不具合
* メソッドが処理の途中で訳の分からない例外で失敗
* 何も言わずに誤った結果を計算
* いくつかのブジェクトの状態を不正な状態にした  
まま正常にリターン
  * 全く関係のない箇所でエラーを引き起こす

-- 

### publicなメソッドでの強制方法
1. パラメータの制約を文書化
1. スローされる例外を`@throws`タグで文書化
  * `IllegalArgumentException`
  * `IndexOutOfBoundsException`
  * `NullPointerException`など
1. メソッドのはじめにパラメータをチェック

--

### publicなメソッドでの強制方法
```
/**
 * 値が(this mod m)であるBigIntegerを返します。このメソッドは、
 * remainderメソッドとは異なり、常に負でないBigIntegerを返します。
 *
 * @param m 正でなければならないモジュラス
 * @return this mod m.
 * @throws ArithmeticException m <= 0 の場合。
 */
public BigInteger mod(BigInteger m) {
    if (m.signum() <= 0)
        throw new ArithmeticException("Modulus <= 0: " + m);
    ... // 計算を行う
}

```
--

### privateなメソッドでの強制方法
アサーションでチェックする
* 作成者がどのような状況でメソッドが呼び出されるか管理する
* 作成者が正当なパラメータだけが渡されることを保証できる、保証する
* 明示的に有効化しない限り、コストが発生しない

--

### privateなメソッドでの強制方法
```
// 再帰的ソートのためのprivateのヘルパー関数
private static void sort(long a[], int offset, int length) {
	assert a != null;
	assert offset >= 0 && offset <= a.length;
	assert length >= 0 && length <= a.length - offset;
	... // 計算を行う
}
```
--

### チェックしないでもよい場合
下記の２点を満たすときは、チェックしなくてもよい
* 正当性チェックのコストが高いor現実的でない
* 正当性チェックが計算の処理の中で暗黙に行われる

--

### チェックしないでもよい場合
例) Colletions.sort(List)
* すべてのオブジェクトは相互比較可能
  * 正当性チェックのコストが高いor現実的でない
* 個々のオブジェクトはソートの過程で比較される
  * 正当性チェックが処理の中で暗黙に行われる
  * ここでの例外は文書化したものに一致させる
--

### 項目38. まとめ
* パラメータにどのような制約が存在するか考える
  * パラメータの制約は出来る限り少なくし、  
  メソッドは一般的なものとして設計する
* 制約は文書化する
* メソッド本体の初めでチェックし、制約を強制する
--

### ここからの内容

* 項目38. パラメータの正当性を検査する
* 項目39. 必要な場合には、防御的にコピーする

--

### 項目39. 必要な場合には、防御的にコピーする

* Javaは安全な言語  
= システムのほかの部分で何が起きてもその
クラスの不変式が成立している

* 安全な言語でも他のクラスからの防御は
自分で行う必要がある

--

### 防御は必要

* セキュリティ面
* APIを使用するプログラマの単純な間違いによる予期せぬ振る舞いを防ぐ

--

### 不完全な不変クラス
```
public final class Period {
	private final Date start:
	private final Date end;

	/**
	 * @param start 期間の開始
	 * @param end 期間の終わりで、開始より前であってはならない。
	 * @throws IlleagalArgumentException startがendの後の場合。
	 * @throws NullPointerException startかendがnullの場合。
	 */
	public Period(Date start, Date end) {
		if (start.compareTo(end) > 0)
		    throw new IllegalArgumentException(start + " after " + end);
		this.start = start;
		this.end = end;
	}

	public Date start() {
		return start;
	}

	public Date end() {
		return end;
	}

	... //残りは省略
}
```

--

###　実は不変ではない
`Date`は可変
```
Date start = new Date();
Date end = new Date();
Period p = new Period(start, end);
end.setYear(78); // pの内部を変更する
```
対策
```
public Period(Date start, Date end) {
    this.start = new Date(start.getTime());
    this.end = new Date(end.getTime());

	if (start.compareTo(end) > 0)
	    throw new IllegalArgumentException(start + " after " + end);
}
```
※ Dateのcloneメソッドを使用してはいけない

--

### まだ不変ではない
アクセッサーが可変なアクセスを提供
```
Date start = new Date();
Date end = new Date();
Period p = new Period(start, end);
p.end().setYear(78); // pの内部を変更する
```
対策
```
public Date start() {
	return new Date(start.getTime());
}

public Date end() {
	return new Date(end.getTime());
}
```
これで不変になった
--

### 防御的コピーの必要性
* 不変なクラスのためだけでない
* 内部データにクライアントからのオブジェクトを格納する場合は、格納後にオブジェクトに変更があってもクラスに問題がないか考える
* 同様に、内部データへの参照をクライアントに返す前にもコピーすべきか考える





















