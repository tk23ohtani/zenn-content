---
title: Rust でのテスト駆動開発（TDD）を学んだ復習の記録 §6
emoji: ⚙
type: tech
topics:
  - TDD
  - Rust
published: false
---

TODO Listsを確認しておこう。

```markdown
- [x] $5 * 2 = $10
- [x] amountのprivateにする
- [x] Dollarの副作用をどうする？
- [ ] Moneyの丸め処理をどうする？
- [x] equals()
- [ ] hashCode()
- [ ] nullとの等価性比較
- [ ] 他のオブジェクトとの等価性比較
- [x] 5 CHF * 2 = 10 CHF
- [ ] DollarとFrancの重複
- [ ] equalsの一般化
- [ ] timesの一般化
```

この章は本通りに進むのは難しい。
Rustには継承ががないので、代わりにTraitを使う。

```rust:lib.rs
trait MoneyTrait {}
```

✅testはGreenのままだ。  
DollarにTraitを適用する。  
ついでに、一カ所SelfになっていたのでDollarに統一しておく。

```rust:lib.rs
impl MoneyTrait for Dollar {
    fn new(_amount: i32) -> Dollar {
        Dollar { amount: _amount }
    }
    fn times(&mut self, _multiplier: i32) -> Dollar {
        Dollar {
            amount: self.amount * _multiplier,
        }
    }
    fn equals(&self, _other: Dollar) -> bool {
        self.amount == _other.amount
    }
}
```

❌ビルドエラーになるので、Traitに宣言を追加する。

```rust:lib.rs
trait MoneyTrait {
    fn new(_amount: i32) -> Dollar;
    fn times(&mut self, _multiplier: i32) -> Dollar;
    fn equals(&self, _other: Dollar) -> bool;
}
```

❌ビルドエラーが続く。  
テストに、`use crate::MoneyTrait;`を追加する。

```rust:lib.rs
mod tests {
    use crate::Dollar;
    use crate::Franc;
    use crate::MoneyTrait;  // 追加
    // ...
}
```

✅testはGreenに戻った。  
ここまでが、継承の代わりになる。（P.36）

ここで、フィールドをMoneyに移動したい。Money構造体を作る。

```rust:lib.rs
pub struct Money {
    amount: i32,
}
```

✅testはGreenのままだ。まだテストは変えない。  

が、Rustには継承が無いので、そのままではMoneyのフィードを使っていることにはならない。
フィールドの移動よりも先に、DollarをMoneyにする必要がある。

```rust:lib.rs
trait MoneyTrait {
    fn new(_amount: i32) -> Money;
    fn times(&mut self, _multiplier: i32) -> Money;
    fn equals(&self, _other: Money) -> bool;
}

impl MoneyTrait for Dollar {
    fn new(_amount: i32) -> Money {
        Money { amount: _amount }
    }
    fn times(&mut self, _multiplier: i32) -> Money {
        Money {
            amount: self.amount * _multiplier,
        }
    }
    fn equals(&self, _other: Money) -> bool {
        self.amount == _other.amount
    }
}
```

❌ビルドエラーになる。
Moneyにequalsメソッドが無いからだ。
Moneyのequalsメソッドを実装する。
実装と言っても、`impl MoneyTrait for Dollar`からのコピペだ。


```rust:lib.rs
impl Money {
    fn equals(&self, _other: Money) -> bool {
        self.amount == _other.amount
    }
}
```

❌ビルドエラーになる。
Moneyにtimesメソッドが無いからだ。
Moneyのtimesメソッドを実装する。
こちらも、`impl MoneyTrait for Dollar`からのコピペだ。

```rust:lib.rs
impl Money {
    fn times(&mut self, _multiplier: i32) -> Money {
        Money {
            amount: self.amount * _multiplier,
        }
    }
    // ...
}
```

❌ビルドエラーになる。
Moneyにも比較演算子のオーバロードが必要だからだ。

```rust:lib.rs
impl PartialEq for Money {
    fn eq(&self, _other: &Money) -> bool {
        self.amount == _other.amount
    }
}
```

オーバロードのために必要な処理も入れる。

```rust:lib.rs
#[derive(Debug)]
pub struct Money {
    amount: i32,
}
```

