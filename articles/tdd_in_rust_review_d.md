---
title: Rust でのテスト駆動開発（TDD）を学んだ復習の記録 §13
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
- [x] $5 + $5 = $10
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

Sumインスタンスを作る？

```rust:lib.rs
mod tests {
    // ...
    #[test]
    fn test_plus_Return_sum() {
        let five = Money::dollar(5);
        let result = five.plus(Money::dollar(5));
        let sum: Sum = result;
        assert_eq!(five, sum.augend);
        assert_eq!(five, sum.addend);
    }
}
```

❌Sumを作る。

```rust:lib.rs
struct Sum {
    augend: Money,
    addend: Money,
}
```

❌plusメソッドでSumを返す。

```rust:lib.rs
impl Money {
    fn plus(&self, _addend: Money) -> Sum {
        Sum {
            augend: Money::new(self.amount, self.currency),
            addend: _addend,
        }
        // Box::new(Money::new(self.amount + _addend.amount, self.currency))
    }
}
```

❌reduceメソッドに引数が合わなくなってしまったので修正する。

```rust:lib.rs
impl Bank {
    // ...
    fn reduce(&self, _source: Sum, _to: &str) -> Money {
        Money::new(10, "USD")
    }
}
```

✅テストはGreenになった！（拍手）
足し算の通貨も同じになる。

```rust:lib.rs
mod tests {
    // ...
    #[test]
    fn test_reduce_sum() {
        let sum = Sum {
            augend: Money::dollar(3),
            addend: Money::dollar(4),
        };
        let bank = Bank::new();
        let result = bank.reduce(sum, "USD");
        assert_eq!(Money::dollar(7), result);
    }
}
```

❌reduceが仮実装なので失敗する。

```rust:lib.rs
impl Bank {
    // ...
    fn reduce(&self, _source: Sum, _to: &'static str) -> Money {
        let sum = _source;
        let amount = sum.augend.amount + sum.addend.amount;
        Money::new(amount, _to)
    }
}
```

✅テストはGreenになった！（拍手）

```rust:lib.rs
impl Bank {
    fn new() -> Bank {
        Bank
    }
    fn reduce(&self, _source: Sum, _to: &'static str) -> Money {
        let sum = _source;
        // let amount = sum.augend.amount + sum.addend.amount;
        // Money::new(amount, _to)
        sum.reduce(_to)
    }
}
```

❌Sum.reduceを実装する。

```rust:lib.rs
impl Sum {
    fn reduce(&self, _to: &'static str) -> Money {
        let amount = self.augend.amount + self.addend.amount;
        Money::new(amount, _to)
    }
}
```

✅テストはGreenになった！（拍手）
Cloneをつかって、本に近い形のテストになるようにしておく。

```rust:lib.rs
#[derive(Debug, Clone)]
pub struct Money {
    // ...
}
// ...
mod tests {
    // ...
    #[test]
    fn test_simple_addition() {
        let five = Money::dollar(5);
        let sum = five.plus(five.clone());
        let bank = Bank::new();
        let reduced = bank.reduce(sum, "USD");
        assert_eq!(Money::dollar(10), reduced);
    }

    #[test]
    fn test_plus_return_sum() {
        let five = Money::dollar(5);
        let result = five.plus(five.clone());
        let sum: Sum = result;
        assert_eq!(five, sum.augend);
        assert_eq!(five, sum.addend);
    }
}
```

✅テストはパスする。  
Bank.reduce(Money)のテストを書こう。

```rust:lib.rs
mod tests {
    // ...
    #[test]
    fn test_reduce_money() {
        let bank = Bank::new();
        let result = bank.reduce(Money::dollar(1), "USD");
        assert_eq!(Money::dollar(1), result);
    }
}
```

❌ビルドエラー
Bank::reduceをExpressionトレイトで受けられるようにすることで、SumでもMoneyでも受け入れられるようにする。

```rust:lib.rs
impl Bank {
    // ...
    fn reduce<T: Expression>(&self, _source: T, _to: &'static str) -> Money {
        _source.reduce(self, _to)
    }
}
```

❌Expressionトレイトにreduceメソッドを宣言する。


```rust:lib.rs
pub trait Expression {
    fn reduce(&self, bank: &Bank, to: &'static str) -> Money;
}
```

❌MoneyにExpression::reduceメソッドを実装する。

```rust:lib.rs
impl Expression for Money {
    fn reduce(&self, _bank: &Bank, _to: &'static str) -> Money {
        Money::new(self.amount, _to)
    }
}
```

❌SumのExpression::reduceメソッドも修正する。

```rust:lib.rs
impl Expression for Sum {
    fn reduce(&self, _bank: &Bank, _to: &'static str) -> Money {
        let aug = self.augend.reduce(_bank, _to);
        let add = self.addend.reduce(_bank, _to);
        let amount = aug.amount + add.amount;
        Money::new(amount, _to)
    }
}
```

✅Greenになった！  
TODOに変更しておこう。

```markdown
- [ ] $5 + 10 CHF = $10
- [ ] $5 + $5 = $10
- [ ] $5 + $5 がMoneyを返す
- [x] Bank.reduce(Money)
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
