---
title: Rust でのテスト駆動開発（TDD）を学んだ復習の記録 §10
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
- [x] FrancとDollarを比較する
- [x] 通過の概念
- [ ] testFrancMultiplicationを削除する？
- [x] Fnancの通貨判定テストを復元する
```

timesメソッドはすでに一般化してしまったので、前半はスキップしよう。
が、通貨の概念が対応していないのだ。
本とはズレてしまうが、テストを作って、timesメソッドも通貨対応していこう。

```rust:lib.rs
mod tests {
    // ...
    #[test]
    fn test_currency() {
        assert_eq!("USD", Money::dollar(1).currency());
        assert_eq!("CHF", Money::franc(1).currency());
        assert_eq!("USD", Money::dollar(1).times(2).currency());
    }
    // ...
}
```

❌Money::times()の通貨が”JPY"で返ってきていることが分かる。
前章で実施したcurrency()と同じ実装をすればいいはずだ。

```rust:lib.rs
impl Money {
    // ...
    fn times(&mut self, _multiplier: i32) -> Money {
        Money {
            amount: self.amount * _multiplier,
            typeid: self.typeid,
            currency: self.currency,
        }
    }
    // ...
}
```

✅テストはGreenに戻る。   
Fnancのテストも追加して、Greenのままであることを確認しておこう。

```rust:lib.rs
    fn test_currency() {
        assert_eq!("USD", Money::dollar(1).currency());
        assert_eq!("CHF", Money::franc(1).currency());
        assert_eq!("USD", Money::dollar(1).times(2).currency());
        assert_eq!("CHF", Money::franc(1).times(2).currency());
    }
```

✅テストはGreenに戻る。問題ない。
本ではここでtimesメソッドをMoneyのnewで返すように変更している。

```rust:lib.rs
    fn times(&mut self, _multiplier: i32) -> Money {
        Money::new(self.amount * _multiplier, self.typeid, self.currency)
        // Money {
        //     amount: self.amount * _multiplier,
        //     typeid: self.typeid,
        //     currency: self.currency,
        // }
    }
```

❌当然テストはRedだ。
まて、Moneyにnewメソッドが無い。

```rust:lib.rs
impl Money {
    fn new(_amount: i32, _typeid: TypeId, _currency: &'static str) -> Money {
        Money {
            amount: _amount,
            typeid: _typeid,
            currency: _currency,
        }
    }
    // ...
}
```

✅よし、Greenに戻った。
本では、MoneyとFrancが一致してしまうことが問題になっているが、我々のコードでは、equalsメソッドと比較演算子のオーバロードで別実装となっているため、これを統一しておく必要があるだろう。テストを書く。

```rust:lib.rs
    fn test_equality() {
        assert!(Money::dollar(5).equals(Money::dollar(5)));
        assert!(!Money::dollar(5).equals(Money::dollar(6)));
        assert!(Money::franc(5).equals(Money::franc(5)));
        assert!(!Money::franc(5).equals(Money::franc(6)));
        assert!(!Money::franc(5).equals(Money::dollar(5)));
        assert_ne!(Money::franc(5), Money::dollar(5));
    }
```

❌当然テストはRedだ。
equalsメソッドと比較演算子のオーバロードを同じ実装に統一しておく

```rust:lib.rs
impl Money {
    // ...
    fn equals(&self, _other: Money) -> bool {
        self == &_other
        // self.amount == _other.amount && self.typeid == _other.typeid
    }
    // ...
}
// ...
impl PartialEq for Money {
    fn eq(&self, _other: &Money) -> bool {
        self.amount == _other.amount && self.typeid == _other.typeid
    }
}
```

✅Greenに戻せた。
クラスの比較から通貨の比較に変更しよう。


```rust:lib.rs
// use std::any::{Any, TypeId};

#[derive(Debug)]
pub struct Money {
    amount: i32,
    currency: &'static str,
}

impl Money {
    fn new(_amount: i32, _currency: &'static str) -> Money {
        Money {
            amount: _amount,
            currency: _currency,
        }
    }
    fn equals(&self, _other: Money) -> bool {
        self == &_other
        // self.amount == _other.amount && self.typeid == _other.typeid
    }
    fn times(&mut self, _multiplier: i32) -> Money {
        Money::new(self.amount * _multiplier, self.currency)
    }
    fn dollar(_amount: i32) -> Money {
        Money {
            amount: _amount,
            currency: "USD",
        }
    }
    fn franc(_amount: i32) -> Money {
        Money {
            amount: _amount,
            currency: "CHF",
        }
    }
    fn currency(&self) -> &'static str {
        self.currency
    }
}

impl PartialEq for Money {
    fn eq(&self, _other: &Money) -> bool {
        self.amount == _other.amount && self.currency == _other.currency
    }
}
```

✅一気にいってしまったが、Greenが保たれている。
TODOに変更はなかった。

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
- [ ] DollarとFrancの重複
- [x] equalsの一般化
- [x] timesの一般化
- [x] FrancとDollarを比較する
- [x] 通過の概念
- [ ] testFrancMultiplicationを削除する？
- [x] Fnancの通貨判定テストを復元する
```
