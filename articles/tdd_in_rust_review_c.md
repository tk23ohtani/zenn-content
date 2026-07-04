---
title: Rust でのテスト駆動開発（TDD）を学んだ復習の記録 §12
emoji: ⚙
type: tech
topics:
  - TDD
  - Rust
published: false
---

TODO Listsを確認しておこう。

```markdown
- [ ] $5 + 10 CHF = $10
- [ ] $5 + $5 = $10
- [x] $5 * 2 = $10
- [x] amountのprivateにする
- [x] Dollarの副作用をどうする？
- [ ] Moneyの丸め処理をどうする？
- [x] equals()
- [ ] hashCode()
- [ ] nullとの等価性比較
- [ ] 他のオブジェクトとの等価性比較
- [x] 5 CHF * 2 = 10 CHF
- [x] DollarとFrancの重複
- [x] equalsの一般化
- [x] timesの一般化
- [x] FrancとDollarを比較する
- [x] 通過の概念
- [x] testFrancMultiplicationを削除する？
- [x] Fnancの通貨判定テストを復元する
```

不要になったら消す章だが、DollarもFrancも参照されない構造体が残っているだけなので、そのまま消して良さそうだ。

```rust:lib.rs
mod tests {
    // ...
    #[test]
    fn test_simple_addition() {
        let sum = Money::dollar(5).plus(Money::dollar(5));
        assert_eq!(Money::dollar(10), sum);
    }
}
```

❌Money::plus()が当然実装されていない。  
実装スピードを加速するらしいｗ

```rust:lib.rs
impl Money {
    fn plus(&self, _addend: Money) -> Money {
        Money::new(self.amount + _addend.amount, self.currency)
    }
}
```

✅テストはGreenになった！（拍手）
為替レートを適用するようだ。

```rust:lib.rs
mod tests {
    // ...
    fn test_simple_addition() {
        let sum = Money::dollar(5).plus(Money::dollar(5));
        assert_eq!(Money::dollar(10), sum);
        let reduced = bank.reduce(sum, "USD");
        assert_eq!(Money::dollar(10), reduced);
    }
    // ...
}
```

❌テストがRedのままでテストを書き足して、やりたいことを書いていっている。

```rust:lib.rs
mod tests {
    // ...
    fn test_simple_addition() {
        let mut sum = Money::dollar(5).plus(Money::dollar(5));
        assert_eq!(Money::dollar(10), sum);
        let five = Money::dollar(5);
        sum = five.plus(five);
        let bank = Bank::new();
        let reduced = bank.reduce(sum, "USD");
        assert_eq!(Money::dollar(10), reduced);
    }
    // ...
}
```

❌テストがRedのままでテストを書き足して、やりたいことを書いていっている。
ExpressionをTraitとして定義する。

```rust:lib.rs
pub trait Expression {}

impl Expression for Money {}
```

❌MoneyのplusメソッドはExpressionを返す必要がある。

```rust:lib.rs
impl Money {
    // ...
    fn plus(&self, _addend: Money) -> Box<dyn Expression> {
        Box::new(Money::new(self.amount + _addend.amount, self.currency))
    }
    // ...
}
```

❌Bankを作る。

```rust:lib.rs
pub struct Bank;

impl Bank {
    fn new() -> Bank {
        Bank
    }
}

mod tests {
    use crate::Bank;
    // ...
}
```

❌Bankに空のreduceを定義しておく

```rust:lib.rs
impl Bank {
    // ...
    fn reduce(&self, _source: Box<dyn Expression>, _to: &str) -> Money {
        Money::new(10, "USD")   // 仮の実装
    }
}
```

✅当然テストはGreenのままだ。  
TODOに変更しておこう。

```markdown
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
- [x] DollarとFrancの重複
- [x] equalsの一般化
- [x] timesの一般化
- [x] FrancとDollarを比較する
- [x] 通過の概念
- [x] testFrancMultiplicationを削除する？
- [x] Fnancの通貨判定テストを復元する
```
