---
title: Rust でのテスト駆動開発（TDD）を学んだ復習の記録 §7
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
- [ ] FrancとDollarを比較する
```

FrancとDollarの型を比較していく。

```rust:lib.rs
mod tests {
    // ...
    #[test]
    fn test_equality() {
        assert!(Dollar::new(5).equals(Dollar::new(5)));
        assert!(!Dollar::new(5).equals(Dollar::new(6)));
        assert!(Franc::new(5).equals(Franc::new(5)));
        assert!(!Franc::new(5).equals(Franc::new(6)));
        assert!(!Franc::new(5).equals(Dollar::new(5)));
    }
    // ...
}
```

❌失敗してしまう。
`std::any::Any`の`type_id()`を使ってみよう。

```rust:lib.rs
use std::any::Any;
// ...
impl Money {
    // ...
    fn equals(&self, _other: Money) -> bool {
        self.amount == _other.amount
        && self.type_id() == _other.type_id()
    }
}
```

❌おっとテストが通らない。MoneyとMoneyの比較になっているようだ。
Moneyにそれぞれの型番号を持たせるしかないのか。
まずは、Moneyに型フィールドを設ける。

```rust:lib.rs
use std::any::{Any, TypeId};
// ...
pub struct Money {
    amount: i32,
    typeid: TypeId,
}
```

インスタンス化した時の型を比較することにする。

```rust:lib.rs
impl Money {
    fn equals(&self, _other: Money) -> bool {
        self.amount == _other.amount
        && self.typeid == _other.typeid
    }
    // ...
}
```

なので、インスタンス化した時に型を持たせる。

```rust:lib.rs
impl MoneyTrait for Dollar {
    fn new(_amount: i32) -> Money {
        Money {
            amount: _amount,
            typeid: TypeId::of::<Dollar>()
        }
    }
}
// ...
impl MoneyTrait for Franc {
    fn new(_amount: i32) -> Money {
        Money {
            amount: _amount,
            typeid: TypeId::of::<Franc>()
        }
    }
}
```

❌ビルドエラー？！  
timesメソッドでもインスタンス化しているな。

```rust:lib.rs
impl Money {
    fn times(&mut self, _multiplier: i32) -> Money {
        Money {
            amount: self.amount * _multiplier,
            typeid: self.typeid
        }
    }
    // ...
}
```

✅testがGreenになり、FrancとDollarの比較が等しくならないようになった。もちろんレートが1:1ではない時ではある。

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
```



```rust:lib.rs
```

```rust:lib.rs
```

```rust:lib.rs
```

✅testはGreenのままだ。  

```rust:lib.rs
```

```rust:lib.rs
```

```rust:lib.rs
```

```rust:lib.rs
```

```rust:lib.rs
```

```rust:lib.rs
```

```rust:lib.rs
```

```rust:lib.rs
```