✅testはGreenに戻った。  
テストは宣言を追加した以外の修正は行っていない。
リファクタリングの範疇だ。
これでようやくフィールドをMoneyに移動できる、かもしれない。
Dollarからフィールドを消してみる。

```rust:lib.rs
pub struct Dollar {
    // amount: i32,
}
```

❌ビルドエラーになる。
`impl MoneyTrait for Dollar`のtimesメソッドとequalsメソッドがフィールド無しエラーになっている。それはそうだ。だがすでにtimesメソッドとequalsメソッドはMoneyに移動しているので、ここにこれらのメソッドは必要なくなっているはずだ。消そう。

```rust:lib.rs
impl MoneyTrait for Dollar {
    fn new(_amount: i32) -> Money {
        Money { amount: _amount }
    }
    // fn times(&mut self, _multiplier: i32) -> Money {
    //     Money {
    //         amount: self.amount * _multiplier,
    //     }
    // }
    // fn equals(&self, _other: Money) -> bool {
    //     self.amount == _other.amount
    // }
}
```

❌ビルドエラーになる。
なるほど、宣言しているのにけしてしまったからだ。宣言も消そう。

```rust:lib.rs
trait MoneyTrait {
    fn new(_amount: i32) -> Money;
    // fn times(&mut self, _multiplier: i32) -> Money;
    // fn equals(&self, _other: Money) -> bool;
```

❌まだビルドエラーになる。
Dollarの比較演算子のオーバロードも不要になる。

```rust:lib.rs
// impl PartialEq for Dollar {
//     fn eq(&self, _other: &Dollar) -> bool {
//         self.amount == _other.amount
//     }
// }
```

```rust:lib.rs
// #[derive(Debug)]
pub struct Dollar {
    // amount: i32,
}
```

✅testはGreenに戻った。  
仕方なくtimesメソッドもMoneyに移してしまったが、ここまででP.38だ。

ページを進めよう。
Fnanc同士の比較を、テストに追加する。

```rust:lib.rs
mod tests {
    // ...
    fn test_equality() {
        assert!(Dollar::new(5).equals(Dollar::new(5)));
        assert!(!Dollar::new(5).equals(Dollar::new(6)));
        assert!(Franc::new(5).equals(Franc::new(5)));
        assert!(!Franc::new(5).equals(Franc::new(6)));
    }
    // ...
}
```

✅testはGreenに保たれている。  
FnansもMoneyにしてみよう。
手順はDollarと同じ順番で進むがかなり省略できる。
まず、FnansにMoneyTraitを適用し、timesメソッドとequalsメソッドはMoneyにすでに存在するので、消そう。

```rust:lib.rs
impl MoneyTrait for Franc {
    fn new(_amount: i32) -> Franc {
        Franc { amount: _amount }
    }
    // fn times(&mut self, _multiplier: i32) -> Franc {
    //     Franc {
    //         amount: self.amount * _multiplier,
    //     }
    // }
    // fn equals(&self, _other: Self) -> bool {
    //     self.amount == _other.amount
    // }
}
```

❌型のエラーになるので、newメソッドも修正しておく。

```rust:lib.rs
impl MoneyTrait for Franc {
    fn new(_amount: i32) -> Money {
        Money { amount: _amount }
    }
}
```

フィールドは不要になるので消しておく。

```rust:lib.rs
pub struct Franc {
    // amount: i32,
}
```

ついでにオーバロードも消そう。

```rust:lib.rs
// impl PartialEq for Franc {
//     fn eq(&self, _other: &Franc) -> bool {
//         self.amount == _other.amount
//     }
// }
```

```rust:lib.rs
// #[derive(Debug)]
pub struct Franc {}
```

✅testはGreenに保たれている。  
Rustの堅実さによってtimesメソッドとequalsメソッドの両方同時に一般化せざるを得なかった。


```
- [ ] $5 + 10 CHF = $10
- [x] $5 * 2 = $10
- [x] amountのprivateにする
- [x] Dollarの副作用をどうする？
- [ ] Moneyの丸め処理をどうする？
- [x] equals()
- [ ] hashCode()
- [ ] nullとの等価性比較
- [ ] 他のオブジェクトとの等価性比較
- [x] 5 CHF * 2 = 10 CHF
- [ ] DollarとFrancの重複
- [x] equalsの一般化
- [x] timesの一般化
- [ ] FrancとDollarを比較する
```

